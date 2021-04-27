# GC系列

## 283. 移动零
- 其实提供的思路就是partition
```
给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。

示例:

输入: [0,1,0,3,12]
输出: [1,3,12,0,0]

链接：https://leetcode-cn.com/problems/move-zeroes
```
- 参考题解链接
  - https://leetcode-cn.com/problems/move-zeroes/solution/kan-dao-zhe-ti-rang-wo-xiang-qi-liao-gczhong-de-bi/
- 思路
```
特判： 过滤掉空数组；
定义一个指针p表示已经遇到的非0元素的个数，初始值为0；
遍历：
    遇到非0的元素就放在p的位置上，然后p++；
```
- 实现
  - 用i遍历数组nums,初始化p=0
  - 当nums中所有元素都为非0元素时,p和i指向位置相同
    - 当nums[i]非0时,交换nums[i]和nums[p]
      - 完成交换后p++
        - 实现的效果是在完成元素交换后,p指向下一个元素
        - 如果下一个元素nums[i+1]也非零,则i,p同步增加
    - 当nums[i+1]为0,只有i会自增,而p的位置不发生改变
- 不变性
  - [0,p-1]中的元素始终非0

- GC的思路和partiton类似
```
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        if not nums:
            return None
        lo, hi = 0, len(nums) - 1
        j = lo # mark pivot value and position
        # [0, j-1] < pivot
        # [0, j-1] != 0
        for i in range(lo, hi+1):
            # if nums[i] < pivot
            if nums[i] != 0:
                # j += 1 partiton中j是先自增
                nums[i], nums[j] = nums[j], nums[i]
                j += 1 
        # partiton中最后还有一步交换
        # nums[lo], nums[j] = nums[j], nums[lo] 
```
## 剑指 Offer 21. 调整数组顺序使奇数位于偶数前面
- 这道题用selectionSort和insertionSort的方式都能做
- 类似题922. 按奇偶排序数组 II(貌似不能用GC来做)