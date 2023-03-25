![http请求消息格式](http-request-message-format.png)

根据HTTP协议，服务器接受到的请求大概是这样的：

```http
POST /a?b=1&c=2 HTTP/1.1
Host: www.jinceon.com
Content-Type: application/x-www-form-urlencoded
cache-control: no-cache
d=3&e=4
```
假设我们已经有了一个很完美的解析工具，将上面的http报文解析成下面的json。

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
   一个个问题自己先思考，想想自己会怎样做，然后去看 express.js, koa.js, springmvc, struts 的代码的时候，对比下别人是怎样做的。