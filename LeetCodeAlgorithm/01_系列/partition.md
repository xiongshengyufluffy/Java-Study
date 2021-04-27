# partition

- 模板
```
def partition(nums, lo, hi):
    idx = random.randint(lo, hi)
    pivot = nums[idx]
    nums[idx], nums[lo] = nums[lo], nums[idx]
    j = lo
    # 注意这里的范围是lo,hi+1!!(而非lo,hi)
    for i in range(lo, hi + 1):
        if nums[i] < pivot:
            j += 1
            nums[i], nums[j] = nums[j], nums[i]
    nums[j], nums[lo] = nums[lo], nums[j]
    return j
```

## 1122. 数组的相对排序
- 参考题解

- 已看链接
https://leetcode-cn.com/problems/relative-sort-array/solution/jie-zhu-kuai-pai-fen-qu-han-shu-de-si-xiang-de-jie/

- 未看链接
https://leetcode-cn.com/problems/relative-sort-array/solution/ming-que-bi-jiao-fang-shi-hou-xiang-zen-yao-pai-ji/

- 思路
- j用来遍历arr1中**未按照arr2调整顺序部分**(即arr1[k]~arr1[length1-1])的各个元素,i用来遍历arr2中的各个元素
- k的约定:用来标志arr1中**已按照arr2调整顺序**的元素位置
  - 在<b>arr1[0]~arr1[k-1]</b>中的元素已经按照arr2[0]~arr2[idx]的大小顺序调整完毕
  - 而<b>arr1[k]~arr1[length1-1]</b>中的元素尚未调整顺序
- 对于arr2[i]来说
  - 用j向后遍历arr1[k]~arr1[length1-1],一旦发现arr1[j] == arr2[k]
    - 交换arr1[k],arr1[j]
    - 此时arr1[0]~arr1[k]中的元素已经按照arr2[0]~arr2[i]的大小顺序排序完毕
      - k自增一个单位
      - --> 此时arr1[0]~arr1[k-1]中的元素已经按照arr2[0]~arr2[i]的大小顺序调整完毕,k的约定再次得到满足
  
- Python版本
```
class Solution:
    def relativeSortArray(self, arr1: List[int], arr2: List[int]) -> List[int]:
        length1, length2 = len(arr1), len(arr2)
        k = 0 # arr1[0,k-1]中的元素是已经按照arr2[0,j-1]的元素的位置排好序的
        for j in range(length2):
            for i in range(k, length1):
                if arr2[j] == arr1[i]:
                    self.swap(arr1, i, k)
                    k += 1
        # 此时arr1[0]~a[k-1]中的元素已经有序
        return arr1[:k] + sorted(arr1[k:])
    
    def swap(self, arr, idx1, idx2):
        arr[idx1], arr[idx2] = arr[idx2], arr[idx1]

```
## 1365. 有多少小于当前数字的数字
https://leetcode-cn.com/problems/how-many-numbers-are-smaller-than-the-current-number/

- 非暴力思路
  - 可以使用partition,参考官方题解
  - (1)对于nums,先生成一个二维数组data
    - data[i][0], data[i][1] = nums[i], i
```
nums = [8,1,2,2,3]
data = [[8, 0], [1, 1], [2, 2], [2, 3], [3, 4]]
```
  - (2)对data[i][0]升序排序
```
data = [[1, 1], [2, 2], [2, 3], [3, 4], [8, 0]]
# data[i][0] -- 升序排序后的元素,i表示的是元素排序后的大小顺序
#            -- 第0大的元素,第1大的元素,第i大的元素,...,最大的元素
# data[i][1] -- 升序排序后的元素在原nums中的索引
#            -- 第0大的元素在原nums中的索引,第1大的元素在原nums中的索引,...,最大的元素在nums中的索引
```
  - (3)判断
    - ret为返回结果,其中ret[i]表示原数组第i个元素存在的<该元素的个数
    - 初始化prev=0,表示第0大的元素前面有prev个元素<该元素
    - i 遍历升序排序后的元素
      - 对于第i大的元素data[i][0]
        - 如果data[i][0] != data[i-1][0]:
          - prev = i 
          - 只在data[i][0]严格>data[i-1][0]更新prev
            - 例如,对于[1,[2],2]
              - 无论是索引为1的[2]还是索引为2的[2],都只存在1个元素<[2]
        - 第i大的元素在原数组中的索引为data[i][1]
          - ret[data[i][1]] = prev

