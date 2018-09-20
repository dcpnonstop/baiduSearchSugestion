 # jsonp跨域获取数据实现百度搜索
 ---
 ### [demo在线演示](https://dcpnonstop.github.io/baiduSearchSugestion/)
## 简单聊聊跨域
跨域问题是由于JavaScript语言安全限制中的同源策略/SOP（Same origin policy）造成的，同源策略是指一段脚本只能读取来自同一来源的窗口和文档的属性,由Netscape公司1995年引入浏览器，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到XSS、CSFR等攻击。所谓同源是指"协议+域名+端口"三者相同，即便两个不同的域名指向同一个ip地址，也非同源。同源策略保证用户信息安全，防止恶意的网站窃取数据。

一个域名的组成：

 |  http:// |www  |abc.com|:8080|/script/iquire.js|
 | --------   | -----:   | -----:   | -----:   | :----: |
 | 协议| 子域名| 主域名| 端口号| 请求资源地址|

当**协议、子域名、主域名、端口号**中任意一个不相同时，都算作不同的域。不同的域之间相互请求资源，属于**跨域**
<!--more-->

同源策略限制以下几种行为：
> 1.Cookie、LocalStorage 和 IndexDB 无法读取

>2.DOM 和 Js对象无法获得

>3.AJAX 请求不能发送

假设没有同源策略，那么我在A网站下的cookie就可以被任何一个网站拿到；那么这个网站的所有者，就可以使用我的cookie(也就是我的身份)在A网站下进行操作。同源策略可以算是 web 前端安全的基石，如果缺少同源策略，浏览器也就没有了安全性可言。


同源策略做了很严格的限制，但是在实际的场景中，又确实有很多地方需要突破同源策略的限制，也就是我们常说的跨域。

#### 解决跨域的几种方式
1.    document.domain + iframe      (只有在主域相同的时候才能使用该方法)。两个网页一级域名相同，二级域名不同，为了共享cookies，可以设置document.domain共享cookie
2.	 location.hash + iframe 原理是利用location.hash来进行传值。假设域名a.com下的文件cs1.html要和cnblogs.com域名下的cs2.html传递信息。1) cs1.html首先创建自动创建一个隐藏的iframe，iframe的src指向cnblogs.com域名下的cs2.html页面;2) cs2.html响应请求后再将通过修改cs1.html的hash值来传递数据;3) 同时在cs1.html上加一个定时器，隔一段时间来判断location.hash的值有没有变化，一旦有变化则获取获取hash值
注：由于两个页面不在同一个域下IE、Chrome不允许修改parent.location.hash的值，所以要借助于a.com域名下的一个代理iframe

3.	window.name + iframe 浏览器窗口有window.name属性。这个属性的最大特点是，无论是否同源，只要在同一个窗口里，前一个网页设置了这个属性，后一个网页可以读取它。
4. postMessage（HTML5中的XMLHttpRequest Level 2中的API）	跨文档通信API（Cross-document messaging）这个API为window对象新增了一个window.postMessage方法，允许跨窗口通信，不论这两个窗口是否同源。
5.	同源政策规定，AJAX请求只能发给同源的网址，否则就报错，针对该限制：
JSONP(JSONP是服务器与客户端跨源通信的常用方法它的基本思想是，网页通过添加一个```<script>```元素，向服务器请求JSON数据，这种做法不受同源政策限制；服务器收到请求后，将数据放在一个指定名字的回调函数里传回来), jsonp虽然很简单，但是有如下缺点：1）安全问题(请求代码中可能存在安全隐患)2）要确定jsonp请求是否失败并不容易
6. websocket(一种通信协议，使用ws://（非加密）和wss://（加密）作为协议前缀。该协议不实行同源政策，只要服务器支持，就可以通过它进行跨源通信),web sockets是一种浏览器的API，它的目标是在一个单独的持久连接上提供全双工、双向通信。(同源策略对web sockets不适用)web sockets原理：在JS创建了web socket之后，会有一个HTTP请求发送到浏览器以发起连接。取得服务器响应后，建立的连接会使用HTTP升级从HTTP协议交换为web sockt协议。
7. CORS(ORS是跨源资源分享（Cross-Origin Resource Sharing）的缩写。它是W3C标准，是跨源AJAX请求的根本解决方法。相比JSONP只能发GET请求，CORS允许任何类型的请求。)CORS背后的思想，就是使用自定义的HTTP头部让浏览器与服务器进行沟通，从而决定请求或响应是应该成功，还是应该失败。


## 使用jsonp跨域


由于同源策略，一般来说位于 server1.example.com 的网页无法与不是 server1.example.com的服务器沟通，而HTML的<script> 元素是一个例外。利用<script>元素的这个开放策略，网页可以得到从其他来源动态产生的 JSON资料，而这种使用模式就是所谓的 JSONP。用 JSONP 抓到的资料并不是 JSON，而是任意的JavaScript，用 JavaScript 直译器执行而不是用 JSON 解析器解析。


示例代码

```
function handleResponse(response) {
   alert(`You get the data : ${response}`)
}
const script = document.createElement('script')
script.src = 'http://somesite.com/json/?callback=handleResponse'
document.body.insertBefore(script, document.body.firstChild)

```

这里的callback回调函数很重要，动态添加在body中的script标签可以使用被加载的文件与HTML文件下的其他JS文件共享一个全局作用域。也就是说，<scritp>标签加载到的资源是可以被全局作用域下的函数所使用的！


# 利用jsonp跨域获取数据实现百度搜索

百度有一个对外暴露的数据接口：https://sp0.baidu.com/5a1Fazu8AA54nxGko9WTAnF6hhy/su?wd=1

在chrome浏览器中打开百度主页，在开发者工具在 netkwork 可以找到:
![图片1](https://user-gold-cdn.xitu.io/2018/6/24/164326dfdd4cb1fa?imageslim)


我们可以直接拿来使用，配合jsonp就能实现跨域获取输入框内容相关热点数据并点击跳转了，具体实现请看Github项目源码

#### 实现效果：
![图片2](https://user-gold-cdn.xitu.io/2018/6/24/164324308cb57b18?imageslim)


页面结构非常简单，如图:


![图片2](https://user-gold-cdn.xitu.io/2018/6/24/164323b50e137b4d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

jsonp跨域实现代码：

```
  document.onkeyup = function () {
    var val = text.value
    var script = document.createElement('script')
    script.src = `https://sp0.baidu.com/5a1Fazu8AA54nxGko9WTAnF6hhy/su?wd=${val}&cb=dosomething`;
    document.body.appendChild(script)
  }
  function dosomething (data) {
    var oUl = document.querySelector('#lists ul')
    oUl.innerHTML = ''
    data.s.map(function (html) {
      var oLi =  document.createElement('li')
      oLi.innerHTML = html
      oLi.onclick = function () {
        window.location.href = `http://www.baidu.com/s?wd=${html}`
      }
      oUl.appendChild(oLi)
    })
}

```


仅仅是一个利用jsonp实现跨域的简单小demo，便于和我一样的新手学习

