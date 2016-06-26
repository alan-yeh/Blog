---
layout: post
date: 2016-06-27
title: 使用Gradle构建Java工程
tags: Java-Web
---
　　这篇博客会带你一起来使用Gradle构建Java工程。

### 创建一个Java工程
　　首先，新建一个Java工程，以便我们尝示如何使用Gradle构建来构建它。为了把我们的精力更专注于如何使用Gralde，我们将这个工程建得非常简单。

　　创建一个工程根目录，然后在`Terminal`打开它。

```bash
$ mkdir gs-gradle
$ cd gs-gradle
```
　　使用命令`mkdir -p src/main/java/hello`创建工程结构。你会得到以下的目录。

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

### 安装Gradle到你的工程
　　[上一篇博客](/blog/macos-setup-gradle)介绍到如何在Mac下配置Gradle环境。如果你已经完成配置Gradle环境了，那可以接着做以下的步骤。

　　回到项目根目录，在你的`Terminal`输入`gradle`。如果以上步骤都没有错的话，你会得到以下信息。

```bash
$ gradle
:help

Welcome to Gradle 2.14.

To run a build, run gradle <task> ...

To see a list of available tasks, run gradle tasks

To see a list of command-line options, run gradle --help

To see more detail about a task, run gradle help --task <task>

BUILD SUCCESSFUL

Total time: 1.428 secs

This build could be faster, please consider using the Gradle Daemon: https://docs.gradle.org/2.14/userguide/gradle_daemon.html
```
　　现在，你的项目可以使用Gradle来构建了。
### 查看Gradle可以做什么
　　现在，Gradle已经安装好了，那么来看看Gradle可以做些什么。无论你有没有创建`build.gradle`文件，你都可以使用以下命令查看Gradle有哪些任务是可用的。

```bash
bradle tasks
```
　　可以看到，Gralde列出了一系列可用的任务。由于之前创建的工程中，没有`build.gradle`文件，因此，你应该会得到以下信息。

```bash
$ gradle tasks
:tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'gs-gradle'.
components - Displays the components produced by root project 'gs-gradle'. [incubating]
dependencies - Displays all dependencies declared in root project 'gs-gradle'.
dependencyInsight - Displays the insight into a specific dependency in root project 'gs-gradle'.
help - Displays a help message.
model - Displays the configuration model of root project 'gs-gradle'. [incubating]
projects - Displays the sub-projects of root project 'gs-gradle'.
properties - Displays the properties of root project 'gs-gradle'.
tasks - Displays the tasks runnable from root project 'gs-gradle'.

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>

BUILD SUCCESSFUL

Total time: 1.122 secs

This build could be faster, please consider using the Gradle Daemon: https://docs.gradle.org/2.14/userguide/gradle_daemon.html
```
　　以上列出来的任务都是可用的。但是，使用`gradle.build`可以使任务变得更有用。可用任务列表会随着你添加插件到`build.gradle`文件而变得更多。

### Build Java code
　　在项目根目下，创建一个最简单的`build.gradle`文件，文件只有以下一行内容：

```bash
apply plugin: 'java'
```
　　这简单的一行配置为工程带来很大的改变。再运行一次`gradle tasks`命令，可以看到任务列表多出了很多项，包括`Build`、`JavaDoc`、`tests`等。

　　你会经常使用到`gradle build`。这项任务会编译、测试，并且会将代码组装成JAR文件。

```bash
gradle build
```

　　经过几秒时间的运行，`BUILD SUCCESSFUL`代表着`build`任务已经完成了。

　　我们来看看，这次任务的运行结果。打开项目根目录，可以看到，多了一个`build`文件夹。`build`文件夹下面，将会包括以下这几个文件夹。

- *classes.* 项目的编译后的.class文件
- *reports.* 编译产生的报告（例如测试报告）
- *libs.* 项目编译后的包（通常是jar或war包）

　　`classes`文件夹下保存编译后的Java代码。你可以在这里找到`HelloWorld.class`和`Greeter.class`。

　　由于这个项目没有依赖任何其它库，所以`dependency_cahce`文件夹下面是空的。

　　`libs`文件夹下应该有一个与项目同名的JAR文件。将来，你可以自己指定JAR包的文件和版本号。
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

　　如果你现在使用`gradle build`来构建项目，这个任务将会失败，因为你还没有声明项目需要依赖`Joda Time`库。

　　你现在需要为第三方库添加源。在`build.gradle`中添加以下内容。

```
repositories {
    mavenCentral()
}
```
　　`repositories`块指定了构建应该使用Maven中央仓库来解决项目的依赖。Gradle依靠许多约定和Maven已建立好的设施，包括使用Maven的中央库来作为库依赖源。

　　现在，我们来声明第三方依赖。在`build.gradle`中添加以下内容。

```
sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile "joda-time:joda-time:2.2"
}
```
　　在`dependencies`块里，我们为项目声明了一个依赖`Joda Time`。并且，你的请求指定了去获取`joda-time`分组下的版本为2.2的`joda-time`库（以`:`分隔，第一个是分组，第二个是库名，第三个是版本号）。

