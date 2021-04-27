# 写一个Demo 验证volatile可以保证数据的可见性
## 思路
- 资源类
    - myData
        - int number 和 setTo60方法

- 主线程
    - 初始化资源类
    - 启动线程AAA
- 线程A
    - 保证主线程随后可以获得资源类的num
    - 使用setTo60, 将num 由 0 -> 60
- 主线程
    - 启动A线程后,一直循环,当num不为0是退出

```
class MyData{
    int number = 0;
    public void setTo60(){
        this.number = 60;
    }
}

  public static void volatileVisibilityDemo(){
        System.out.println("可见性测试");
        MyData myData = new MyData();
        // 启动一个线程操作共享数据
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t come in");
            // 让主线程抢占到时间片
            try {
                TimeUnit.SECONDS.sleep(3);
                myData.setTo60();
                System.out.println(Thread.currentThread().getName() + "update number value" + myData.number);

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t come out");
        }, "AAA").start();

        while(myData.number == 0){
            // main线程中获得myData.number 的值为0时,一直自旋
        }
        System.out.println(Thread.currentThread().getName() + "main get number val: " + myData.number);
    }
```
- 当number未使用volatile修饰,会一直在死循环中
```
可见性测试
AAA	 come in
AAAupdate number value60
AAA	 come out
```
- 当number使用volatile修饰
```
可见性测试
AAA	 come in
AAAupdate number value60
AAA	 come out
mainmain get number val: 60
```