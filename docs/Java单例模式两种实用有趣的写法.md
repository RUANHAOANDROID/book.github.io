---
title: Java单例模式两种实用有趣的写法
categories: -技术
tags: 
- Java 
- Thinking in Java 
- Java编程思想
- Java
- 单例模式
date: 2016.04.01 09:52:14
---
# Java单例模式两种实用有趣的写法
## 1.内部类实现
``` java
Class Singleton{
    
   private static class HelperHolder{

   public static final Helper helper=new Helper();
       
   }
    
   public static Helper newInstance(){
        return HelperHolder.helper;
   }
}
```
为什么说有趣呢，内部类的实现能够延迟初始化（Lazy initialization）,并且多线程安全，还保证了高性能。

为什么会延迟初始化呢，因为java的语言特性，内部类只有在使用的时候才会去加载，从而初始化内部静态变量。

为什么没有加线程锁会是线程安全的呢，Java运行环境自动给你保证的，加载的时候会自动隐形同步。

为什么是高性能呢,在访问对象时，不需要同步java虚拟机，又会自动给你取消息同步，所以效率高。

## 2.枚举实现
``` java
public enum Singleton{

    INSTANCE;

    private Singleton(){

    }

    public Helper newInstance(){

    return new Helper()；

    }

}
```
枚举的实现 即使使用反射机制也无法多次实例化一个枚举量，也是线程安全的



<p style="text-align:right"> 摘自 Thinking in Java《JAVA编程思想》 </p>

