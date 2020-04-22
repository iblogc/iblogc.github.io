title: curl常用命令
comments: true
date: 2015-12-17 21:29:29
categories: [程序猿]
tags: [curl, 教程]
---
curl是利用URL语法在命令行方式下工作的开源文件传输工具。它被广泛应用在Unix、多种Linux发行版中，并且有DOS和Win32、Win64下的移植版本。
<!--more-->
### 访问url并输出结果
```
curl http://www.example.com
```

### 传递参数
默认curl使用GET方式请求数据，这种方式下直接通过URL传递数据
可以通过 --data/-d 方式指定使用POST方式传递数据
```
# GET
curl -u username https://api.github.com/user?access_token=XXXXXXXXXX

# POST
curl -u username -d "param1=value1&param2=value" https://api.github.com

# 也可以指定一个文件，将该文件中的内容当作数据传递给服务器端
curl -d @filename https://github.api.com/authorizations
```
注：默认情况下，通过POST方式传递过去的数据中若有特殊字符，首先需要将特殊字符转义在传递给服务器端，如value值中包含有空格，则需要先将空格转换成%20，如：
```
curl -d "value%201" http://hostname.com
```
在新版本的CURL中，提供了新的选项 --data-urlencode，通过该选项提供的参数会自动转义特殊字符。
```
curl --data-urlencode "value 1" http://hostname.com
```

### 指定请求方式
除了使用GET和POST协议外，还可以通过 -X 选项指定其它协议，如：
```
curl -I -X DELETE https://api.github.com
```

### 设置请求头信息
``` 
curl -H 'Accept-Language: zh' http://cnn.com
```
`-H`或被多次指定
``` 
curl -H 'Host: 157.166.226.25'-H 'Accept-Language: zh'-H 'Cookie: ID=1234' http://cnn.com
```
对于"User-Agent", "Cookie", "Host"这类标准的HTTP头部字段，通常会有另外一种设置方法。curl命令提供了特定的选项来对这些头部字段进行设置：
-A (or --user-agent): 设置 "User-Agent" 字段.
-b (or --cookie): 设置 "Cookie" 字段.
-e (or --referer): 设置 "Referer" 字段. 
```
curl -H "User-Agent: my browser" http://cnn.com
curl -A "my browser" http://cnn.com
```

### 查看响应头信息
```
curl -I http://www.baidu.com
```



### 提交表单
```
curl --form "fileupload=@filename.txt" http://hostname/resource
```

### 访问url并奖结果保存到本地文件中
`-o`: 将文件保存为命令行中指定的文件名到本地
`-O`: 使用url中默认的文件名保存文件到本地
```
curl -o index.html http://www.example.com
# 或
curl  http://www.baidu.com > index.html
# 在windows上没成功
curl -O http://www.example.com
```

### 忽略证书错误
工作中，经常需要用自签的假证书搭建开发环境。cURL在遇到证书错误时罢工，使用 -k 参数就可以让它不做证书校验。
```
curl -k https://www.example.com
```

### 获取重定向后的页面
如果url重定向的话，curl默认是不会去获取重定向后的url页面的，使用`-L`可进行强制重定向
```
curl -L http://www.example.com
```

### 发送压缩的请求
 
cURL提供了一个 –compress 参数，可以用来发送支持压缩的请求。但使用了–compress之后，虽然传输过程是压缩的，cURL的输出还是解压之后的，难以看到效果。
 
自己写一个 Accept-Encoding 字段在头信息中。
```
curl -H "Accept-Encoding: gzip" http://www.kuqin.com/
```

如果直接运行上面的命令，会得到一堆乱码，因为cURL输出的内容，是压缩后的数据。不妨在后面接一个gunzip试试。
```
# 使用gunzip解压
curl -H "Accept-Encoding: gzip" http://www.kuqin.com/ | gunzip
```

使用gunzip解压之后，信息又被还原了。

### 断点续传
通过使用-C选项可对大文件使用断点续传功能
```
# 未下载完成即中断该进程
curl -o a.zip http://www.example.com/bigfile.zip

# 后面可以通过-C来继续下载
curl -C -o a.html http://www.example.com/bigfile.zip
```

### 下载限速
使用-limit-rate进行限速
```
# 限速为100k/s
curl --limit-rate 1000k -o a.zip http://www.example.com/bigfile.zip
```

### 根据文件修改时间来判断是否进行下载
```
# 若文件的修改时间在2011/12/11之后，则下载
curl -z 21-Dec-11 http://www.example.com/bigfile.zip
```

### 授权
在访问需要授权的页面时，可通过`-u`来提供用户名和密码进行授权
```
curl -u username:password http://www.example.com
```

### ftp操作
```
# 列出指定目录下的所有文件
curl -u ftpuser:ftppw -O ftp://ftp_server/public_html/

# 下载文件
curl -u ftpuser:ftppw -O
ftp://ftp_server/public_hmtl/bigfile.zip

# 上传文件
curl -u ftpuser:ftppw -T myfile.txt ftp://ftp_server/public_html/

# 上传多个文件
curl -u ftpuser:ftppw -T "{myfile1.txt, myfile2.txt}" ftp://ftp_server/public_html/

# 从标准输入获取内容保存到服务器的指定文件中
curl -u ftpuser:ftppw -T - ftp://ftp_server/public_html/1.txt
```

### 设置代理
```
curl -x proxyserver.com:1080 http://www.example.com
```

### 保存与使用网站的cookie信息
```
# 将网站的cookies信息保存到example_cookies文件中
curl -D example_cookies http://www.example.com

# 使用cookies信息访问url
curl -b example_cookies http://www.example.com/user/
```


