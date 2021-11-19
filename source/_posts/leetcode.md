---
title: leetcode刷题记录
date: 2021-11-01 13:50:37
tags:
---

# [575. 分糖果](https://leetcode-cn.com/problems/distribute-candies/)

---

难度 `简单` | 标签 `数组` `哈希表` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给定一个<strong>偶数</strong>长度的数组，其中不同的数字代表着不同种类的糖果，每一个数字代表一个糖果。你需要把这些糖果<strong>平均</strong>分给一个弟弟和一个妹妹。返回妹妹可以获得的最大糖果的种类数。</p>
<p><strong>示例 1:</strong></p>
<pre><strong>输入:</strong> candies = [1,1,2,2,3,3]
<strong>输出:</strong> 3
<strong>解析: </strong>一共有三种种类的糖果，每一种都有两个。
     最优分配方案：妹妹获得[1,2,3],弟弟也获得[1,2,3]。这样使妹妹获得糖果的种类数最多。
</pre>
<p><strong>示例 2 :</strong></p>
<pre><strong>输入:</strong> candies = [1,1,2,3]
<strong>输出:</strong> 2
<strong>解析:</strong> 妹妹获得糖果[2,3],弟弟获得糖果[1,1]，妹妹有两种不同的糖果，弟弟只有一种。这样使得妹妹可以获得的糖果种类数最多。
</pre>
<p><strong>注意:</strong></p>
<ol>
	<li>数组的长度为[2, 10,000]，并且确定为偶数。</li>
	<li>数组中数字的大小在范围[-100,000, 100,000]内。
	<ol>
	</ol>
	</li>
</ol>
</section>

## My Solution

假设糖果的数量为`n`，由于妹妹只能分到一半的苹果，所以答案不会超过`n/2`。

假设糖果种类一共有`m`种，答案也不会超过`m`

综上所述，答案为$min(m,\cfrac{n}{2} )$




```csharp
public class Solution {
    public int DistributeCandies(int[] candyType)
    {
        ISet<int> set = new HashSet<int>();
        foreach (var i in candyType)
        {
            set.Add(i);
        }

        return Math.Min(set.Count, candyType.Length / 2);
    }
}
```



# [237. 删除链表中的节点](https://leetcode-cn.com/problems/delete-node-in-a-linked-list/)

---

难度 `简单` | 标签 `链表` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>请编写一个函数，用于 <strong>删除单链表中某个特定节点</strong> 。在设计函数时需要注意，你无法访问链表的头节点&nbsp;<code>head</code> ，只能直接访问 <strong>要被删除的节点</strong> 。</p>
<p>题目数据保证需要删除的节点 <strong>不是末尾节点</strong> 。</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<img style="width: 450px; height: 322px;" src="https://assets.leetcode.com/uploads/2020/09/01/node1.jpg" alt="">
<pre><strong>输入：</strong>head = [4,5,1,9], node = 5
<strong>输出：</strong>[4,1,9]
<strong>解释：</strong>指定链表中值为&nbsp;5&nbsp;的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -&gt; 1 -&gt; 9
</pre>
<p><strong>示例 2：</strong></p>
<img style="width: 450px; height: 354px;" src="https://assets.leetcode.com/uploads/2020/09/01/node2.jpg" alt="">
<pre><strong>输入：</strong>head = [4,5,1,9], node = 1
<strong>输出：</strong>[4,5,9]
<strong>解释：</strong>指定链表中值为&nbsp;1&nbsp;的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -&gt; 5 -&gt; 9</pre>
<p><strong>示例 3：</strong></p>
<pre><strong>输入：</strong>head = [1,2,3,4], node = 3
<strong>输出：</strong>[1,2,4]
</pre>
<p><strong>示例 4：</strong></p>
<pre><strong>输入：</strong>head = [0,1], node = 0
<strong>输出：</strong>[1]
</pre>
<p><strong>示例 5：</strong></p>
<pre><strong>输入：</strong>head = [-3,5,-99], node = -3
<strong>输出：</strong>[5,-99]
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li>链表中节点的数目范围是 <code>[2, 1000]</code></li>
	<li><code>-1000 &lt;= Node.val &lt;= 1000</code></li>
	<li>链表中每个节点的值都是唯一的</li>
	<li>需要删除的节点 <code>node</code> 是 <strong>链表中的一个有效节点</strong> ，且 <strong>不是末尾节点</strong></li>
</ul>
</section>

## My Solution

