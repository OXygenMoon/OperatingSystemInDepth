<p align='center'>
<a href="https://oxygenpanda.github.io/" target="_blank"><img alt="Website" src="https://img.shields.io/badge/博客-劳振煜的知識倉儲-faf2f2.svg?style=flat-square&logo=Blogger"></a>
<a href="https://www.github.com/OXygenPanda" target="_blank"><img src="https://img.shields.io/badge/Github-@劳振煜-f3e1e1.svg?style=flat-square&logo=GitHub"></a>
<a href="https://i.loli.net/2020/11/11/SBZ2mFJGKLjUtTO.jpg" target="_blank"><img src="https://img.shields.io/badge/微信-@OXygen-f1d1d1.svg?style=flat-square&logo=WeChat"></a>

# 深入理解操作系统 第一章

>   第一章的主要内容是 : 操作系统的一些知识

## 操作系统是什么？

用户角度：操作系统是一个控制软件

*   管理应用程序
*   为应用程序提供服务
*   杀死应用程序

程序角度：操作系统是资源管理器

*   管理外设、分配资源
*   抽象
    *   将CPU抽象成进程
    *   将磁盘抽象成文件
    *   将内存抽象成地址空间

## 操作系统层次

位于硬件之上，应用程序之下。

## 操作系统的界面和内核

Linux Windows Android 的界面属于外壳(Shell) ，而不是内核(kernel)。操作系统研究的是内核，处于Shell之下。

## 操作系统内部组件

*   CPU调度器
*   物理内存管理
*   虚拟内存管理
*   文件系统管理
*   中断处理与设备驱动

## 操作系统特征

*   并发
    *   一段时间内运行多个进程（并行 : 一个时间点运行多个进程，一般要求有多个CPU)
    *   需要OS管理和调度
*   共享
    *   “同时”共享
    *   互斥共享
*   虚拟
    *   让每一个用户觉得的有一个计算机专门为他服务
*   异步
    *   程序是走走停停，而不是一直运行


