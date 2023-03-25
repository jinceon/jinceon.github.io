+++
title = "从零开始设计一个web框架"
date = "2023-03-25T13:07:42.6668061Z"
Description = "多年前写的半成品文章"
Tags = ["框架", "web"]
Categories = ["程序设计"]

+++
## 背景
在做web开发的时候，我们往往不会直接去处理http请求，而是使用各种各样的web框架。

如Java的SpringMVC、NodeJS的express/koa等。

以前总觉得这类框架很牛逼，而网上在解读这种框架的时候大多数是钻进细节从源代码入手。

带你去看各个类的设计和流转，用了什么牛逼的设计模式。

而我一向是认为技术是为需求服务的。单纯从技术角度去研究分析它，视角会非常低。

为什么框架会被设计成这样，是为了解决什么问题？我觉得从需求（问题）的角度出发会更接近设计者的思路。

另外，就我个人阅读源代码的习惯，我喜欢找框架最原始的版本，看框架的雏形，这时它的脉络往往也是最清晰的。

## 起航

根据HTTP协议，服务器接受到的请求大概是这样的：

```http
POST /a?b=1&c=2 HTTP/1.1
Host: www.jinceon.com
Content-Type: application/x-www-form-urlencoded
cache-control: no-cache
d=3&e=4
```
假设我们已经有了一个很完美的网络处理+http报文解析工具，将上面的http报文解析成下面的json。

```js
req = {
  method: 'POST',
  url: '/a',
  query: { b: 1, c: 2 },
  body: { d: 3, e: 4 },
  headers: {
    host: 'www.jinceon.com',
    contentType: 'application/x-www-form-urlencoded',
    cacheControl: 'no-cache'
  }
}
```

一个项目当然不只一个接口啦，不同的接口会有不同的逻辑。
我们二话不说，写出了一堆if-else。

```js
function process(req, res){
    if(req.url === '/a'){
       res.write('逻辑a')
    }else if(req.url === '/b'){
       res.write('逻辑b')
    }else if(req.url === '/c'){
       res.write('逻辑c')
    }else{
       // 此
       // 处
       // 省
       // 略
       // 200
       // 个
       // if-else
    }
}
```

写完回头一看，这个方法长到一眼看不到尽头。
想想不行，得按功能模块拆分一下。

```js
// a.js
a = {
  '/a1':function(req, res){res.write(1)},
  '/a2':function(req, res){res.write(2)},
  '/a3':function(req, res){res.write(3)}
}
// b.js
b = {
  '/b1':function(req, res){res.write(1)},
  '/b2':function(req, res){res.write(2)},
  '/b3':function(req, res){res.write(3)}
}
// c.js
c = {
  '/c1':function(req, res){res.write(1)},
  '/c2':function(req, res){res.write(2)},
  '/c3':function(req, res){res.write(3)}
}
// app.js
app = {...a, ...b, ...c}  //将所有的路由收集汇总起来
function process(req, res){
   let fn = app[req.url]
   if(fn){
     fn(req, res)
   }
}
```
新需求来了，想遵守restful的url风格。
比如 `/users/123`, `/users/124`
得加上通配符的支持。
那`app.js`里`app[req.url]`就不好使了。
得改改，在收集路由的时候，得把路由改成正则表达式。

```js
//app.js
app = []
for(let module in a,b,c){
  for(let url in module){
     let regex = url2regex(url)
     app.push({regex:regex, fn:module[url]})
  }
}
function url2regex(url){
  // 假设将 /a 转成 /^\/a$/
  // 假设将 /a/* 转成 /^\/a\/(\w+)$/
  return regex;
}
function process(req, res){
  for(let route of app){
    if(route.regex.test(req.url)){
      route.fn(req, res)
    }
  }
}
```
需求源源不断的来。
1. 对于同一个url，比如 `/users/123`，当`GET`时是查询id=123的用户，POST时是新建用户用户。
2. 占位符解析，比如 `/users/*` 写成 `/users/:id`，当请求`/users/123`进来时，将匹配到的123赋值给id，通过 `req.params.id` 就可以得到 `123`。
3. 希望对所有请求都加一个前置拦截，判断用户有没有登录，有没有权限。
4. 加后置拦截，等等。  

随着一个个问题（新需求）的到来，这个web框架就会越来越完善，最终大多数会殊途同归到如今流行的各种web框架了。