题目保证`node`是**优先节点**，且不是**末尾节点**，所以将当前节点的值改为下一个节点的值，再将当前节点指向下下节点即可完成覆盖。

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
    public void DeleteNode(ListNode node) {
        node.val = node.next.val;
        node.next = node.next.next;
    }
}
```

# [407. 接雨水 II](https://leetcode-cn.com/problems/trapping-rain-water-ii/)

---

难度 `困难` | 标签 `广度优先搜索` `数组` `矩阵` `堆（优先队列）` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给你一个&nbsp;<code>m x n</code>&nbsp;的矩阵，其中的值均为非负整数，代表二维高度图每个单元的高度，请计算图中形状最多能接多少体积的雨水。</p>
<p>&nbsp;</p>
<p><strong>示例 1:</strong></p>
<p><img src="https://assets.leetcode.com/uploads/2021/04/08/trap1-3d.jpg" alt=""></p>
<pre><strong>输入:</strong> heightMap = [[1,4,3,1,3,2],[3,2,1,3,2,4],[2,3,3,2,3,1]]
<strong>输出:</strong> 4
<strong>解释:</strong> 下雨后，雨水将会被上图蓝色的方块中。总的接雨水量为1+2+1=4。
</pre>
<p><strong>示例&nbsp;2:</strong></p>
<p><img src="https://assets.leetcode.com/uploads/2021/04/08/trap2-3d.jpg" alt=""></p>
<pre><strong>输入:</strong> heightMap = [[3,3,3,3,3],[3,2,2,2,3],[3,2,1,2,3],[3,2,2,2,3],[3,3,3,3,3]]
<strong>输出:</strong> 10
</pre>
<p>&nbsp;</p>
<p><strong>提示:</strong></p>
<ul>
	<li><code>m == heightMap.length</code></li>
	<li><code>n == heightMap[i].length</code></li>
	<li><code>1 &lt;= m, n &lt;= 200</code></li>
	<li><code>0 &lt;= heightMap[i][j] &lt;= 2 * 10<sup>4</sup></code></li>
</ul>
<p>&nbsp;</p>
</section>

## My Solution

首先思考一下什么样的方块一定可以接住水：

- 该方块不为最外层的方块
- 该方块的自身高度比其上下左右四个相邻的方块接水

我们假设初始时矩阵的每个格子都接满了水，且高度均为 $maxHeight$，其中 $maxHeight$ 为矩阵中高度最高的格子。我们知道方块接水后的高度为 $water[i][j]$，它的求解公式与方法一样。方块 $(i,j)$ 的接水后的高度为：
$$
water[i][j]=max(heightMap[i],min(water[i-1][j],water[i+1][j],water[i][j-1],water[i][j+1]))
$$


我们知道方块 $(i,j)$ 实际接水的容量计算公式为$water[i][j]-heightMap[i][j]$ 。
我们首先假设每个方块 $(i,j)$ 的接水后的高度均为 $water[i][j]=maxHeight$，首先我们知道最外层的方块的肯定不能接水，所有的多余的水都会从最外层的方块溢出，我们每次发现当前方块 $(i,j)$ 的接水高度$water[i][j]$小于与它相邻的4个模块的接水高度时，则我们将进行调整接水高度，我们将其相邻的四个方块的接水高度调整与$(i,j)$高度保持一致，我们不断重复的进行调整，直到所有的方块的接水高度不再有调整时即为满足要求。



```csharp
public class Solution {
    public int TrapRainWater(int[][] heightMap)
    {
        int m = heightMap.Length;
        int n = heightMap[0].Length;
        int[] dirs = { -1, 0, 1, 0, -1 };
        int maxHeight = 0;
        for (int i = 0; i < m; i++)
        {
            for (int j = 0; j < n; j++)
            {
                maxHeight = Math.Max(maxHeight, heightMap[i][j]);
            }
        }

        int[,] water = new int[m, n];
        for (int i = 0; i < m; i++)
        {
            for (int j = 0; j < n; j++)
            {
                water[i, j] = maxHeight;
            }
        }

        Queue<int[]> queue = new Queue<int[]>();
        for (int i = 0; i < m; i++)
        {
            for (int j = 0; j < n; j++)
            {
                if (i != 0 && i != m - 1 && j != 0 && j != n - 1) continue;
                if (water[i, j] <= heightMap[i][j]) continue;
                water[i, j] = heightMap[i][j];
                queue.Enqueue(new[]{i, j});
            }
        }

        while (queue.Count > 0)
        {
            int[] curr = queue.Dequeue();
            int x = curr[0];
            int y = curr[1];
            for (int i = 0; i < 4; i++)
            {
                int nx = x + dirs[i], ny = y + dirs[i + 1];
                if (nx < 0 || nx > m - 1 || ny < 0 || ny > n - 1)
                    continue;
                if (water[x, y] < water[nx, ny] && water[nx, ny] > heightMap[nx][ny])
                {
                    water[nx, ny] = Math.Max(water[x, y], heightMap[nx][ny]);
                    queue.Enqueue(new[] { nx, ny });
                }
            }
        }

        int res = 0;
        for (int i = 0; i < m; i++)
        {
            for (int j = 0; j < n; j++)
            {
                res += water[i, j] - heightMap[i][j];
            }
        }

        return res;
    }
}
```



# [367. 有效的完全平方数](https://leetcode-cn.com/problems/valid-perfect-square/)

---

难度 `简单` | 标签 `数学` `二分查找` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给定一个 <strong>正整数</strong> <code>num</code> ，编写一个函数，如果 <code>num</code> 是一个完全平方数，则返回 <code>true</code> ，否则返回 <code>false</code> 。</p>
<p><strong>进阶：不要</strong> 使用任何内置的库函数，如&nbsp; <code>sqrt</code> 。</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<pre><strong>输入：</strong>num = 16
<strong>输出：</strong>true
</pre>
<p><strong>示例 2：</strong></p>
<pre><strong>输入：</strong>num = 14
<strong>输出：</strong>false
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>1 &lt;= num &lt;= 2^31 - 1</code></li>
</ul>
</section>

## My Solution

二分搜索来优化遍历过程

```csharp
public class Solution {
    public bool IsPerfectSquare(int num) {
        int left = 0, right = num;
        while (left <= right) {
            int mid = (right - left) / 2 + left;
            long square = (long) mid * mid;
            if (square < num) {
                left = mid + 1;
            } else if (square > num) {
                right = mid - 1;
            } else {
                return true;
            }
        }
        return false;
    }
}

