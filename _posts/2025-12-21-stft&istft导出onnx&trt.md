---
layout: post
title:  "stft/istft 导出 onnx/tensorrt 模型"
author: shuai
categories: [ onnx,onnxruntime ]
image: assets/images/13.jpg
---

当 torch 写的神经网络里面有 stft 和 istft 操作时,转换为 onnx/trt 模型会有一些困难.stft 操作在似乎在最新的 tensorrt 版本里面才被支持,而 istft 我没有查到在哪个版本支持了导出 onx 和 tensorrt 模型.本文提供了和 stft/istft 等效的实现,支持转换为 onnx/tensorrt模型

## STFT 的可导出的等效实现

```python
class STFT(torch.nn.Module):
    def __init__(self, filter_length=1024, hop_length=512, win_length=None,
                 window='hann'):
        """
        This module implements an STFT using 1D convolution and 1D transpose convolutions.
        This is a bit tricky so there are some cases that probably won't work as working
        out the same sizes before and after in all overlap add setups is tough. Right now,
        this code should work with hop lengths that are half the filter length (50% overlap
        between frames).
        
        Keyword Arguments:
            filter_length {int} -- Length of filters used (default: {1024})
            hop_length {int} -- Hop length of STFT (restrict to 50% overlap between frames) (default: {512})
            win_length {[type]} -- Length of the window function applied to each frame (if not specified, it
                equals the filter length). (default: {None})
            window {str} -- Type of window to use (options are bartlett, hann, hamming, blackman, blackmanharris) 
                (default: {'hann'})
        """
        super(STFT, self).__init__()
        self.filter_length = filter_length
        self.hop_length = hop_length
        self.win_length = win_length if win_length else filter_length
        self.window = window
        self.forward_transform = None
        self.pad_amount = int(self.filter_length / 2)
        scale = self.filter_length / self.hop_length
        fourier_basis = np.fft.fft(np.eye(self.filter_length))

        cutoff = int((self.filter_length / 2 + 1))
        fourier_basis = np.vstack([np.real(fourier_basis[:cutoff, :]),
                                   np.imag(fourier_basis[:cutoff, :])])
        forward_basis = torch.FloatTensor(fourier_basis[:, None, :])
        inverse_basis = torch.FloatTensor(
            np.linalg.pinv(scale * fourier_basis).T[:, None, :])

        assert(filter_length >= self.win_length)
        # get window and zero center pad it to filter_length
        fft_window = get_window(window, self.win_length, fftbins=True)
        fft_window = pad_center(fft_window, size=filter_length)
        fft_window = torch.from_numpy(fft_window).float()

        # window the bases
        forward_basis *= fft_window
        inverse_basis *= fft_window

        self.register_buffer('forward_basis', forward_basis.float())
        self.register_buffer('inverse_basis', inverse_basis.float())

    def transform(self, input_data):
        """Take input data (audio) to STFT domain.
        
        Arguments:
            input_data {tensor} -- Tensor of floats, with shape (num_batch, num_samples)
        
        Returns:
            magnitude {tensor} -- Magnitude of STFT with shape (num_batch, 
                num_frequencies, num_frames)
            phase {tensor} -- Phase of STFT with shape (num_batch, 
                num_frequencies, num_frames)
        """
        num_batches = input_data.shape[0]
        num_samples = input_data.shape[-1]

        # self.num_samples = num_samples

        # similar to librosa, reflect-pad the input
        input_data = input_data.view(num_batches, 1, num_samples)
        
        input_data = F.pad(
            input_data.unsqueeze(1),
            (self.pad_amount, self.pad_amount, 0, 0),
            mode='reflect')
        input_data = input_data.squeeze(1)

        forward_transform = F.conv1d(
            input_data,
            self.forward_basis,
            stride=self.hop_length,
            padding=0)

        cutoff = int((self.filter_length / 2) + 1)
        real_part = forward_transform[:, :cutoff, :]
        imag_part = forward_transform[:, cutoff:, :]

        return real_part,imag_part
```
该实现是在某个开源仓库里面找到的,找不到是哪一个仓库了

