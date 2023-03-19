# aService直接调用bDao合适吗？
## 背景
做Web开发写CRUD的Java程序员，Controller、Service、Dao是耳熟能详，天天写代码也是写这些东西。

但是代码分层到底对不对、合不合理，我真的很好奇，有几个团队是内部完全达成了共识的？

Controller相对还好，基本大家都比较认可了就是做参数校验等工作，别在Controller里写业务逻辑，逻辑要放在Service。

但是Service和Dao是真的划分的一塌糊涂。尤其是那些不复杂的业务逻辑。

不少人为了简单省事，在Service层继承了mybatis-plus的IService，直接在Service层里快速地进行CRUD。

还有部分人是代码生成器的拥趸，建好表后二话不说就根据数据库表生成Entity、Dao、Service、Controller。

PS：我个人是非常讨厌代码生成器的。

这里不多说，先来看一个具体的问题，也是本文想讨论的问题。

## 问题
aService应该直接调用bDao，还是aService调用bService再由bService调用bDao？

## 案例
假设有一个订单列表页，每个订单里可以包含多个商品。

显然大家无需思考就可以建出2个表来：订单表(order)和订单商品项表(order_item)。

既然有2个表，那对应有2个Entity也很合理吧？嗯，Order 和 OrderItem。

既然有2个Entity，那有2个对应的Dao也很合理吧？嗯，OrderDao 和 OrderItemDao。

那再分别来2个Service和2个Controller也很合理吧？
OrderController、OrderItemController、OrderService、OrderItemService

打住。先停下来。好好想想，这里有没有问题？

## 思考
大家有没有发现，其实这一步根本就没关注需求。
似乎建好表后照着表一一对应建立相应的controller、service、dao绝对是天经地义的事。

如果照restful风格，2个api也够了
```
/orders
/order/:id
```
但这2个api都归在OrderController里，不比拆成OrderController和OrderItemController香？

![订单列表页截图](images/order_list.jpg)

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

## 结论
Service是写业务逻辑的地方。使用了2个不同的Dao不代表这是2个业务。  
在一个Service里调用2个Dao是毫无问题的。  
反而在aServcie先调用bService再由bService调用bDao是很奇怪的事。