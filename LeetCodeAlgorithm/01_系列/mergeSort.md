# MergeSort

- 思路
  - Python版本
```
class Solution:
    def fun(self, nums):
        length = len(nums)
        temp = [0] * length 
        self.divide(nums, 0, length-1, temp)
        return nums

    def divide(self, nums, lo, hi, temp):
        if lo == hi:
            return 0
        mid = lo + (hi - lo) // 2
        self.divide(nums, lo, mid, temp)
        self.divide(nums, mid+1, hi, temp)
        self.merge(nums, lo, mid, hi, temp)
    
    def merge(self, nums, lo, mid, hi, temp):
        # left [lo, mid], right[mid+1, hi] 
        for i in range(lo, hi+1):
            temp[i] = nums[i]
        i, j = lo, mid+1
        for k in range(lo, hi+1):
            if i > mid:
                nums[k] = temp[j]
                j += 1
            elif j > hi:
                nums[k] = temp[i]
                i += 1
            elif temp[i] <= temp[j]:
                nums[k] = temp[i]
                i += 1
            else:
                nums[k] = temp[j]
                j += 1
```

- Java版本见
  https://leetcode-cn.com/problems/sort-an-array/solution/fu-xi-ji-chu-pai-xu-suan-fa-java-by-liweiwei1419/


## 剑指 Offer 51. 数组中的逆序对

```
class Solution:
    def reversePairs(self, nums: List[int]) -> int:
        if not nums:
            return 0
        return self.divide(nums, 0, len(nums)-1, [0]*len(nums))

    def divide(self, nums, lo, hi, temp):
        if lo == hi:
            return 0
        mid = lo + (hi - lo) // 2
        left_num = self.divide(nums, lo, mid, temp)
        right_num = self.divide(nums, mid+1, hi, temp)
        if nums[mid] <= nums[mid+1]:
            return left_num + right_num
        cross_num = self.merge_count(nums, lo, mid, hi, temp)
        return left_num + right_num + cross_num
    
    def merge_count(self, nums, lo, mid, hi, temp):
        # left [lo, mid], right[mid+1, hi] 
        for i in range(lo, hi+1):
            temp[i] = nums[i]
        i, j = lo, mid+1
        res = 0
        for k in range(lo, hi+1):
            if i > mid:
                nums[k] = temp[j]
                j += 1
            elif j > hi:
                nums[k] = temp[i]
                i += 1
            elif temp[i] <= temp[j]:
                nums[k] = temp[i]
                i += 1
            else:
                nums[k] = temp[j]
                res += mid - i + 1
                j += 1
        return res
```

## 327. 区间和的个数

- 参考链接

https://leetcode-cn.com/problems/count-of-range-sum/solution/shuo-ming-yi-xia-guan-fang-gui-bing-pai-xu-by-sing/

```
class Solution:
    def countRangeSum(self, nums: List[int], lower: int, upper: int) -> int:
        if not nums:
            return 0
        # 先对nums处理成前缀和
        length = len(nums)
        for i in range(1, length):
            nums[i] += nums[i - 1]
        # print(nums)
        temp = [0] * length
        # self.result = 0
        self.lower, self.upper = lower, upper
        result = self.divide(nums, 0, length - 1, temp)
        return result

    def divide(self, nums, lo, hi, temp):
        if lo == hi:
            return 1 if self.lower <= nums[lo] and nums[lo] <= self.upper else 0

        mid = lo + (hi - lo) // 2
        left_result = self.divide(nums, lo, mid, temp)
        right_result = self.divide(nums, mid + 1, hi, temp)
        cross_result = self.merge(nums, lo, mid, hi, temp)
        return left_result + right_result + cross_result

    def merge(self, nums, lo, mid, hi, temp):
        result = 0
        # left [lo, mid], right[mid+1, hi]
        # 在这里统计下标对的数量
        for left in range(lo, mid + 1):
            # right_max(即nums[hi]) < nums[left] + self.lower --> right整体过小,可以跳过
            # right_min(即nums[mid+1]) > nums[left] + self.upper时 --> 当前的left对应的nums[left]较小,可以跳过本轮的left,continue
            if nums[hi] < nums[left] + self.lower or nums[mid+1] > nums[left] +  self.upper:
                continue
            for right in range(mid + 1, hi + 1):
                if self.lower <= nums[right] - nums[left] and nums[right] - nums[left] <= self.upper:
                    result += 1
                if nums[right] - nums[left] > self.upper:
                    break
        
        for i in range(lo, hi + 1):
            temp[i] = nums[i]
        i, j = lo, mid + 1
        for k in range(lo, hi + 1):
            if i > mid:
                nums[k] = temp[j]
                j += 1
            elif j > hi:
                nums[k] = temp[i]
                i += 1
            elif temp[i] <= temp[j]:
                nums[k] = temp[i]
                i += 1
            else:
                nums[k] = temp[j]
                j += 1
        return result
```

