# C#的webclient的使用总结与初识api

# 前言
今天在B站看到一个up主发布的关于用C语言写b站粉丝数显示器的视频，它使我颇受启发。我第一次了解到了[wget](https://www.cnblogs.com/zhaojiedi1992/p/zhaojiedi_linux_004.html)这个下载文件的命令行工具以及它的使用，对浏览器的开发者工具也有了进一步了解(在network下按<kbd>ctrl+r</kbd>重新加载元素)和在此界面寻找api链接。
当然我认为他用C语言写的那个代码太不简洁，用C#会更方便些。于是我尝试用C#写了这个程序。

# WebClient的使用
好长时间没用C#了，关于[WebClient](https://docs.microsoft.com/zh-cn/dotnet/api/system.net.webclient?redirectedfrom=MSDN&view=netframework-4.8)的使用几乎忘得差不多了，于是我在网上查了它的使用。
## 示例
```csharp
using System;
using System.Net;
using System.IO;

public class Test
{
    public static void Main (string[] args)
    {
        WebClient client = new WebClient ();

        // Add a user agent header in case the 
        // requested URI contains a query.

        client.Headers.Add ("user-agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.2; .NET CLR 1.0.3705;)");

        Stream data = client.OpenRead (args[0]);
        StreamReader reader = new StreamReader (data);
        string s = reader.ReadToEnd ();
        Console.WriteLine (s);
        data.Close ();
        reader.Close ();
    }
}
```
## 属性

|属性|作用|
|:--|:--|
|[BaseAddress](https://docs.microsoft.com/zh-cn/dotnet/api/system.net.webclient.baseaddress?view=netframework-4.8#System_Net_WebClient_BaseAddress)|获取或设置 WebClient 发出请求的基 URI。|
|[Encoding](https://docs.microsoft.com/zh-cn/dotnet/api/system.net.webclient.encoding?view=netframework-4.8#System_Net_WebClient_Encoding)|获取或设置用于上传和下载字符串的 Encoding|
|[Headers](https://docs.microsoft.com/zh-cn/dotnet/api/system.net.webclient.encoding?view=netframework-4.8#System_Net_WebClient_Encoding)|获取或设置用于上传和下载字符串的 Encoding|
|[IsBusy](https://docs.microsoft.com/zh-cn/dotnet/api/system.net.webclient.isbusy?view=netframework-4.8#System_Net_WebClient_IsBusy)|了解是否存在进行中的 Web 请求。|
|[Proxy](https://docs.microsoft.com/zh-cn/dotnet/api/system.net.webclient.proxy?view=netframework-4.8#System_Net_WebClient_Proxy)|获取或设置此 WebClient 对象使用的代理。|
|[QueryString](https://docs.microsoft.com/zh-cn/dotnet/api/system.net.webclient.querystring?view=netframework-4.8#System_Net_WebClient_QueryString)|获取与响应关联的标头名称/值对集合。|
|[site](https://docs.microsoft.com/zh-cn/dotnet/api/system.componentmodel.component.site?view=netframework-4.8#System_ComponentModel_Component_Site)|获取或设置 ISite 的 Component。 (继承自 Component)|

##  方法

|方法|作用|
|:--|:--|
|Dispose()|释放 Component 使用的所有资源。 (继承自 Component)
|DownloadData(String)|从指定 URI 下载资源作为 Byte 数组。
|DownloadFile(String, String)|将具有指定 URI 的资源下载到本地文件。
|DownloadString(String)|以 String 形式下载请求的资源。 以包含 URI 的 String 的形式指定要下载的资源。
|OpenRead(String)|为从具有 String 指定的 URI 的资源下载的数据打开一个可读的流。
|OpenWrite(String)|打开一个流以将数据写入指定的资源。
|UploadData(String, Byte[])|将数据缓冲区上载到由 URI 标识的资源。
|UploadFile(String, String)|将指定的本地文件上载到具有指定 URI 的资源。
|UploadString(String, String)|使用 POST 方法将指定的字符串上载到指定的资源.

# 获取网站API
以B站为例，打开个人中心，打开开发者模式，选择Network，按下<kbd>ctrl+r</kbd>![在这里插入图片描述](https://img-blog.csdnimg.cn/20200206154533666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9qdW42NjY=,size_16,color_FFFFFF,t_70)
复制以上链接地址。来写一个简单的程序：
```csharp
using System;
using System.Text;
using System.IO;
using System.Net;


namespace test
{
    class Programe
    {
        static void Main(string[] args)
        {//https://api.bilibili.com/x/relation/stat?vmid=352047271&jsonp=jsonp&callback=__jp4
        //https://api.bilibili.com/x/space/upstat?mid=352047271&jsonp=jsonp&callback=__jp5
            WebClient wc = new WebClient();
            wc.Encoding = Encoding.UTF8;
            string txt = wc.DownloadString("https://api.bilibili.com/x/relation/stat?vmid=352047271");
            Console.WriteLine("stat:"+txt);
            txt = wc.DownloadString("https://api.bilibili.com/x/space/upstat?mid=352047271");
            Console.WriteLine("upstat:"+txt);
            //结果为：
            // stat:{"code":0,"message":"0","ttl":1,"data":{"mid":352047271,"following":19,"whisper":0,"black":0,"follower":5}}
            // upstat:{"code":0,"message":"0","ttl":1,"data":{"archive":{"view":957},"article":{"view":0},"likes":26}}
        }
    }
}
```
以上两个字符串表明我关注了19人，有5个粉丝，播放量957，有26个赞。通过一个正则表达式即可提取出来。
> 这是鄙人的B站账号：==[斗木君](https://space.bilibili.com/352047271?from=search&seid=7402953021674044678)==,欢迎关注😀