## 973.最接近原点的K个点
```
import random 
class Solution:
    def kClosest(self, points: List[List[int]], K: int) -> List[List[int]]:
        length = len(points)
        distance = [point[0] ** 2 + point[1] ** 2 for point in points]
        # 对索引进行排序
        indexs = [i for i in range(length)]
        # 【partition的作用】
        # [lo, hi]是索引的范围
        # 从[lo, hi]中随机选择一个元素作为pivot,返回索引j
        # nums[lo, j]<pivot
        # pivot<=nums[j+1, hi]
        # 以下写法错误
        # for i in range(K):
        #     self.partition(distance, 0, i, indexs)
        # ----
        # 该写法同样错误
        # 以上两种写法的错误原因在于,本题中partition中初始轴点位置的选取是[lo,hi]之间随机选择的
        # for i in range(length):
        #     self.partition(distance, 0, i, indexs)
        # 或者
        # 以下写法错误 
        # self.quick(distance, 0, K-1, indexs) 该写法也是错误的 --> 实现的是对distance中[0]~[K-1]号元素进行排序
        # ----
        # 该写法可行
        self.quick(distance, 0, length-1, indexs)
        return [points[index] for index in indexs[:K]]
        
    
    def quick(self, nums, lo, hi, indexs):
        if lo >= hi:
            return 
        idx = self.partition(nums, lo, hi, indexs)
        self.quick(nums, lo, idx-1, indexs)
        self.quick(nums, idx+1, hi, indexs)

    
    def partition(self, nums, lo, hi, indexs):
        idx = random.randint(lo, hi)
        indexs[idx], indexs[lo] = indexs[lo], indexs[idx]
        pivot = nums[indexs[lo]]
        j = lo
        for i in range(lo, hi+1):
            if nums[indexs[i]] < pivot:
                j += 1
                indexs[i], indexs[j] = indexs[j], indexs[i]
        indexs[j], indexs[lo] = indexs[lo], indexs[j]
        return j
```

## 215.数组中的第K个最大的元素
```
import random
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        lo, hi = 0, len(nums) - 1
        target_pos = len(nums) - k
        while True:
            idx = self.partition(nums, lo, hi)
            # 假设数组不重复,在partition完成后
            # nums[lo, idx-1]中的元素都<nums[idx]
            # nums[idx+1, hi]中的元素都>nums[idx]
            if idx < target_pos:
                lo = idx + 1
                # 下一个需要搜索的索引范围在[idx+1,hi]中,因为此时nums[idx+1, hi]中的元素都>nums[idx]
            if idx > target_pos:
                hi = idx - 1
                # 下一个需要搜索的索引范围在[lo,idx-1]中,因为此时nums[lo, idx-1]中的元素都<nums[idx]
            if idx == target_pos:
                break
        return nums[idx]
    
    def partition(self, nums, lo, hi):
        idx = random.randint(lo, hi)
        nums[idx], nums[lo] = nums[lo], nums[idx] # swap pivot value and nums[lo]
        pivot, j = nums[lo], lo # mark pivot value and position
        
        for i in range(lo+1, hi+1):
            if nums[i] < pivot:
                j += 1
                nums[i], nums[j] = nums[j], nums[i]
        nums[lo], nums[j] = nums[j], nums[lo] 
        return j
```

## 75.颜色分类
https://leetcode-cn.com/problems/sort-colors/solution/kuai-su-pai-xu-partition-guo-cheng-she-ji-xun-huan/

- 思路
  - 借助partition,使得
    - all in [0, zero) == 0
    - all in [zero, i) == 1
    - all in [two, len-1] == 2
  - 循环终止条件: i == two
    - 当i==two时,三个区间分别为:[0, zero], [zero, two), [two, len-1]
    - 可以覆盖到nums的所有区间,所以循环终止条件为i == two
    - 循环继续条件则为i < two
  - 初始化
    - 区间[0,zero)长度为0,zero初始化为0,此时nums[zero]可以取值,先swap(nums, i, zero),再增加zero
    - 区间[two,len-1]长度为0,two初始化为len,此时nums[two]无法取值,先减少two,再swap(nums, i, two)
    - 区间[zero,i)长度为0,**因此**i初始为0,因为区间```[zero,i)==[zero,i-1]```,当zero初始化为0时,区间```[zero,i-1]==[0,-1]```,取到的为空区间
  - 区间构成:
    - 只有在nums[i] == 2时不用移动i,推测是因为2构成的区间和i无关
  - 或者使得
    - all in [0, zero] == 0
    - all in (zero, i) == 1
    - all in (two, len-1] == 2
  - 循环终止条件: i == two + 1
    - 当i==two时,三个区间分别为:[0, zero],(zero, two), (two, len-1]
    - 此时nums中元素nums[two]尚未被覆盖,因此循环终止条件为i == two + 1
    - 循环继续条件则为i <= two
  - 初始化
    - 区间[0,zero]长度为空,zero初始化为-1,此时nums[zero]无法取值,应先增加zero,再swap(nums, i, zero)
    - 区间(two,len-1]长度为空,two初始化为len-1,此时nums[two]可以取值,应该先swap(nums, i, two),再减少two
    - 区间(zero, i)(或者可以这么理解为[zero+1,i-1]),需要做到长度为0,**因此**i初始为0,这样```[zero+1, i-1]==[0,-1],取到的元素为空```