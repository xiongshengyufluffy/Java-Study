# 写一个Demo 实现线程安全的单例模式

- 分析
  - 写一个单例模式的Demo
    - 构造方法中输出 调用构造方法   
    - 在main中创建1000个线程调用单例模式的getInstance()方法
  
- 代码
```
public class SingletonDemo {
    private static volatile SingletonDemo singletonDemo = null;
    private SingletonDemo(){
        System.out.println(Thread.currentThread().getName() + "进入构造方法");
    }
    public static SingletonDemo getInstance(){
        if(singletonDemo == null){
            synchronized (SingletonDemo.class){
                if(singletonDemo == null){
                    singletonDemo = new SingletonDemo();
                }
            }
        }
        return singletonDemo;
    }

    public static void main(String[] args) {
        for (int i = 0; i < 1000; i++) {
            new Thread(()->{
                SingletonDemo.getInstance();
            }, String.valueOf(i)).start();
        }
    }
}

```

- 输出
```
0进入构造方法
```