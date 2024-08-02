---
title: 交换器 Exchanger
date: 2018-05-22 16:56
tags: 
  - Java
  - Concurrent
  - Exchanger
categories:
  - [Java,Concurrent]
---

# 交换器 Exchanger

**Exchanger** 是 Java 并发包中的一个同步工具类，位于 **java.util.concurrent** 包下。它提供了一个用于在两个线程之间交换数据的同步点。Exchanger 用于当两个线程希望在某个同步点交换数据时，可以使用它来实现数据交换。


#### Exchanger 的工作机制如下：

- 两个线程通过调用 exchange 方法来到达同步点。
- 第一个线程调用 exchange 方法时，会等待第二个线程也调用 exchange 方法。
- 当两个线程都调用 exchange 方法时，它们会交换数据，即第一个线程将自己的数据传递给第二个线程，同时接收第二个线程的数据。


下面是一个简单的示例，展示了如何使用 Exchanger 在两个线程之间交换数据：

```
import java.util.concurrent.Exchanger;

public class ExchangerExample {
    public static void main(String[] args) {
        Exchanger<String> exchanger = new Exchanger<>();

        // 线程A
        new Thread(() -> {
            try {
                String dataA = "Data from A";
                System.out.println("Thread A is exchanging data: " + dataA);
                String receivedData = exchanger.exchange(dataA);
                System.out.println("Thread A received: " + receivedData);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }).start();

        // 线程B
        new Thread(() -> {
            try {
                String dataB = "Data from B";
                System.out.println("Thread B is exchanging data: " + dataB);
                String receivedData = exchanger.exchange(dataB);
                System.out.println("Thread B received: " + receivedData);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }).start();
    }
}


```

**Exchanger** 通常用于一些需要线程之间进行数据交换的场景，比如生产者-消费者模型中，当生产者和消费者希望交换缓冲区时，可以使用 Exchanger 来实现。