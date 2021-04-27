# SelectionSort
- 思路
  - Python版本
```
class Solution:
    def selectionSort(self, nums):
        length = len(nums)
        # 循环不变量:[0,i)有序,且该区间里所有元素就是最终排定的样子
        for i in range(length):
            # 选择区间[i,length-1]中最小的元素的索引,交换到下标i
            minIndex = i
            for j in range(i+1, length):
                if nums[j] < nums[minIndex]:
                    minIndex = j
            self.swap(nums, i, minIndex)

    def swap(self, nums, idx1, idx2):
        nums[idx1], nums[idx2] = nums[idx2], nums[idx1]
```

## 922. 按奇偶排序数组 II

```
给定一个非负整数数组 A， A 中一半整数是奇数，一半整数是偶数。

对数组进行排序，以便当 A[i] 为奇数时，i 也是奇数；当 A[i] 为偶数时， i 也是偶数。

你可以返回任何满足上述条件的数组作为答案。

示例：

输入：[4,2,5,7]
输出：[4,5,2,7]
解释：[4,7,2,5]，[2,5,4,7]，[2,7,4,5] 也会被接受。
```
- 思路
  - 用i遍历A,依次比较数组A[i]与i的奇偶性,若相同则继续
  - 否则,此时A[i]与i的奇偶性不同: 
    - ~~forSwapIndex = i~~(大可不必这么写,只是为了风格和selectionSort统一)
    - 用j遍历A[i+1]~A[length-1]
      - 一旦发现A[j]和**i**的奇偶性相同
        - ~~forSwapIndex = j~~
        - 退出遍历
    - 交换i,j坐标 swap(A, i, j) ~~swap(A, i, forSwapIndex)~~

```
class Solution:
    def sortArrayByParityII(self, A: List[int]) -> List[int]:
        length = len(A)
        for i in range(length):
            if ((i % 2) != (A[i] % 2)):
                for j in range(i+1,length):
                    if ((i % 2) == (A[j] % 2)):
                        self.swap(A, i, j)
                        break
        return A
    
    def swap(self, nums, idx1, idx2):
        nums[idx1], nums[idx2] = nums[idx2], nums[idx1]
```