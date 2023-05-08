+++
title = "闲聊Java枚举的实际使用"
date = "2023-04-16"

Description = ""
Tags = ["Java"]
Categories = ["程序开发"]

+++
方式一姑且叫它`原生枚举`。
```java
public enum Gender {
    MALE,FEMALE,OTHER
}
```

方式二因被广大程序员普遍使用，暂且叫它`通用枚举`（mybatis-plus也这样叫）。
```java
public enum Gender {
    MALE(1, "Male"),
    FEMALE(2, "Female"),
    OTHER(3, "Other");

    private final Integer value;
    private final String desc;

    Gender(Integer value,String desc) {
        this.value = value;
        this.desc = desc;
    }

    public Integer getValue() {
        return value;
    }

    public String getDesc() {
        return desc;
    }
}
```
## 你喜欢原生枚举还是通用枚举？
**我个人比较喜欢原生枚举**。

Java 枚举本身已经包含了 name 和 ordinal 两个属性，用来表示枚举值在代码中的名称和顺序。  

有些开发者可能会认为这两个属性并不能满足业务需求，需要使用额外的 value 属性来表示更具体的信息，如状态码、字符串等。
这种情况下，使用 value 属性确实可以提供更多的信息，但是也存在一些问题。
首先，枚举本身就是一个带有固定取值范围的数据类型，如果在枚举中定义了 value 属性，那么这个取值范围就不再固定，这会破坏枚举的本质。
其次，如果多个枚举值有相同的 value 值，就会造成混淆和错误。

我在工作中就经常遇到同事不同分支里开发最终代码合并导致的value重复。  
如一个枚举已经用了value 1、2、3、4、5，这时张三在分支A里增加6，李四在分支B里增加6，于是就出现value重复了。

## 前后端交互及持久化的时候是用Name还是Value?
首先不存在争议的就是，别用ordinal。
因为ordinal是按枚举定义的顺序取下标，如果有人改变了枚举的定义顺序，或后期追加的时候不是从末尾加而是在中间插入，就会导致ordinal发生变化。

**我是倾向是用Name的**，非常直观。就拿本文开头的枚举例子来说，前后端直接用MALE、FEMALE来交互，比用1、2是要清晰的多。
springmvc也是原生默认支持的。  
至于持久化到数据库里，我个人也倾向直接存MALE、FEMALE。  
mybatis也是默认支持的，org.apache.ibatis.type.EnumTypeHandler。  
但不少人喜欢在数据库存成1、2，觉得省存储，但是现在的研发环境，存储值几个钱？

对于这些喜欢省存储省带宽流量的人，原生枚举无法满足他们。他们只能选择使用通用枚举。  
mybatis-plus也是针对这种用法提供了com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler