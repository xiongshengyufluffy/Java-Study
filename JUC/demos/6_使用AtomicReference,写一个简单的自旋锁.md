# 写一个Demo,模拟两个线程使用自旋锁来设置引用
- 分析
  - 资源为线程类型的原子引用atomicReference
  - MyLock() 方法
    - 以CAS的方式将atomicReference: null -> 当前线程
      - 若CAS失败,则一直自旋(表示没有获取到锁)
  - MyUnlock() 方法
    - 以CAS的方式将atomicReference: 当前线程 -> null
  - 流程
    - 在main中初始化资源类  
    - 线程A启动,获得锁,线程B启动,尝试获得锁
      - 保证线程A一定先于线程B获得锁
    - 线程A获得锁之后需要5s才会释放锁,线程B获得锁之后需要1s才会释放锁
    - 线程B自旋等待线程A释放锁
    - 线程A释放锁后
      - 线程B获得锁,线程B释放锁

- 代码
```
package demo05;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;

public class mySpinLock {
    AtomicReference<Thread> atomicReference = new AtomicReference<> ();

    public static void main(String[] args) {
        mySpinLock mySpinLock = new mySpinLock();
        new Thread(() -> {
            mySpinLock.myLock();
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            mySpinLock.myUnLock();
        }, "A").start();
        try {
            TimeUnit.SECONDS.sleep(1); // 确保线程A一定比线程B先开始执行
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> {
            mySpinLock.myLock();
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            mySpinLock.myUnLock();
        }, "B").start();
    }
    public void myLock(){
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "尝试获得锁");
        int i = 0;
        while(!atomicReference.compareAndSet(null, thread)){
            if(i == 0){
                System.out.println(thread.getName() + "尝试获得锁失败, 进入自旋");
                i++;
            }
        }
        System.out.println(thread.getName() + "获得锁成功");
    }

    public void myUnLock(){
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "马上释放锁");
        atomicReference.compareAndSet(thread, null);
    }
}

```

- 输出
```
A尝试获得锁
A获得锁成功
B尝试获得锁
B尝试获得锁失败, 进入自旋
A马上释放锁
B获得锁成功
B马上释放锁
```