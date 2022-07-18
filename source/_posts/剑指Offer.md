---
title: 剑指Offer
date: 2022-07-11 02:55:27
tags: 算法
categories: 计算机基础知识
---

# 第一天

## [剑指 Offer 09. 用两个栈实现队列](https://leetcode.cn/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

### 题解

```csharp
/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.AppendTail(value);
 * int param_2 = obj.DeleteHead();
 */
public class CQueue
{
    private Stack<int> _a, _b;

    public CQueue()
    {
        _a = new Stack<int>();
        _b = new Stack<int>();
    }

    public void AppendTail(int value)
    {
        _a.Push(value);
    }

    public int DeleteHead()
    {
        if (_b.Count > 0)
            return _b.Pop();
        if (_a.Count == 0)
            return -1;
        while (_a.Count > 0)
            _b.Push(_a.Pop());
        return _b.Pop();
    }
}
```

### 后记

没什么，经典双栈拼接队列。

## [剑指 Offer 30. 包含min函数的栈 - 力扣（LeetCode）](https://leetcode.cn/problems/bao-han-minhan-shu-de-zhan-lcof/)

### 题解

```csharp
public class MinStack
{
    private Stack<int> _a, _minStack;

    /** initialize your data structure here. */
    public MinStack()
    {
        _a = new Stack<int>();
        _minStack = new Stack<int>();
    }


    public void Push(int x)
    {
        _a.Push(x);
        if (_minStack.Count == 0 || _minStack.Peek() >= x)
            _minStack.Push(x);
    }

    public void Pop()
    {
        if (_a.Pop() == _minStack.Peek())
        {
            _minStack.Pop();
        }
    }

    public int Top()
    {
        return _a.Peek();
    }

    public int Min()
    {
        return _minStack.Peek();
    }
}
```

### 后记

其实这个玩意 最好还是直接记录一下，但是感觉没必要。

# 第二天

## [剑指 Offer 06. 从尾到头打印链表 - 力扣（LeetCode）](https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

## 题解

```csharp
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     public int val;
 *     public ListNode next;
 *     public ListNode(int x) { val = x; }
 * }
 */
public class Solution
{
    private Stack<int> _stack = new Stack<int>();

    public int[] ReversePrint(ListNode head)
    {
        while (head != null)
        {
            _stack.Push(head.val);
            head = head.next;
        }

        int[] res = new int[_stack.Count];
        for (int i = 0; i < res.Length; i++)
            res[i] = _stack.Pop();
        return res;
    }
}
```

## 后记

最大的难点居然是有人`Length`拼了四次才拼对。

## [剑指 Offer 24. 反转链表 - 力扣（LeetCode）](https://leetcode.cn/problems/fan-zhuan-lian-biao-lcof/)

### 题解

```csharp
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     public int val;
 *     public ListNode next;
 *     public ListNode(int x) { val = x; }
 * }
 */
public class Solution {
    public ListNode ReverseList(ListNode head) {
        ListNode pre = null;
        ListNode cur = head;
        while(cur != null)
        {
            ListNode temp = cur.next;
            cur.next = pre;
            pre = cur;
            cur = temp;
        }
        return pre;
    }
}
```

### 后记

我是废物！

## [剑指 Offer 35. 复杂链表的复制 - 力扣（LeetCode）](https://leetcode.cn/problems/fu-za-lian-biao-de-fu-zhi-lcof/)

### 题解

#### 哈希表 + 迭代

```csharp
/*
// Definition for a Node.
public class Node {
    public int val;
    public Node next;
    public Node random;
    
    public Node(int _val) {
        val = _val;
        next = null;
        random = null;
    }
}
*/

public class Solution
{
    private Dictionary<Node, Node> _nodeDic;

    public Node CopyRandomList(Node head)
    {
        _nodeDic = new Dictionary<Node, Node>();

        if (head == null)
        {
            return null;
        }

        Node tempNode = head;

        while (tempNode != null)
        {
            _nodeDic.Add(tempNode, new Node(tempNode.val));
            tempNode = tempNode.next;
        }

        tempNode = head;
        while (tempNode != null)
        {
            if (tempNode.next != null)
                _nodeDic[tempNode].next = _nodeDic[tempNode.next];
            if (tempNode.random != null)
                _nodeDic[tempNode].random = _nodeDic[tempNode.random];
            tempNode = tempNode.next;
        }

        return _nodeDic[head];
    }
}
```

