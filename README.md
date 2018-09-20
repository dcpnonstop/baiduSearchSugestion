 # jsonp跨域获取数据实现百度搜索
 ---
 ### [demo在线演示](https://dcpnonstop.github.io/baiduSearchSugestion/)
## 简单聊聊跨域
跨域是由同源策略引起的，同源策略/SOP（Same origin policy）是一种约定，由Netscape公司1995年引入浏览器，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到XSS、CSFR等攻击。所谓同源是指"协议+域名+端口"三者相同，即便两个不同的域名指向同一个ip地址，也非同源。


同源策略限制以下几种行为：
> 1.Cookie、LocalStorage 和 IndexDB 无法读取

>2.DOM 和 Js对象无法获得

>3.AJAX 请求不能发送

假设没有同源策略，那么我在A网站下的cookie就可以被任何一个网站拿到；那么这个网站的所有者，就可以使用我的cookie(也就是我的身份)在A网站下进行操作。同源策略可以算是 web 前端安全的基石，如果缺少同源策略，浏览器也就没有了安全性可言。


同源策略做了很严格的限制，但是在实际的场景中，又确实有很多地方需要突破同源策略的限制，也就是我们常说的跨域。
跨域的方法有很多（如接下来要玩的jsonp跨域，还有cors跨域资源共享，反向代理等等）。

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

