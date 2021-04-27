# 写一个Demo,解决ABA问题
- 思路
  - 使用AtomicStampedReference,初始化为(exp1, ver1)
  - 确保线程A, B都能获得初始版本
  - A是动作比较快的线程
    - 先将exp1 -> exp2, ver1 -> ver1 + 1,输出修改成功标志, 当前值和当前版本
    - 再讲exp2 -> exp1, ver1 + 1 -> ver1 + 2,输出修改成功标志, 当前值和当前版本
  - B是动作比较慢的线程
    - 尝试讲exp1 -> exp2, ver1 -> ver1 + 1, 输出修改成功标志, 当前值和当前版本

```
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicStampedReference;

public class ABASolution {
    static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference< >(100, 1);

    public static void main(String[] args) {
        new Thread(()->{
            int exp1 = 100, exp2 = 101;
            int stamp =atomicStampedReference.getStamp(); // 获得第一次版本号的拷贝
            System.out.println(Thread.currentThread().getName() + "\t第一次版本号" + atomicStampedReference.getStamp());
            try {
                TimeUnit.SECONDS.sleep(1); // 确保线程B可以获得最初的版本号
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean f1 = atomicStampedReference.compareAndSet(exp1, exp2, stamp, stamp + 1);
            System.out.println(Thread.currentThread().getName() + "第一次修改将" + exp1 + "-->" + exp2 + "是否成功" + f1 +
                    " 此时的当前值为: " + atomicStampedReference.getReference() + "此时的当前版本号为" + atomicStampedReference.getStamp());
            boolean f2 = atomicStampedReference.compareAndSet(exp2, exp1, stamp + 1, stamp + 2);
            System.out.println(Thread.currentThread().getName() + "第二次修改将" + exp2 + "-->" + exp1 + "是否成功" + f2 +
                    " 此时的当前值为: " + atomicStampedReference.getReference() + "此时的当前版本号为" + atomicStampedReference.getStamp());
        }, "A").start();

        new Thread(()->{
           int exp1 = 100, exp2 = 101;
           int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t第一次版本号" + atomicStampedReference.getStamp());
            try {
                TimeUnit.SECONDS.sleep(4); // 确保线程A可以完成ABA的操作
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean f1 = atomicStampedReference.compareAndSet(exp1, exp2, stamp, stamp + 1);
            System.out.println(Thread.currentThread().getName() + "第一次修改将" + exp1 + "-->" + exp2 + "是否成功" + f1 +
                    " 此时的当前值为: " + atomicStampedReference.getReference() + "此时的当前版本号为" + atomicStampedReference.getStamp());
        }, "B").start();
    }
}
```

- 输出
```
A	第一次版本号1
B	第一次版本号1
A第一次修改将100-->101是否成功true 此时的当前值为: 101此时的当前版本号为2
A第二次修改将101-->100是否成功true 此时的当前值为: 100此时的当前版本号为3
B第一次修改将100-->101是否成功false 此时的当前值为: 100此时的当前版本号为3
```