#### 拼接 + 拆分

```csharp
    public Node CopyRandomList(Node head)
    {
        if (head == null)
            return null;

        Node tempNode = head;
        while (tempNode != null)
        {
            Node temp = new Node(tempNode.val);
            temp.next = tempNode.next;
            tempNode.next = temp;

            tempNode = tempNode.next.next;
        }

        tempNode = head;
        while (tempNode != null)
        {
            if (tempNode.random != null)
                tempNode.next.random = tempNode.random.next;
            tempNode = tempNode.next.next;
        }

        Node cur = head.next;
        Node res = head.next;
        tempNode = head;

        while (cur.next != null)
        {
            tempNode.next = cur.next;
            cur.next = cur.next.next;

            cur = cur.next;
            tempNode = tempNode.next;
        }

        tempNode.next = null;
        return res;
    }
}
```

### 后记

第一种解法偏常规，主要需要判空。

第二种解法你会发现题目十分阴间，比如37行的`tempNode.next = null`，如果没有这一句话，该题目错误。因为你必须还原原链表，但是我明明只需要返回新链表啊！！你怎么能检测原链表啊！！！你是怎么做到的啊。**第二种解法还是需要多看看。**

# 第三天

## [剑指 Offer 05. 替换空格 - 力扣（LeetCode）](https://leetcode.cn/problems/ti-huan-kong-ge-lcof/)

### 硬写

```csharp
public class Solution
{
    public string ReplaceSpace(string s)
    {
        StringBuilder sb = new StringBuilder();
        foreach (char c in s)
        {
            if (c == ' ')
                sb.Append("%20");
            else
                sb.Append(c);
        }

        return sb.ToString();
    }
}
```

### 后记

没什么东西，标准题解的语言中大部分没有可变字符串。我是懒鬼，我选择可变。

## [剑指 Offer 58 - II. 左旋转字符串 - 力扣（LeetCode）](https://leetcode.cn/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/)

### 硬写

```csharp
public class Solution
{
    public string ReverseLeftWords(string s, int n)
    {
        StringBuilder sb = new StringBuilder();
        for (int i = n; i < n + s.Length; i++)
        {
            sb.Append(s[i % s.Length]);
        }

        return sb.ToString();
    }
}
```

### 后记

sb yyds。

# 第四天

## [剑指 Offer 03. 数组中重复的数字 - 力扣（LeetCode）](https://leetcode.cn/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

### 题解

```csharp
public class Solution
{
    public int FindRepeatNumber(int[] nums)
    {
        int i = 0;
        while (i < nums.Length)
        {
            // 当前索引满足条件
            if (nums[i] == i)
            {
                i++;
                continue;
            }

            if (nums[nums[i]] == nums[i])
                return nums[i];
            (nums[nums[i]], nums[i]) = (nums[i], nums[nums[i]]);
        }

        return 0;
    }
}
```

### 后记

想笑啊。排行最优的算法是桶排序。

## [剑指 Offer 53 - I. 在排序数组中查找数字 I - 力扣（LeetCode）](https://leetcode.cn/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/)

### 题解

```csharp
public class Solution {
    public int Search(int[] nums, int target) {
        int left = 0, right = nums.Length - 1;
        while(left <= right)
        {
            int mid = right - (right - left) / 2;
            if(nums[mid] <= target)
                left = mid + 1;
            else
                right = mid - 1;
        }

        int resRight = left;
        if(right >= 0 && nums[right] != target)
            return 0;
        left = 0;
        right = nums.Length - 1;
        while(left <= right)
        {
            int mid = right - (right - left) / 2;
            if(nums[mid] < target)
                left = mid + 1;
            else
                right = mid - 1;
        }

        int resLeft = right;
        return resRight - resLeft - 1;
    }
}
```

### 后记

二分好难啊。但是？这玩意在leetcode不如一句`return nums.Count(num => num == target);`，Linq太强啦呜呜呜。

## [剑指 Offer 53 - II. 0～n-1中缺失的数字 - 力扣（LeetCode）](https://leetcode.cn/problems/que-shi-de-shu-zi-lcof/)

### 题解

```csharp
public class Solution
{
    public int MissingNumber(int[] nums)
    {
        int left = 0, right = nums.Length - 1;
        while (left <= right)
        {
            int mid = right - (right - left) / 2;
            if (nums[mid] == mid)
                left = mid + 1;
            else
                right = mid - 1;
        }

        return left;
    }
}
```

