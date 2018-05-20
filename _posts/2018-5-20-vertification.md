---
title: 用Java服务器代码自动生成验证码功能
date: 2018-5-19 11:11:10
categories:
- java web
# comments: true
---
# 为什么我们要生成验证码？
在我们登录的一般都可以看见需要手机验证或者是验证码登录注册，那么这是为什么呢？我粗略统计了以下几个原因：
+ 有一些机构需要实名制认证，所以必须电话短信验证，以及为了后期的账号安全也需要手机验证。
+ 防止用户使用恶意程序登录，如果你没有设置验证码的话，有一些黑客很可能会利用一些代码或者漏洞来无限制次数登录等等，这将对你的服务器产生巨大的影响。
+ 从生活经验我们经常发现一些验证码样子十分奇怪，好像特意不让别人看清一样，其实这是为了防止别人使用扫描程序轻易获得到验证码。

# 验证码样式展示：
实现每次点击验证码图片即可更换一张图片功能
![](http://p8i28834i.bkt.clouddn.com/Vertification-1.png)
实现点击文字更换图片
![](http://p8i28834i.bkt.clouddn.com/vertification-3.png)
# 实现步骤分析：
![](http://p8i28834i.bkt.clouddn.com/vertification-2.png)
# 代码实现：
## servlet部分：
```java
/**
*核心servlet类实现读取new_word文件中的成语
然后处理这些成语是之生成验证码图片。
*/
package Checking;

import java.awt.Color;
import java.awt.Font;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.image.BufferedImage;
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

import javax.imageio.ImageIO;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
* 验证码生成程序
*
*
*
*/
public class CheckImageServlet extends HttpServlet {

// 集合中保存所有成语
private List<String> words = new ArrayList<String>();

@Override
public void init() throws ServletException {
// 初始化阶段，读取new_words.txt
// web工程中读取 文件，必须使用绝对磁盘路径
String path = getServletContext().getRealPath("/WEB-INF/new_word.txt");
try {
BufferedReader reader = new BufferedReader(new FileReader(path));
String line;
while ((line = reader.readLine()) != null) {
words.add(line);
}
reader.close();
} catch (IOException e) {
e.printStackTrace();
}
}

public void doGet(HttpServletRequest request, HttpServletResponse response)
throws ServletException, IOException {
// 禁止缓存
// response.setHeader("Cache-Control", "no-cache");
// response.setHeader("Pragma", "no-cache");
// response.setDateHeader("Expires", -1);

int width = 120;
int height = 30;

// 步骤一 绘制一张内存中图片
BufferedImage bufferedImage = new BufferedImage(width, height,
BufferedImage.TYPE_INT_RGB);

// 步骤二 图片绘制背景颜色 ---通过绘图对象
Graphics graphics = bufferedImage.getGraphics();// 得到画图对象 --- 画笔
// 绘制任何图形之前 都必须指定一个颜色
graphics.setColor(getRandColor(200, 250));
graphics.fillRect(0, 0, width, height);

// 步骤三 绘制边框
graphics.setColor(Color.WHITE);
graphics.drawRect(0, 0, width - 1, height - 1);

// 步骤四 四个随机数字
Graphics2D graphics2d = (Graphics2D) graphics;
// 设置输出字体
graphics2d.setFont(new Font("宋体", Font.BOLD, 18));

Random random = new Random();// 生成随机数
int index = random.nextInt(words.size());
String word = words.get(index);// 获得成语

// 定义x坐标
int x = 10;
for (int i = 0; i < word.length(); i++) {
// 随机颜色
graphics2d.setColor(new Color(20 + random.nextInt(110), 20 + random
.nextInt(110), 20 + random.nextInt(110)));
// 旋转 -30 --- 30度
int jiaodu = random.nextInt(60) - 30;
// 换算弧度
double theta = jiaodu * Math.PI / 180;

// 获得字母数字
char c = word.charAt(i);

// 将c 输出到图片
graphics2d.rotate(theta, x, 20);
graphics2d.drawString(String.valueOf(c), x, 20);
graphics2d.rotate(-theta, x, 20);
x += 30;
}

// 将验证码内容保存session
request.getSession().setAttribute("checkcode_session", word);

// 步骤五 绘制干扰线
graphics.setColor(getRandColor(160, 200));
int x1;
int x2;
int y1;
int y2;
for (int i = 0; i < 30; i++) {
x1 = random.nextInt(width);
x2 = random.nextInt(12);
y1 = random.nextInt(height);
y2 = random.nextInt(12);
graphics.drawLine(x1, y1, x1 + x2, x2 + y2);
}

// 将上面图片输出到浏览器 ImageIO
graphics.dispose();// 释放资源

//将图片写到response.getOutputStream()中
ImageIO.write(bufferedImage, "jpg", response.getOutputStream());

}

public void doPost(HttpServletRequest request, HttpServletResponse response)
throws ServletException, IOException {
doGet(request, response);
}

/**
* 取其某一范围的color
*
* @param fc
* int 范围参数1
* @param bc
* int 范围参数2
* @return Color
*/
private Color getRandColor(int fc, int bc) {
// 取其随机颜色
Random random = new Random();
if (fc > 255) {
fc = 255;
}
if (bc > 255) {
bc = 255;
}
int r = fc + random.nextInt(bc - fc);
int g = fc + random.nextInt(bc - fc);
int b = fc + random.nextInt(bc - fc);
return new Color(r, g, b);
}

}

```
## HTML部分：
```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>验证码登录测试页面</title>
<script type="text/javascript">
window.onload = function (ev) {
}
function changeImg(obj) {
// 利用参数，实现点击更换验证码功能参数不能一样，所以利用取当前时间的方式。
obj.src = "/Checking?time=" + new Date().getTime();
}
</script>
</head>
<body>
<form action="" method="post">
用户名：<input type="text" name="username"><br/>
密码：<input type="password",name="password"><br>
验证码: <input type="text" name="username">
<!--<img onclick="changeImg(this)" src="/Checking">-->
<img id="image" src="/Checking">
<a href="#" onclick="changeImg(image)">看不清,换一张</a>
<br/>
<input type="submit" value="登录" ><br>

</form>
</body>
</html>
```
__JavaScript代码很重要,利用JavaScript代码实现点击更换图片的功能__
## 成语文本文件部分
![](http://p8i28834i.bkt.clouddn.com/vertificaion-4.png)

生成验证码功能完成，目前还没有实现验证登录核实验证码。