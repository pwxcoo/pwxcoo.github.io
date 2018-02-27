---
title: leetcode150题
date: 2018-02-25 22:15:39
categories: 备忘
tags:
- 算法
- leetcode
---

刷了 10 多天，刷了 leetcode 上前面 150 题。

因为很多题以前做过，所以做的还是比较快的。因为想熟悉一下 Java ,所以全部都用 Java 写的。leetcode 很多都是面试题，所以题目都是基本，没有什么特别难的题目，几道 hard 也没有特别难的。但是很多题目很经典，而且社区很活跃，Discuss 里很多神仙。有几道题目真的让我感觉到“还有这种操作？”的感受，因为思路太秀了。

之所以特地现在做一个中断，是因为前面 150 道比较经典，虽然很多类型的题目还没出现过，但是最新的题目我做过一些，有凸包，有最短路径什么的，而且 150-200 题之前很多 easy 和 medium，太水了，我懒得做了。我大概浏览了一下 150-200 的 hard，知道了基本的思路，然后就打算先这样中断一下，不打算最近撸了，之后如果没什么特别需要的地方，可能就每周做一下 Contest 吧。

因为有几道题目很帅，所以特地记录一下，有空可以在做一遍这几道。

## leetcode32. Longest Valid Parentheses
可以用 DP ，也可以用栈来做。我用了 DP 。官方 Solution 很详细了。
```Java
class Solution {
    public int longestValidParentheses(String s) {
        int len = s.length();
        int[] dp = new int[len];
        int res = 0;
        
        for(int i = 0; i < len; i++)
        {
            if(i - 1 >= 0 && s.charAt(i) == ')')
            {
                if(s.charAt(i - 1) == '(') 
                {
                    if(i >= 2) dp[i] = dp[i - 2] + 2;
                    else dp[i] = 2;
                }
                else if(s.charAt(i - 1) == ')')
                {
                    if(i - 1 - dp[i - 1] >= 0 && s.charAt(i - 1 - dp[i - 1]) == '(') 
                    {
                        dp[i] = dp[i - 1] + 2;
                        if(i - 2 - dp[i - 1] >= 0) dp[i] += dp[i - 2 - dp[i - 1]];
                    }
                }
                res = Math.max(res, dp[i]);
            }
        }
        
        return res;
    }
}
```

## leetcode42. Trapping Rain Water
用两个指针向中间收敛来维护一个变量。官方的 Solution 也很详细。也可以用 DP 或者栈来做。
```Java
class Solution {
    public int trap(int[] height) {
        if(height.length <= 2) return 0;
        int left = 0, right = height.length - 1;
        int res = 0, level = Math.min(height[left], height[right]);
        
        while(left < right)
        {
            if(height[left] <= height[right])
            {
                left++;
                if(height[left] > level)
                {
                    level = Math.min(height[left], height[right]);
                }
                else 
                {
                    res += level - height[left];
                }
            }
            else 
            {
                right--;
                if(height[right] > level)
                {
                    level = Math.min(height[left], height[right]);
                }
                else 
                {
                    res += level - height[right];
                }
            }
        }
        return res;      
    }
}
```

## leetcode69. Sqrt(x)
题目贴的 tag 是 binary search，想考查的是二分查找，用二分查找可以。但是一般标准库里用的是 sqrt() 用的是牛顿迭代法。
```Java
/**
 *  1. 二分查找 
 */
class Solution {
    public int mySqrt(int x) {
        if (x == 0)
            return 0;
        int left = 1, right = Integer.MAX_VALUE;
        while (true) {
            int mid = left + (right - left)/2;
            if (mid > x/mid) {
                right = mid - 1;
            } else {
                if (mid + 1 > x/(mid + 1))
                    return mid;
                left = mid + 1;
            }
        }
    }
}


/**
 *  2. 牛顿迭代法
 */
class Solution {
    public int mySqrt(int x) {
        long r = x;
        while (r*r > x)
            r = (r + x/r) / 2;
        return (int) r;
    }
}
```

## leetcode84. Largest Rectangle in Histogram
可以转化成最大01矩阵问题，但是会超时。当然用栈来模拟。
```Java
class Solution {
    public int largestRectangleArea(int[] heights) {
        Stack<Integer> st = new Stack<>();
        int res = 0, cnt = 0;
        for(int i = 0; i < heights.length; i++)
        {
            if(st.empty() || heights[i] >= st.peek())
            {
                st.push(heights[i]);
            }
            else 
            {
                cnt = 0;
                while(!st.empty() && heights[i] < st.peek())
                {
                    res = Math.max(res, ++cnt * st.pop());
                }
                while(cnt-- >= 0) st.push(heights[i]);
            }
        }
        cnt = 0;
        while(!st.empty())
        {
            res = Math.max(res, ++cnt * st.pop());
        }
        return res;
    }
}
```

