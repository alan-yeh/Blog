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
　　根据[前面的博客](/blog/macos-setup-maven)配置Maven环境。如果你已经完配置环境了，那可以接着做以下的步骤。

### 定义一个简单的Maven构建
　　现在Maven已经安装好了，你需要将工程转换成Maven工程。Maven工程使用一个名为`pom.xml`文件来定议。这个文件定义了工程名字、版本、外部依赖等。

　　在项目根目录创建一个`pom.xml`文件，并将以下内容复制到里面。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework</groupId>
    <artifactId>gs-maven</artifactId>
    <packaging>jar</packaging>
    <version>0.1.0</version>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>hello.HelloWorld</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```
　　除了可选的`<packaging>`元素之外，这可能是最简单的Java工程*pom.xml*配置了。它包含了以下项目配置细节：

- `<modelVersion>`. POM model version（一般来说都是4.0.0）。
- `<groupId>`. 工程所属的分组或组织。通常使用反序域名作为groupId。
- `<artifactId>`. 工程打包名。（比如JAR、WAR文件的名字）
- `<version>`. 工程版本号。
- `<packaging>`. 标记工程以什么方式打包。默认使用`jar`来打包JAR包，使用`war`来打包WAR文件。

　　到此为止，你已经将工程转成Maven工程了。
### Build Java code
　　Maven现在已经准备好构建工程了。你现在可以执行一系列Maven的构建周期目标（build lifecycle goal）了，包括编译工程代码，创建库包(library package)，以及将库安装到Maven本地依赖库中。

　　使用以下命令来构建项目。

```bash
mvn compile
```
　　这个命令会开始运行Maven，并让它去执行编译目标(compile goal)。当命令执行完毕之后，你可以在*target/classes*目录下找到编译好的*.class*文件。

　　你应该不会想要直接使用*.class*文件，你应该会将它们打包后再使用：

```bash
mvn package
```
　　打包目标(package goal)会编译你的Java代码，运行所有测试，然后将`target`下编译好的代码打包成JAR文件。JAR文件会根据`pom.xml`中的`<artifactId>`和`<version>`来命名。在这个例子中，JAR文件最后会被命名为*gs-maven-0.1.0.jar*。

　　Maven会在你本地机器里持有一个仓库，用于存放你的依赖(通常在*.m2/repository*目录下)，以便可以快速解决项目依赖。如果你想把你的工程打包好的JAR文件添加到这个仓库中，你可以使用`install`命令。

```bash
mvn install
```
　　安装目标(install goal)将会编译、测试、将工程打包起来，最后将它复制到本地他库中，这样其它工程就可以引用这个包作为它们的依赖了。

　　即然谈到了依赖，那么现在，我们来为工程声明一个依赖，看Maven是如何工作的。
### 声明依赖
　　刚才那个`Hello World`项目是一个非常简单的项目，没有依赖任何外部库。但是，大多数应用会依赖外部库来处理命令和复杂的功能。

　　继续刚才的`Hello World`项目。假设你现在需要让你的应用打印当前的日期和时间。当然，你可以使用Java原生的日期时间库来完成这个功能，但是，如果你使用`Joda Time`库来完成这个功能的话，会更有意思。

　　首先，将`HelloWorld.java`文件修改成以下的样子。

```java
package hello;

import org.joda.time.LocalTime;

public class HelloWorld {
  public static void main(String[] args) {
    LocalTime currentTime = new LocalTime();
    System.out.println("The current local time is: " + currentTime);

    Greeter greeter = new Greeter();
    System.out.println(greeter.sayHello());
  }
}
```
　　现在，`HelloWorld`使用`Joda Time`的`LocalTime`类来获取以及显示当前的时间。

　　如果你现在使用`maven compile`来构建项目，这个任务将会失败，因为你还没有声明项目需要使用依赖`Joda Time`库。你可以通过把下面的内容加到*pom.xml*文件下的`<project>`节点修复这个问题。

```xml
    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>joda-time</groupId>
            <artifactId>joda-time</artifactId>
            <version>2.2</version>
        </dependency>
    </dependencies>
```
　　以上的XML为项目声明了一个依赖列表。现在这个列表只有一项`Joda Time`依赖。在`<dependency>`元素中，依赖坐标(dependency coordinates)由三个元素组成：

- `<groupId>`. 这个依赖归属的分组或组织
- `<artifactId>`. 这个依赖的名字
- `<version>`. 这个依赖的版本号

　　Maven默使使用`compile`依赖。这意味着，他们在编译时(compile-time)应该是可用的(如果你是在打包WAR文件，这些依赖会被包含到*/WEB-INF/libs*目录下)。除了`compile`之类，`<scope>`可以指定为以下类型：

- `provided`. 仅在编译时依赖，这个库会在容器运行时提供（比如`Java Servlet API`）。
- `test`. 仅在运行测试用例时依赖，在打包或运行项目时不需要依赖。

　　现在，你再次运行`mvn compile`或者`mvn package`，Maven就可以从Maven中央库(Maven Central repository)里找到你需要的`Joda Time`依赖，并正确执行任务。

　　以下是完成之后的`pom.xml`文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework</groupId>
    <artifactId>gs-maven</artifactId>
    <packaging>jar</packaging>
    <version>0.1.0</version>

    <!-- tag::joda[] -->
    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>joda-time</groupId>
            <artifactId>joda-time</artifactId>
            <version>2.2</version>
        </dependency>
    </dependencies>
    <!-- end::joda[] -->

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>hello.HelloWorld</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

> 注意： 这个**pom.xml**文件使用[Maven Shade Plugin](http://maven.apache.org/plugins/maven-shade-plugin/)来生成可执行的JAR文件。这篇博客我们更关注如何入门Maven，而不是怎么来使用特定的插件。

### 总结
　　恭喜你，你现在已经可以使用Maven来简单、高效地构建Java项目了。