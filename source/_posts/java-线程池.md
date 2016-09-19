---
title: Java 线程池
date: 2016-09-18 14:29:35
tags: java
---
# ThreadPoolExecutor
**Executor**是一个顶层接口，在它里面只声明了一个方法execute(Runnable)；
**ExecutorService**是一个接口，继承了Executor接口，并声明了一些方法：submit、invokeAll、invokeAny以及shutDown等；
**AbstractExecutorService**是一个抽象类，实现了ExecutorService接口，基本实现了ExecutorService中声明的所有方法；
**ThreadPoolExecutor**继承了类AbstractExecutorService。

**在ThreadPoolExecutor类中有几个非常重要的方法：**
**execute()**
execute()方法实际上是Executor中声明的方法，在ThreadPoolExecutor进行了具体的实现，这个方法是ThreadPoolExecutor的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池去执行。 
**submit()**
submit()方法是在ExecutorService中声明的方法，在AbstractExecutorService就已经有了具体的实现，在ThreadPoolExecutor中并没有对其进行重写，这个方法也是用来向线程池提交任务的，但是它和execute(）方法不同，它能够返回任务执行的结果，去看submit()方法的实现，会发现它实际上还是调用的execute()方法，只不过它利用了Future来获取任务执行结果。
**shutdown()**
**shutdownNow()**
shutdown()和shutdownNow()是用来关闭线程池的。
# Executors
Java通过Executors提供四种线程池，分别为：
**newCachedThreadPool**
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
**newFixedThreadPool**
创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
**newScheduledThreadPool**
创建一个定长线程池，支持定时及周期性任务执行。
**newSingleThreadExecutor**
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。