## leetcode132. Palindrome Partitioning II
两种DP。
```Java
class Solution {
    public int minCut(String s) {
        boolean[][] isPalindrome = new boolean[s.length()][s.length()];
        int[] dp = new int[s.length() + 1];
        for(int i = 0; i <= s.length(); i++) dp[i] = s.length() - i - 1;
        
        for(int i = s.length() - 1; i >= 0; i --)
        {
            isPalindrome[i][i] = true;
            dp[i] = dp[i + 1] + 1;
            for(int j = i + 1; j < s.length(); j++)
            {
                if(s.charAt(i) == s.charAt(j) && (i + 1 == j || isPalindrome[i + 1][j - 1]))
                {
                    isPalindrome[i][j] = true;
                    if(dp[j + 1] + 1 < dp[i])
                    {
                        dp[i] = dp[j + 1] + 1;
                    }
                }
            }
        }
        return dp[0];
    }
}

class Solution {
    public int minCut(String s) {
        int[] dp = new int[s.length() + 1];
        for(int i = 0; i <= s.length(); i++) dp[i] = i - 1;
        
        for(int i = 0; i < s.length(); i++)
        {
            for(int j = 0; i - j >= 0 && i + j < s.length() && s.charAt(i - j) == s.charAt(i + j); j++) // odd 
                dp[i + j + 1] = Math.min(dp[i + j + 1], dp[i - j] + 1);
            for(int j = 0; i - j - 1 >= 0 && i + j < s.length() && s.charAt(i - j - 1) == s.charAt(i + j); j++) // even
                dp[i + j + 1] = Math.min(dp[i + j + 1], dp[i - j - 1] + 1);
        }
        return dp[s.length()];
    }
}
```

## leetcode137. Single Number II
感受到布尔代数的魅力了。一波位运算毁天灭地。
```Java
class Solution {
    public int singleNumber(int[] nums) {
        int one = 0, two = 0;
        for(int num : nums)
        {
            one = (one ^ num) & ~two;
            two = (two ^ num) & ~one;
        }
        return one;
    }
}
```

## leetcode146. LRU Cache
实现一个 LRU(Least Recently Used) 的 cache 类。突然感受了一下学操作系统学到的东西，这个看起来很简单的东西，实现起来还是有点秀的。用双向链表和哈希表来实现，让查询和插入都为O(1)。
```Java
class LRUCache {

    class DLinkedNode {
        int key;
        int value;
        DLinkedNode pre;
        DLinkedNode post;
    }

    /**
     * Always add the new node right after head;
     */
    private void addNode(DLinkedNode node){
        node.pre = head;
        node.post = head.post;

        head.post.pre = node;
        head.post = node;
    }

    /**
     * Remove an existing node from the linked list.
     */
    private void removeNode(DLinkedNode node){
        DLinkedNode pre = node.pre;
        DLinkedNode post = node.post;

        pre.post = post;
        post.pre = pre;
    }

    /**
     * Move certain node in between to the head.
     */
    private void moveToHead(DLinkedNode node){
        this.removeNode(node);
        this.addNode(node);
    }

    // pop the current tail. 
    private DLinkedNode popTail(){
        DLinkedNode res = tail.pre;
        this.removeNode(res);
        return res;
    }

    private Map<Integer, DLinkedNode> cache = new HashMap<Integer, DLinkedNode>();
    private int count;
    private int capacity;
    private DLinkedNode head, tail;

    public LRUCache(int capacity) {
        this.count = 0;
        this.capacity = capacity;
        
        head = new DLinkedNode();
        head.pre = null;

        tail = new DLinkedNode();
        tail.post = null;

        head.post = tail;
        tail.pre = head;
    }

    public int get(int key) {

        DLinkedNode node = cache.get(key);
        if(node == null){
            return -1; // should raise exception here.
        }

        // move the accessed node to the head;
        this.moveToHead(node);

        return node.value;
    }


    public void put(int key, int value) {
        DLinkedNode node = cache.get(key);

        if(node == null){

            DLinkedNode newNode = new DLinkedNode();
            newNode.key = key;
            newNode.value = value;

            this.cache.put(key, newNode);
            this.addNode(newNode);

            ++count;

            if(count > capacity){
                // pop the tail
                DLinkedNode tail = this.popTail();
                this.cache.remove(tail.key);
                --count;
            }
        }else{
            // update the value.
            node.value = value;
            this.moveToHead(node);
        }

    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```

## leetcode166. Fraction to Recurring Decimal
写一个除法生成分数。比较难的用无线循环小数的时候，用哈希表存储余数，key存储余数，value存储起始的位置。如果重复就在起始位置插入'('。

## leetcode187. Repeated DNA Sequences
用三位就可以表示 A，C，G，T。10个的话用 30 位表示一个序列保存到哈希表中。


## 总结
这几道是真的很秀。我这几天的代码全在[这里](https://github.com/pwxcoo/ac-game)了。
