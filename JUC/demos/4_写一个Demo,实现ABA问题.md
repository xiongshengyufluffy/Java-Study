# 写一个Demo,实现ABA问题
- 思路
  - 线程t1是速度较快的线程
    - 将100 -> 101,输出修改成功标志
    - 将101 -> 100,输出修改成功标志
  - 线程t2是速度较慢的线程
    - 先睡眠10s
    - 将100 -> 101,输出修改成功标志

- 代码
```
import java.util.concurrent.atomic.AtomicReference;

public class ABA {
    public static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
    public static void main(String[] args) {
        new Thread(()->{
            int a = 100, b = 101;
            System.out.println(Thread.currentThread().getName() + " a : " + a + "-->" + "b : " + b + " -- " +atomicReference.compareAndSet(a, b));
            System.out.println(Thread.currentThread().getName() + " b : " + b + "-->" + "a : " + a + " -- " +atomicReference.compareAndSet(b, a));
        }, "t1").start();

        new Thread(()->{
            int a = 100, c = 102;
            try {
                TimeUnit.SECONDS.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " a : " + a + "-->" + "c : " + c + " -- " + atomicReference.compareAndSet(a, c));
            System.out.println(Thread.currentThread().getName() + " 当前值: " + atomicReference.get());
        }, "t2").start();

    }
}
```

- 输出
```
t1 a : 100-->b : 101 -- true
t1 b : 101-->a : 100 -- true
t2 a : 100-->c : 102 -- true
t2 当前值: 102
``` 