```

# [1218. 最长定差子序列](https://leetcode-cn.com/problems/longest-arithmetic-subsequence-of-given-difference/)

---

难度 `中等` | 标签 `数组` `哈希表` `动态规划` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给你一个整数数组&nbsp;<code>arr</code>&nbsp;和一个整数&nbsp;<code>difference</code>，请你找出并返回 <code>arr</code>&nbsp;中最长等差子序列的长度，该子序列中相邻元素之间的差等于 <code>difference</code> 。</p>
<p><strong>子序列</strong> 是指在不改变其余元素顺序的情况下，通过删除一些元素或不删除任何元素而从 <code>arr</code> 派生出来的序列。</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<pre><strong>输入：</strong>arr = [1,2,3,4], difference = 1
<strong>输出：</strong>4
<strong>解释：</strong>最长的等差子序列是 [1,2,3,4]。</pre>
<p><strong>示例&nbsp;2：</strong></p>
<pre><strong>输入：</strong>arr = [1,3,5,7], difference = 1
<strong>输出：</strong>1
<strong>解释：</strong>最长的等差子序列是任意单个元素。
</pre>
<p><strong>示例 3：</strong></p>
<pre><strong>输入：</strong>arr = [1,5,7,8,5,3,4,2,1], difference = -2
<strong>输出：</strong>4
<strong>解释：</strong>最长的等差子序列是 [7,5,3,1]。
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>1 &lt;= arr.length &lt;= 10<sup>5</sup></code></li>
	<li><code>-10<sup>4</sup> &lt;= arr[i], difference &lt;= 10<sup>4</sup></code></li>
</ul>
</section>

## My Solution

$$
dp[i] = 
\begin{cases}
dp[i - difference] + 1,& dp[i-difference] \neq -1\\
1& dp[i-difference] = -1
\end{cases}
$$



```csharp
public class Solution {
   public int LongestSubsequence(int[] arr, int difference)
    {
        int res = 0;
        Dictionary<int, int> dictionary = new Dictionary<int, int>();
        foreach (var t in arr)
        {
            int number = dictionary.ContainsKey(t - difference) ? dictionary[t - difference] : 0;
            if (dictionary.ContainsKey(t))
            {
                dictionary[t] = number + 1;
            }
            else
            {
                dictionary.Add(t, number + 1);
            }

            res = Math.Max(res, dictionary[t]);
        }

        return res;
    }
}
```

# [268. 丢失的数字](https://leetcode-cn.com/problems/missing-number/)

---

难度 `简单` | 标签 `位运算` `数组` `哈希表` `数学` `排序` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给定一个包含 <code>[0, n]</code>&nbsp;中&nbsp;<code>n</code>&nbsp;个数的数组 <code>nums</code> ，找出 <code>[0, n]</code> 这个范围内没有出现在数组中的那个数。</p>
<ul>
</ul>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<pre><strong>输入：</strong>nums = [3,0,1]
<strong>输出：</strong>2
<b>解释：</b>n = 3，因为有 3 个数字，所以所有的数字都在范围 [0,3] 内。2 是丢失的数字，因为它没有出现在 nums 中。</pre>
<p><strong>示例 2：</strong></p>
<pre><strong>输入：</strong>nums = [0,1]
<strong>输出：</strong>2
<b>解释：</b>n = 2，因为有 2 个数字，所以所有的数字都在范围 [0,2] 内。2 是丢失的数字，因为它没有出现在 nums 中。</pre>
<p><strong>示例 3：</strong></p>
<pre><strong>输入：</strong>nums = [9,6,4,2,3,5,7,0,1]
<strong>输出：</strong>8
<b>解释：</b>n = 9，因为有 9 个数字，所以所有的数字都在范围 [0,9] 内。8 是丢失的数字，因为它没有出现在 nums 中。</pre>
<p><strong>示例 4：</strong></p>
<pre><strong>输入：</strong>nums = [0]
<strong>输出：</strong>1
<b>解释：</b>n = 1，因为有 1 个数字，所以所有的数字都在范围 [0,1] 内。1 是丢失的数字，因为它没有出现在 nums 中。</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>n == nums.length</code></li>
	<li><code>1 &lt;= n &lt;= 10<sup>4</sup></code></li>
	<li><code>0 &lt;= nums[i] &lt;= n</code></li>
	<li><code>nums</code> 中的所有数字都 <strong>独一无二</strong></li>
</ul>
<p>&nbsp;</p>
<p><strong>进阶：</strong>你能否实现线性时间复杂度、仅使用额外常数空间的算法解决此问题?</p>
</section>

## My Solution

```csharp
public class Solution {
    public int MissingNumber(int[] nums)
    {
        int res = nums.Length;
        for (var i = 0; i < nums.Length; i++)
        {
            res += i;
            res -= nums[i];
        }

        return res;
    }
}
```

# [598. 范围求和 II](https://leetcode-cn.com/problems/range-addition-ii/)

---

难度 `简单` | 标签 `数组` `数学` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给定一个初始元素全部为&nbsp;<strong>0</strong>，大小为 m*n 的矩阵&nbsp;<strong>M&nbsp;</strong>以及在&nbsp;<strong>M&nbsp;</strong>上的一系列更新操作。</p>
<p>操作用二维数组表示，其中的每个操作用一个含有两个<strong>正整数&nbsp;a</strong> 和 <strong>b</strong> 的数组表示，含义是将所有符合&nbsp;<strong>0 &lt;= i &lt; a</strong> 以及 <strong>0 &lt;= j &lt; b</strong> 的元素&nbsp;<strong>M[i][j]&nbsp;</strong>的值都<strong>增加 1</strong>。</p>
<p>在执行给定的一系列操作后，你需要返回矩阵中含有最大整数的元素个数。</p>
<p><strong>示例 1:</strong></p>
<pre><strong>输入:</strong> 
m = 3, n = 3
operations = [[2,2],[3,3]]
<strong>输出:</strong> 4
<strong>解释:</strong> 
初始状态, M = 
[[0, 0, 0],
 [0, 0, 0],
 [0, 0, 0]]
执行完操作 [2,2] 后, M = 
[[1, 1, 0],
 [1, 1, 0],
 [0, 0, 0]]
执行完操作 [3,3] 后, M = 
[[2, 2, 1],
 [2, 2, 1],
 [1, 1, 1]]
M 中最大的整数是 2, 而且 M 中有4个值为2的元素。因此返回 4。
</pre>
<p><strong>注意:</strong></p>
<ol>
	<li>m 和 n 的范围是&nbsp;[1,40000]。</li>
	<li>a 的范围是 [1,m]，b 的范围是 [1,n]。</li>
	<li>操作数目不超过 10000。</li>
</ol>
</section>

## My Solution

```csharp
public class Solution {
    public int MaxCount(int m, int n, int[][] ops) {
        int mina = m, minb = n;
        foreach (int[] op in ops) {
            mina = Math.Min(mina, op[0]);
            minb = Math.Min(minb, op[1]);
        }
        return mina * minb;
    }
}
```

# [299. 猜数字游戏](https://leetcode-cn.com/problems/bulls-and-cows/)

---

难度 `中等` | 标签 `哈希表` `字符串` `计数` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>你在和朋友一起玩 <a href="https://baike.baidu.com/item/%E7%8C%9C%E6%95%B0%E5%AD%97/83200?fromtitle=Bulls+and+Cows&amp;fromid=12003488&amp;fr=aladdin">猜数字（Bulls and Cows）</a>游戏，该游戏规则如下：</p>
<p>写出一个秘密数字，并请朋友猜这个数字是多少。朋友每猜测一次，你就会给他一个包含下述信息的提示：</p>
<ul>
	<li>猜测数字中有多少位属于数字和确切位置都猜对了（称为 "Bulls", 公牛），</li>
	<li>有多少位属于数字猜对了但是位置不对（称为 "Cows", 奶牛）。也就是说，这次猜测中有多少位非公牛数字可以通过重新排列转换成公牛数字。</li>
</ul>
<p>给你一个秘密数字&nbsp;<code>secret</code> 和朋友猜测的数字&nbsp;<code>guess</code> ，请你返回对朋友这次猜测的提示。</p>
<p>提示的格式为 <code>"xAyB"</code> ，<code>x</code> 是公牛个数， <code>y</code> 是奶牛个数，<code>A</code> 表示公牛，<code>B</code>&nbsp;表示奶牛。</p>
<p>请注意秘密数字和朋友猜测的数字都可能含有重复数字。</p>
<p>&nbsp;</p>
<p><strong>示例 1:</strong></p>
<pre><strong>输入:</strong> secret = "1807", guess = "7810"
<strong>输出:</strong> "1A3B"
<strong>解释:</strong> 数字和位置都对（公牛）用 '|' 连接，数字猜对位置不对（奶牛）的采用斜体加粗标识。
"1807"
  |
"<em><strong>7</strong></em>8<em><strong>10</strong></em>"</pre>
<p><strong>示例 2:</strong></p>
<pre><strong>输入:</strong> secret = "1123", guess = "0111"
<strong>输出:</strong> "1A1B"
<strong>解释: </strong>数字和位置都对（公牛）用 '|' 连接，数字猜对位置不对（奶牛）的采用斜体加粗标识。
"1123"        "1123"
  |      or     |
"01<em><strong>1</strong></em>1"        "011<em><strong>1</strong></em>"
注意，两个不匹配的 1 中，只有一个会算作奶牛（数字猜对位置不对）。通过重新排列非公牛数字，其中仅有一个 1 可以成为公牛数字。</pre>
<p><strong>示例 3：</strong></p>
<pre><strong>输入：</strong>secret = "1", guess = "0"
<strong>输出：</strong>"0A0B"
</pre>
<p><strong>示例 4：</strong></p>
<pre><strong>输入：</strong>secret = "1", guess = "1"
<strong>输出：</strong>"1A0B"
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>1 &lt;= secret.length, guess.length &lt;= 1000</code></li>
	<li><code>secret.length == guess.length</code></li>
	<li><code>secret</code> 和 <code>guess</code> 仅由数字组成</li>
</ul>
</section>

## My Solution

```csharp
public class Solution {
    public string GetHint(string secret, string guess) {
        int bulls = 0;
        int[] cntS = new int[10];
        int[] cntG = new int[10];
        for (int i = 0; i < secret.Length; ++i) {
            if (secret[i] == guess[i]) {
                ++bulls;
            } else {
                ++cntS[secret[i] - '0'];
                ++cntG[guess[i] - '0'];
            }
        }
        int cows = 0;
        for (int i = 0; i < 10; ++i) {
            cows += Math.Min(cntS[i], cntG[i]);
        }
        return bulls.ToString() + "A" + cows.ToString() + "B";
    }
}
```

# [495. 提莫攻击](https://leetcode-cn.com/problems/teemo-attacking/)

---

难度 `简单` | 标签 `数组` `模拟` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>在《英雄联盟》的世界中，有一个叫 “提莫” 的英雄。他的攻击可以让敌方英雄艾希（编者注：寒冰射手）进入中毒状态。</p>
<p>当提莫攻击艾希，艾希的中毒状态正好持续&nbsp;<code>duration</code> 秒。</p>
<p>正式地讲，提莫在 <code>t</code> 发起发起攻击意味着艾希在时间区间 <code>[t, t + duration - 1]</code>（含 <code>t</code> 和 <code>t + duration - 1</code>）处于中毒状态。如果提莫在中毒影响结束 <strong>前</strong> 再次攻击，中毒状态计时器将会 <strong>重置</strong> ，在新的攻击之后，中毒影响将会在 <code>duration</code> 秒后结束。</p>
<p>给你一个 <strong>非递减</strong> 的整数数组 <code>timeSeries</code> ，其中 <code>timeSeries[i]</code> 表示提莫在 <code>timeSeries[i]</code> 秒时对艾希发起攻击，以及一个表示中毒持续时间的整数 <code>duration</code> 。</p>
<p>返回艾希处于中毒状态的 <strong>总</strong> 秒数。</p>
&nbsp;
<p><strong>示例 1：</strong></p>
<pre><strong>输入：</strong>timeSeries = [1,4], duration = 2
<strong>输出：</strong>4
<strong>解释：</strong>提莫攻击对艾希的影响如下：
- 第 1 秒，提莫攻击艾希并使其立即中毒。中毒状态会维持 2 秒，即第 1 秒和第 2 秒。
- 第 4 秒，提莫再次攻击艾希，艾希中毒状态又持续 2 秒，即第 4 秒和第 5 秒。
艾希在第 1、2、4、5 秒处于中毒状态，所以总中毒秒数是 4 。</pre>
<p><strong>示例 2：</strong></p>
<pre><strong>输入：</strong>timeSeries = [1,2], duration = 2
<strong>输出：</strong>3
<strong>解释：</strong>提莫攻击对艾希的影响如下：
- 第 1 秒，提莫攻击艾希并使其立即中毒。中毒状态会维持 2 秒，即第 1 秒和第 2 秒。
- 第 2 秒，提莫再次攻击艾希，并重置中毒计时器，艾希中毒状态需要持续 2 秒，即第 2 秒和第 3 秒。
艾希在第 1、2、3 秒处于中毒状态，所以总中毒秒数是 3 。
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>1 &lt;= timeSeries.length &lt;= 10<sup>4</sup></code></li>
	<li><code>0 &lt;= timeSeries[i], duration &lt;= 10<sup>7</sup></code></li>
	<li><code>timeSeries</code> 按 <strong>非递减</strong> 顺序排列</li>
</ul>
</section>

## My Solution

```csharp
public class Solution {
    public int FindPoisonedDuration(int[] timeSeries, int duration) {
        int ans = 0;
        int expired = 0;
        for (int i = 0; i < timeSeries.Length; ++i) {
            if (timeSeries[i] >= expired) {
                ans += duration;
            } else {
                ans += timeSeries[i] + duration - expired;
            }
            expired = timeSeries[i] + duration;
        }
        return ans;
    }
}
```

# [629. K个逆序对数组](https://leetcode-cn.com/problems/k-inverse-pairs-array/)

---

难度 `困难` | 标签 `动态规划` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给出两个整数&nbsp;<code>n</code>&nbsp;和&nbsp;<code>k</code>，找出所有包含从&nbsp;<code>1</code>&nbsp;到&nbsp;<code>n</code>&nbsp;的数字，且恰好拥有&nbsp;<code>k</code>&nbsp;个逆序对的不同的数组的个数。</p>
<p>逆序对的定义如下：对于数组的第<code>i</code>个和第&nbsp;<code>j</code>个元素，如果满<code>i</code>&nbsp;&lt;&nbsp;<code>j</code>且&nbsp;<code>a[i]</code>&nbsp;&gt;&nbsp;<code>a[j]</code>，则其为一个逆序对；否则不是。</p>
<p>由于答案可能很大，只需要返回 答案 mod 10<sup>9</sup>&nbsp;+ 7 的值。</p>
<p><strong>示例 1:</strong></p>
<pre><strong>输入:</strong> n = 3, k = 0
<strong>输出:</strong> 1
<strong>解释:</strong> 
只有数组 [1,2,3] 包含了从1到3的整数并且正好拥有 0 个逆序对。
</pre>
<p><strong>示例 2:</strong></p>
<pre><strong>输入:</strong> n = 3, k = 1
<strong>输出:</strong> 2
<strong>解释:</strong> 
数组 [1,3,2] 和 [2,1,3] 都有 1 个逆序对。
</pre>
<p><strong>说明:</strong></p>
<ol>
	<li>&nbsp;<code>n</code>&nbsp;的范围是 [1, 1000] 并且 <code>k</code> 的范围是 [0, 1000]。</li>
</ol>
</section>

## My Solution



```csharp
public class Solution {
    public int KInversePairs(int n, int k) {
        const int MOD = 1000000007;
        int[,] f = new int[2, k + 1];
        f[0, 0] = 1;
        for (int i = 1; i <= n; ++i) {
            for (int j = 0; j <= k; ++j) {
                int cur = i & 1, prev = cur ^ 1;
                f[cur, j] = (j - 1 >= 0 ? f[cur, j - 1] : 0) - (j - i >= 0 ? f[prev, j - i] : 0) + f[prev, j];
                if (f[cur, j] >= MOD) {
                    f[cur, j] -= MOD;
                } else if (f[cur, j] < 0) {
                    f[cur, j] += MOD;
                }
            }
        }
        return f[n & 1, k];
    }
}

