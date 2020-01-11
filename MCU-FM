@[TOC](基于单片机的FM收音机 附带测温功能)

# 介绍
本次制作为我在单片机课程设计这一门课的作业，该设计用STC89C52RC型号的单片机控制FM收音模块TEA5767在特定频率下搜台，并通过数字温度传感器DS18B20读取当前温度。下图为成品图片。![成品图正面](https://img-blog.csdnimg.cn/20191109164916371.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9qdW42NjY=,size_16,color_FFFFFF,t_70)
![成品图反面](https://img-blog.csdnimg.cn/20191109173309755.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9qdW42NjY=,size_16,color_FFFFFF,t_70)
本制作需要的主要元器件有：==STC89C52RC,TEA5767,LM386,LCD1602,DS18B20==。其余元件有==电容，电阻，电位器，喇叭，导线，轻触开关，晶振==。下图为该收音机的电路原理图，为了便于观看，里面省略了复位电路和时钟电路。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191122225609954.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9qdW42NjY=,size_16,color_FFFFFF,t_70)
从上面的实物图可以看出，收音机的天线用一根导线代替即可。我使用的是从网线里面抽出的一根长约半米的包着皮的细铜线。至于使用的电源，使用手机充电器供电或电脑供电均可。
# 可能会遇到的问题
1. 买tea5767时不要贪便宜，最初我在淘宝上找到了一家店卖￥0.49一个，买回来后发现它和常见的tea5767不一样，根本无法判断它的引脚，而且卖家提供的资料链接是失效的。我追问卖家，结果。。。![无良卖家](https://img-blog.csdnimg.cn/20191109171702536.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9qdW42NjY=,size_16,color_FFFFFF,t_70)
2. 制作过程中的难点之一便是TEA5767的焊接。由于TEA5767体积比较小，相邻管脚的距离约为2mm，而常见万用板相邻两孔的间距约为２.５ｍｍ，故TEA5767很难直接焊接，尤其对我这种手抖的人而言。在老师的指导下，我用AD给它设计了一个底座，TEA5767可以很简便地焊接在底座上，这大大降低了制作难度，底座的形状如下图：![tea5767底座](https://img-blog.csdnimg.cn/20191109170519825.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9qdW42NjY=,size_16,color_FFFFFF,t_70)
3. 制作过程的另一大难点为程序的编写，无论是DS18B20，LCD1602还是TEA5767都需要比较复杂的程序去控制。在网上可以很方便地找到这三者的手册和相关程序。如果你不想关注它们使用的具体细节，可以仅从函数的功能（而非函数的具体实现）入手，调用相关函数实现整体功能。如果你想要本实例的程序以及 上面底座的PCB文件，请到微信公众号<kbd>技术杂学</kbd>中回复<kbd>单片机FM</kbd>。另外附上运行视频：[点击此处](https://www.bilibili.com/video/av74780614/)

本文在今日头条以[东方建木](https://www.toutiao.com/i6756184839075922444/)的身份首发。