### 后记

最快的还是暴力，我想笑啊。

# 第五天

## [剑指 Offer 04. 二维数组中的查找 - 力扣（LeetCode）](https://leetcode.cn/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

### 题解

```csharp
public class Solution
{
    public bool FindNumberIn2DArray(int[][] matrix, int target)
    {
        if (matrix == null || matrix.Length == 0)
            return false;

        int x = 0;
        int y = matrix[0].Length - 1;

        while (x < matrix.Length && y >= 0)
        {
            if (matrix[x][y] == target)
                return true;
            else if (matrix[x][y] < target)
                x++;
            else
                y--;
        }

        return false;
    }
}
```

### 后记

我好烦判空啊！

## [剑指 Offer 11. 旋转数组的最小数字 - 力扣（LeetCode）](https://leetcode.cn/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

### 题解

```csharp
public class Solution
{
    public int MinArray(int[] numbers)
    {
        int left = 0;
        int right = numbers.Length - 1;

        while (left < right)
        {
            int mid = left - (left - right) / 2;

            if (numbers[mid] < numbers[right])
            {
                right = mid;
            }
            else if (numbers[mid] > numbers[right])
            {
                left = mid + 1;
            }
            else
            {
                right--;
            }
        }

        return numbers[left];
    }
}
```

### 后记

呜呜呜，我真的不会二分。

## [剑指 Offer 50. 第一个只出现一次的字符 - 力扣（LeetCode）](https://leetcode.cn/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

### 题解

```csharp
public class Solution {
    public char FirstUniqChar(string s) {
        if (s.Length != 0) {
            Dictionary<char, bool> dic = new Dictionary<char, bool>();
            foreach(char c in s) {
                if (dic.ContainsKey(c)) 
                    dic[c] = false;
                else 
                    dic.Add(c, true);
            }
            foreach (var d in dic)
            {
                if (d.Value) 
                    return d.Key;
            }
        }
        return ' ';
    }
}
```

### 后记

没什么，感觉不如原神。

# 第六天

## [剑指 Offer 32 - I. 从上到下打印二叉树 - 力扣（LeetCode）](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

### 题解

```csharp
public class Solution
{
    public int[] LevelOrder(TreeNode root)
    {
        if (root == null)
            return new int[0];

        Queue<TreeNode> treeNodes = new Queue<TreeNode>();
        List<int> res = new List<int>();
        treeNodes.Enqueue(root);
        while (treeNodes.Count > 0)
        {
            TreeNode now = treeNodes.Dequeue();
            res.Add(now.val);
            if (now.left != null)
                treeNodes.Enqueue(now.left);
            if (now.right != null)
                treeNodes.Enqueue(now.right);
        }

        return res.ToArray();
    }
}
```

### 后记

`treeNodes.Enqueue(root);` 添加元素

`treeNodes.Dequeue();` 取出元素

狠狠牢记API

## [剑指 Offer 32 - II. 从上到下打印二叉树 II - 力扣（LeetCode）](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)

### 题解

```csharp
public class Solution
{
    public IList<IList<int>> LevelOrder(TreeNode root)
    {
        var ans = new List<IList<int>>();
        if (root == null) return ans;
        var q = new Queue<TreeNode>();
        q.Enqueue(root);
        while (q.Count > 0)
        {
            var level = new List<int>();
            for (int cnt = q.Count; cnt > 0; cnt--)
            {
                var cur = q.Dequeue();
                level.Add(cur.val);
                if (cur.left != null) q.Enqueue(cur.left);
                if (cur.right != null) q.Enqueue(cur.right);
            }

            ans.Add(level);
        }

        return ans;
    }
}
```

### 后记

这个BFS有点意思啊。

## [剑指 Offer 32 - III. 从上到下打印二叉树 III - 力扣（LeetCode）](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

### 题解

