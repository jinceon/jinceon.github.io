+++
title = "我为什么不喜欢MyBatis-Plus"
date = "2023-05-26"

Description = ""
Tags = [ "Java"]
Categories = ["程序开发"]

+++
我个人不喜欢MyBatis-Plus。

我觉得里面的很多设计都不好。挑2个典型的来说

## 1.代码生成器
程序是用来解决重复的，不是用来制造重复的。

你有100个表，要将其生成Java Entity文件，设计一个代码生成器来解决这些重复劳动，一点问题都没有，非常合理。

但是MyBatis-Plus在解决重复的同时，也生成了很多重复的代码。具体体现在生成了Controller、Service等代码。

为什么生成Entity是合理的，生成Controller、Service却是不合理的？

因为Entity的生成没法再抽象了。从生成的代码来看，每个Entity、每个Entity里的字段、类型都是不一样的，你无法再提取抽象了。

但Controller、Service呢？这些纯粹是重复的代码。按照代码重构改善坏味道的原则，这些相同的代码是可以提取抽象方法的。如果你的需求就是简单的crud，那最终的方案应该是类似spring data rest这种，一个controller加通配符和注解就可以搞定，而不是把controller代码复制100遍。

当然这个观点不一定被mybatis-plus的拥护者们认可。他们会反驳，我只是提供了工具，怎么用是用户的事。用户如果觉得不合理可以选择不用。

然而事实上，不反对就相当于是鼓励了。何况你还提供工具。

## 2.IService接口和ActiveRecord 模式的Model接口
学Java的人都学过，代码要分层。大家也按照这个模式来分了controller、service和dao(mapper)，但说实话，实际工作中没见过几个人分的对的。

或者不谈对错，就说在一个团队里，能否达成一致达成共识。

显然不能，我见过很多逻辑有些人认为应该写在service，有些人觉得应该写在dao，有些人觉得无所谓。

说到底，是大家对层的划分没有一个确切的原则。遇到这种带争议的，leader也只能来一句，就事论事，视实际情况而定。

有办法提炼出原则吗？

当然有，最简单的办法就是替换法。

替换法的意思就是，当你不确定这个代码要写在service还是dao的时候，你就想想，如果现在要求你把dao层的框架（比如把mytatis-plus替换成jpa)，能不能不影响service？或尽可能不影响service?

在实际编程中，如果是一条sql，大家都不会有分歧，肯定把它归在dao层。

但是在service却经常出现
```java
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.eq("name",  "查询名字");
Page<User> page = new Page<>(1, 10);
IPage<user> userList = userMapper.selectPage(page, wrapper);
```
甚至是
```java
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.eq("name",  "查询名字");
Page<User> page = new Page<>(1, 10);
IPage<user> userList = userService.selectPage(page, wrapper);
// 注意这里是 userService
```
假设真的需要替换dao层的框架，你会发现service一大堆东西需要调整。dao层的框架影响了service层，合理吗？

广大的crud码农本来就没有能力区分service和dao层的边界，mybatis-plus提供的这些 IService 更是推了一把。

诚然，mybatis-plus的初衷是为了提供便利。但这种便利，用一个不恰当的比喻来说就是，

> 门出入，窗通风。你给人提供个梯子在窗边，让人更方便地从窗户出入。
~~这个例子不太好。再举一个~~

> 你办事的时候遇到困难，首先想到的解决方法就是托人找关系。但你未必有熟人，未必有渠道。为了方便大家找关系，它特意提供了个送礼工具。通过它这个工具，你就可以直接把礼送到要送的人手里。

这个比喻还是不好。

感觉mybatis-plus就是太过纵容开发人员的习惯。就好比用户爱刷小视频、爱刷小作文，抖音头条之类的就拼命地按你的喜欢去给你推。
mybatis-plus也是这样。你爱这样写service，那我就给你个iservice让你更方便地写crud咯
这种便利，作为一个流行的工具库非但不应该提供，还应该提醒大家不要这样做。

而是靠自己的影响力，努力引导大家往正确的合理的方式上去编程。

## 写在最后
目前国内的现状就是大量的crud码农，用着面向对象的Java，写着面向数据库编程的代码。

而MyBatis-Plus就是在这样一个畸形的需求下，给大家提供了便利。

而我认为，MyBatis-Plus本可以做的，是给大家一个当头棒喝。



有兴趣的可以参看我另外两篇文章

[不该用的代码生成器](不该用的代码生成器.md)

[aService直接调用bDao合适吗？](aService直接调用bDao合适吗？)
