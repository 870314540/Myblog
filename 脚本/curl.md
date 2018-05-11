##curl

###1简介

curl命令是一个利用URL规则在命令行下工作的文件传输工具。它支持文件的上传和下载，所以是综合传输工具，但按传统，习惯称curl为下载工具。

作为一款强力工具，curl支持包括HTTP、HTTPS、ftp等众多协议，还支持POST、cookies、认证、从指定偏移处下载部分文件、用户代理字符串、限速、文件大小、进度条等特征。做网页处理流程和数据检索自动化，curl可以祝一臂之力。



###2命令及示例：太强大，用到什么学什么
【语法】 
    curl [options] [URL...] 

- curl -d  “value=value”  url  :  post 请求

-d/--data <data>    为 POST 数据包指定要向 HTTP 服务器发送的数据并发送出去。这个过程和在浏览器中点击“submit”按钮是一样的，且数据将以 `content-type application/x-www-form-urlencoded `的方式被编码。 

在同一条命令中多次使用 -d 参数，参数后的内容会被符号 & 拼接起来。比如 `-d name=daniel -d skill=lousy `会被转化为 `name=daniel&skill=lousy` 
                                   
如果<data>的内容以符号 `@`开头，其后的字符串将被解析为文件名，curl 命令会从这个文件中读取数据发送，但文件中的内容必须是 URL 编码的格式；如果内容以符号 - 开头，curl 命令将从 stdin 读取数据发送
                                   


```
curl -d "jdbId=11111"  http://localhost:8080/log

返回：
{"error":{"returnCode":1,"returnMessage":"startTime  is null","returnUserMessage":"startTime  is null"},"data":null,"logId":"03535170960027"}

```





###3问题整理




