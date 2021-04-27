# 扩容流程
![arrayList的add过程](https://xiongshengyu-1256692535.cos.ap-beijing.myqcloud.com/photos/arrayList的add过程.png)

- 图中问题的解答
 - 对于这种分配到length为Integer.MAX_VALUE,但是从理论上来说,原本存数组length 和 8 byte head words的位置就没有了的情况的处理方式, 和VM的实现有关
 - 就是说可以给你分配Integer.MAX_VALUE的长度,但是可能会抛出OOM异常

# 扩容效率的比较
- arrayList 和 LinkedList

```
public class Test1 {
    public static long arr(int time){
        List<Integer> li = new ArrayList<>();
        long startTime = System.currentTimeMillis(); //获取开始时间
        for (int i = 0; i < time; i++) {
            li.add(i);
        }

        long endTime = System.currentTimeMillis(); //获取结束时间
        System.out.println("程序运行时间： " + (endTime - startTime) + "ms");
        return endTime - startTime;
    }
    public static long list(int time){
        List<Integer> li = new LinkedList<>();
        long startTime = System.currentTimeMillis(); //获取开始时间
        for (int i = 0; i < time; i++) {
            li.add(i);
        }

        long endTime = System.currentTimeMillis(); //获取结束时间
        System.out.println("程序运行时间： " + (endTime - startTime) + "ms");
        return endTime - startTime;
    }

    public static void main(String[] args) {
        int[] times = new int[] {1000, 10000, 100000, 1000000, 10000000};
        for (int i = 0; i < times.length; i++) {
            System.out.println("插入" + times[i] + "次");
            long t1 = arr(times[i]);
            long t2 = list(times[i]);
            System.out.println("arraylist 扩容时间是否 < linkedlist 扩容时间" + (t1 < t2));
        }
    }
}
```
- 本地运行结果
```
插入1000次
程序运行时间： 0ms
程序运行时间： 0ms
arraylist 扩容时间是否 < linkedlist 扩容时间false
插入10000次
程序运行时间： 1ms
程序运行时间： 1ms
arraylist 扩容时间是否 < linkedlist 扩容时间false
插入100000次
程序运行时间： 4ms
程序运行时间： 4ms
arraylist 扩容时间是否 < linkedlist 扩容时间false
插入1000000次
程序运行时间： 14ms
程序运行时间： 16ms
arraylist 扩容时间是否 < linkedlist 扩容时间true
插入10000000次
程序运行时间： 180ms
程序运行时间： 6811ms
arraylist 扩容时间是否 < linkedlist 扩容时间true
```

- 结论
    - 大概1000000次是分界点,有的时候arraylist会快一点,有的时候linkedlist会快一点
    - 10000000次对比就会很明显了

- 原因
    - array 的 add操作涉及赋值(耗时最短)以及可能涉及(扩容 + 复制)(耗时最长) , linkedlist只涉及在尾部的追加一个新建节点(耗时中等的)
    - array 的 add操作在数据量小的时候容易频繁引起(扩容 + 复制)的耗时最长的操作,因此在数据量小的时候耗时较linkedlist长,扩容效率低
    - array 的 add操作在数据量大的时候多以耗时最短的在尾部赋值为主,因此在数据量大的情况下,array的扩容效率反而高于linkedlist