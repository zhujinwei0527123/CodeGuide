---
layout: post
category: itstack-code-life
title: 一个Bug，让我发现了 Java 界的.AJ(锥)！
tagline: by 小傅哥
tag: [java,itstack-code-life]
excerpt: 伙伴学小傅哥的中间件，说写个切面运行不了，排查BUG原尾过程分析还学到了新知识。接下来我带着大家一起看看什么是快乐星球，他是怎么一顿骚操作让切面拦截不到的！
lock: need
---

# 一个Bug，让我发现了 Java 界的.AJ(锥)！

作者：小傅哥
<br/>博客：[https://bugstack.cn](https://bugstack.cn)
<br/>原文：[https://mp.weixin.qq.com/s/ArZX-gA1WTon864tjtvjag](https://mp.weixin.qq.com/s/ArZX-gA1WTon864tjtvjag)

> 沉淀、分享、成长，让自己和他人都能有所收获！😄

## 一、前言

`话我放这，踩过的坑越多头发越少！`

说来也是奇怪，只要是学编程的，从初次接触的 Java 到安装 JDK、IDEA、MYSQL， 再到接触 Spring、MyBatis、RPC、MQ，哪怕有时候在浅的坑也会跳进去尝尝鲜，一遍抓着头发，一手点着鼠标也几乎是你的常态。*你的键盘里总是有很多被抓碎的头发！*

但，哪怕是抓了这么头发，还是遇到了一个满脑子都是骚操作的小伙。**“傅哥，我的切面怎么拦截不到？我是照着你的[《SpringBoot 中间件设计和开发》](https://bugstack.cn/itstack-ark-middleware/2021/03/31/SpringBoot-%E4%B8%AD%E9%97%B4%E4%BB%B6%E8%AE%BE%E8%AE%A1%E5%92%8C%E5%BC%80%E5%8F%91-%E4%B8%93%E6%A0%8F%E5%B0%8F%E5%86%8C%E4%B8%8A%E7%BA%BF%E5%95%A6.html)专栏写的，你给我看看吧，我都弄了一天了”**

接下来我带着大家一起看看什么是快乐星球，他是怎么一顿骚操作让切面拦截不到的！

## 二、满脑子都是骚操作

### 1. 遇到问题

上周，谢飞机(~~化名~~)发过来了自己的手撸的中间件源码，说这代码都没有啥怎么就不能切面呢？

![](https://bugstack.cn/assets/images/story/story-2-01.png)

- 最开始我大意了，让谢飞机发了一些代码截图。
- 看截图的代码，这完全就和我写的中间件里的代码一毛一样，没啥问题呀，包路径也能扫描到，咋就不能切面了？
- 我说你打个断点调试下，看看怎么切不到了呢？嘿，调试了，直接通过，就是没切面到。
- 此时我思考了JDK版本、环境配置、Spring上下文、切面的定义、包的路径以及这小子是否忽悠我？
- 最后我抱着这小子忽悠我的心里，把源码要过来了。

### 2. 发现问题

看了几遍源码没发现问题，开始调试，还真它哈拉哨的不进这个切面，接下来；
- 谢飞机的源码保留，复制出来一份新的。
- 我的目的要先让他跑起来，在研究。接下来我把自己的工程里的 `DoJoinPoint` 拷贝过来粘贴进去，噗察一下贴进去了，没提示替换，虽然有报错但两个类能共存，如下：
 ![](https://bugstack.cn/assets/images/story/story-2-02.png)
  - 这就神奇了哈，我当时怀疑是不它那 `DoJoinPoint` 不是一个正经 Java 类，路径不对？有看不见的特殊字符？
- 既然发现这个类不对，那行先删掉。让程序先跑起来，确保除了这个类其他的内容没有问题，这样也好排查问题。
- 还别说，去掉这个错误类，程序可以正常运行，拦截到切面内容了。
- 既然程序能跑了，我就想着这可以看看问题出在哪了，没想到就只打开个文件夹，就发现了一个神奇的**AJ**！这货压根就不是 Java 类！
  ![](https://bugstack.cn/assets/images/story/story-2-03.png)

### 3. 排查问题

要不是IDEA把 `.aj` 这货显示成 C 类的图标，可能早就发现问题了。紧接着把这错误类的截图发给了谢飞机，问它你是怎么创建的？*他说实话了*
- 谢飞机先说自己偷懒了，哈哈哈，让人怪不好意思的！
- 他说在创建 `DoJoinPoint` 时，看到一个 Aspect 的选项，以为这个就是创建切面的快捷操作，如图；
  ![](https://bugstack.cn/assets/images/story/story-2-04.png)
- 创建完成以后发现有点不对，不是 class 类型的，是个 aspect，于是他手动把 aspect 改成了 class，如图；
  ![](https://bugstack.cn/assets/images/story/story-2-05.png)
- 所以，谢飞机实际创建出来的是一个 aspect 的以 `.aj` 结尾的类，并不是一个正经的 Java 类，所以切面不到，也根本没有对应的 class 文件。  
  
## 三、如何正确使用 Aspect 的 .aj 类

**AspectJ**，简称 AJ  ~~我自己说的~~

AspectJ 其实也是 AOP 的一种实现技术，功能类似于拦截器，在集成在 IntelliJ IDEA 开发工具里。在使用 IntelliJ IDEA 编写 AspectJ 代码之前需要本机先安装 AspectJ 工具包。*否则你的 `.aj` 类不能运行，同时IDEA类显示出来的 `.aj` 类，也是C的标识*

接下来我们就来聊聊关于这个东西怎么使用，别再被 `.aj` 骗了。

### 1. 安装 AspectJ 

在使用 AspectJ 之前，需要去官网下载一个安装包，地址：[https://www.eclipse.org/aspectj/downloads.php](https://www.eclipse.org/aspectj/downloads.php) *如果官网下载的很慢，可以从我提供的源码中获取，也可以从其他途径搜索下载 aspectj-1.9.4.jar*

下载完成安装；
- 双击安装
- 命令安装 `java -jar aspectj-1.9.4.jar`
- 配置说明：没有配置，傻瓜式下一步就可以了

![](https://bugstack.cn/assets/images/story/story-2-06.png)

- 默认配置安装完成以后会在C盘创建出一个文件夹 `C:\aspectj1.9`，包括：bin、doc、lib等，后面我们就会使用到这些内容。

### 2. AspectJ 插件

在专业版 IDEA 中开发 AspectJ，需要安装以下两个插件：

- `Spring AOP/@AspectJ`
- `AspectJ Support`

![](https://bugstack.cn/assets/images/story/story-2-07.png)

### 3. 添加依赖 aspectjrt.jar

开始之前需要在项目中添加 `aspectjrt.jar` 依赖，`aspectjrt.jar` 即 AspectJ 安装目录中`lib`目录下的jar包。*你可以复制到工程中引入，也可以直接引入*

![](https://bugstack.cn/assets/images/story/story-2-08.png)

1. 在工程上鼠标右键，点击 Open Module Setting 打开 `Project Structure`
2. 点击 Libraries 选项卡，和上面的 + 号，创建 New Project Library
3. 选择 `C:\aspectj1.9\lib\aspectjrt.jar` 路径，点击即可配置完成

### 4. 配置AspectJ编译器

IDEA 默认使用 `javac` 编译器，这里需要配置 AspectJ 的编译器 `ajc`，在 IDEA 中做相应配置。

![](https://bugstack.cn/assets/images/story/story-2-09.png)

1. 打开 `IDEA -> File  -> Settings` 对话
2. 选择 `Build，Execution，Deployment -> Compiler -> Java Compiler`
3. Use complier：选择 Ajc
4. 在 `Path to aspectjtools.jar` 里配置路径 `C:\aspectj1.9\lib\aspectjtools.jar`

### 5. 案例测试

**创建 Aspect 类**

```java
public aspect DoAspect {

    pointcut logPointcut():call(* ApiTest.hi(..));

    void around():call(void ApiTest.hi(..)){
        System.out.println("call 开始...");
        proceed();
        System.out.println("call 结束...");
    }

    before(): logPointcut(){
        System.out.println("方法执行 before");
    }

    after(): logPointcut(){
        System.out.println("方法执行 after");
    }

}
```

**测试类**

```java
public class ApiTest {

    public void hi(){
        System.out.println("Hi Aspect");
    }

    public static void main(String[] args) {
        ApiTest apiTest = new ApiTest();
        apiTest.hi();
    }

}
```

**测试结果**

```java
call 开始...
方法执行 before
Hi Aspect
call 结束...
方法执行 after

Process finished with exit code 0
```

- 到这，才是一个关于 Aspect 类的正确打开方式，关于 Aspect 的使用也可以尝试搞搞，此篇还只是关于此类切面写法的一个入门。

## 四、总结

- 你的代码越粗犷、越豪放、越骚气，几乎你遇到的问题也是越多的，可能就是因为没有遵守一定的研发执行规范，所以遇到的这些有点傻的问题，几乎会浪费掉你一个上午或者一天。
- 但有些时候如果你能认真对待你弄出来的bug，深入分析下它是如何产生的，并把它复现出来一点点深入研究下，可能也会得到意想不到的收获，也说不定。所以凡是认真，凡事没有坏事。
- 关于切面、关于源码、关于开发，可能并不应该只注重于功能实现，甚至有时候要想办法逃离日复一日没有成长的工作内容。多在那些有价值的技术上下功夫，那你的收获也是最多的。
