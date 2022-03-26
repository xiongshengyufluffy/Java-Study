# 一条update语句是如何执行的
## 第一阶段:加载数据到Buffer Pool
- 涉及到的数据结构:
    - free链表
    - lru链表
![一条update语句是如何执行的--1](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/一条update语句是如何执行的--1.png)

## 第二阶段:在InnoDB中执行更新操作
- 涉及到的数据结构:
  - lru链表
  - flush链表

![20210411182531](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210411182531.png)

## 第三阶段:缓冲池内存不足触发脏页刷盘

- 涉及到的数据结构
  - lru链表
  - flush链表
  - free链表

![6](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/6.png)
![](../../Tools/shortcut/photos/shotcut/1.png)

![20210411193018](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/20210411193018.png)