- merge的另外一种写法
```
def merge(self, nums, lo, mid, hi, temp):
        result = 0
        # left [lo, mid], right[mid+1, hi]
        # 在这里统计下标对的数量
        for left in range(lo, mid + 1):
            # right_max(即nums[hi]) < nums[left] + self.lower --> right整体过小,此时没有必要尝试更大的left --> break
            # right_min(即nums[mid+1]) > nums[left] + self.upper时 --> 当前的left对应的nums[left]较小,可以跳过本轮的left,尝试更大的left --> continue
            if nums[hi] < nums[left] + self.lower:
                break
            if nums[mid+1] > nums[left] +  self.upper:
                continue
            for right in range(mid + 1, hi + 1):
                if self.lower <= nums[right] - nums[left] and nums[right] - nums[left] <= self.upper:
                    result += 1
                if nums[right] - nums[left] > self.upper:
                    break
```

## 315. 计算右侧小于当前元素的个数

- 参考链接
https://leetcode-cn.com/problems/count-of-smaller-numbers-after-self/solution/gui-bing-pai-xu-suo-yin-shu-zu-python-dai-ma-java-/


- 思路
  - 在归并时使用索引数组indexs(即,不修改原数组,得到原数组的排序结果对应的索引值)
  - 在归并时(假设不考虑使用数组索引进行归并)
    - i指向左区间([lo, mid])中元素,j指向右区间([mid+1, hi])中元素
    - 当左区间([lo, mid])中的i号元素(```lo <= i <= mid```)被归并入辅助数组temp中时
      - 存在关系nums[i] <= nums[j]
      - 假设辅助数组temp中已经存在从右区间中[mid+1,j-1]的元素,共**j-1-(mid+1)+1,即j-1-mid**个来自右区间的元素
      - (**注意,nums[j]并未被归并入辅助数组中,所以为j-1-mid个元素,而非j-mid个元素**)
  - 在归并时(考虑使用索引数组indexs进行归并)
    - i指向左索引区间([lo, mid])中元素,j指向右索引区间([mid+1, hi])中元素
    - 当左索引区间([lo, mid])中的i号元素(```lo <= i <= mid```)被归并入辅助索引数组temp中时
      - 存在关系nums[indexs[i]] <= nums[indexs[j]] (**和上方在判断时只有这里存在区别**)
      - 假设辅助数组temp中已经存在从右索引数组区间中[mid+1,j-1]的元素,共**j-1-(mid+1)+1,即j-1-mid**个来自右区间的元素
      - (**注意,nums[j]并未被归并入辅助数组中,所以为j-1-mid个元素,而非j-mid个元素**)

```
class Solution:
    def countSmaller(self, nums: List[int]) -> List[int]:
        if not nums:
            return nums
        length = len(nums)
        self.result = [0] * length
        indexs = [i for i in range(length)]
        self.divide(nums, 0, length - 1, [0] * length, indexs)
        return self.result
    
    def divide(self, nums, lo, hi, temp, indexs):
        if lo == hi:
            return 
        mid = lo + (hi - lo) // 2
        self.divide(nums, lo, mid, temp, indexs)
        self.divide(nums, mid+1, hi, temp, indexs)
        self.merge(nums, lo, mid, hi, temp, indexs)

    
    def merge(self, nums, lo, mid, hi, temp, indexs):
        i, j = lo, mid + 1
        # [lo, mid] [mid+1, hi]
        for k in range(lo, hi + 1):
            temp[k] = indexs[k]
        for k in range(lo, hi + 1):
            if i > mid:
                indexs[k] = temp[j]
                j += 1
            elif j > hi:
                indexs[k] = temp[i]
                i += 1
                # 左区间被并入时
                # 考虑累加辅助数组中已存的来自右区间的元素个数
                self.result[indexs[k]] += hi - mid
                
            elif nums[temp[i]] <= nums[temp[j]]:
                indexs[k] = temp[i]
                i += 1
                # 左区间被并入
                self.result[indexs[k]] += j - mid - 1
            else:
                indexs[k] = temp[j]
                j += 1
```
- 以上几道题既可以用mergeSort的方式做,又可以用树状数组做
  