以下代码给出了 STFT 的用法,并和 torch 原生的stft 的结果进行了对比:
```python
import torch,sys,os
import numpy as np
from scipy.signal import get_window
from librosa.util import pad_center, tiny

def _stft( x):
    spec = torch.stft(
        x,
        istft_params["n_fft"], istft_params["hop_len"], istft_params["n_fft"], window=stft_window.to(x.device),
        return_complex=False)
    # spec = torch.view_as_real(spec)  # [B, F, TT, 2]
    return spec[..., 0], spec[..., 1]

stft_custom= STFT(filter_length=istft_params["n_fft"], 
                                          hop_length=istft_params["hop_len"], 
                                          win_length=istft_params["n_fft"],
                                          window='hann')
x=torch.rand(1,istft_params["n_fft"]*10)*1000
y1_real,y1_img=_stft(x)
y2_real,y2_img=stft_custom.transform(x)
diff=torch.abs(y1_real-y2_real)
print(f'diff max:{diff.max()},diff min:{diff.min()}')
print(y1_real.shape)
# 
```
两者的结果差异很小:diff max:0.0001678466796875,diff min:0.0,在使用的时候使用 STFT 替换掉 torch.stft,即可实现导出 onnx,并能转换到 tensorrt 模型

## ISTFT 的可导出的等效实现

istft 的等效实现来源于开源仓库[https://github.com/biendltb/torch-istft-onnx.git](https://github.com/biendltb/torch-istft-onnx.git)

```python
import torch,sys,os
import numpy as np
from scipy.signal import get_window
sys.path.append(os.path.dirname(__file__)+"/third_party/torch-istft-onnx") # noqa: E402
from torch_istft_onnx.istft import ISTFT # noqa: E402

istft_params={}

istft_params["n_fft"]=1024
istft_params["hop_len"]=256
stft_window=torch.from_numpy(get_window("hann", istft_params["n_fft"], fftbins=True).astype(np.float32))
def _stft( x):
    spec = torch.stft(
        x,
        istft_params["n_fft"], istft_params["hop_len"], istft_params["n_fft"], window=stft_window.to(x.device),
        return_complex=False)
    # spec = torch.view_as_real(spec)  # [B, F, TT, 2]
    return spec[..., 0], spec[..., 1]

def _istft( magnitude, phase):
    real = magnitude * torch.cos(phase)
    img = magnitude * torch.sin(phase)
    inverse_transform = torch.istft(torch.complex(real, img), istft_params["n_fft"], istft_params["hop_len"],
                                    istft_params["n_fft"], window=stft_window.to(magnitude.device))
    return inverse_transform
istft_custom = ISTFT(
            n_fft=istft_params["n_fft"],
            hop_length=istft_params["hop_len"],
            window=stft_window,
            normalized=False,
            max_frames=1000000
        )
def _istft_custom( magnitude, phase):
    real = magnitude * torch.cos(phase)
    img = magnitude * torch.sin(phase)
    real=real.unsqueeze(3)
    img=img.unsqueeze(3)
    spec=torch.cat([real,img],dim=3)
    inverse_transform = istft_custom(spec)
    return inverse_transform

x=torch.rand(1,istft_params["n_fft"]*10)*10
y1_real,y1_img=_stft(x)

magnitude=torch.abs(y1_real+1j*y1_img)
phase=torch.angle(y1_real+1j*y1_img)

y1=_istft_custom(magnitude,phase)[:,istft_params["hop_len"]:-istft_params["hop_len"]]
y2=_istft(magnitude,phase)[:,istft_params["hop_len"]:-istft_params["hop_len"]]
diff=torch.abs(y1-y2)
print(f'diff max:{diff.max()},diff min:{diff.min()}')
```

以上代码给出了 自定义的 ISTFT 的用法,并和 torch 原生的 istft 的结果进行了对比,两者的结果差异也很小:`diff max:4.9114227294921875e-05,diff min:0.0`,不过上面在对比输出结果时裁剪掉了两端的一些数据,因为两端的误差有些大,不过两端的数据有差异在我使用的场景是完全可以忽略的,暂时没有深究为什么有差异.

在使用的时候用 ISTFT 替换掉原生的 istft 即可实现导出 onnx,并转换为 tensorrt 模型