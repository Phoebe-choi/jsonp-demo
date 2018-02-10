
# 一个简易的jsonp跨域请求


### 页面数据交互的发展过程

付款是我们日常中常见的一种金钱交易，用户在页面中点击付款按钮，网页提交请求给服务器，服务器收到请求后在数据库对金额进行扣减，然后将消息返回给页面，告诉用户给付款成功。

在讲JSONP之前，我们先看看以前的前端程序员是用什么办法实现一个网页数据交互过程的。

### 用form表单向服务器发起请求 

- 页面代码如下：用表单提交post请求，请求路径为/pay
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h5>您的账户余额是<span id="amount">&&&amount&&&</span></h5>
<form action="/pay" method="post">//用表单提交post请求，请求路径为/pay
    <input type="submit" value="付款">
</form>
</body>
</html>
```
- 服务器代码

当请求的路径为/pay时，数据库进行扣减金额一块钱，然后给页面返回success
```
if(path === '/pay'&& method.toUpperCase()==='POST'){
//如果的路径是path，而且你的方法是post
    var amount = fs.readFileSync('./db', 'utf8') //100
    //那么我就去读一下你当前有多少钱
    var newAmount = amount - 1
    //然后把这个钱扣一块，扣了一块之后就是99
    fs.writeFileSync('./db', newAmount)
    //然后再把剩下的钱存到数据库里
    response.write('success')
    //然后告诉你扣费成功了
    response.end()
    //结束
  }
```
**这里有一个比较重要的细节**，在点击付款按钮的form表单，一旦提交了，就一定会刷新当前页面，不管是成功付款还是失败付款，都会刷新页面。这样用户体验效果就很差了，我们可以用另外一种方式优化一下，但是这个方式也是比较古老的，就是用`iframe`来实现。
```
<h5>您的账户余额是<span id="amount">&&&amount&&&</span></h5>
<form action="/pay" method="post" target="result">
    <input tpe="submit" value="付款">
</form>

