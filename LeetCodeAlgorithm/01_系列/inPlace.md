# 剑指 Offer 03. 数组中重复的数字

- 这道题不能用二分
- 
  该方法对于本题不适用，但是题目类似，长度为n的数组，且数字都在1~n-1之间，且一定有数字是重复的。每一遍二分的过程，统计nums元素中1~m的数量(m <=n-1, 改变m的值，遍历的是整个数组)，如果数量大于这个值，这说明重复的元素一定在1~m中。
但实际此种方法不可行，例如[1, 1, 1, 2, 4, 5, 6, 7, 8, 9]，对这个样例，则无法找到正确的答案。
- 注:
  - 如果不存在重复数,返回-1

- 参考思路
  
https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/solution/mian-shi-ti-03-shu-zu-zhong-zhong-fu-de-shu-zi-yua/

```
class Solution:
    def findRepeatNumber(self, nums: [int]) -> int:
        i = 0
        while i < len(nums):
            if nums[i] == i:
                i += 1
                continue
            # 起到类似于字典判断重复的作用
            if nums[nums[i]] == nums[i]: return nums[i]
            # 起到的作用是是的nums中
            # 0~i坐标对应的nums[0]~nums[i]的元素值为[0~i]
            nums[nums[i]], nums[i] = nums[i], nums[nums[i]]
        return -1

```