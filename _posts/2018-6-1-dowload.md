---
title: Servlet实现文件的上传与下载
date: 2018-5-31 20:30:10
categories:
- java web,Servlet
# comments: true
---
# 利用Servlet技术实现文件的上传和下载
在我们日常上网的时候会经常碰到下载，或者上传文件，相信很久之前就从网页上面下载过文件了吧，也在个中作业提交系统上面提交过文件吧？那么这其中的原理是什么的？我们应该怎么去实现呢？

+ 利用servletContext寻找路径
+ 利用文件拷贝代码模块
+ 用访问链接的形式去访问资源提供下载
## 利用Srvlet技术实现客户端下载服务器端文件
效果图如下：
![](http://xiaolitongxue.top/Servlet%E5%AE%9E%E7%8E%B0%E6%96%87%E4%BB%B6%E7%9A%84%E4%B8%8B%E8%BD%BD-1.png)
### 第一种方式-直接利用<a />标签来实现文件的下载
Servlet代码部分：
```java
String path = this.getServletContext().getRealPath("download/filename.jpg");
/**
* 获得该文件的输入，输出流 通过response获得的输出流，用于向客户端写内容
*/
InputStream in = new FileInputStream(path);
ServletOutputStream out = response.getOutputStream();
/**
* 文件拷贝代码（可以封装成模板）多次使用
*/
int len = 0;
byte[] buffer = new byte[1024];
while ((len = in.read(buffer)) > 0) {
out.write(buffer,0,len);
}
in.close();
out.close();
```
HTML代码部分：
```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Title</title>
</head>

<body>
<h1>第一种方式：利用a标签直接指向服务器资源</h1>
<a href="/download/a.flv">a.flv</a><br />
<a href="/download/a.jpg">a.jpg</a><br />
<a href="/download/a.mp3">a.mp3</a><br />
<a href="/download/a.mp4">a.mp4</a><br />
<a href="/download/a.txt">a.txt</a><br />
<a href="/download/a.zip">a.zip</a>
<!--<h1 align="center">第一种直接贴链接的方式有缺陷，遇到不能解析的文件它才会显示下载否则仅仅打开</h1>-->
<h1>第一种直接贴链接的方式有缺陷，遇到不能解析的文件它才会显示下载否则仅仅打开</h1>
<hr>
</body>
</html>
```
__解释：这种方式是直接利用的HTML超链接的方法直接去服务器端寻找资源并实现下载，但是这样会有弊端。例如：__
![](http://xiaolitongxue.top/Servlet%E5%AE%9E%E7%8E%B0%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD-2.png)
所以说这种方法并不是最好的方法.
### 第二种方式，利用Servlet的respones和request实现文件下载
__原理：前台的文件链接上面多加一个参数，参数中包含需要下载文件的名称，后台利用request接受到前台发送的参数，然后用response来设置消息头来保证文件只实现下载而不是打开。（注意乱码问题）__
Servlet部分代码实现：
```java
package DownLoadServlet;

import sun.misc.BASE64Encoder;

import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.URLEncoder;

@WebServlet(name = "DownServlet")
public class DownServlet extends HttpServlet {
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
doGet(request,response);
}

protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
/**
* 1.获取需要下载文件的文件名
*/
String fileName = request.getParameter("filename");
fileName = new String(fileName.getBytes("ISO8859-1"), "UTF-8");
/**
* 利用ServletContext获得文件的绝对路径
*/
String path = this.getServletContext().getRealPath("download/" + fileName);

//获得请求头中的User-Agent
String agent = request.getHeader("User-Agent");
//根据不同浏览器进行不同的编码
String filenameEncoder = " ";
if (agent.contains("MSIE")) {
// IE浏览器
filenameEncoder = URLEncoder.encode(fileName, "utf-8");
filenameEncoder = filenameEncoder.replace("+", " ");
} else if (agent.contains("Firefox")) {
// 火狐浏览器
BASE64Encoder base64Encoder = new BASE64Encoder();
filenameEncoder = "=?utf-8?B?"
+ base64Encoder.encode(fileName.getBytes("utf-8")) + "?=";
} else {
// 其它浏览器
filenameEncoder = URLEncoder.encode(fileName, "utf-8");
}

/**
* 要下载文件的类型---客户端通过文件的mime类型去区分
*/
response.setContentType(this.getServletContext().getMimeType(fileName));
/**
* 设置头信息-告诉客户端该文件是不是直接解析，而是以附件形式打开（下载）
*/
response.setHeader("Content-Disposition","attachement;filename="+filenameEncoder);

/**
* 获得该文件的输入，输出流 通过response获得的输出流，用于向客户端写内容
*/
InputStream in = new FileInputStream(path);
ServletOutputStream out = response.getOutputStream();
/**
* 文件拷贝代码（可以封装成模板）多次使用
*/
int len = 0;
byte[] buffer = new byte[1024];
while ((len = in.read(buffer)) > 0) {
out.write(buffer,0,len);
}
in.close();
out.close();
}
}

```

HTML代码部分：
```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Title</title>
</head>

<body>
<h1>第一种方式：利用a标签直接指向服务器资源</h1>
<a href="/download/a.flv">a.flv</a><br />
<a href="/download/a.jpg">a.jpg</a><br />
<a href="/download/a.mp3">a.mp3</a><br />
<a href="/download/a.mp4">a.mp4</a><br />
<a href="/download/a.txt">a.txt</a><br />
<a href="/download/a.zip">a.zip</a>
<!--<h1 align="center">第一种直接贴链接的方式有缺陷，遇到不能解析的文件它才会显示下载否则仅仅打开</h1>-->
<h1>第一种直接贴链接的方式有缺陷，遇到不能解析的文件它才会显示下载否则仅仅打开</h1>
<hr>
<h1>第二种方式：利用服务器端编码实现下载文件</h1>
<a href="/DownServlet?filename=a.flv">a.flv</a><br />
<a href="/DownServlet?filename=a.jpg">a.jpg</a><br />
<a href="/DownServlet?filename=a.mp3">a.mp3</a><br />
<a href="/DownServlet?filename=a.mp4">a.mp4</a><br />
<a href="/DownServlet?filename=a.txt">a.txt</a><br />
<a href="/DownServlet?filename=a.zip">a.zip</a><br/>
<a href="/DownServlet?filename=嗨.jpg">嗨.jpg</a><br/>

</body>
</html>
```
__代码中的解释比较详细，这里就不多再解释代码了__


----
## 利用Servlet实现文件从客户端上传到服务器端
图片演示结果：
![](http://xiaolitongxue.top/Servlet%E5%AE%9E%E7%8E%B0%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A02.png)
![](http://xiaolitongxue.top/Servlet%E5%AE%9E%E7%8E%B0%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E5%8A%9F%E8%83%BD1.png)

代码实现：
```java
package UpLoadServlet;

import org.apache.commons.fileupload.FileItem;
import org.apache.commons.fileupload.disk.DiskFileItemFactory;
import org.apache.commons.fileupload.servlet.ServletFileUpload;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.*;
import java.util.List;
@WebServlet(name = "UpLoadServlet")
public class UpLoadServlet extends HttpServlet {
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
doGet(request, response);

}

protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
request.setCharacterEncoding("utf-8"); //设置编码

//获得磁盘文件条目工厂
DiskFileItemFactory factory = new DiskFileItemFactory();
//获取文件需要上传到的路径
// String path = request.getRealPath("/upload");
// String path = this.getServletContext().getRealPath("upload");
String path ="E:\\Java Pro\\J2EE\\WebResponse\\web\\upload";
//如果没以下两行设置的话，上传大的 文件 会占用 很多内存，
//设置暂时存放的 存储室 , 这个存储室，可以和 最终存储文件 的目录不同
/**
* 原理 它是先存到 暂时存储室，然后在真正写到 对应目录的硬盘上，
* 按理来说 当上传一个文件时，其实是上传了两份，第一个是以 .tem 格式的
* 然后再将其真正写到 对应目录的硬盘上
*/
factory.setRepository(new File(path));
//设置 缓存的大小，当上传文件的容量超过该缓存时，直接放到 暂时存储室
factory.setSizeThreshold(1024*1024) ;

//高水平的API文件上传处理
ServletFileUpload upload = new ServletFileUpload(factory);

try {
//可以上传多个文件
// List<FileItem> list = (List<FileItem>)upload.parseRequest(request);
List<FileItem> list = upload.parseRequest(request);
for(FileItem item : list)
{
//获取表单的属性名字
String name = item.getFieldName();

//如果获取的 表单信息是普通的 文本 信息
if(item.isFormField())
{
//获取用户具体输入的字符串 ，名字起得挺好，因为表单提交过来的是 字符串类型的
String value = item.getString() ;
request.setAttribute(name, value);
}
//对传入的非 简单的字符串进行处理 ，比如说二进制的 图片，电影这些
else
{
/**
* 以下三步，主要获取 上传文件的名字
*/
//获取路径名
String value = item.getName() ;
//索引到最后一个反斜杠
int start = value.lastIndexOf("\\");
//截取 上传文件的 字符串名字，加1是 去掉反斜杠，
String filename = value.substring(start+1);

request.setAttribute(name, filename);

//真正写到磁盘上
//它抛出的异常 用exception 捕捉

//item.write( new File(path,filename) );//第三方提供的

//手动写的
OutputStream out = new FileOutputStream(new File(path,filename));

InputStream in = item.getInputStream() ;

int length = 0 ;
byte [] buf = new byte[1024] ;

System.out.println("获取上传文件的总共的容量："+item.getSize()+" b");

// in.read(buf) 每次读到的数据存放在 buf 数组中
while( (length = in.read(buf) ) != -1)
{
//在 buf 数组中 取出数据 写到 （输出流）磁盘上
out.write(buf, 0, length);

}

in.close();
out.close();
}
}
} catch (Exception e) {
// TODO Auto-generated catch block
e.printStackTrace();
}
//显示页面
// request.getRequestDispatcher("filedemo.jsp").forward(request, response);
}
}
```
JSP页面部分：
```html
<%--
Created by IntelliJ IDEA.
User: 育 Danger
Date: 2018/5/29
Time: 11:20
To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
<title>上传文件</title>
</head>
<body>
<!-- enctype 默认是 application/x-www-form-urlencoded -->
<form action="/upload" enctype="multipart/form-data" method="post" >

用户名：<input type="text" name="usename"> <br/>
上传文件：<input type="file" name="file1"><br/>
上传文件： <input type="file" name="file2"><br/>
<input type="submit" value="提交"/>

</form>
</body>
</html>
```
__注意！！！！：在Servlet代码中我使用了绝对路径，请读者自行更改__

由于端水平一般暂时前端页面很简陋，请见谅.....