```

# [391. 完美矩形](https://leetcode-cn.com/problems/perfect-rectangle/)

---

难度 `困难` | 标签 `数组` `扫描线` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给你一个数组 <code>rectangles</code> ，其中 <code>rectangles[i] = [x<sub>i</sub>, y<sub>i</sub>, a<sub>i</sub>, b<sub>i</sub>]</code> 表示一个坐标轴平行的矩形。这个矩形的左下顶点是 <code>(x<sub>i</sub>, y<sub>i</sub>)</code> ，右上顶点是 <code>(a<sub>i</sub>, b<sub>i</sub>)</code> 。</p>
<p>如果所有矩形一起精确覆盖了某个矩形区域，则返回 <code>true</code> ；否则，返回 <code>false</code> 。</p>
&nbsp;
<p><strong>示例 1：</strong></p>
<img style="width: 300px; height: 294px;" src="https://assets.leetcode.com/uploads/2021/03/27/perectrec1-plane.jpg" alt="">
<pre><strong>输入：</strong>rectangles = [[1,1,3,3],[3,1,4,2],[3,2,4,4],[1,3,2,4],[2,3,3,4]]
<strong>输出：</strong>true
<strong>解释：</strong>5 个矩形一起可以精确地覆盖一个矩形区域。 
</pre>
<p><strong>示例 2：</strong></p>
<img style="width: 300px; height: 294px;" src="https://assets.leetcode.com/uploads/2021/03/27/perfectrec2-plane.jpg" alt="">
<pre><strong>输入：</strong>rectangles = [[1,1,2,3],[1,3,2,4],[3,1,4,2],[3,2,4,4]]
<strong>输出：</strong>false
<strong>解释：</strong>两个矩形之间有间隔，无法覆盖成一个矩形。</pre>
<p><strong>示例 3：</strong></p>
<img style="width: 300px; height: 294px;" src="https://assets.leetcode.com/uploads/2021/03/27/perfectrec3-plane.jpg" alt="">
<pre><strong>输入：</strong>rectangles = [[1,1,3,3],[3,1,4,2],[1,3,2,4],[3,2,4,4]]
<strong>输出：</strong>false
<strong>解释：</strong>图形顶端留有空缺，无法覆盖成一个矩形。</pre>
<p><strong>示例 4：</strong></p>
<img style="width: 300px; height: 294px;" src="https://assets.leetcode.com/uploads/2021/03/27/perfecrrec4-plane.jpg" alt="">
<pre><strong>输入：</strong>rectangles = [[1,1,3,3],[3,1,4,2],[1,3,2,4],[2,2,4,4]]
<strong>输出：</strong>false
<strong>解释：</strong>因为中间有相交区域，虽然形成了矩形，但不是精确覆盖。</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>1 &lt;= rectangles.length &lt;= 2 * 10<sup>4</sup></code></li>
	<li><code>rectangles[i].length == 4</code></li>
	<li><code>-10<sup>5</sup> &lt;= x<sub>i</sub>, y<sub>i</sub>, a<sub>i</sub>, b<sub>i</sub> &lt;= 10<sup>5</sup></code></li>
</ul>
</section>

## My Solution

```csharp
public bool IsRectangleCover(int[][] rectangles)
{
    int n = rectangles.Length;
    int[][] rs = new int[2 * n][];
    for (int i = 0, idx = 0; i < n; i++)
    {
        int[] re = rectangles[i];
        rs[idx++] = new int[] { re[0], re[1], re[3], 1 };
        rs[idx++] = new int[] { re[2], re[1], re[3], -1 };
    }

    Array.Sort(rs, ((a, b) =>
    {
        if (a[0] != b[0])
        {
            return a[0] - b[0];
        }

        return a[1] - b[1];
    }));
    n *= 2;
    List<int[]> l1 = new List<int[]>(), l2 = new List<int[]>();
    for (int l = 0; l < n;)
    {
        int r = l;
        l1.Clear();
        l2.Clear();
        while (r < n && rs[r][0] == rs[l][0])
            r++;
        for (int i = l; i < r; i++)
        {
            int[] cur = new[] { rs[i][1], rs[i][2] };
            List<int[]> list = rs[i][3] == 1 ? l1 : l2;
            if (list.Count <= 0)
            {
                list.Add(cur);
            }
            else
            {
                int[] prev = list[^1];
                if (cur[0] < prev[1])
                {
                    return false;
                }
                else if (cur[0] == prev[1])
                {
                    prev[1] = cur[1];
                }
                else
                {
                    list.Add(cur);
                }
            }
        }

        if (l > 0 && r < n)
        {
            if (l1.Count != l2.Count)
            {
                return false;
            }

            for (int i = 0; i < l1.Count; i++)
            {
                if (l1[i][0] == l2[i][0] && l1[i][1] == l2[i][1])
                {
                    continue;
                }

                return false;
            }
        }
        else
        {
            if (l1.Count + l2.Count != 1)
            {
                return false;
            }
        }

        l = r;
    }

    return true;
}
```

# [318. 最大单词长度乘积](https://leetcode-cn.com/problems/maximum-product-of-word-lengths/)

---

难度 `中等` | 标签 `位运算` `数组` `字符串` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给定一个字符串数组&nbsp;<code>words</code>，找到&nbsp;<code>length(word[i]) * length(word[j])</code>&nbsp;的最大值，并且这两个单词不含有公共字母。你可以认为每个单词只包含小写字母。如果不存在这样的两个单词，返回 0。</p>
<p>&nbsp;</p>
<p><strong>示例&nbsp;1:</strong></p>
<pre><strong>输入:</strong> <code>["abcw","baz","foo","bar","xtfn","abcdef"]</code>
<strong>输出: </strong><code>16 
<strong>解释:</strong> 这两个单词为<strong> </strong></code><code>"abcw", "xtfn"</code>。</pre>
<p><strong>示例 2:</strong></p>
<pre><strong>输入:</strong> <code>["a","ab","abc","d","cd","bcd","abcd"]</code>
<strong>输出: </strong><code>4 
<strong>解释: </strong></code>这两个单词为 <code>"ab", "cd"</code>。</pre>
<p><strong>示例 3:</strong></p>
<pre><strong>输入:</strong> <code>["a","aa","aaa","aaaa"]</code>
<strong>输出: </strong><code>0 
<strong>解释: </strong>不存在这样的两个单词。</code>
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>2 &lt;= words.length &lt;= 1000</code></li>
	<li><code>1 &lt;= words[i].length &lt;= 1000</code></li>
	<li><code>words[i]</code>&nbsp;仅包含小写字母</li>
</ul>
</section>

## My Solution

```csharp
public class Solution {
    public int MaxProduct(string[] words) {
        int res = 0;
        int[] masks = new int[words.Length];
        for(int i = 0; i < words.Length; i++)
        {
            for(int j = 0; j < words[i].Length; j++)
            {
                masks[i] |= 1 << (words[i][j] - 'a');
            }
        }
        for(int i = 0; i < words.Length; i++)
        {
            for(int j = i; j < words.Length; j++)
            {
                if((masks[i] & masks[j]) == 0)
                    res = Math.Max(res, words[i].Length * words[j].Length);
            }
        }
        return res;
    }
}
```

# [563. 二叉树的坡度](https://leetcode-cn.com/problems/binary-tree-tilt/)

---

难度 `简单` | 标签 `树` `深度优先搜索` `二叉树` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给定一个二叉树，计算 <strong>整个树 </strong>的坡度 。</p>
<p>一个树的<strong> 节点的坡度 </strong>定义即为，该节点左子树的节点之和和右子树节点之和的 <strong>差的绝对值 </strong>。如果没有左子树的话，左子树的节点之和为 0 ；没有右子树的话也是一样。空结点的坡度是 0 。</p>
<p><strong>整个树</strong> 的坡度就是其所有节点的坡度之和。</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<img style="width: 712px; height: 182px;" src="https://assets.leetcode.com/uploads/2020/10/20/tilt1.jpg" alt="">
<pre><strong>输入：</strong>root = [1,2,3]
<strong>输出：</strong>1
<strong>解释：</strong>
节点 2 的坡度：|0-0| = 0（没有子节点）
节点 3 的坡度：|0-0| = 0（没有子节点）
节点 1 的坡度：|2-3| = 1（左子树就是左子节点，所以和是 2 ；右子树就是右子节点，所以和是 3 ）
坡度总和：0 + 0 + 1 = 1
</pre>
<p><strong>示例 2：</strong></p>
<img style="width: 800px; height: 203px;" src="https://assets.leetcode.com/uploads/2020/10/20/tilt2.jpg" alt="">
<pre><strong>输入：</strong>root = [4,2,9,3,5,null,7]
<strong>输出：</strong>15
<strong>解释：</strong>
节点 3 的坡度：|0-0| = 0（没有子节点）
节点 5 的坡度：|0-0| = 0（没有子节点）
节点 7 的坡度：|0-0| = 0（没有子节点）
节点 2 的坡度：|3-5| = 2（左子树就是左子节点，所以和是 3 ；右子树就是右子节点，所以和是 5 ）
节点 9 的坡度：|0-7| = 7（没有左子树，所以和是 0 ；右子树正好是右子节点，所以和是 7 ）
节点 4 的坡度：|(3+5+2)-(9+7)| = |10-16| = 6（左子树值为 3、5 和 2 ，和是 10 ；右子树值为 9 和 7 ，和是 16 ）
坡度总和：0 + 0 + 0 + 2 + 7 + 6 = 15
</pre>
<p><strong>示例 3：</strong></p>
<img style="width: 800px; height: 293px;" src="https://assets.leetcode.com/uploads/2020/10/20/tilt3.jpg" alt="">
<pre><strong>输入：</strong>root = [21,7,14,1,1,2,2,3,3]
<strong>输出：</strong>9
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li>树中节点数目的范围在 <code>[0, 10<sup>4</sup>]</code> 内</li>
	<li><code>-1000 &lt;= Node.val &lt;= 1000</code></li>
</ul>
</section>

## My Solution

实现思路如下：

- 从根节点开始遍历，设当前节点为`node`
- 遍历`node`的左右节点`left`和`right`，将两者节点之和的差的绝对值累加到结果`res`
- 返回`node`作为根节点的树的节点之和`left + right + node.val`

```csharp
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     public int val;
 *     public TreeNode left;
 *     public TreeNode right;
 *     public TreeNode(int val=0, TreeNode left=null, TreeNode right=null) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
