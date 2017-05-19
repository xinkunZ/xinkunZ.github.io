---
layout: post
title: Java ClassPath
tags:
- classpath
- exception & error
categories: java
description: 理解java classpath，以及Exception与error
---
# classpath到底是什么
* classpath字面意思，类路径，到底如何理解呢
    * 首先，我们可以把java程序启动之后，运行的class集合当成一个独立的操作系统，classpath就是他们的根路径“/”，在运行的这个java程序中，所有的资源文件都是相对于这个/来寻找的
    * 因此也就不难理解为什么jpos中的FileUtils可以从jpos-oem和jpoe-oem1.jar中解压复制文件，因为他们都被加载到了classpath中，可以通过根路径/查询
    * 相当于java在启动时，会将所有的jar和非jar的文件全部复制到同一个根路径/下面，按照包名排排好，所有的com放在一起，所有的org放在一起，逐层往下。这个根路径就是classpath

## classpath设计有什么好处呢
* 可以让我们在自己程序中不受限制的使用不同jar中的资源文件，示例如下：
* 比如A.jar中有目录/resource/1.png,那么可以在任意一个类中使用`this.getClass().getResource("/resource/1.png");`来获取到这个文件的URL，也可以使用`getResourceAsStream()`方法将该对象读成流，从而做一些文件流的操作
* 而需要注意的是，上述`getResource()`方法读取到的URL，不可以用`new File(URL.getPath())`的方式获取文件，因为这个path：`this.getClass().getResourceAsStream("/jpos-loader.xml")`是针对java运行环境特殊处理的，`new File(path)`是取不到这个文件的
* 这种实现方式归功于classloader，要如此使用，首先要确保jar文件在运行时被加载。也就是在eclipse中配置的java build path，为工程添加jar

# 反射
* 反射内部类时要使用com.package.Class$InnerClass的方式
* 实例化对象时可以使用Class.getConstructor()，使用指定参数的构造方法创建对象
```java
  public static void main(String[] args) throws Exception {
    Class c = StringBuilder.class;
    Constructor constructor = c.getConstructor(String.class);
    Object s = constructor.newInstance("hello");
    System.out.println(s.toString());
  }
```
## 内部类和外部类的反射
* 内部类调用外部类的方法时，如果当前对象是外部类的子类，并且重写了上述方法，那么这个重写不会实现。即，父类中的内部类仍然会调用父类的方法，不会调用子类中override的方法


# 关于Exception和Error 
* 并不是所有的error都会在编译时暴露，比如`java.lang.UnsatisfiedLinkError`和`java.lang.NoClassDefFoundError`
* 如果在动态调用时没有注意这个问题，那么当出现上述error的时候，`Exception`是捕获不到的，需要使用`Throwable`才行
* 顺带一提：`classNotFoundException`和`NoClassDefFoundError`的区别：
    * 前者是编译时编译器的报错，基本上就是jar包添加的问题，容易排查。后者是运行时的错误
* 如果你的代码里明明报了错，但是`Exception`没有捕获到，那么你可以看看是不是发生了`Error`，需要将`catch`中改成`Throwable`


