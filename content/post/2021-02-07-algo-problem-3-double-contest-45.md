---
title: 编程题目(3)—No.45-双周赛
date: '2021-02-07'
slug: algo-problem-3-double-contest-45
categories:
  - programing
tags:
  - algo
---

## 题目A ([5657. 唯一元素的和][1])

**题目描述：**

给你一个整数数组 nums 。数组中唯一元素是那些只出现 恰好一次 的元素。
请你返回 nums 中唯一元素的 和 。
   
**示例 1：**
输入：nums = [1,2,3,2]
输出：4
解释：唯一元素为 [1,3] ，和为 4 。

**解题思路：**
模拟。
使用hash表记录每个元素的出现的次数，然后通过一次遍历取得所有出现一次的所有元素的sum。

**解题代码：**

    class Solution { 
    public:
        int sumOfUnique(vector<int>& nums) {
            unordered_map<int, int> hash;
            for (auto x : nums) hash[x] ++;
            int res = 0;
            for (auto [k, v] : hash)
                if (v == 1)
                    res += k;
            return res;
        } };


## 题目B([5658. 任意子数组和的绝对值的最大值][2])
**题目描述：**

    给你一个整数数组 nums 。一个子数组 [numsl, numsl+1, ..., numsr-1, numsr] 的 和的绝对值 为 abs(numsl + numsl+1 + ... + numsr-1 + numsr) 。
    
    请你找出 nums 中 和的绝对值 最大的任意子数组（可能为空），并返回该 最大值 。
    
    abs(x) 定义如下：
    
    如果 x 是负整数，那么 abs(x) = -x 。
    如果 x 是非负整数，那么 abs(x) = x 。
    
    **示例 1：**
    输入：nums = [1,-3,2,3,-4]
    输出：5
    解释：子数组 [2,3] 和的绝对值最大，为 abs(2+3) = abs(5) = 5 。

**解题思路：**
前缀和。
整体思路：
根据前缀和数组能够快速求取区间和，而绝对值的最大值无非是找到这个区间和中的最大值和最小值，做一次遍历，求取差值的最大情况即可。一般绝对值需要利用区间的最大和最小值两种情况来分析替代。

第1步：求数组的前缀和数组sum
第2步： 遍历前缀和数组，每次截取与区间最小值的差值，以及与区间最大值的差值。维护这两个差值中的较大值。
第3步：返回区间差值最大值。

**解题代码：**

    class Solution {
    public:
        int maxAbsoluteSum(vector<int>& nums) {
            int n = nums.size();
            vector<int> s(n + 1);
            for (int i = 1; i <= n; i ++ ) s[i] = s[i - 1] + nums[i - 1];
            int res = 0;
            int minv = 0, maxv = 0;
            for (int i = 1; i <= n; i ++ ) {
                res = max(res, abs(s[i] - minv));
                res = max(res, abs(s[i] - maxv));
                minv = min(minv, s[i]);
                maxv = max(maxv, s[i]);
            }
            return res;
        }
    };

## 题目C([5659. 删除字符串两端相同字符后的最短长度][3])
**题目描述：**

    给你一个只包含字符 'a'，'b' 和 'c' 的字符串 s ，你可以执行下面这个操作（5 个步骤）任意次：
    
    选择字符串 s 一个 非空 的前缀，这个前缀的所有字符都相同。
    选择字符串 s 一个 非空 的后缀，这个后缀的所有字符都相同。
    前缀和后缀在字符串中任意位置都不能有交集。
    前缀和后缀包含的所有字符都要相同。
    同时删除前缀和后缀。
    请你返回对字符串 s 执行上面操作任意次以后（可能 0 次），能得到的 最短长度 。
    
    示例1：
    输入：s = "ca"
    输出：2
    解释：你没法删除任何一个字符，所以字符串长度仍然保持不变。

**解题思路：**
贪心，模拟，双指针。
指针p从前扫描，指针q从后扫描，将前后相同字符的部分删掉。
需要特判的情况，当最后只剩下一个字符是是不能继续删除的，因为前后待删区间是不能存在交集的。
只需要一遍的扫描，时间复杂度是O(n)。

**解题代码：**

    class Solution {
    public:
        int minimumLength(string s) {
            int i = 0, j = s.size() - 1;
            while (i < j) {
                if (s[i] != s[j]) break;
                char c = s[i];
                while (i <= j && s[i] == c) i ++ ;
                while (i <= j && s[j] == c) j -- ;
            }
            return j - i + 1;
        }
    };


##题目D:([5660. 最多可以参加的会议数目 II][4])
**题目描述：**

    给你一个 events 数组，其中 events[i] = [startDayi, endDayi, valuei] ，表示第 i 个会议在 startDayi 天开始，第 endDayi 天结束，如果你参加这个会议，你能得到价值 valuei 。同时给你一个整数 k 表示你能参加的最多会议数目。
    
    你同一时间只能参加一个会议。如果你选择参加某个会议，那么你必须 完整 地参加完这个会议。会议结束日期是包含在会议内的，也就是说你不能同时参加一个开始日期与另一个结束日期相同的两个会议。
    请你返回能得到的会议价值 最大和 。
    **示例 1：**
    ![][5]
    输入：events = [[1,2,4],[3,4,3],[2,3,1]], k = 2
    输出：7
    解释：选择绿色的活动会议 0 和 1，得到总价值和为 4 + 3 = 7 。

**解题思路：**
DP。题目给出若干区间，每个区间有权值，从中选出若干区间且不想相交，使得权值最大。



  [1]: https://leetcode-cn.com/problems/sum-of-unique-elements/
  [2]: https://leetcode-cn.com/problems/maximum-absolute-sum-of-any-subarray/
  [3]: https://leetcode-cn.com/problems/minimum-length-of-string-after-deleting-similar-ends/
  [4]: https://leetcode-cn.com/problems/maximum-number-of-events-that-can-be-attended-ii/
  [5]: https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2021/02/06/screenshot-2021-01-11-at-60048-pm.png
  [6]: https://leetcode-cn.com/problems/sum-of-unique-elements/
  [7]: https://leetcode-cn.com/problems/maximum-absolute-sum-of-any-subarray/