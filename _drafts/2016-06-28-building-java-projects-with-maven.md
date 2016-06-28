---
layout: post
date: 2016-06-28
title: 使用Maven构建Java工程
tags: Java-Web
---
　　这篇博客会带你一起来使用Maven构建Java工程。

### 创建一个Java工程
　　首先，新建一个Java工程，用于演示如何使用Maven来构建它。为了让我们更关注如何使用Maven，我们将这个工程建得非常简单。

　　创建一个工程根目录，然后在`Terminal`打开它。

```bash
$ mkdir gs-maven
$ cd gs-maven
```
　　使用命令`mkdir -p src/main/java/hello`创建工程目录结构。你会得到以下的目录。

```
└── src
    └── main
        └── java
            └── hello
```

　　在`src/main/java/hello`目录下，你可以创建任意你想要的Java类。为了方便接下来的演示，这里仅创建两个简单的类：`HelloWorld.java`和`Greeter.java`。

`src/main/java/hello/HelloWorld.java`

```java
package hello;

public class HelloWorld {
  public static void main(String[] args) {
    Greeter greeter = new Greeter();
    System.out.println(greeter.sayHello());
  }
}
```

`src/main/java/hello/Greeter.java`

```java
package hello;

public class Greeter {
  public String sayHello() {
    return "Hello world!";
  }
}
```
### 安装Maven到你的工程
　　在这篇博客