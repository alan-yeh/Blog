---
layout: post
date: 2016-10-23
title: Gradle管理多子项目
tags: Java-Web
---

# Gradle管理多子项目
　　当项目达到一定规模时，我们一般会将项目拆分成各个小的项目，以便进行管理。本篇主要介绍Gradle如何便利地管理多子项目。

## 项目结构
　　拆分项目时，可能会拆分成如下结构:

```
└── gradle-subprojects-example
    └── service-api
    └── service-impl
    └── web
```

　　其中，`service-impl`依赖于`service-api`，而`web`依赖于前者两个。那么此时，如何使用Gradle来管理这些项目呢？

## settings文件
　　settings文件声明了项目的层次结构。在默认情况下，这个文件被命名为settings.gradle，并且和根项目的build.gradle文件放在一起。

`gradle-subprojects-example/settings.gradle`:

```groovy
rootProject.name = 'gradle-subprojects-example'
include 'service-api'
include 'service-impl'
include 'web'
```

　　如果你的项目结构层次比较深，比如你想要映射`api/service/user-service-api`目录的工程，可以使用`include 'api:service:user-service-api'`方式来添加子项目。

## 集中式配置子项目
　　集中式配置就是将所有项目的依赖、任务都集中在根项目中进行配置管理。

`gradle-subprojects-example/build.gradle`:

```groovy
group = 'cn.yerl'
version = '1.0.0-SNAPSHOT'

project(':service-api') {
    group = 'cn.yerl'
    version = '1.0.0-SNAPSHOT'
    apply plugin: 'java'
}
project(':service-impl') {
    group = 'cn.yerl'
    version = '1.0.0-SNAPSHOT'
    apply plugin: 'java'
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        testCompile group: 'junit', name: 'junit', version: '4.11'
        compile project(':service-api')
    }
}
project(':service-api') {
    group = 'cn.yerl'
    version = '1.0.0-SNAPSHOT'
    apply plugin: 'war'
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        testCompile group: 'junit', name: 'junit', version: '4.11'
        compileOnly 'javax.servlet:servlet-api:2.5'
        compile 'org.springframework:spring-webmvc:4.3.3.RELEASE'
    }
}
```
> 上面可以发现，重复写了好几次group、version、repositories等等，这些在本文后面有提到如何进行优化。

　　以上便是集中式项目配置管理的例子，将所有子项目相关的配置信息都存放于根项目的`build.gradle`中，便于管理。
## 分布式配置子项目
　　当项目达到一定规模时，子项目越来越多，自定义任务也越来越多的时候，集中式项目配置的缺陷就暴露出来了。此时可以使用分布式配置子项目。

　　分布式配置子项目的意思就是，将`build.gradle`中的内部拆分到每个子项目的`build.gradle`文件中。拆分后，文件内容如下：

`gradle-subprojects-example/build.gradle`

```groovy
group = 'cn.yerl'
version = '1.0.0-SNAPSHOT'
```

--

`gradle-subprojects-example/service-api/build.gradle`

```groovy
group = 'cn.yerl'
version = '1.0.0-SNAPSHOT'
apply plugin: 'java'

sourceCompatibility = 1.6
```

--

`gradle-subprojects-example/service-impl/build.gradle`

```groovy
group = 'cn.yerl'
version = '1.0.0-SNAPSHOT'
apply plugin: 'java'

sourceCompatibility = 1.6

repositories {
    mavenCentral()
}
    
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.11'
    compile project(':service-api')
}
```

--

`gradle-subprojects-example/web/build.gradle`

```groovy
group = 'cn.yerl'
version = '1.0.0-SNAPSHOT'
apply plugin: 'war'

sourceCompatibility = 1.6

repositories {
    mavenCentral()
}
    
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.11'
    compileOnly 'javax.servlet:servlet-api:2.5'
    compile 'org.springframework:spring-webmvc:4.3.3.RELEASE'
}
```

## 定义公共行为
　　在上面的代码清单中，需要为每个项目定议group、version等属性，需要将java插件分别应用于每个子项目，需要为每个项目都配置一次repositories等。在相当小的项目中，这似乎不是什么大的问题，但是如果子项目有十数个时，就会变成体力劳动，相当无趣。

　　如何定义公共行为呢，这里用到了`allprojects`和`subprojects`方法来改进现有的代码。

- allprojects: 包含根项目的所有项目
- subprojects: 不包含根项目的所有子项目

`gradle-subprojects-example/build.gradle`

```groovy
allprojects {
    group = 'cn.yerl'
    version = '1.0.0-SNAPSHOT'
}

subprojects {
    apply plugin: 'java'
    
    jar {
        sourceCompatibility = 1.6
        targetCompatibility = 1.6
    }
    
    repositories {
        mavenLocal()
        mavenCentral()
    }
    
    dependencies {
        testCompile group: 'junit', name: 'junit', version: '4.11'
    }
}
```
--

`gradle-subprojects-example/service-api/build.gradle`

```groovy
```
--

`gradle-subprojects-example/service-impl/build.gradle`

```groovy
dependencies {
    compile project(':service-api')
}
```
--

`gradle-subprojects-example/web/build.gradle`

```groovy
dependencies {
    compileOnly 'javax.servlet:servlet-api:2.5'
    compile 'org.springframework:spring-webmvc:4.3.3.RELEASE'
}
```

　　可以看到，通过定义公共行为，子项目仅需关注自己与别的项目的不同的行为即可，这样可以大大减少代码的冗余量。