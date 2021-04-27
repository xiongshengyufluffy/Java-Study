# 写一个Demo 验证volatile不能保证原子性

- 作为对比,使用AtomicInteger验证其可以保证原子性
## 思路

- 资源类
  - myData
    - int number
    - addPlusPlus 将number每次++
    - AtomicInteger atomicInteger
    - addAtomic 将atomicInteger自增1

- 主线程
  - 初始化资源类
  - 启动1~20号子线程
    - 每个子线程将number 累加1000次, 将atomicInteger 累加1000次
  - 当CPU切换回主线程时,检查当前活着的线程数是否>2(确保20个子线程全部都执行完毕的一种方式)
    - 若 > 2,表明还有子线程还没有执行完成
  - 输出最终的number, atomicInteger累加结果

- 代码
```
class MyData{
    int number = 0;
    AtomicInteger atomicInteger = new AtomicInteger();
    public void addPlusPlus(){
        number ++;
    }
    public void addAtomic(){
        atomicInteger.getAndIncrement();
    }
}

public static void atomicDemo(){
        System.out.println("原子性测试");
        MyData myData = new MyData();

        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    myData.addPlusPlus();
                    myData.addAtomic();
                }
            }, String.valueOf(i)).start();
        }

        while(Thread.activeCount() > 2) Thread.yield();
        System.out.println(Thread.currentThread().getName() + "final number val" + myData.number);
        System.out.println(Thread.currentThread().getName() + "final AtomicInteger val" + myData.atomicInteger);
    }
```
- 效果
```
原子性测试
mainfinal number val16494
mainfinal AtomicInteger val20000
```