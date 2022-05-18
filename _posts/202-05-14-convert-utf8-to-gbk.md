---
layout: post
title:  "使用c/c++实现utf8编码转gbk编码的两种方法"
author: sal
categories: [ iconv,fatfs ]
image: assets/images/16.jpg
---
最近工作中遇到一个问题，需要把中文语句转换成对应的音素，如“增大音量”需要转换成“[z-eng] [d-a] [ii-in] [l-iang]”。boss给我提供了一个python实现的程序，需要我用c/c++实现一遍。python实现中大致分为两步：汉字转拼音、拼音转音素。其中汉字转拼音python中是由一个库实现的，要如何用c来实现这一功能呢？我在网上找到一个把gbk编码的汉字映射成拼音的表，但是程序要求的输入是utf8编码的汉字，所以就需要实现utf8编码转gbk编码。

## 使用libiconv实现编码转换

libiconv库是一个基于GNU协议的开源库，能够实现不同编码的转换。使用起来很简单,就是对一下三个函数的调用:

```c
#include <iconv.h>
iconv_t iconv_open(const char *tocode, const char *fromcode);
size_t iconv(iconv_t cd, char **inbuf, size_t *inbytesleft, char **outbuf, size_t *outbytesleft);
int iconv_close(iconv_t cd);
```
其中需要注意的是`iconv`函数会修改指针**inbuf 和指针outbuf的值**。因此需要保存原输入、输出内存分配的地址值。大致的调用如下:
```c
//pin,pout均为char*类型,为指向输入输出空间的指针
iconv_t cd;
// 保存原输入、输出内存分配的地址值
char *pin_bak = pin;
char *pout_bak = pout;

cd = iconv_open("gbk", "utf8");
if (cd == 0)
{
    //open failed,do something
}
if (iconv(cd, &pin, &pin_len, &pout, &pout_len) == -1){
    // convert failed ,do something
}
iconv_close(cd);
//pout_bak指向转换后的结果,pout-pout_bak得转换后的长度
```
该代码在Ubuntu上运行良好,但是通过NDK编译时,却发现NDK中限制了只有android api等级大于等于28才可以使用iconv,所以这个方案就被pass了。

## 移植fatfs实现字符编码转换

libiconv不可以使用,我又开始查找不用第三方库实现编码转换的方法.我发现在一些单片机的项目中也使用到了编码的转换,如果在单片机上能实现,那么在安卓上肯定也能实现.顺着这些蛛丝马迹,我最后找到了github上的fatfs项目,该项目中也有转换编码的需求.

fatfs中对于简体中文支持基于两个超大的数组所记录的映射.其中一个就是unicode映射gbk的数组:
```c
static const WCHAR uni2oem936[] = {	/* Unicode --> GBK pairs */
    ...
}
```
Unicode是一个字符集，规定了每一个字符对应一个二进制代码，至于这个二进制代码如何存储则没有任何规定。而UTF8则是在unicode的基础上,规定了这个二进制代码在内存中如何存储.utf-8编码是一种变长编码, 中文字符用三个byte来存储，而编码范围在 0 到 0x7f 则使用1个字节存储.更多细节看这个表就知道了.

| Number of bytes |Bits for code point|First code point  |	Last code point	|Byte1	    |Byte2	    |Byte3	    |Byte4
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|        1	       |           7	   |      U+0000	  |       U+007F	|0xxxxxxx	|		    |           |
|        2	       |           11      |      U+0080	  |       U+07FF	|110xxxxx	|10xxxxxx	|	        |
|        3	       |           16      |      U+0800	  |       U+FFFF	|1110xxxx	|10xxxxxx	|10xxxxxx	|
|        4	       |           21      |      U+10000	  |       U+10FFFF	|11110xxx	|10xxxxxx	|10xxxxxx	|10xxxxxx

按上表所示,对于一个utf8编码的汉字,它在内存中按二进制看是这么存放的:`1110xxxx,10xxxxxx,10xxxxxx`,我们要做的就是把这里面的`x`提取出来组成一个新的数字,这个数字就是这个汉字的unicode编码.比如汉字`习`,在内存中是这样的`0xe4,0xb9,0xa0`,换成二进制是这样的:`0b11100100,0b10111001,0b10100000`,那么它对应的unicode编码是`0b0100111001100000`,也即0x4e60.把这个数字传入如下函数(参数代码页cp是936),返回值即为gbk编码对应的数字.

```c
WCHAR ff_uni2oem (	/* Returns OEM code character, zero on error */
	DWORD	uni,	/* UTF-16 encoded character to be converted */
	WORD	cp		/* Code page for the conversion */
)
```

综上所诉,只需要把上面那个映射的数组uni2oem936和函数ff_uni2oem复制出来,再把ff_uni2oem函数稍作修改,就轻松实现utf8转gbk了~

<!-- ## Full HTML

Perhaps the best part of Markdown is that you're never limited to just Markdown. You can write HTML directly in the Markdown editor and it will just work as HTML usually does. No limits! Here's a standard YouTube embed code as an example:

<p><iframe style="width:100%;" height="315" src="https://www.youtube.com/embed/Cniqsc9QfDo?rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe></p> -->