public class Solution {
    public int res = 0;
    public int FindTilt(TreeNode root) {
        DFS(root);
        return res;
    }
    public int DFS(TreeNode node)
    {
        if(node == null)
            return 0;
        int left = DFS(node.left);
        int right = DFS(node.right);
        res += Math.Abs(left - right);
        return left + right + node.val;
    }
}
```

# [397. 整数替换](https://leetcode-cn.com/problems/integer-replacement/)

---

难度 `中等` | 标签 `贪心` `位运算` `记忆化搜索` `动态规划` 

---

## Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给定一个正整数&nbsp;<code>n</code> ，你可以做如下操作：</p>
<ol>
	<li>如果&nbsp;<code>n</code><em>&nbsp;</em>是偶数，则用&nbsp;<code>n / 2</code>替换&nbsp;<code>n</code><em> </em>。</li>
	<li>如果&nbsp;<code>n</code><em>&nbsp;</em>是奇数，则可以用&nbsp;<code>n + 1</code>或<code>n - 1</code>替换&nbsp;<code>n</code> 。</li>
</ol>
<p><code>n</code><em>&nbsp;</em>变为 <code>1</code> 所需的最小替换次数是多少？</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<pre><strong>输入：</strong>n = 8
<strong>输出：</strong>3
<strong>解释：</strong>8 -&gt; 4 -&gt; 2 -&gt; 1
</pre>
<p><strong>示例 2：</strong></p>
<pre><strong>输入：</strong>n = 7
<strong>输出：</strong>4
<strong>解释：</strong>7 -&gt; 8 -&gt; 4 -&gt; 2 -&gt; 1
或 7 -&gt; 6 -&gt; 3 -&gt; 2 -&gt; 1
</pre>
<p><strong>示例 3：</strong></p>
<pre><strong>输入：</strong>n = 4
<strong>输出：</strong>2
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>1 &lt;= n &lt;= 2<sup>31</sup> - 1</code></li>
</ul>
</section>


## My Solution

**方法一**

- 当$n$为偶数时，我们只有唯一的办法将$n$替换为$\dfrac{n}{2}$
- 当n为奇数时，我们可以选择将$n$增加1或减少1。由于这两种方法都会将$n$变为偶数，那么下一步一定是除2，因此这里可以堪称使用两次操作，将$n$变为$\frac{n+1}{2}$或$\frac{n-1}{2}$

这里需要注意$n = 2^{31}-1$时，计算$n+1$会溢出，所以用$\lfloor \frac{n}{2} \rfloor + 1$和$\lfloor \frac{n}{2} \rfloor$分别计算$\frac{n+1}{2}$或$\frac{n-1}{2}$。

```csharp
public class Solution {
    Dictionary<int, int> memo = new Dictionary<int, int>();