<iframe name="result" src="about:blank" frameborder="0" height=200></iframe>
//使用iframe来实现，这个方法也很古老
```

### img发请求法
- 浏览器有一个特点，一旦发现你在内存里面创建了一个img，它就会去请求这个img的source.
- 这种办法有一个缺陷，就是没办法post，所以它只能get
- 使用用img发起一个请求，服务器会同时执行两句话

![](http://upload-images.jianshu.io/upload_images/8532417-e5e69b85e0da77d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/8532417-a3b602dbeb6540c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 页面代码
```
用图片发起get请求
button.addEventListener('click', (e) => {
    let image = document.createElement('img')
    image.src = '/pay'
    image.onload = function() {
      alert('打钱成功')
      amount.innerText = amount.innerText - 1
    }
    image.onerror = function() {
      alert('打钱失败')
    }
}
```
- 服务器代码
```
if(Math.random()>0.5{
    fs.writeFileSync('./db', newAmount)
    response.setHeader('Content-Type','image/ipg')
    response.statusCode = 200
    response.write('fs.writeFileSync('./dog.jpg'))
    //只能传送一张真的图片，才能让服务器成功发起请求
}else{
    response.status=400
    response.write('fail')
}
response.end()
```

### script发请求法(SRJ)
- 尝试用script发起请求，但是到服务器运行后发现没有丝毫变化

![](http://upload-images.jianshu.io/upload_images/8532417-1ded582b1149729d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过其他前端工程师不断地尝试，发现只有把`<script>`放到body(页面)里面，服务器才能发起请求，与img发起请求法不同，img是不需要这一步的，直接就可以向服务器发请求，而用`script`请求法是需要这一步的

```
button.addEventListener('click', (e) => {
let script = document.createElement('script')
    script.src = '/pay'
    document.body.appendChild(script)
    script.onload=function(){
        alert('success')
    }
    script.onerror = function() {
        alert('fail')
    }

```
```
if(path === '/pay'){
    var amount = fs.readFileSync('./db', 'utf8') //100
    var newAmount = amount - 1
    if(Math.random()>0.5{
        fs.writeFileSync('./db', newAmount)
        response.setHeader('Content-Type', 'application/javascript')
        response.statusCode = 200
    response.write('')
    }else{
        response.status=400
        response.write('fail')
    }
  response.end()
```

通过以上方法，我们可以用`script`成功向服务器发起请求，比用img发起请求法要快一些，不需要再返回一张图片了。

但是，这种方法有一个问题还没解决。就是当我们向服务器请求了`script`之后，这个`script`是会执行的呀。什么叫会执行？请看下图：

![](https://i.loli.net/2018/01/23/5a66cb3ac58b7.png
)

下图可以看到，当我们执行完多出来的`script`的内容后，才会继续执行onload事件(success)

![](http://upload-images.jianshu.io/upload_images/8532417-26efd460db1b223b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/8532417-0e19fde0827b9077.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当我们点击打钱，就会创建一个`script`标签，由于页面中多出了一个`script`，于是浏览器就会执行pay这个`script`里面的内容。也就是`alert`里面的"我是pay"，然后才继续执行onload事件。

既然先执行响应内容，然后再调用onload，连续触发两次alert事件，如此多此一举。我们为何不直接把所有的内容都放进pay里面？同时本地页面监听script标签的onload，onerror事件来删除动态创建的script标签，节省内存。

![](http://upload-images.jianshu.io/upload_images/8532417-68d4474b475406bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/8532417-9e25bb9cb909d129.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**这个方案就叫做SRJ(Server Rendered JavaScript)，服务器返回的JavaScript**

- 一些重要的操作，比如付款、取款，都不要用get来操作，一律都用post操作。

*以上就是一个成用数据库实现一个付款扣费的方案，和失败的付款扣费方案，但是上面的方法是旧的时代前后端用数据库配合的方法，下面我们用现在流行的方案再来实现一遍*

---

### JSONP
JSONP(JSON with Padding) 是 json 的一种"使用模式"，可以让网页从别的域名（网站）那获取资料，即跨域读取数据。

上面的SRJ方案有一个很大的缺陷，就是双方网站的代码耦合度太高，为了代码的解耦，可以使用JSONP方案。


JSONP完整过程：
- 请求方：前端程序员(浏览器)
>请求方动态创建一个`script`标签，`script`的src路径指向响应方，同时传一个查询参数callback函数，根据约定必须是callback，函数名可随机化

```
<script>
button.addEventListener('click', (e) => {
let script = document.createElement('script')
//创建一个随机的函数名
let functionName ='tina'+parseInt(Math.random()*100000,10);
script.src = 'http://xxx.com:8002/pay?callback=' + functionName; 
//xxx.com为响应网站的域名，8002为响应网站的端口号

window[callback] = function(result){ //定义函数
        if(result === 'success'){
            amount.innerText = amount.innerText - 1;
        }
    }
    script.onload = function(e){
        e.currentTarget.remove();//执行结束删除script标签
        delete window[callback]; //执行结束删除函数
    }
    script.onerror = function (e) {
        alert('fail');
        e.currentTarget.remove();//执行结束删除script标签
        delete window[callback];//执行结束删除函数
    }
})

```

- 响应方：后端程序员(服务器) 
>.响应方根据查询参数callback找到函数名后，回调这个函数，做出响应。有下面两种形式：<br>
>函数名.call(undefined,'你要的数据')<br>
>函数名('你要的数据')
```
//服务器端只需要这样就可以了，后台并不关心你写的函数名字
response.write(`
   ${query.callback}.call(undefined, 'success')
`)
```
浏览器接收到响应后，就会执行`函数名.call(undefined,'你要的数据')`<br>
那么请求方得到了响应的数据，就会返回到页面上来

大致流程如下：

>JSONP<br>
请求方：前端程序员(浏览器)<br>
响应方：后端程序员(服务器)<br>
>1. 请求方创建script标签，src路径指向响应方，同时传一个查询参数`?callback=' + functionName;`
>2. 响应方根据查询参数callback，构造形如下
>- 函数名.call(undefined,'你要的数据')
>- 函数名('你要的数据')<br>
>这样的响应
>3. 浏览器接收到响应后，就会执行`函数名.call(undefined,'你要的数据')`
>4. 那么请求方就知道了他要的数据<br>
>
>这就是JSONP

上面的JSONP太复杂了，有没有更简单的办法？有的，用jQuery实现

```
$.ajax({
  url: "http://想访问的另一个网站:端口号/pay",
  dataType: "jsonp",
  success: function (response) {
    if(response === 'success') {
    amount.innerText = amount.innerText - 1 
    }  
  }
})
```
这里值得注意一下JQuery封装的JSONP虽然起了一个与ajax相近的名字，但是与ajax没有任何关系，它只是一个动态的`script`，`script`标签是不受域名限制的，而Ajax是受域名限制的。

### JSON
- JSON的语法就一个特点，就是要加双引号，这一部分就叫JSON，JSON前面这一部分叫做左padding，右边叫做右padding，中间是jSON。这个技术名字就叫做：
>JSON + padding = JSONP  也可以理解为
>
>String + padding = StringP

![](https://i.loli.net/2018/01/23/5a67036c3c384.png
)

可以简单的理解为 带callback的json就是jsonp.

### 一个面试题

请问JSONP为什么不支持POST请求？
- JSONP通过动态创建script来实现的
- 动态创建script的时候，只能用get，没办法用post


完~


