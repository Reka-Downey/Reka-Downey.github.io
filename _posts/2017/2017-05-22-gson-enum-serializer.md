---
layout: post
title: Gson 优雅实现多个枚举的自定义（反）序列化过程
category: 2017年5月
tags: Gson
keywords: Gson, Enum
description: 通过 Gson 实现 Enum 的通用（反）序列化实现类
---

## 版本说明

　　`JDK`版本

![JDK版本说明](https://ooo.0o0.ooo/2016/08/23/57bc4f3e6ee15.jpg)

　　`Gson`版本

```xml
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
        <version>2.7</version>
    </dependency>
```

## Gson （反）序列化例子

　　`Gson`是实现对象序列化和反序列化的利器，但是`Gson`在（反）序列化枚举时默认是根据枚举值的名称来实现的，如果你想要在（反）序列化枚举时输出自己定义的枚举属性，那么此时至少有两种选择：

- 继承`TypeAdapter`抽象类并实现其中的抽象方法
- 实现`JsonSerializer` 和/或`JsonDeserializer` 接口

　　我通常是选择第二种方式，即实现接口的方式来自定义（反）序列化过程。比如下面这个实例：

　　针对`JDK1.8`新的时间`API`中的`LocalDate`进行自定义（反）序列化过程：

``` java
import com.google.gson.*;
import java.lang.reflect.Type;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

public class LocalDateTypeAdapter implements JsonSerializer<LocalDate>, JsonDeserializer<LocalDate> {

    private final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");

    @Override
    public JsonElement serialize(LocalDate src, Type typeOfSrc, JsonSerializationContext context) {
        return null == src ? null : new JsonPrimitive(formatter.format(src));
    }

    @Override
    public LocalDate deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        return null == json ? null : LocalDate.parse(json.getAsString(), formatter);
    }

}
```

　　当注册了该`TypeAdapter`的`Gson`在序列化`LocalDate`类型时，会调用其中的`serialize`方法，此时如果`LocalDate`实例非空的话，我们将调用`formatter.format(src)`方法将`LocalDate`实例格式化成指定形式的字符串，然后封装成`JsonPrimitive`实例并返回`Gson`，`Gson`再将返回的`JsonPrimitive`写入到 json 字符串中。

　　同理，当`Gson`反序列化时遇到`LocalDate`类型时，会调用其中的`deserialize`方法完成反序列化。

## 枚举（反）序列化通用实现

　　好了，扯完基本（反）序列化实现后，现在有一个问题：那就是针对单个枚举使用这种方式那是无可厚非的。但是当你的项目中应用到的枚举数量比较多的时候，如果依然采用这种方式，那么随着而来的问题是类的增加以及出现一些小范围的重复代码。

　　要如何解决这个问题就是写作本文的意图了。

　　本文是基于`JsonSerializer` 和`JsonDeserializer` 接口来解决问题的，当然网上有针对`TypeAdapter`的工厂类`TypeAdapterFactory`来解决该问题，但是该解决方案有个不足之处，那就是需要修改`Gson`的源码，这里就不讲该方案了，有需要的同学自行搜索。 

　　第一步：一个接口，该接口定义了`Gson`在（反）序列化枚举时将调用的方法：

```java
public interface GsonEnum<E> {

    String serialize();

    E deserialize(String jsonEnum);

}
```

　　接口中定义接收一个泛型`E`，该泛型用来表示枚举类型，里面有两个抽象方法，`String serialize()`方法表示将枚举序列化成字符串，`E deserialize(String jsonEnum)`表示将字符串反序列化成枚举并返回特定的枚举`E`。

　　第二步：定义枚举类并实现第一步中声明的接口，这里为了演示效果，定义了如下两个枚举：

　　派别枚举`Faction`：

```java
public enum Faction implements GsonEnum<Faction> {

    ABNEGATION("无私派"), AMITY("和平派"), DAUNTLESS("无畏派"), CANDOR("诚实派"), ERUDITE("博学派");

    private final String name;

    Faction(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public static Faction parse(String faction) {
        switch (faction) {
            case "无私派":
                return Faction.ABNEGATION;
            case "和平派":
                return Faction.AMITY;
            case "无畏派":
                return Faction.DAUNTLESS;
            case "诚实派":
                return Faction.CANDOR;
            case "博学派":
                return Faction.ERUDITE;
            default:
                throw new IllegalArgumentException("There is not enum names with [" + faction + "] of type Faction exists! ");
        }
    }

    @Override
    public Faction deserialize(String jsonEnum) {
        return Faction.parse(jsonEnum);
    }

    @Override
    public String serialize() {
        return this.getName();
    }

}
```

　　性别枚举`Gender`：

```java
public enum Gender implements GsonEnum<Gender> {

    MALE("男"), FEMALE("女");

    private final String type;

    Gender(String type) {
        this.type = type;
    }

    public String getType() {
        return type;
    }

    public static Gender parse(String type) {
        switch (type) {
            case "男":
                return Gender.MALE;
            case "女":
                return Gender.FEMALE;
            default:
                throw new IllegalArgumentException("There is not enum names with [" + type + "] of type Gender exists! ");
        }
    }

    @Override
    public Gender deserialize(String jsonEnum) {
        return Gender.parse(jsonEnum);
    }

    @Override
    public String serialize() {
        return this.getType();
    }

}
```

　　对于这两个枚举内部对`GsonEnum`接口的实现方法比较简单，所以这里就不解释了。

　　第三步：自定义`Gson`（反）序列化过程的`GsonEnumTypeAdapter`，先看具体代码实现：

```java
public class GsonEnumTypeAdapter<E> implements JsonSerializer<E>, JsonDeserializer<E> {

    private final GsonEnum<E> gsonEnum;

    public GsonEnumTypeAdapter(GsonEnum<E> gsonEnum) {
        this.gsonEnum = gsonEnum;
    }

    @Override
    public JsonElement serialize(E src, Type typeOfSrc, JsonSerializationContext context) {
        if (null != src && src instanceof GsonEnum) {
            return new JsonPrimitive(((GsonEnum) src).serialize());
        }
        return null;
    }

    @Override
    public E deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        if (null != json) {
            return gsonEnum.deserialize(json.getAsString());
        }
        return null;
    }

}
```

　　代码看起来很简单，唯一的构造器要求传入一个`GsonEnum`的实现类。

　　首先看一个[测试](http://lib.csdn.net/base/softwaretest)用例：

　　（反）序列化测试时用到的类`Person`：

```java
public class Person implements Serializable {

    private String name;
    private Gender gender;
    private Faction faction;
    private LocalDate birthday;

    public Person() {
    }

    public Person(String name, Gender gender, Faction faction, LocalDate birthday) {
        this.name = name;
        this.gender = gender;
        this.faction = faction;
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", gender=" + gender +
                ", faction=" + faction +
                ", birthday=" + birthday +
                '}';
    }

    // 省略 getter 和 setter
}
```

## 实现效果测试

　　`JUnit`测试实例

```java
    @Test
    public void testGson() throws Exception {
        Gson gson = new GsonBuilder()
                .serializeNulls()
                .registerTypeAdapter(LocalDate.class, new LocalDateTypeAdapter())
                .registerTypeAdapter(Gender.class, new GsonEnumTypeAdapter<>(Gender.FEMALE))
                .registerTypeAdapter(Faction.class, new GsonEnumTypeAdapter<>(Faction.ABNEGATION))
                .create();

        Person p1 = new Person("雷卡", Gender.MALE, Faction.DAUNTLESS, LocalDate.of(1994, 10, 11));
        System.out.println("调用 toString 方法：\n" + p1);
        String jsonText = gson.toJson(p1);
        System.out.println("将 person 转换成 json 字符串：\n" + jsonText);

        System.out.println("-------------------");

        Person p2 = gson.fromJson(jsonText, Person.class);
        assert p2 != p1;
        System.out.println("根据 json 字符串生成 person 实例：\n" + p2);

    }
```

　　测试结果为： 

![测试结果](https://ooo.0o0.ooo/2016/08/23/57bc3b38b0830.jpg)

## 实现原理说明

　　测试结果表明我们想要的效果已经达到了，现在讲一下`GsonEnumTypeAdapter`的实现原理。 

　　以`Faction`为例，首先它接收`E`时我们传递的是`Faction`枚举的一个枚举值`Faction.ABNEGATION`，这个枚举值没有任何限定，可以是该枚举类中的任意一个。 

　　序列化`Faction`类型时，`Gson`调用其中的`serialize`方法，注意此时我们传入的`Faction.ABNEGATION`并不发挥作用，我们直接调用的是序列化时遇到的`Faction`实例（比如`Faction.AMITY`，`Faction.CANDOR`）的`serialize`实例方法，那么此时返回的必然是该枚举实例的`name`属性值； 

　　反序列化遇到`Faction`类型时，`Gson`调用其中的`deserialize`方法，此时我们实际上操作的最开始传入的枚举类型`Faction.ABNEGATION`的`deserialize`方法，注意此时该方法需要传入一个字符串，刚好可以用到`JsonElement`自身提供的`getAsString`方法。再看`Faction`类中对`deserialize`方法的实现，实际上该方法并不关注调用者是谁，它真正关心的传入值是什么，根据传入的值，它可以很快获取得到对应的枚举或者抛出异常。

　　可能上面表述得不是很清楚，但实际上都是一些基础的知识，即泛型，接口，枚举的特殊性的综合应用，多看几遍应该就懂了，另外最好打上几个断点，然后`debug`一遍，看整个流程是怎么跑的，那样基本就清晰了。 

　　另外，为确保`GsonEnumTypeAdapter`只接收枚举并且`GsonEnum`只被枚举类所实现，可以添加泛型参数的边界，如下： 

　　`GsonEnum`的类声明

```java
public interface GsonEnum<E extends Enum<E>>
```

　　`GsonEnumTypeAdapter`的类声明

```java
public class GsonEnumTypeAdapter<E extends Enum<E>> implements JsonSerializer<E>, JsonDeserializer<E>
```

　　声明了泛型上限为`Enum`之后可以确保在使用该泛型时只有枚举类能够被处理。

源码地址：[GitHub](https://github.com/Reka-Downey/java-miscellaneous) 

|                 |
| --------------: |
| 编写日期：2016-08-23 |
| 发布日期：2017-05-22 |