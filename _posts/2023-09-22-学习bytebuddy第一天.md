---
title: ByteBuddy学习
tags: [ByteBuddy]
key: byteBuddy001
lang: zh
cover:
    src: 
---

# ByteBuddy入门



## 起步

### 什么是ByteBuddy

[Byte Buddy](https://github.com/raphw/byte-buddy) 是一个代码生成和操作库，用于在Java应用程序运行时创建和修改Java类，而无需编译器的帮助。

除了Java类库附带的代码生成实用程序外，Byte Buddy还允许创建任意类，并且不限于实现用于创建运行时代理的接口。

此外，Byte Buddy提供了一种方便的API，可以使用Java代理或在构建过程中手动更改类。

至于性能，在官网他们给出了对比数据，您可自行阅读

### maven安装

在pom文件引入依赖

```xml
<!--    当前最新版本为1.14.8 可替换为最新版本    -->
<dependency>
    <groupId>net.bytebuddy</groupId>
    <artifactId>byte-buddy</artifactId>
    <version>1.14.8</version>
</dependency>
```

### Hello Word

惯例都是从Hello Word开始，我也从这里编写入门代码

```java
public void HelloByteBuddySave(){
        try {
            new ByteBuddy()
              			//继承父类
                    .subclass(Object.class)
              			//文件名和包名
                    .name("com.feimatu.HelloWorld")
              			//生成
                    .make()
              			//保存文件
              			.saveIn(new File("/Users/IdeaProjects/feimatu/agent/src/test/java/"));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```

上述代码就是一个最简单的入门代码，使用ByteBuddy都需要从 new ByteBuddy() 来开始，此代码生成了一个HelloWorld的类，并将class文件保存在了指定路径下，下面给出的生成的类的代码，是通过反编译得到的

```java
package com.feimatu;

public class HelloWorld {
    public HelloWorld() {
    }
}
```

生成了一个只有构造函数的类，就是上述代码的全部功能，接下来我们一步步去充实类



```java
public void HelloByteBuddyDefineMethod(){
        try {
            Object o = new ByteBuddy()
                    .subclass(Object.class)
                    .name("com.feimatu.HelloWorld")
                    .defineMethod("helloWorld",String.class, Modifier.PUBLIC)
                    .intercept(FixedValue.value("hello word!"))
                    .make()
                    .load(this.getClass().getClassLoader())
                    .getLoaded().newInstance();
            Method method = o.getClass().getMethod("helloWorld");
            Object invoke = method.invoke(o);
            System.out.println(invoke);
        } catch (InstantiationException | IllegalAccessException | NoSuchMethodException | InvocationTargetException e) {
            throw new RuntimeException(e);
        }

    }
```

此处通过defineMethod 声明了一个方法helloWorld，方法的返回值为String，修饰符为PUBLIC，通过intercept，使此方法返回

hello word!，上述代码最终执行结果就是打印输出

hello word!

***defineMethod***作用就是定义一个方法，三个参数描述很简单就能看出里是什么意思，如果我们想接收参数，则需要在调用此方法后再次调用***withParameter***来进行指定

```java
new ByteBuddy()
  .subclass(Object.class)
  .name("com.feimatu.HelloWorld")
  .defineMethod("helloWorld",String.class, Modifier.PUBLIC)
  .withParameter(String.class)
```

***interceptor***拦截方法或修改实现，在上述代码中我们通过FixedValue.value("hello word!")定义方法的返回值为期望值，我们可以通过此方法放入自己需要的任何自定义逻辑

而且使用这项技术，基本就是修改已有逻辑，或者在已经存在的逻辑中织入自己的逻辑

