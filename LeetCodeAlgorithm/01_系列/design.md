# 381. O(1) 时间插入、删除和获取随机元素 - 允许重复
## ps: 这道题应该也可以归类于Hash

设计一个支持在平均 时间复杂度 O(1) 下， 执行以下操作的数据结构。

注意: 允许出现重复元素。

insert(val)：向集合中插入元素 val。

remove(val)：当 val 存在时，从集合中移除一个 val。

getRandom：从现有集合中随机获取一个元素。每个元素被返回的概率应该与其在集合中的数量呈线性相关。

```
// 初始化一个空的集合。
RandomizedCollection collection = new RandomizedCollection();

// 向集合中插入 1 。返回 true 表示集合不包含 1 。
collection.insert(1);

// 向集合中插入另一个 1 。返回 false 表示集合包含 1 。集合现在包含 [1,1] 。
collection.insert(1);

// 向集合中插入 2 ，返回 true 。集合现在包含 [1,1,2] 。
collection.insert(2);

// getRandom 应当有 2/3 的概率返回 1 ，1/3 的概率返回 2 。
collection.getRandom();

// 从集合中删除 1 ，返回 true 。集合现在包含 [1,2] 。
collection.remove(1);

// getRandom 应有相同概率返回 1 和 2 。
collection.getRandom();
```