    public int IntegerReplacement(int n) {
        if (n == 1) {
            return 0;
        }
        if (!memo.ContainsKey(n)) {
            if (n % 2 == 0) {
                memo.Add(n, 1 + IntegerReplacement(n / 2));
            } else {
                memo.Add(n, 2 + Math.Min(IntegerReplacement(n / 2), IntegerReplacement(n / 2 + 1)));
            }
        }
        return memo[n];
    }
}

```

**方法二**

上述两种做法，我们不可避免地在每个回合枚举了所有我们可以做的决策：主要体现在对$x$为奇数时的处理，我们总是处理$x + 1$和$x - 1$两种情况。

可以从二进制的角度进行分析：**给定起始值$n$，求解将其变为$(000...0001)_2$的最小步数**

- 对于偶数（二进制最低位为0）而言，我们只能进行一种操作，其作用是将当前值$x$进行一个单位的右移
- 对于奇数（二进制最低位为1）而言，我们能进行 **+1**和 **-1**操作，分析两种操作对$x$产生的影响
  - 对于 **+1**操作而言：最低位必然为1，此时如果次低位为0的话，**+1**相当于将最低位和次低位交换；如果次低位为1的话， **+1操作将将从最低位开始，连续一段的1进行消除（置零）**，并在连续一段的最高一位添加一个1
  - 对于 **-1**操作而言：最低位必然为1，其作用是将最低位的1进行消除

因此，对于$x$为奇数所能执行的两种操作， **+1**能够消除连续一段的1，只要次低位为1（存在连续端），应当优先使用  **+1**操作，但需要注意边界$x=1$的情况（此时选择 **-1**操作）

此方法个人觉得不如官方题解给的方案，这里给附上网址自行查阅[整数替换 - 整数替换 - 力扣（LeetCode） (leetcode-cn.com)](https://leetcode-cn.com/problems/integer-replacement/solution/zheng-shu-ti-huan-by-leetcode-solution-swef/)

```csharp
public class Solution {
    public int IntegerReplacement(int n) {
        int ans = 0;
        long tempN = n;
        while(tempN != 1)
        {
            if(tempN % 2 == 0)
                tempN >>= 1;
            else
            {
                if(tempN != 3 && (tempN >> 1 & 1) == 1)
                    tempN++;
                else
                    tempN--;
            }
            ans++;
        }
        return ans;
    }
}
```
