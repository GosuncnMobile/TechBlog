---
title: Android编译时注解
date: 2017-12-27 11:40:17
categories:
- android
tags:
- android
- 注解
- 线程切换
---

# Android编译时注解


[TOC]

## 前言
> 相信大家经常都使用到注解，如果使用过AndroidAnnotations,Dagger2,EventBus,RxJava,BufferKnife等开源项目，对注解应该更为深刻，这些项目的原理基本都是基于编译时注解动态生成代码，效果等同于手写代码，因此相对反射来说性能基本无影响。

> 另外，已经实现了[注解轻松实现线程切换开源项目](https://github.com/android-coding-well/AwesomeTool)，欢迎fork&star.
## 了解注解
### 注解的概念
> 注解（Annotation），也叫元数据（Metadata），是Java5的新特性，JDK5引入了Metadata很容易的就能够调用Annotations。注解与类、接口、枚举在同一个层次，并可以应用于包、类型、构造方法、方法、成员变量、参数、本地变量的声明中，用来对这些元素进行说明注释。
<!--more-->
### 注解的语法与定义

 1. 以@interface关键字定义
 2. 注解可以包含成员，成员以无参数的方法的形式被声明，其方法名和返回值定义了该成员的名字和类型。
 3. 成员赋值是通过@Annotation(name=value)的形式。
 4. 注解需要标明注解的生命周期，注解的修饰目标等信息，这些信息是通过元注解实现。

以 java.lang.annotation 中定义的 Target 注解为例：
``` java
@Retention(value = RetentionPolicy.RUNTIME)
@Target(value = { ElementType.ANNOTATION_TYPE } )
public @interface Target {
    ElementType[] value();
}
```
源码分析如下：
第一：元注解@Retention，成员value的值为RetentionPolicy.RUNTIME。
第二：元注解@Target，成员value是个数组，用{}形式赋值，值为ElementType.ANNOTATION_TYPE
第三：成员名称为value，类型为ElementType[]
另外，需要注意一下，如果成员名称是value，在赋值过程中可以简写。如果成员类型为数组，但是只赋值一个元素，则也可以简写。如上面的简写形式为：
``` java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```
### 注解的分类

 **1** 基本内置注解，是指Java自带的几个Annotation，如@Override、Deprecated、@SuppressWarnings等；
 **2** 元注解（meta-annotation），是指负责注解其他注解的注解，JDK 1.5及以后版本定义了4个标准的元注解类型，如下：
``` java
@Target
@Retention
@Documented
@Inherited
```
 **3** 自定义注解，根据需要可以自定义注解，自定义注解需要用到上面的meta-annotation


### 元注解
* Java定义了4个标准的元注解（ <span  style="color:#F00">java8之后新增了**@Repeatable**元注解</span>）：
* **@Documented**：标记注解，注解表明这个注解应该被 javadoc工具记录. 默认情况下,javadoc是不包括注解的. 但如果声明注解时指定了 @Documented,则它会被 javadoc 之类的工具处理, 所以注解类型信息也会被包括在生成的文档中。
``` java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}
```
---
* **@Inherited**：标记注解，允许子类继承父类的注解，此注解理解有难度，可以参考[这里](http://blog.csdn.net/lemon89/article/details/47836783)，总的来说就是子类在继承父类时如果父类上的注解有此@Inherited标记，那么子类就能把父类的这个注解继承下来，如果没有@Inherited标记，那么子类在继承父类之后并没有继承父类的注解（不知道有没有说明白了，不明白就还是点进链接去看下吧）。
``` java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```
---
* **@Retention**：指Annotation被保留的时间长短，标明注解的生命周期
``` java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    RetentionPolicy value();
}
```
> 注解需要标明注解的生命周期，这些信息是通过元注解 @Retention 实现，注解的值是 enum 类型的 RetentionPolicy，包括以下几种情况：
``` java
public enum RetentionPolicy {
    /**
     * 注解只保留在源文件，当Java文件编译成class文件的时候，注解被遗弃.
     * 这意味着注解仅存在于编译器处理期间，编译器处理完之后，该注解就没用了，在class文件找不到了
     */
    SOURCE,

    /**
     * 注解被保留到class文件，但jvm加载class文件时候被遗弃，这是默认的生命周期.
     * 简单来说就是你在class文件中还能看到注解
     */
    CLASS,

    /**
     * 注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在，
     * 保存到class对象中，可以通过反射来获取
     */
    RUNTIME
}
```
---
* **@Target**：标明注解的修饰目标（ <span  style="color:#F00">java8为ElementType枚举增加了**TYPE_PARAMETER**、**TYPE_USE**两个枚举值</span>）
``` java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}

// ElementType取值
public enum ElementType {
    /** 类、接口（包括注解类型）或枚举 */
    TYPE,
    /** field属性，也包括enum常量使用的注解 */
    FIELD,
    /** 方法 */
    METHOD,
    /** 参数 */
    PARAMETER,
    /** 构造函数 */
    CONSTRUCTOR,
    /** 局部变量 */
    LOCAL_VARIABLE,
    /** 注解上使用的元注解 */
    ANNOTATION_TYPE,
    /** 包 */
    PACKAGE
}
```
## 注解处理器(Annotation Processor)
### 概述
> 注解处理器是javac的一个工具，它用来在编译时扫描和处理注解（Annotation）。你可以自定义注解，并注册到相应的注解处理器，由注解处理器来处理你的注解。一个注解的注解处理器，以Java代码（或者编译过的字节码）作为输入，生成文件（通常是.java文件）作为输出。这些生成的Java代码是在生成的.java文件中，所以你不能修改已经存在的Java类，例如向已有的类中添加方法。这些生成的Java文件，会同其他普通的手动编写的Java源代码一样被javac编译。
> 
> 简单来说，在源代码编译阶段，通过注解处理器，将标记了注解的类、方法等作为输入内容，经过注解处理器进行处理，产生需要的java代码。
>
>Android Gradle插件2.2版本发布后，Android 官方提供了annotationProcessor来代替android-apt，annotationProcessor支持 javac 和 jack 编译方式，而android-apt只支持 javac 编译方式。
### 使用
* 直接在Module中使用，比之前Android-apt使用方式更加简单。
```
dependencies {
	        compile 'com.github.huweijian5:AwesomeTool:latest_version'
		annotationProcessor 'com.github.huweijian5:AwesomeTool-compiler:latest_version'
}
```


## 实例说明
> 接下来以本人写的一个注解实现线程切换的项目为例，讲解下编译时注解的编码过程。
### 项目结构
> 本项目主要分为三个Module，分别为lib_api，lib_annotation，lib_compiler。
> 其中lib_api主要存放供外界使用的接口，是对外开放的
> lib_annotation里指定了自定义注解的定义
> lib_compiler里实现注解处理器，是本项目的核心

*  项目目录结构如下图：
![这里写图片描述](http://img.blog.csdn.net/20171227091018608?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaHV3ZWlqaWFuNQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 依赖关系图如下：
``` sequence
app->lib_api:dependence
lib_api->lib_annotation: dependence
lib_compiler->lib_annnotaion: dependence
```
> 值得注意的是，lib_annotation和lib_compiler都是java工程（apply plugin: 'java'），而lib_api是android工程（apply plugin: 'com.android.library'）
### lib_annotation
> 此Module主要实现自定义的注解定义
``` java
/**
 * 注入对象实例
 * Created by HWJ on 2017/3/12.
 */
@Documented
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.FIELD)
public @interface InjectObject {

    /**
     * 线程优先级-20~19,-20代表优先级最高，详见android.os.Process,默认为THREAD_PRIORITY_DEFAULT(0)
     * @return
     */
    int priority() default 0;
}
```
``` java
/**
 * 后台线程注解
 * Created by HWJ on 2017/3/12.
 */
@Documented
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.METHOD)
public @interface WorkInBackground {

}
```
``` java
/**
 1. UI线程注解
 2. Created by HWJ on 2017/3/12.
 */
@Documented
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.METHOD)
public @interface WorkInMainThread {
}
```
### lib_api
 * 定义接口
``` java
 public interface IObjectInjector<T> {
    void inject(T t);
}
```
*  利用反射进行注入实例
``` java
public static void inject(Object target) {

        //获取生成类全称
        Class<?> clazz = target.getClass();
        String proxyClassFullName = clazz.getName() + ConstantValue.SUFFIX;
        Class<?> proxyClazz = null;
        try {
            //反射生成类实例对象并进行注入
            proxyClazz = Class.forName(proxyClassFullName);
            IObjectInjector objectInjector = (IObjectInjector) proxyClazz.newInstance();
            objectInjector.inject(target);
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("注入失败："+e.getMessage());
        }
    }
```
### lib_compiler
> lib_compiler中实现注解处理器，是项目的核心，本项目是使用Handler和HandlerThread实现线程切换，对于HandlerThread线程切换的使用网络文章已经有很多，本文不再赘述。

**1 添加以下依赖：**
```
dependencies {

.

//auto-service库可以帮我们去生成META-INF等信息
compile 'com.google.auto.service:auto-service:1.0-rc4'
//用于生成源代码
compile 'com.squareup:javapoet:1.9.0'

}

//只有android N支持java8，如果你写1.8之后，强制要你使用buildToolsVersion为24.0.0
sourceCompatibility = "1.7"
targetCompatibility = "1.7"

.


```
> 注意此Module为java工程，而不是android工程，如果弄错了就会报找不到类AbstractProcessor的错误

---
**2 继承AbstractProcessor,并实现方法init,getSupportedAnnotationTypes,getSupportedSourceVersion,process四个方法即可，参考代码如下：**
``` java
@AutoService(Processor.class)
public class AwesomeToolProcessor extends AbstractProcessor {
    private static final String TAG = "AwesomeToolProcessor";
    private Filer mFileUtils;//跟文件相关的辅助类，生成JavaSourceCode
    private Elements elementUtils;//跟元素相关的辅助类，帮助我们去获取一些元素相关的信息
    private Messager messager;//跟日志相关的辅助类
    private Map<String, AwesomeToolProxyInfo> proxyInfoMap = new HashMap<String, AwesomeToolProxyInfo>();//key为注解所在类的全名

    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        messager = processingEnv.getMessager();
        elementUtils = processingEnv.getElementUtils();
        mFileUtils=processingEnv.getFiler();
    }

    /**
     * 支持的注解类型
     *
     * @return
     */
    @Override
    public Set<String> getSupportedAnnotationTypes() {
        HashSet<String> supportTypes = new LinkedHashSet<>();
        supportTypes.add(InjectObject.class.getCanonicalName());
        supportTypes.add(WorkInBackground.class.getCanonicalName());
        supportTypes.add(WorkInMainThread.class.getCanonicalName());
        return supportTypes;
    }

    /**
     * 注解处理器支持到的JAVA版本
     * @return
     */
    @Override
    public SourceVersion getSupportedSourceVersion() {
        printMessage("SupportedSourceVersion=%s",SourceVersion.latestSupported().name());
        return SourceVersion.latestSupported();
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        printMessage("process:annotations size=%d", annotations.size());
        proxyInfoMap.clear();
        handleInjectObjectAnnotation(roundEnv);
        handleWorkInBackgroundAnnotation(roundEnv);
        handleWorkInMainThreadAnnotation(roundEnv);

        printMessage("AwesomeToolProxyInfo Map size=%d", proxyInfoMap.size());

        generateSourceFiles();
        return false;//如果返回true,当有两个注解作用在同一方法上，那么第一个处理完了之后就不会再处理第二个
    }

```

> 注解@AutoService(Processor.class)可以自动帮我们处理一些工作，简化代码
> 
> init中可以获得Messager用来打印信息，打印的信息会显示在AndroidStudio的Gradle Console窗口
> 同时也可以获得Elements，用来获取元素的相关信息，还有Filer,可以用来生成代码
> 
> getSupportedAnnotationTypes里需要返回支持的注解类型，就是lib_annotation中定义的注解
>
>getSupportedSourceVersion为注解处理器支持到的java版本
>
> process里处理注解元素作用的类方法等，根据自己的业务逻辑处理并生成相应代码

---
**3 处理注解**
> 通过RoundEnvironment.getElementsAnnotatedWith()可以获得注解所在的方法类等，如下
``` java
 Set<? extends Element> elesWithBind = roundEnv.getElementsAnnotatedWith(WorkInMainThread.class);
```
* 其中Element的类型及说明如下：

| 类型 | 说明 |
| :-------- | ---- |
|ExecutableElement |Represents a method, constructor, or initializer (static or instance) of a class or interface, including annotation type elements.|
|VariableElement|Represents a field, {@code enum} constant, method or constructor parameter, local variable, resource variable, or exception parameter.|
|PackageElement |Represents a package program element.  Provides access to information about the package and its members.|


*  获取方法参数,参考代码如下：
``` java
 for (VariableElement variableElement : executableElement.getParameters()) {
                System.out.println("参数类型及名称：" + variableElement.asType() + "," + variableElement.getSimpleName());
            }
```
---
**4 生成代码**
> 生成代码的方式可以通过手动拼接字符串，也可以通过开源库javapoet实现。
``` java
            try {
                JavaFileObject jfo = processingEnv.getFiler().createSourceFile(
                        proxyInfo.getProxyClassFullName(),//类名全称
                        proxyInfo.getTypeElement());//类元素
                Writer writer = jfo.openWriter();
                writer.write(proxyInfo.generateJavaCode());
                writer.flush();
                writer.close();
            } catch (IOException e) {
                error(proxyInfo.getTypeElement(),
                        "Unable to write injector for type %s: %s",
                        proxyInfo.getTypeElement(), e.getMessage());
            }

```
---
> 至此已经走完了编译时注解的整个流程，最后贴下生成的代码：
``` java
//Generated code. Do not modify!
//自动生成代码，请勿修改！
package com.junmeng.aad;

import android.os.Handler;
import android.os.HandlerThread;
import android.os.Message;

import com.junmeng.api.inter.IObjectInjector;

import java.lang.ref.WeakReference;

import java.util.ArrayList;
import java.util.List;

public class BlankFragmentHelper implements IObjectInjector<BlankFragment> {
    public static final int MESSAGE_NEEDWORKINTHREAD = 1;
    public static final int MESSAGE_NEEDWORKINMAINTHREAD = 2;
    private Handler mainHandler;
    private Handler workHandler;
    private HandlerThread handlerThread;
    private WeakReference<BlankFragment> target;

    @Override
    public void inject(final BlankFragment target) {
        if (target.blankFragmentHelper != null) {
            target.blankFragmentHelper.quit();
        }
        target.blankFragmentHelper = new BlankFragmentHelper();
        target.blankFragmentHelper.init(target);
    }

    public void init(final BlankFragment target) {
        this.target = new WeakReference<BlankFragment>(target);
        handlerThread = new HandlerThread("thread_BlankFragmentHelper", -16);
        handlerThread.start();
        mainHandler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                List<Object> params;
                switch (msg.what) {
                    case MESSAGE_NEEDWORKINMAINTHREAD:
                        params = (List<Object>) msg.obj;
                        target.needWorkInMainThread();
                        break;
                }
            }
        };
        workHandler = new Handler(handlerThread.getLooper()) {
            @Override
            public void handleMessage(Message msg) {
                List<Object> params;
                switch (msg.what) {
                    case MESSAGE_NEEDWORKINTHREAD:
                        params = (List<Object>) msg.obj;
                        target.needWorkInThread((java.lang.String) params.get(0), (int) params.get(1), (double) params.get(2), (com.junmeng.aad.Test) params.get(3));
                        break;
                }
            }
        };
    }

    public void needWorkInThread(java.lang.String str, int i, double d, com.junmeng.aad.Test test) {
        List<Object> params = new ArrayList<>();
        params.add(str);
        params.add(i);
        params.add(d);
        params.add(test);
        workHandler.sendMessage(workHandler.obtainMessage(MESSAGE_NEEDWORKINTHREAD, params));
    }

    public void needWorkInMainThread() {
        List<Object> params = new ArrayList<>();
        mainHandler.sendMessage(mainHandler.obtainMessage(MESSAGE_NEEDWORKINMAINTHREAD, params));
    }

    /**
     * 在不用时务必调用此方法，防止内存泄漏
     */
    public void quit() {
        if (handlerThread != null && handlerThread.isAlive()) {
            handlerThread.quitSafely();
        }
    }
}

```
> 另外，说明下google的auto-service实际上会帮助我们生成jar包并添加META-INF信息，如下图
![这里写图片描述](http://img.blog.csdn.net/20171227112008839?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaHV3ZWlqaWFuNQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
# 如有错误之处请指正，谢谢。
# 待解决的关键问题
* 按照上面的实现只能在build的时候才能生成代码，有没有即时生成的技术呢？还没找到答案。
# 参考：
* http://blog.csdn.net/wzgiceman/article/details/54580745
* http://blog.csdn.net/lmj623565791/article/details/43452969
* http://blog.csdn.net/github_35180164/article/details/52107204
* https://www.jianshu.com/p/28edf5352b63





