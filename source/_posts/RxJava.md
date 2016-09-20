---
title: RxJava
date: 2016-09-20 15:19:32
tags:
---
# RxJava 的观察者模式
RxJava 有四个基本概念：Observable (可观察者，即被观察者)、 Observer (观察者)、 subscribe (订阅)、事件。Observable 和 Observer 通过 subscribe() 方法实现订阅关系，从而 Observable 可以在需要的时候发出事件来通知 Observer。

与传统观察者模式不同， RxJava 的事件回调方法除了普通事件 onNext() （相当于 onClick() / onEvent()）之外，还定义了两个特殊的事件：onCompleted() 和 onError()。

- onCompleted(): 事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的onNext() 发出时，需要触发 onCompleted() 方法作为标志。
- onError(): 事件队列异常。在事件处理过程中出异常时，onError() 会被触发，同时队列自动终止，不允许再有事件发出。
- 在一个正确运行的事件序列中, onCompleted() 和 onError() 有且只有一个，并且是事件序列中的最后一个。需要注意的是，onCompleted() 和 onError() 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。

create() 方法是 RxJava 最基本的创造事件序列的方法。基于这个方法， RxJava 还提供了一些方法用来快捷创建事件队列，例如：
- just(T...): 将传入的参数依次发送出来。
Observable observable = Observable.just("Hello", "Hi", "Aloha");
- from(T[]) / from(Iterable<? extends T>) : 将传入的数组或 Iterable 拆分成具体对象后，依次发送出来。

<!-- more -->
# 线程控制 —— Scheduler
在不指定线程的情况下， RxJava 遵循的是线程不变的原则，即：在哪个线程调用 subscribe()，就在哪个线程生产事件；在哪个线程生产事件，就在哪个线程消费事件。如果需要切换线程，就需要用到 Scheduler （调度器）。

Scheduler 的 API
在RxJava 中，Scheduler ——调度器，相当于线程控制器，RxJava 通过它来指定每一段代码应该运行在什么样的线程。RxJava 已经内置了几个 Scheduler ，它们已经适合大多数的使用场景：
- Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
- Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。
- Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。
- Schedulers.computation(): 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。
- 另外， Android 还有一个专用的 AndroidSchedulers.mainThread()，它指定的操作将在 Android 主线程运行。

有了这几个 Scheduler ，就可以使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制了。
- subscribeOn(): 指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程。
- observeOn(): 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。

# Demo
```
        Map<String, String> map = new HashMap<>();
        map.put("1", "1");
        map.put("2", "2");
        map.put("3", "3");
        map.put("4", "4");
        map.put("5", "5");

        Observable.from(map.values()).map(new Func1<String, String>() {
            @Override
            public String call(String s) {
                Log.d("lll", Thread.currentThread().getName() + "  map->" + s);
                return s;
            }
        }).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<String>() {
                    @Override
                    public void onCompleted() {
                        Log.d("lll", Thread.currentThread().getName() + "  ->onCompleted");
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(String s) {
                        Log.d("lll", Thread.currentThread().getName() + "  ->next:" + s);
                    }
                });
```
```
09-20 15:19:08.509 6209-6248/shinelee.test D/lll: RxIoScheduler-2  map->5
09-20 15:19:08.509 6209-6248/shinelee.test D/lll: RxIoScheduler-2  map->4
09-20 15:19:08.509 6209-6248/shinelee.test D/lll: RxIoScheduler-2  map->1
09-20 15:19:08.509 6209-6248/shinelee.test D/lll: RxIoScheduler-2  map->3
09-20 15:19:08.510 6209-6248/shinelee.test D/lll: RxIoScheduler-2  map->2
09-20 15:19:08.526 6209-6209/shinelee.test D/lll: main  ->next:5
09-20 15:19:08.526 6209-6209/shinelee.test D/lll: main  ->next:4
09-20 15:19:08.526 6209-6209/shinelee.test D/lll: main  ->next:1
09-20 15:19:08.526 6209-6209/shinelee.test D/lll: main  ->next:3
09-20 15:19:08.526 6209-6209/shinelee.test D/lll: main  ->next:2
09-20 15:19:08.526 6209-6209/shinelee.test D/lll: main  ->onCompleted
```