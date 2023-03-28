+++
title = "闲聊null==a"
date = "2023-03-28"

Description = ""
Tags = ["Java"]
Categories = ["程序开发"]

+++
## 故事
代码评审的时候，是不是还会看到有小伙伴写下`null == a`这样的代码。  
有时心血来潮，我会问一句，为什么这样写？  
如果他回答里有`防止空指针`的话，我可能会对他有点失望。  

## 剖析
为什么会出现这样的写法？防止空指针又是什么骚操作？  
我们来看看下面的两个例子。

```java
String a = null;

a.equals("something");   // ---> null.equals("something")
"something".equals(a);   // ---> "something".equals(null);     
```

显然null是没有`equals`方法的，所以会出现NullPointerException。  
这种场景下把变量放在`equals`右侧确实可以避免NullPointerException或减少一次null判断。

再来看两个例子。
```java
String a = null;

a == null;    // ---> null == null
null == a;    // ---> null == null
```
我们会发现，`对于Java来说`变量写在`==`左边或右边都没有区别。  

所以对于`Java程序员`来说，一刀切地把变量写在比较符右边就是一种典型的想当然。  

我猜也许发展过程是这样：  
一开始的时候，写`a.equals("something");`出现过NullPointerException。  
这时有前辈指导他写成`"something".equals(a);`。  
然后他自己死记硬背记住了，把变量写在`比较符`右边。  
最后把比较符从`equals`扩展到`==`。

## 彩蛋
眼尖的小伙伴可能发现了，上面特意高亮了`对于Java来说`等字眼，
其他语言难道有不一样吗？

还真是！比如javascript
```javascript
if(a == null){
    console.log(1)
}else{
    console.log(2)
}
```
如果`==`漏写了个`=`，代码就变成了
```javascript
if(a = null){
    console.log(1)
}else{
    console.log(2)
}
```
这段代码在编译时不会报错，在执行时相当于把 `==` 相等判断变成了 `=` 变量赋值语句。