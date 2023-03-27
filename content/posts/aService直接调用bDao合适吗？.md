+++
title = "aService直接调用bDao合适吗？"
date = "2023-03-19"

Description = ""
Tags = ["Java"]
Categories = ["程序开发"]

+++
## 背景
做Web开发写CRUD的Java程序员，Controller、Service、Dao是耳熟能详，天天写代码也是写这些东西。

但是代码分层到底对不对、合不合理，我真的很好奇，有几个团队是内部完全达成了共识的？

Controller相对还好，基本大家都比较认可了就是做参数校验等工作，别在Controller里写业务逻辑，逻辑要放在Service。

但是Service和Dao是真的划分的一塌糊涂。尤其是那些不复杂的业务逻辑。

不少人为了简单省事，在Service层继承了mybatis-plus的IService，直接在Service层里快速地进行CRUD。

还有部分人是代码生成器的拥趸，建好表后二话不说就根据数据库表生成Entity、Dao、Service、Controller。

PS：我个人是非常讨厌代码生成器的。

这里不多说，先来看一个具体的问题，也是本文想讨论的问题。

## 问题1
aService应该直接调用bDao，还是aService调用bService再由bService调用bDao？

## 案例1
假设有一个订单列表页，每个订单里可以包含多个商品。

显然大家无需思考就可以建出2个表来：订单表(order)和订单商品项表(order_item)。

既然有2个表，那对应有2个Entity也很合理吧？嗯，Order 和 OrderItem。

既然有2个Entity，那有2个对应的Dao也很合理吧？嗯，OrderDao 和 OrderItemDao。

那再分别来2个Service和2个Controller也很合理吧？
OrderController、OrderItemController、OrderService、OrderItemService

打住。先停下来。好好想想，这里有没有问题？

## 思考1
大家有没有发现，其实这一步根本就没关注需求。
似乎建好表后照着表一一对应建立相应的controller、service、dao绝对是天经地义的事。

如果照restful风格，2个api也够了
```
/orders
/order/:id
```
但这2个api都归在OrderController里，不比拆成OrderController和OrderItemController香？

![订单列表页截图](/images/order_list.jpg)

如果UI是这样设计，在列表页就需要将订单商品全部展示出来，甚至会设计成一个api。

```json5
[
  { 
    id: "1", 
    items: [//订单1的商品清单
      {},
      {}
    ]
  },
  {
    id: "2",
    items: [//订单2的商品清单
      {},
      {}
    ]
  },
]
```

典型的面向数据库编程就是这样，不需要管需求是什么，不管有没有需要，
直接照着数据库表设计就用代码生成器生成一堆没用的垃圾代码。

在这样的背景下，才会有人说出`自己的Dao只能被自己的Service调用`这种话。  
aService不能直接调用bDao，bDao只能被自己的bService调用。  
大哥，哪来的`自己的Servcie`这么一个说法？

查出订单及订单里的商品明细，是两个业务？  
对于用户来说，会不会绕过主订单直接查询订单里的商品明细？  
显然不会。  
商品明细是依附于订单存在的，根本就不应该为它建立一个OrderItemService。  

我再换个角度问，如果你用的不是MySQL这种关系型数据库，而是Mongo这种文档型数据库，
你可能就不会建2个文档类型（Order和OrderItem）。
那自然也不会有2个Dao，2个Service,2个Controller了。

这样看问题是不是更明显？同样的业务需求，就因为数据库技术选型不同，直接影响了你的代码逻辑？

当你用MySQL的时候你必须建两个Controller和两个Service，
但当你用Mongo的时候你只需要一个Controller和一个Service就够了。

不滑稽吗？

## 结论1
Service是写业务逻辑的地方。使用了2个不同的Dao不代表这是2个业务。  
在一个Service里调用2个Dao是毫无问题的。  
反而在aServcie先调用bService再由bService调用bDao是很奇怪的事。

Controller是贴近用户的，更接近业务的抽象（什么人要做什么事）；  
Dao是远离用户的，是技术的抽象；  
而Service就是两个抽象的粘合（转换）层。

所以如果你就是单纯的做个CRUD管理系统，不然不会出现xxEntity、xxDao、xxService、xxController一条龙顺过去。  

我一向讨厌代码生成器，是因为我觉得代码生成器最多生成xxEntity、xxDao，  
Controller和Service几乎不可能是通用的。

如果Controller和Service都能通用，那你也不需要生成，因为能靠生成的代码都是重复的，大可以再抽象一层，  
而这类抽象已经有了，比如 Spring Data Rest。

## 质疑
这例子似乎太简单了。

bService里没有其他逻辑，仅调用了bDao，所以可以改成直接在aService里调用bDao。  
那如果bService里除了调用bDao外还有其他逻辑呢？也不能调用bService吗？  
是不是会出现aService里既引入bService又引入bDao的情况？

先来明确一个原则。  
Controller调用Service，Service调用Dao。  
Controller绝不允许调用Controller，Dao绝不允许调用Dao。  
Service慎重调用Service。

如果你真的有需要，在Service里调用Service也是可以的。

但是建议先思考，这是否是必须的？

## 案例2

信用卡新卡申请的时候，有新用户自主申请、、新用户邀请申请等。  
如果是邀请申请，那么邀请人也会获得活动奖励。

我相信不少同学会这样写代码。

```java
public class CardService{

    private BonusService bonusService;

    public void register(ApplyForm form){
      // some logic
      if(form.isInvited()){
          // 填写了邀请码
          bonusService.pay();//给邀请人发奖励
      }
    }
}
```
先想想，这里有没有问题？

## 问题2
非得在 CardService 里调用 BonusService吗？

## 思考2
做法1：事件解耦。  
邀请奖励和新卡申请其实是2个业务。完全可以通过事件解耦。  
伪代码参考如下：

```java
public class CardService{

    private BonusService bonusService;

    public void register(ApplyForm form){
      // some logic
      publishEvent(xxxx); //用spring的事件或mq等均可
    }
}

public class RegisterEventListener {
    public void onRegister(xxxx){
        if(xxxx.isInvited()){
          // 填写了邀请码
          bonusService.pay();//给邀请人发奖励
      }
    }
}
```

做法2：对于那些有业务关联的逻辑，也可以额外抽一个Service。

```java
public class RegisterService{

    private CardService cardService;
    private BonusService bonusService;
    
    public void register(ApplyForm form){
        cardService.register(form);
        bonus.pay();
    }
}
```
如果想用pmd等自定义规则来校验，绝对禁止Service里调用Service，可以将这种包含多个Service调用的类统一一个新的命名，
举例叫 *ServiceManager，如 RegisterServiceManager 。  

关于自定义pmd规则，可以看看我的另外一个Demo项目[simple-code-checker](https://gitee.com/jinceon/simple-code-checker)。