　　另外一个需要注意的是，这个依赖是一个`compile`依赖，这指明了它在编译时（compile-time）应该是可用的（如果你是在编译WAR包，那么这个库应该被包含在`/WEB-INF/libs`目录下）。除了`compile`依赖，还存在以下两种依赖：

- `providedCompile`。仅在编译时依赖，这个库会在容器运行时提供（比如`Java Servlet API`）。
- `testCompile`。仅在运行测试用例时依赖，在打包或运行项目时不需要依赖。

　　最后，我们来指定生成的JAR包的名字。在`build.gradle`中添加以下内容。

```
jar {
    baseName = 'gs-gradle'
    version =  '0.1.0'
}
```
　　`jar`块里指定了JAR文件会如何被命名。在这个例子里，它最终会变成`gs-gradle-0.1.0.jar`。

　　现在，你再来运行`gradle build`命令，Gradle已经可以在Maven中央库里解决`Joda Time`的依赖问题，并编译成功。在`build/libs`文件夹里，看看是否存在`gs-gradle-0.1.0.jar`文件？

### 使用Gradle Wrapper来构建项目
　　`Gradle Wrapper`是Gradle推荐的打包方式。它包含了Windows批处理脚本、OS X/Linux的Shell脚本。这些脚本允许你在没有安装Gradle环境下使用Gradle打包你的项目。在`build.gradle`下添加以下内容。

```
task wrapper(type: Wrapper) {
    gradleVersion = '2.14'
}
```
　　添加后，使用`gradle wrapper`命令生效。它会去下载并初始化wrapper脚本。命令完成之后，你会发现根目录下多了几个文件。

```
└── gradlew
└── gradlew.bat
└── gradle
    └── wrapper
        └── gradle-wrapper.jar
        └── gradle-wrapper.properties
```
　　现在，`Gradle Wrapper`已经准备好构建项目了。把wrapper添加到你的版本控制系统，那么其他人clone你的项目下来之后，就可以以相同方式、相同的Gradle版本来构建项目了。和之前使用gradle命令相似，运行wrapper脚本来执行任务构建任务：

```bash
./gradlew build
```
　　当你第一次运行指定版本的Gradle wrapper，它会去下载并缓存该版本的Gradle的安装文件。`Gradle Wrapper`设计之初，就是为了让它能够被提交到版本控制系统中，任何人在没有配置该版本的Gradle都可以正常构建项目。

　　现在，你应该可以构建你的代码了。

```
build
├── classes
│   └── main
│       └── hello
│           ├── Greeter.class
│           └── HelloWorld.class
├── dependency-cache
├── libs
│   └── gs-gradle-0.1.0.jar
└── tmp
    └── jar
        └── MANIFEST.MF
```
　　来看看`gs-gradle-0.1.0.jar`里面的内容：

```bash
$ jar tvf build/libs/gs-gradle-0.1.0.jar
  0 Mon Jun 27 05:32:34 CST 2016 META-INF/
 25 Mon Jun 27 05:32:34 CST 2016 META-INF/MANIFEST.MF
  0 Mon Jun 27 05:32:34 CST 2016 hello/
369 Mon Jun 27 05:32:34 CST 2016 hello/Greeter.class
988 Mon Jun 27 05:32:34 CST 2016 hello/HelloWorld.class
```
　　我们可以看到，class文件被打包进来了。但是我们注意到，它并不能运行，即使你声明了你需要依赖`joda-time`，但是库并没有被打包到这个JAR文件中。

　　如果需要让这个JAR变得可运行，我们需要借助于gradle的`application`插件。把以下内容添加到`build.gradle`文件中。

```
apply plugin: 'application'

mainClassName = 'hello.HelloWorld'
```
　　现在，你可以运行应用了！

```bash
$ ./gradlew run
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:run
The current local time is: 05:39:55.523
Hello world!

BUILD SUCCESSFUL

Total time: 2.327 secs

This build could be faster, please consider using the Gradle Daemon: https://docs.gradle.org/2.14/userguide/gradle_daemon.html
```
　　依赖打包有不同的方式。举个例子，我们要构建一个WAR包，可以使用gradle的[WAR插件](https://docs.gradle.org/current/userguide/war_plugin.html)来将第3方依赖打包在一起。

　　此时，你的`build.gradle`文件应该是这样的。

```
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'application'

mainClassName = 'hello.HelloWorld'

// tag::repositories[]
repositories {
    mavenCentral()
}
// end::repositories[]

// tag::jar[]
jar {
    baseName = 'gs-gradle'
    version =  '0.1.0'
}
// end::jar[]

// tag::dependencies[]
sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile "joda-time:joda-time:2.2"
}
// end::dependencies[]

// tag::wrapper[]
task wrapper(type: Wrapper) {
    gradleVersion = '2.3'
}
// end::wrapper[]
```

### 总结
　　恭喜你，你现在已经可以使用Gradle build文件来简单、高效地构建Java项目了。