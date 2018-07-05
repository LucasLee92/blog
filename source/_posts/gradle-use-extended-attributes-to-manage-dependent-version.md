---
title: Gradle使用扩展属性管理依赖版本号
date: 2018-07-05 13:44:34
tags: 
    - gradle
    - maven
---

## Maven预设变量
使用过`Maven`的人应该都知道，我们在`Maven`项目中添加依赖的一般性做法。就是打开`pom.xml`文件，在`<dependencies>`节点下添加
```xml
<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-core</artifactId>
    <version>5.5.0</version>
</dependency>
```
包含坐标和版本号的内容，那么在项目中，就可以引用`Lucene-core`Jar包中的各种类了。但是要注意一点，这里面的版本号是以硬编码的形式存在，作为一个合格的软件开发者，要尽量在你的代码中避免硬编码的情况，方便版本管理
在Maven中使用预设变量的方式来解决：
```xml
<properties>
    <version.lucene>6.0.0</version.lucene>
</properties>
<dependencies>
    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-core</artifactId>
        <version>${version.lucene}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.lucene</groupId>
        <artifactId>lucene-queries</artifactId>
        <version>${version.lucene}</version>
    </dependency>
</dependencies>
```
那么以后再遇到项目升级的情况，只需要手动修改`<version.lucene>6.0.0</version.lucene>`一行代码即可搞定，所有引用到该版本变量的依赖都自动升级。

## Gradle单模块

同样作为后起之秀的`Gradle`如何优雅地解决类似问题呢？硬编码的写法如下

```gradle
dependencies {
    compile "org.apache.lucene:lucene-core:5.5.0"
}
```

优雅的写法①如下，修改`build.gradle`文件

```gradle
ext {
    luceneVersion = '6.5.0'
}
dependencies {
    compile "org.apache.lucene:lucene-core:$luceneVersion"
}
```

一定要注意包含`$`符号时，要用**双引号**。在Gradle中单引号和双引号都是合法的，但是略有不同。单引号中的内容严格对应Java中的String，不对$符号进行转义。双引号的内容则和脚本语言处理有点像，如果字符中有$号的话，则它会先对$表达式求值。在Gradle中，其实还有三引号的情形，这代表什么呢？三引号中的字符串支持任意换行，比如

```gradle
def multieLines = ''' begin
  line  1
  line  2
  line  3
  end '''
```

除了①，还有优雅的写法②，使用字典类型，修改`build.gradle`文件

```gradle
ext {
    javaSourceCompatibility = '1.8'
    libVersions = [
            junit : '4.12',
            lucene: '6.5.0',
            guava : '20.0'
    ]
}
dependencies {
    compile "junit:junit:$libVersions.junit"
    compile "com.google.guava:guava:$libVersions.guava"
}
```

## Gradle多模块
当在一个根项目下有多个子模块，那么一种简单的做法是在每个子模块中都定义`ext`代码块，声明需要使用到的版本号变量，这样做当然可以。但是当遇到需要升级版本号的情况时，需要手动修改所有的子模块，其实还有更优雅的解决方案。就是在根项目中定义`ext`代码块，打开根项目的`build.gradle`定义

```gradle
ext {
    luceneVersion = '6.5.0'
    libVersions = [
            junit : '4.12',
            guava : '20.0'
    ]
}
```

然后在每个子模块的`build.gradle`中都可以直接引用之，方式如下

```gradle
dependencies {
    compile "junit:junit:${rootProject.libVersions.junit}"
}
```

转载至[Gradle使用扩展属性管理依赖版本号](http://codepub.cn/2017/05/09/Gradle-uses-extended-attributes-to-manage-dependent-version-numbers/)