```csharp
public class Solution
{
    public IList<IList<int>> LevelOrder(TreeNode root)
    {
        List<IList<int>> list = new List<IList<int>>();
        Queue<TreeNode> queue = new Queue<TreeNode>();
        if (root != null)
        {
            queue.Enqueue(root);
        }

        while (queue.Count > 0)
        {
            List<int> tempList = new List<int>();
            int count = queue.Count;
            for (int i = 0; i < count; i++)
            {
                TreeNode tree = queue.Dequeue();
                if (list.Count % 2 == 0)
                {
                    tempList.Add(tree.val);
                }
                else
                {
                    tempList.Insert(0, tree.val);
                }

                if (tree.left != null)
                {
                    queue.Enqueue(tree.left);
                }

                if (tree.right != null)
                {
                    queue.Enqueue(tree.right);
                }
            }

            list.Add(tempList);
        }

        return list;
    }
}
```

### 后记

其实今天这几个题，都是BFS。。。

# 第七天

## [剑指 Offer 26. 树的子结构 - 力扣（LeetCode）](https://leetcode.cn/problems/shu-de-zi-jie-gou-lcof/)

### 题解

```csharp
public class Solution {
    public bool IsSubStructure(TreeNode A, TreeNode B)
    {
        if (A == null || B == null)
            return false;
        return Helper(A, B) || IsSubStructure(A.left, B) || IsSubStructure(A.right, B);
    }

    public bool Helper(TreeNode a, TreeNode b)
    {
        if (b == null)
            return true;
        if (a == null || a.val != b.val)
            return false;
        return Helper(a.left, b.left) && Helper(a.right, b.right);
    }
}
```

### 后记

怎么又是搜索。

## [剑指 Offer 27. 二叉树的镜像 - 力扣（LeetCode）](https://leetcode.cn/problems/er-cha-shu-de-jing-xiang-lcof/)

### 题解

```csharp
public class Solution
{
    public TreeNode MirrorTree(TreeNode root)
    {
        if (root == null)
            return null;
        Stack<TreeNode> stack = new Stack<TreeNode>();
        stack.Push(root);
        while (stack.Count > 0)
        {
            var nowRoot = stack.Pop();
            if (nowRoot.left != null)
                stack.Push(nowRoot.left);
            if (nowRoot.right != null)
                stack.Push(nowRoot.right);

            (nowRoot.left, nowRoot.right) = (nowRoot.right, nowRoot.left);
        }

        return root;
    }
}
```

### 后记

突然发现自己雀氏老了

## [剑指 Offer 28. 对称的二叉树 - 力扣（LeetCode）](https://leetcode.cn/problems/dui-cheng-de-er-cha-shu-lcof/)

### 题解

```csharp
public class Solution
{
    public bool IsSymmetric(TreeNode root)
    {
        if (root == null) return true;
        return (Helper(root.left, root.right));
    }

    private bool Helper(TreeNode a, TreeNode b)
    {
        if (a == null && b == null) return true;
        if (a == null || b == null) return false;
        if (a.val != b.val) return false;
        return Helper(a.left, b.right) && Helper(a.right, b.left);
    }
}
```

### 后记

麻，我真的不喜欢递归。

# 第八天

## [剑指 Offer 10- I. 斐波那契数列 - 力扣（LeetCode）](https://leetcode.cn/problems/fei-bo-na-qi-shu-lie-lcof/)

### 题解

```csharp
public class Solution
{
    public int Fib(int n)
    {
        int a = 0, b = 1, sum = 0;
        for (int i = 0; i < n; i++)
        {
            sum = (a + b) % 1000000007;
            a = b;
            b = sum;
        }

        return a;
    }
}
```

### 后记

递归会爆。

## [剑指 Offer 10- II. 青蛙跳台阶问题 - 力扣（LeetCode）](https://leetcode.cn/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

### 题解

```csharp
public class Solution
{
    public int NumWays(int n)
    {
        int a = 1, b = 1, sum = 0;
        for (int i = 0; i < n; i++)
        {
            sum = (a + b) % 1000000007;
            a = b;
            b = sum;
        }

        return a;
    }
}
```

### 后记

和上面一模一样。

## [剑指 Offer 63. 股票的最大利润 - 力扣（LeetCode）](https://leetcode.cn/problems/gu-piao-de-zui-da-li-run-lcof/)

### 题解

```csharp
public class Solution
{
    public int MaxProfit(int[] prices)
    {
        int res = 0;
        int cost = int.MaxValue;

        foreach (var price in prices)
        {
            if (price < cost)
                cost = price;
            else if (res < price - cost)
                res = price - cost;
        }

        return res;
    }
}
```

### 后记

这题为什么要说是DP（我感觉不来啊）
