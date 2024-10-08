---
title: TOP 命令
date: 2021-02-02 13:22
tags: 
  - Linux
  - Shell
  - Top
categories:
  - [操作系统, Linux]
---

Top程序提供了一个运行系统的动态实时视图。它可以显示系统摘要信息以及当前由Linux内核管理的进程或线程列表。 所显示的系统摘要信息的类型以及为进程显示的信息的类型，顺序和大小都是用户可配置的，并且配置可以在重新启动时保持不变。
该程序为过程操作提供了一个有限的交互式界面，以及一个用于个人配置的更广泛的界面 - 涵盖了其操作的每个方面。

## 一、显示区域介绍

### 系统信息栏
**第一行（TOP）**
```
top - 14:28:52 up 39 days, 20:56,  1 user,  load average: 0.20, 0.42, 0.52
```
- 系统当前时间
- 系统已经运行时间
- 登录当前系统的终端数
- **load average** 为当前系统负载的平均值，后面的三个值分别为1分钟前、5分钟前、15分钟前进程的平均数
>平均负载(load average)是指系统的运行队列的平均利用率，也可以认为是可运行进程的平均数。  
>多核CPU的话，满负荷状态的数字为 "1.00 * CPU核数"，即双核CPU为2.00，四核CPU为4.00。 

**第二行（Tasks）**
```
Tasks: 236 total,   2 running, 234 sleeping,   0 stopped,   0 zombie
```
当前系统的进程情况

字段 | 含义
---|---
total | 为当前系统进程总数
running | 为当前运行中的进程数
sleeping | 为当前处于等待状态中的进程数
stoped | 为被停止的系统进程数
zombie | 为被复原的进程数
 
**第三行（Cpus）**
```
%Cpu(s):  2.9 us,  1.4 sy,  0.0 ni, 95.1 id,  0.5 wa,  0.0 hi,  0.1 si,  0.0 st
```
分别表示了 CPU 当前的使用率

字段 | 含义
---|---
us | 用户空间占用CPU百分比
sy | 内核空间占用CPU百分比
ni | 用户进程空间内改变过优先级的进程占用CPU百分比
id | 空闲CPU百分比
wa | 等待输入输出的CPU时间百分比
hi | 硬件中断
si | 软件中断
st | 实时


**第四行（Mem）**
```
GiB Mem :   62.645 total,    4.274 free,   22.042 used,   36.329 buff/cache
```
分别表示了内存总量、空闲内存量、当前使用量、以及缓冲使用中的内存量

**第五行（Swap）**
```
GiB Swap:   64.000 total,   63.203 free,    0.797 used.   39.949 avail Mem
```
表示类别同第四行（Mem），但此处反映着交换分区（Swap）的使用情况。通常，交换分区（Swap）被频繁使用的情况，将被视作物理内存不足而造成的。

### 进程列表栏
```
 PID USER       PR   NI   VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
23090 root      20   0  9.793g 818716   7460 S  14.6  1.2   1971:30 java
 3617 root      20   0 20.274g 220676   7468 S  12.6  0.3  10874:44 java
23496 root      20   0  9.780g 444548   7524 S   8.3  0.7   1228:19 java
 2648 elastic+  20   0 50.123g 1.527g   7612 S   7.3  2.4 662:56.49 java
```

列名 | 含义
---|---
PID | 进程id
PPID | 父进程id
RUSER | Real user name
UID | 进程所有者的用户id
USER | 进程所有者的用户名
GROUP | 进程所有者的组名
TTY | 启动进程的终端名。不是从终端启动的进程则显示为 ?
PR | 优先级
NI | nice值。负值表示高优先级，正值表示低优先级
P | 最后使用的CPU，仅在多CPU环境下有意义
%CPU | 上次更新到现在的CPU时间占用百分比
TIME | 进程使用的CPU时间总计，单位秒
TIME+ | 进程使用的CPU时间总计，单位1/100秒
%MEM | 进程使用的物理内存百分比
VIRT | 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
SWAP | 进程使用的虚拟内存中，被换出的大小，单位kb。
RES | 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
CODE | 可执行代码占用的物理内存大小，单位kb
DATA | 可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb
SHR | 共享内存大小，单位kb
nFLT | 页面错误次数
nDRT | 最后一次写入到现在，被修改过的页面数。
S | 进程状态。<br>D=不可中断的睡眠状态<br>R=运行<br>S=睡眠<br>T=跟踪/停止<br>Z=僵尸进程
COMMAND | 命令名/命令行
WCHAN | 若该进程在睡眠，则显示睡眠中的系统函数名
Flags | 任务标志，参考 sched.h

>默认情况下仅显示比较重要的  PID、USER、PR、NI、VIRT、RES、SHR、S、%CPU、%MEM、TIME+、COMMAND  列。

可以使用快捷键 **f** 更改显示内容

>有时候进程的cpu使用率可能大于100，那是因为该进程启用了多线程占用了多个核心，所以有时候该值得时候会超过100%，但不会超过总核数*100。

-----

## 二、命令介绍
### 命令行
详细内容可以参考MAN 帮助文档。

命令格式：

top [-] [d] [p] [q] [c] [C] [S]    [n]

参数说明：

d：  指定每两次屏幕信息刷新之间的时间间隔。当然用户可以使用s交互命令来改变之。

p：  通过指定监控进程ID来仅仅监控某个进程的状态。

q：该选项将使top没有任何延迟的进行刷新。如果调用程序有超级用户权限，那么top将以尽可能高的优先级运行。

S： 指定累计模式

s ： 使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险。

i：  使top不显示任何闲置或者僵死进程。

c：  显示整个命令行而不只是显示命令名


### 交互式命令

1 显示所有的CPU核心的使用率

d/s 改变刷新平率

l - 关闭或开启第一部分第一行 top 信息的表示

t - 关闭或开启第一部分第二行 Tasks 和第三行 Cpus 信息的表示

m - 关闭或开启第一部分第四行 Mem 和 第五行 Swap 信息的表示

N - 以 PID 的大小的顺序排列表示进程列表

P - 以 CPU 占用率大小的顺序排列进程列表

M - 以内存占用率大小的顺序排列进程列表

h - 显示帮助

n - 设置在进程列表所显示进程的数量

q - 退出 top

c - 显示进程的命令名称或命令行

Z - 改变颜色映射

z - 显示或关闭颜色

B - 开启/关闭加粗显示

b - 高亮显示排序字段

f - 显示和隐藏字段

F/O - 控制排序的字段

< / > - 控制排序的字段

R - 升序排列或倒序排列

x - 高亮显示排序字段

H - 显示线程

u - 仅显示指定的用户