- 过程分析
  - insert(1)
    - nums: [1]
    - idx: {1: {0}}
  - insert(1)
    - nums: [1,1]
    - idx: {1: {0,1}}
  - insert(2)
    - nums: [1,1,2] 
    - idx: {1: {0, 1}, 2: {2}}
  - getRandom()
    - 使用math.random,生成一个[0,1) 之间的随机数 _random
    - 得到数字的索引 index = (int) (_random * nums.length())
    - 返回nums[index]
  - remove(1)
    - 待删除元素:```val==1```
    - 使用迭代器,从idx中得到val对应的~~第一个~~某一个位置i idx: {1: {<font color=#FF0000> 0</font>, 1}, 2: {2}},```i==0```
      - **注**: 这里并非要求一定是第一个
    - 得到nums中的最后一个元素lastNum [1,1,<font color=#FF0000>2</font>]
    - nums:
      - 将nums中位置i对应的元素设置为lastNum,```lastNum==2```
        - [<font color=#FF0000>2</font>,1,<font color=#FF0000>2</font>]
    - idx
      - 删除val对应的~~第一个~~某一个位置i
        - idx.get(val).remove(i)
          - idx.get(val)返回的是val对应的索引无序集合setObj(```setObj == {0, 1}```)
          - setObj.remove(i)是移除集合中的指定元素
        -  idx: {1: {1}, 2: {2}}
      - 删除lastNum对应的位置  
        - lastNum是nums中最后一个元素,对应的索引为nums.length() - 1
        - idx.get(lastNum).remove(nums.length() - 1)
        - idx: {1: {1}, 2:{}}
      - 在待删除元素出现的~~第一个~~某一个位置i不为nums中最后一个元素的情况下,在idx中补充lastNum的出现位置
        - lastNum新出现在位置i,即idx.get(lastNum).add(i)
        - idx: {1: {1}, 2:{0}}
      - 如果待删除元素val在idx中已经没有存在的索引(即key==val,value=={}),此时在idx中移除键key
    - nums:
      - 删除nums中的最后一个元素 
      - nums --> ```[2,1]```

    - ps:
      - 官方题解思路有点变扭
        - 可以先判断待删除的元素val是否就是刚刚插入的第一个元素
        - (1)如果是
            - nums == [1,2,3], idx = {1:{0}, 2:{1}, 3:{2}}
            - remove(3)之后
              - nums [1,2,3] --> [1,2]
              - idx {1:{0}, 2:{1}, 3:{2}} --> {1:{0}, 2:{1}}
        - (2)否则
          - 在nums中用最后一个元素与待删除元素出现的~~第一个~~**某一个位置**交换 (并不是第一个位置,某一个位置就行)
          - 在idx中保持各个元素坐标的正确
          - 2.a
            - nums == [1,$3_a$,2,2,$3_b$], idx = {1:{0}, 2:{2,3}, 3:{1, 4}}
            - remove(3)
              - (2.1) 对nums处理
                - [1,$3_a$,2,2,$3_b$]
                - [1,$3_b$,2,2,$3_b$]
                - [1,$3_b$,2,2]
              - (2.2) 对idx处理
                - idx = {1:{0}, 2:{2,3}, 3:{4}}
          - 2.b  
           - nums == [1,$3_a$,2,4,2,$3_b$,4], idx = {1:{0}, 2:{2,4}, 3:{1, 5}, 4:{3,6}}
            - remove(3)
              - (2.1) 对nums处理
                - [1,$3_a$,2,4,2,$3_b$,4]
                - [1,4,2,4,2,$3_b$,4]
                - [1,4,2,4,2,$3_b$]
              - (2.2) 对idx处理
                - idx = {1:{0}, 2:{2,4}, 3:{5},4:{1,3}}

https://www.zhihu.com/question/28414001

- 思路
  - 官方题解:Hash

- 代码晚点补充(见提交记录)

# 两部分系列
## 设计的数据结构由两个同样性质的数据结构组成
### 295.数据流中的中位数
```
class MedianFinder {
    private int counts;
    private PriorityQueue<Integer> maxHeap;
    private PriorityQueue<Integer> minHeap; // 默认为小顶堆
    /** initialize your data structure here. */
    public MedianFinder() {
        counts = 0;
        maxHeap = new PriorityQueue<>(new Comparator<Integer>() {
            @Override
            public int compare(Integer t1, Integer t2) {
                return t2 - t1;
            }
        });
        minHeap = new PriorityQueue<>();
    }

    public void addNum(int num) {
        if (counts++ == 0) {
            maxHeap.add(num);
            return;
        }
        if (counts++ == 1) {
            if (maxHeap.peek() <= num){
                minHeap.add(num);
            }else{
                int peek = maxHeap.poll();
                maxHeap.add(num);
                minHeap.add(peek);
            }
            return;
        }
        if(maxHeap.size() == minHeap.size()){
            int rightPeek = minHeap.peek(); //右侧小顶堆最小值
            if(num > rightPeek){
                rightPeek = minHeap.poll();
                maxHeap.add(rightPeek);
                minHeap.add(num);
            }else{
                maxHeap.add(num);
            }
            counts ++;
            return;
        }
        if(maxHeap.size() == minHeap.size() + 1){
            int leftPeek = maxHeap.peek();
            if(num < leftPeek){
                leftPeek = maxHeap.poll();
                minHeap.add(leftPeek);
                maxHeap.add(num);
            }else{
                minHeap.add(num);
            }
            counts ++;
            return;
        }
    }

    public double findMedian() {
//        if (counts == 1) return (double) maxHeap.peek();
//        if (counts % 2 == 1) return (double) maxHeap.peek();
        if (counts % 2 == 0) return (maxHeap.peek() + minHeap.peek()) / 2.0;
        return (double) maxHeap.peek(); // 为奇数
    }
}
```

### 1670.设计前中后队列
```
// java双端队列
// 队列元素个数为奇数时 左队列元素数 == 右队列元素数 + 1
// pushMiddle(e)
// (1.1)左队列队尾出队 --> 右队队首
// (1.2)左队队尾add(e)
// popMiddle()
// (2.1)左队列队尾出队
// 队列元素个数为偶数时 左队列元素数 == 右队列元素数
// pushMiddle(e)
// (1.1)左侧队尾add(e)
// popMiddle()
// (2.1)左队列队尾出队
// (2.2)右队列队首出队 --> 左队队尾
class FrontMiddleBackQueue {
    private int count;
    private Deque<Integer> leftDeque;
    private Deque<Integer> rightDeque;

    public FrontMiddleBackQueue() {
        count = 0;
        leftDeque = new LinkedList<>();
        rightDeque = new LinkedList<>();
    }
    // [1,2,3][4,5] --> [6,1,2][3,4,5]
    // [6,1,2][3,4,5] --> [7,6,1,2][3,4,5]
    public void pushFront(int val) {
        //此时队列长度为奇数
        if(count % 2 == 1){
            leftDeque.addFirst(val);
            rightDeque.addFirst(leftDeque.removeLast());
        }//此时队列长度为偶数
        else{
            leftDeque.addFirst(val);
        }
        count ++;
    }
    // [1,2,3][4,5] --> [1,2,6][3,4,5]
    // [1,2,6][3,4,5] --> [1,2,6,7][3,4,5]
    public void pushMiddle(int val) {
        //此时队列长度为奇数
        if(count % 2 == 1){
            rightDeque.addFirst(leftDeque.removeLast());
            leftDeque.addLast(val);
        }//此时队列长度为偶数
        else{
            leftDeque.addLast(val);
        }
        count ++;
    }
    // [1,2,3][4,5] --> [1,2,3][4,5,6]
    // [1,2,3][4,5,6] --> [1,2,3,4][5,6,7]
    public void pushBack(int val) {
        //此时队列长度为奇数
        if(count % 2 == 1){
            rightDeque.addLast(val);
        }//此时队列长度为偶数
        else{
            rightDeque.addLast(val);
            leftDeque.addLast(rightDeque.removeFirst());
        }
        count ++;

    }
    
    public int popFront() {
        if (leftDeque.size() == 0) return -1;
        int ret;
        if(count % 2 == 1){
            ret = leftDeque.removeFirst();
        }//此时队列长度为偶数
        else{ // [1, 2] [3, 4] --> [2][3,4]
            ret = leftDeque.removeFirst();
            leftDeque.addLast(rightDeque.removeFirst());
        }
        count --;
        return ret;
    }
    
    public int popMiddle() {
        if (leftDeque.size() == 0) return -1;
        int ret;
        // [1,2][3,4] --> [1,3][4]
        // [1,2][3] --> [1][3]
        if(count % 2 == 1){
            ret = leftDeque.removeLast();
        }//此时队列长度为偶数
        else{ 
            ret = leftDeque.removeLast();
            leftDeque.addLast(rightDeque.removeFirst());
        }
        count --;
        return ret;

    }
    
    public int popBack() {
        if (leftDeque.size() == 0) return -1;
        
        int ret;
        // [1][] --> []
        // [1,2][3,4] --> [1,2][3]
        // [1,2][3] --> [1][2]
        if(count % 2 == 1){
            if(rightDeque.size() == 0) ret = leftDeque.removeLast();
            else{
                ret = rightDeque.removeLast();
                rightDeque.addFirst(leftDeque.removeLast());
            }
        }//此时队列长度为偶数
        else{ 
            ret = rightDeque.removeLast();
        }
        count --;
        return ret;

    }
}

```

# 705.设计哈希集合
- 设计哈希表有两种实现方式(见Acwing740)
  - 开放寻址法 和 拉链法
    - 若无删除操作 --> 使用开放寻址法编写较为便捷
      - 否则使用拉链法