~~## 1365. 有多少小于当前数字的数字~~

- 不方便用mergeSort的方式做,可以用树状数组做


## 493. 翻转对

```
class Solution:
    def reversePairs(self, nums: List[int]) -> int:
        if not nums:
            return 0
        return self.divide(nums, 0, len(nums)-1, [0]*len(nums))

    def divide(self, nums, lo, hi, temp):
        if lo == hi:
            return 0
        mid = lo + (hi - lo) // 2
        left_num = self.divide(nums, lo, mid, temp)
        right_num = self.divide(nums, mid+1, hi, temp)
        # if nums[mid] <= 2 * nums[mid+1]:
        #     return left_num + right_num
        cross_num = self.merge_count(nums, lo, mid, hi, temp)
        return left_num + right_num + cross_num
    
    def merge_count(self, nums, lo, mid, hi, temp):
        # left [lo, mid], right[mid+1, hi] 
        for i in range(lo, hi+1):
            temp[i] = nums[i]
        i, j = lo, mid+1
        # if lo == 0 and mid == 2:
        #     print(temp)
        res = 0
        right_begin = mid+1 # 右侧元素的最小者
        left_begin = lo # 左侧元素的最小者
        while left_begin < mid + 1 and right_begin < hi + 1:
            if temp[left_begin] > 2 * temp[right_begin]:
                # print(temp, "左区间", "[", lo, ",", mid, "] 右区间 [", mid+1, ",", hi, "]")
                # 此时[left_begin,mid]中的元素都满足反转对条件
                res += mid - left_begin + 1
                right_begin += 1 # 可以尝试更大的右侧元素
            else:
                left_begin += 1 # 尝试更大的左侧元素
                
        for k in range(lo, hi+1):
            if i > mid:
                nums[k] = temp[j]
                j += 1
            elif j > hi:
                nums[k] = temp[i]
                i += 1
            elif temp[i] <= temp[j]:
                nums[k] = temp[i]
                i += 1
            else:
                nums[k] = temp[j]
                j += 1
        return res
```

## 493. 翻转对
```
class Solution:
    def reversePairs(self, nums: List[int]) -> int:
        if not nums:
            return 0
        return self.divide(nums, 0, len(nums)-1, [0]*len(nums))

    def divide(self, nums, lo, hi, temp):
        if lo == hi:
            return 0
        mid = lo + (hi - lo) // 2
        left_num = self.divide(nums, lo, mid, temp)
        right_num = self.divide(nums, mid+1, hi, temp)
        cross_num = self.merge_count(nums, lo, mid, hi, temp)
        return left_num + right_num + cross_num
    
    def merge_count(self, nums, lo, mid, hi, temp):
        # left [lo, mid], right[mid+1, hi] 
        for i in range(lo, hi+1):
            temp[i] = nums[i]
        i, j = lo, mid+1

        res = 0
        right_begin = mid+1 # 右侧元素的最小者
        left_begin = lo # 左侧元素的最小者
        while left_begin < mid + 1 and right_begin < hi + 1:
            if temp[left_begin] > 2 * temp[right_begin]:
                # print(temp, "左区间", "[", lo, ",", mid, "] 右区间 [", mid+1, ",", hi, "]")
                # 此时[left_begin,mid]中的元素都满足反转对条件
                res += mid - left_begin + 1
                right_begin += 1 # 可以尝试更大的右侧元素
            else:
                left_begin += 1 # 尝试更大的左侧元素
                
        for k in range(lo, hi+1):
            if i > mid:
                nums[k] = temp[j]
                j += 1
            elif j > hi:
                nums[k] = temp[i]
                i += 1
            elif temp[i] <= temp[j]:
                nums[k] = temp[i]
                i += 1
            else:
                nums[k] = temp[j]
                j += 1
        return res
```

## 148. 排序链表(链表版本mergeSort,参见twoPointer.md)