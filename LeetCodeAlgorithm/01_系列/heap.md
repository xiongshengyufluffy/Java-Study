# 堆
## 结构
- 完全二叉树
  - 小顶堆: $root < leftChild \quad \&\& \quad root < rightChild$
  - 大顶堆: $root > leftChild \quad \&\& \quad root > rightChild$
## 功能
- (1) 插入一个数
- (2) 求集合当中的最小值
- (3) 删除最小值
- (4) 删除任意一个元素
- (5) 修改任意一个元素

## 存储
- $idx(leftChild) = 2 * idx(x)$
- $idx(rightChild) = 2 * idx(x) + 1$
<img src="photos\heap\1.jpg">

## 基本操作
### down(x)
- 假设是小顶堆
  - 在修改堆中的某个节点$x$之后(将原来的$x_{small}$替换为新的$x_{big}$),如果此时在$x,leftChild,rightChild$中,$x$不再是三个节点中的最小者.
  - 此时$x$和$min(leftChild,rightChild)$交换位置来使得堆重新满足$x < leftChild \quad \&\& \quad x < rightChild$
<img src="photos\heap\2.jpg">

### up(x)
- 假设是小顶堆
  - 在修改堆中的某个节点$Child$之后(将原来的$Child_{big}$替换为新的$Child_{small}$),,如果此时在$x,leftChild,rightChild$中,$Child$比当前的$x$还小.
  - 此时$x$和$Child$交换位置来使得堆重新满足$x < leftChild \quad \&\& \quad x < rightChild$

<img src="photos\heap\3.jpg">

## 实现堆的功能
### (1)插入一个数
- 1.1 由堆的存储结构有,当想往堆中插入一个数时,实际上是往数组最后插入一个数字
  - ```Size```表示当前堆的大小,```heap```表示堆 
  - ```heap[++Size] = x```

<img src="photos\heap\4.png">

- 1.2 在插入完成之后,将该节点不断上滤
  - ```up(Size)```

### (2)求集合中的最小值
```heap[1]```

### (3)删除最小值
- 3.1 将整个堆的最后一个元素覆盖堆顶
- 3.2 删除堆的最后一个元素(即将堆尾指针前移)
- 3.3 将交换过来的首元素下滤
- 为什么采取<font coilor=red>**交换头尾元素,删除尾元素的方式**</font>
  - 因为堆的存储是使用一维数组,对于一维数组来说,删除第一个元素意味着要搬动整个数组,删除代价过高
  - 而删除一维数组尾部元素直接可以将尾指针前移一个单位来实现
  - 所以一个比较高效的删除堆的最小值的方式是<font coilor=red>**交换头尾元素,删除尾元素的方式**</font>
``` 
heap[1] = heap[Size];
Size--;
down(1);
```

### (4)删除任意一个元素
- 和删除第1个元素的做法类似
- 假设想删除第k个元素

```
heap[k] = heap[Size];
Size--;
```
- 替换之后,新的第k个元素可能上滤,也可能下滤
```
// 反正每次只能走其中一个分支
down(k);
up(k);
```

### (5)修改堆中第k个元素为x
```
heap[k] = x;
down(k);
up(k);
```

## 实现堆的基本功能

## 建堆
### 将一个数组中的元素构建为一个堆-$O(nlogn)$
- 如果每次将数组中的一个元素插入堆中,扫描完一遍数组的时间复杂度为$O(n)$
  - 插入完一个元素后,该元素随即进行上滤$down$,时间复杂度为$O(logn)$
  - 采用这种方式建堆,时间复杂度为$O(nlogn)$

### 将一个数组中的元素构建为一个堆-$O(n)$
- 从数组中第$\frac{n}{2}$个元素一直$down$到第$1$个元素
- 第$\frac{n}{2} + 1$至第$n$个元素自然构成堆的叶子节点

<img src="photos\heap\5.png">

<img src="photos\heap\6.png">

- 为什么时间复杂度为$O(n)$?

<img src="photos\heap\7.png">


<img src="photos\heap\4.jpg">

## down
```
static void down(int n){
        int x = n; // x用于得到n, leftChild, rightChild中值最小的节点
        if(n * 2 <= size && t[x] > t[n * 2])x = n * 2; 
        // 如果当前点有leftChild 且 节点值 > leftChild节点值,更新x
        if(n * 2 + 1 <= size && t[x] > t[n * 2 + 1])x = n * 2 + 1;
        // 如果当前点有rightChild 且 节点值 > rightChild节点值,更新x
        if(n != x){
            swap(n, x);
            down(x);
        }
    }
```

