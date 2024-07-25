---
title: 栈溢出-StackOverflowError
date: 2021-07-13 23:41
tags: 
  - java
categories:
  - [java]
---


### 问题
生产环境栈溢出

```
3282788         at com.chedaia.task.address.util.ParkingPointUtil.getPreviewGpsPoint(ParkingPointUtil.java:410)
3282789         at com.chedaia.task.address.util.ParkingPointUtil.getPreviewGpsPoint(ParkingPointUtil.java:410)
3282790 Exception in thread "pool-250-thread-43" java.lang.StackOverflowError
3282791         at java.util.AbstractCollection.toArray(AbstractCollection.java:183)
3282792         at java.lang.String.split(String.java:2378)
3282793         at com.chedaia.task.address.util.ParkingPointUtil.isGps(ParkingPointUtil.java:32)
3282794         at com.chedaia.task.address.util.ParkingPointUtil.getPreviewGpsPoint(ParkingPointUtil.java:406)
3282795         at com.chedaia.task.address.util.ParkingPointUtil.getPreviewGpsPoint(ParkingPointUtil.java:410)
3282796         at com.chedaia.task.address.util.ParkingPointUtil.getPreviewGpsPoint(ParkingPointUtil.java:410)
3282797         at com.chedaia.task.address.util.ParkingPointUtil.getPreviewGpsPoint(ParkingPointUtil.java:410)
3282798         at com.chedaia.task.address.util.ParkingPointUtil.getPreviewGpsPoint(ParkingPointUtil.java:410)
3282799         at com.chedaia.task.address.util.ParkingPointUtil.getPreviewGpsPoint(ParkingPointUtil.java:410)

```

### 原因
递归调用次数太多导致  
且启动脚本中设置了-Xss
```
-XX:PermSize=128m -Xss256k 
```

### 解决办法
由于涉及到的代码依赖较多，先不改动代码  
加大线程栈大小到2m
```
-XX:PermSize=128m -Xss2m 
```
----
----

##  线程栈大小与递归次数测试

### 代码
```
    public static void main(String[] args) {
        recursion(0);
    }

    public static void recursion(int i) {
        System.out.println(i);
        i++;
        recursion(i);
    }
```

#### 执行结果

结果：966
>此时的栈设置为： -Xss128k

```
963
964
965
966
Exception in thread "main" java.lang.StackOverflowError
	at sun.nio.cs.UTF_8.updatePositions(UTF_8.java:77)
	at sun.nio.cs.UTF_8.access$200(UTF_8.java:57)
	at sun.nio.cs.UTF_8$Encoder.encodeArrayLoop(UTF_8.java:636)
	at sun.nio.cs.UTF_8$Encoder.encodeLoop(UTF_8.java:691)
	at java.nio.charset.CharsetEncoder.encode(CharsetEncoder.java
```

### 不同栈大小下的递归次数

线程栈大小 | 递归次数约为
---|---
-Xss128k | 966
-Xss256k | 2467
-Xss512k | 5447
-Xss1m   | 11404
-Xss2m   | 23323
-Xss3m   | 35247
-Xss4m   | 41306


### 方法中的局部变量会占用栈空间
当递归的方法中定义了一些局部变量时会占用线程栈的空间，从而影响到递归的次数  
如将上面的方法稍微修改一下，增加3个局部变量：
```
    public static void recursion(int i) {
        System.out.println(i);
        long a = 1l;
        long b = 2l;
        long c = a + b;
        i++;
        recursion(i);
    }
```

在栈大小设置为： -Xss128k 时，结果是：623

```
620
621
622
623
java.lang.StackOverflowError
	at sun.nio.cs.UTF_8.updatePositions(UTF_8.java:77)
	at sun.nio.cs.UTF_8.access$200(UTF_8.java:57)
	at sun.nio.cs.UTF_8$Encoder.encodeArrayLoop(UTF_8.java:636)
	at sun.nio.cs.UTF_8$Encoder.encodeLoop(UTF_8.java:691)
```


----

### 官方说明
https://www.oracle.com/technetwork/java/hotspotfaq-138619.html#threads_oom

您可能遇到线程的默认堆栈大小的问题。在Java SE 6中，Sparc的默认值是32位VM中的512k, 64位VM中的1024k。

在x86 Solaris/Linux上，32位虚拟机中是320k, 64位虚拟机中是1024k。

在Windows上，默认的线程堆栈大小是从二进制文件(java.exe)读取的。在Java SE 6中，这个值在32位VM中是320k，在64位VM中是1024k。

您可以通过使用-Xss选项运行来减小堆栈大小。例如:
```
java -server -Xss64k
```
注意，在某些版本的Windows上，操作系统可能使用非常粗的粒度来计算线程堆栈大小。如果请求的大小小于默认大小1K或更大，则将堆栈大小四舍五入到默认大小;否则，堆栈大小将四舍五入为1 MB的倍数。


64k是每个线程允许的最小堆栈空间量。


>不显式设置-Xss或-XX:ThreadStackSize时，或者把-Xss或者-XX:ThreadStackSize设为0，就是使用“系统默认值”。



### 用命令查看
```
# java -XX:+PrintFlagsFinal -version | grep ThreadStackSize
     intx CompilerThreadStackSize                   = 0                                   {pd product}
     intx ThreadStackSize                           = 1024                                {pd product}
     intx VMThreadStackSize                         = 1024                                {pd product}
java version "1.8.0_162"
Java(TM) SE Runtime Environment (build 1.8.0_162-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.162-b12, mixed mode)
```
