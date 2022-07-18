---
title: leetcode刷题记录
date: 2021-11-01 13:50:37
tags: 算法
categories: 计算机基础知识
---

# 搜索

## [1609. 奇偶树](https://leetcode-cn.com/problems/even-odd-tree/)

---

难度 `中等` | 标签 `树` `广度优先搜索` `二叉树` 

---

### Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>如果一棵二叉树满足下述几个条件，则可以称为 <strong>奇偶树</strong> ：</p>
<ul>
	<li>二叉树根节点所在层下标为 <code>0</code> ，根的子节点所在层下标为 <code>1</code> ，根的孙节点所在层下标为 <code>2</code> ，依此类推。</li>
	<li><strong>偶数下标</strong> 层上的所有节点的值都是 <strong>奇</strong> 整数，从左到右按顺序 <strong>严格递增</strong></li>
	<li><strong>奇数下标</strong> 层上的所有节点的值都是 <strong>偶</strong> 整数，从左到右按顺序 <strong>严格递减</strong></li>
</ul>
<p>给你二叉树的根节点，如果二叉树为 <strong>奇偶树 </strong>，则返回 <code>true</code> ，否则返回 <code>false</code> 。</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<p><strong><img style="height: 229px; width: 362px;" src="https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/10/04/sample_1_1966.png" alt=""></strong></p>
<pre><strong>输入：</strong>root = [1,10,4,3,null,7,9,12,8,6,null,null,2]
<strong>输出：</strong>true
<strong>解释：</strong>每一层的节点值分别是：
0 层：[1]
1 层：[10,4]
2 层：[3,7,9]
3 层：[12,8,6,2]
由于 0 层和 2 层上的节点值都是奇数且严格递增，而 1 层和 3 层上的节点值都是偶数且严格递减，因此这是一棵奇偶树。
</pre>
<p><strong>示例 2：</strong></p>
<p><strong><img style="height: 167px; width: 363px;" src="https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/10/04/sample_2_1966.png" alt=""></strong></p>
<pre><strong>输入：</strong>root = [5,4,2,3,3,7]
<strong>输出：</strong>false
<strong>解释：</strong>每一层的节点值分别是：
0 层：[5]
1 层：[4,2]
2 层：[3,3,7]
2 层上的节点值不满足严格递增的条件，所以这不是一棵奇偶树。
</pre>
<p><strong>示例 3：</strong></p>
<p><img style="height: 167px; width: 363px;" src="https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/10/04/sample_1_333_1966.png" alt=""></p>
<pre><strong>输入：</strong>root = [5,9,1,3,5,7]
<strong>输出：</strong>false
<strong>解释：</strong>1 层上的节点值应为偶数。
</pre>
<p><strong>示例 4：</strong></p>
<pre><strong>输入：</strong>root = [1]
<strong>输出：</strong>true
</pre>
<p><strong>示例 5：</strong></p>
<pre><strong>输入：</strong>root = [11,8,6,1,3,9,11,30,20,18,16,12,10,4,2,17]
<strong>输出：</strong>true
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li>树中节点数在范围 <code>[1, 10<sup>5</sup>]</code> 内</li>
	<li><code>1 &lt;= Node.val &lt;= 10<sup>6</sup></code></li>
</ul>
</section>

### My Solution

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
    public bool IsEvenOddTree(TreeNode root) {
        Queue<TreeNode> queue = new Queue<TreeNode>();
        queue.Enqueue(root);

        int layer = 0;
        while(queue.Count > 0)
        {
            int size = queue.Count;
            int prev = layer % 2 == 0 ? int.MinValue : int.MaxValue;
            for(int i = 0; i < size; i++)
            {
                TreeNode node = queue.Dequeue();
                int value = node.val;

                if(layer % 2 == value % 2)
                    return false;
                if((layer % 2 == 0 && value <= prev) || (layer % 2 == 1 && value >= prev))
                    return false;
                prev = value;
                if(node.left != null)
                    queue.Enqueue(node.left);
                if(node.right != null)
                    queue.Enqueue(node.right);
            }
            layer++;
        }
        return true;
    }
}
```

# 字符串

## [1629. 按键持续时间最长的键](https://leetcode-cn.com/problems/slowest-key/)

---

难度 `简单` | 标签 `数组` `字符串` 

---

### Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>LeetCode 设计了一款新式键盘，正在测试其可用性。测试人员将会点击一系列键（总计 <code>n</code> 个），每次一个。</p>
<p>给你一个长度为 <code>n</code> 的字符串 <code>keysPressed</code> ，其中 <code>keysPressed[i]</code> 表示测试序列中第 <code>i</code> 个被按下的键。<code>releaseTimes</code> 是一个升序排列的列表，其中 <code>releaseTimes[i]</code> 表示松开第 <code>i</code> 个键的时间。字符串和数组的 <strong>下标都从 0 开始</strong> 。第 <code>0</code> 个键在时间为 <code>0</code> 时被按下，接下来每个键都 <strong>恰好</strong> 在前一个键松开时被按下。</p>
<p>测试人员想要找出按键 <strong>持续时间最长</strong> 的键。第 <code>i</code><sup> </sup>次按键的持续时间为 <code>releaseTimes[i] - releaseTimes[i - 1]</code> ，第 <code>0</code> 次按键的持续时间为 <code>releaseTimes[0]</code> 。</p>
<p>注意，测试期间，同一个键可以在不同时刻被多次按下，而每次的持续时间都可能不同。</p>
<p>请返回按键 <strong>持续时间最长</strong> 的键，如果有多个这样的键，则返回 <strong>按字母顺序排列最大</strong> 的那个键。</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<pre><strong>输入：</strong>releaseTimes = [9,29,49,50], keysPressed = "cbcd"
<strong>输出：</strong>"c"
<strong>解释：</strong>按键顺序和持续时间如下：
按下 'c' ，持续时间 9（时间 0 按下，时间 9 松开）
按下 'b' ，持续时间 29 - 9 = 20（松开上一个键的时间 9 按下，时间 29 松开）
按下 'c' ，持续时间 49 - 29 = 20（松开上一个键的时间 29 按下，时间 49 松开）
按下 'd' ，持续时间 50 - 49 = 1（松开上一个键的时间 49 按下，时间 50 松开）
按键持续时间最长的键是 'b' 和 'c'（第二次按下时），持续时间都是 20
'c' 按字母顺序排列比 'b' 大，所以答案是 'c'
</pre>
<p><strong>示例 2：</strong></p>
<pre><strong>输入：</strong>releaseTimes = [12,23,36,46,62], keysPressed = "spuda"
<strong>输出：</strong>"a"
<strong>解释：</strong>按键顺序和持续时间如下：
按下 's' ，持续时间 12
按下 'p' ，持续时间 23 - 12 = 11
按下 'u' ，持续时间 36 - 23 = 13
按下 'd' ，持续时间 46 - 36 = 10
按下 'a' ，持续时间 62 - 46 = 16
按键持续时间最长的键是 'a' ，持续时间 16</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>releaseTimes.length == n</code></li>
	<li><code>keysPressed.length == n</code></li>
	<li><code>2 &lt;= n &lt;= 1000</code></li>
	<li><code>1 &lt;= releaseTimes[i] &lt;= 10<sup>9</sup></code></li>
	<li><code>releaseTimes[i] &lt; releaseTimes[i+1]</code></li>
	<li><code>keysPressed</code> 仅由小写英文字母组成</li>
</ul>
</section>

### My Solution

```csharp
public class Solution {
    public char SlowestKey(int[] releaseTimes, string keysPressed) {
        int n = releaseTimes.Length;
        char ans = keysPressed[0];
        int maxTime = releaseTimes[0];
        for (int i = 1; i < n; i++) {
            char key = keysPressed[i];
            int time = releaseTimes[i] - releaseTimes[i - 1];
            if (time > maxTime || (time == maxTime && key > ans)) {
                ans = key;
                maxTime = time;
            }
        }
        return ans;
    }
}


```

## [71. 简化路径](https://leetcode-cn.com/problems/simplify-path/)

---

难度 `中等` | 标签 `栈` `字符串` 

---

### Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给你一个字符串 <code>path</code> ，表示指向某一文件或目录的&nbsp;Unix 风格 <strong>绝对路径 </strong>（以 <code>'/'</code> 开头），请你将其转化为更加简洁的规范路径。</p>
<p class="MachineTrans-lang-zh-CN">在 Unix 风格的文件系统中，一个点（<code>.</code>）表示当前目录本身；此外，两个点 （<code>..</code>）&nbsp;表示将目录切换到上一级（指向父目录）；两者都可以是复杂相对路径的组成部分。任意多个连续的斜杠（即，<code>'//'</code>）都被视为单个斜杠 <code>'/'</code> 。 对于此问题，任何其他格式的点（例如，<code>'...'</code>）均被视为文件/目录名称。</p>
<p>请注意，返回的 <strong>规范路径</strong> 必须遵循下述格式：</p>
<ul>
	<li>始终以斜杠 <code>'/'</code> 开头。</li>
	<li>两个目录名之间必须只有一个斜杠 <code>'/'</code> 。</li>
	<li>最后一个目录名（如果存在）<strong>不能 </strong>以 <code>'/'</code> 结尾。</li>
	<li>此外，路径仅包含从根目录到目标文件或目录的路径上的目录（即，不含 <code>'.'</code> 或 <code>'..'</code>）。</li>
</ul>
<p>返回简化后得到的 <strong>规范路径</strong> 。</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<pre><strong>输入：</strong>path = "/home/"
<strong>输出：</strong>"/home"
<strong>解释：</strong>注意，最后一个目录名后面没有斜杠。 </pre>
<p><strong>示例 2：</strong></p>
<pre><strong>输入：</strong>path = "/../"
<strong>输出：</strong>"/"
<strong>解释：</strong>从根目录向上一级是不可行的，因为根目录是你可以到达的最高级。
</pre>
<p><strong>示例 3：</strong></p>
<pre><strong>输入：</strong>path = "/home//foo/"
<strong>输出：</strong>"/home/foo"
<strong>解释：</strong>在规范路径中，多个连续斜杠需要用一个斜杠替换。
</pre>
<p><strong>示例 4：</strong></p>
<pre><strong>输入：</strong>path = "/a/./b/../../c/"
<strong>输出：</strong>"/c"
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>1 &lt;= path.length &lt;= 3000</code></li>
	<li><code>path</code> 由英文字母，数字，<code>'.'</code>，<code>'/'</code> 或 <code>'_'</code> 组成。</li>
	<li><code>path</code> 是一个有效的 Unix 风格绝对路径。</li>
</ul>
</section>

### My Solution

```csharp
public class Solution {
    public string SimplifyPath(string path) {
        string[] names = path.Split("/");
        IList<string> stack = new List<string>();
        foreach (string name in names) {
            if ("..".Equals(name)) {
                if (stack.Count > 0) {
                    stack.RemoveAt(stack.Count - 1);
                }
            } else if (name.Length > 0 && !".".Equals(name)) {
                stack.Add(name);
            }
        }
        StringBuilder ans = new StringBuilder();
        if (stack.Count == 0) {
            ans.Append('/');
        } else {
            foreach (string name in stack) {
                ans.Append('/');
                ans.Append(name);
            }
        }
        return ans.ToString();
    }
}
```

## [1078. Bigram 分词](https://leetcode-cn.com/problems/occurrences-after-bigram/)

---

难度 `简单` | 标签 `字符串` 

---

### Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给出第一个词&nbsp;<code>first</code> 和第二个词&nbsp;<code>second</code>，考虑在某些文本&nbsp;<code>text</code>&nbsp;中可能以 <code>"first second third"</code> 形式出现的情况，其中&nbsp;<code>second</code>&nbsp;紧随&nbsp;<code>first</code>&nbsp;出现，<code>third</code>&nbsp;紧随&nbsp;<code>second</code>&nbsp;出现。</p>
<p>对于每种这样的情况，将第三个词 "<code>third</code>" 添加到答案中，并返回答案。</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<pre><strong>输入：</strong>text = "alice is a good girl she is a good student", first = "a", second = "good"
<strong>输出：</strong>["girl","student"]
</pre>
<p><strong>示例 2：</strong></p>
<pre><strong>输入：</strong>text = "we will we will rock you", first = "we", second = "will"
<strong>输出：</strong>["we","rock"]
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>1 &lt;= text.length &lt;= 1000</code></li>
	<li><code>text</code>&nbsp;由小写英文字母和空格组成</li>
	<li><code>text</code> 中的所有单词之间都由 <strong>单个空格字符</strong> 分隔</li>
	<li><code>1 &lt;= first.length, second.length &lt;= 10</code></li>
	<li><code>first</code> 和&nbsp;<code>second</code>&nbsp;由小写英文字母组成</li>
</ul>
</section>

### My Solution

```csharp
public class Solution {
    public string[] FindOcurrences(string text, string first, string second) {
        string[] words = text.Split(" ");
        IList<string> list = new List<string>();
        for (int i = 2; i < words.Length; i++) {
            if (words[i - 2].Equals(first) && words[i - 1].Equals(second)) {
                list.Add(words[i]);
            }
        }
        int size = list.Count;
        string[] ret = new string[size];
        for (int i = 0; i < size; i++) {
            ret[i] = list[i];
        }
        return ret;
    }
}

```

## [1576. 替换所有的问号](https://leetcode-cn.com/problems/replace-all-s-to-avoid-consecutive-repeating-characters/)

---

难度 `简单` | 标签 `字符串` 

---

### Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给你一个仅包含小写英文字母和 <code>'?'</code> 字符的字符串 <code>s</code>，请你将所有的 <code>'?'</code> 转换为若干小写字母，使最终的字符串不包含任何 <strong>连续重复</strong> 的字符。</p>
<p>注意：你 <strong>不能</strong> 修改非 <code>'?'</code> 字符。</p>
<p>题目测试用例保证 <strong>除</strong> <code>'?'</code> 字符 <strong>之外</strong>，不存在连续重复的字符。</p>
<p>在完成所有转换（可能无需转换）后返回最终的字符串。如果有多个解决方案，请返回其中任何一个。可以证明，在给定的约束条件下，答案总是存在的。</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<pre><strong>输入：</strong>s = "?zs"
<strong>输出：</strong>"azs"
<strong>解释：</strong>该示例共有 25 种解决方案，从 "azs" 到 "yzs" 都是符合题目要求的。只有 "z" 是无效的修改，因为字符串 "zzs" 中有连续重复的两个 'z' 。</pre>
<p><strong>示例 2：</strong></p>
<pre><strong>输入：</strong>s = "ubv?w"
<strong>输出：</strong>"ubvaw"
<strong>解释：</strong>该示例共有 24 种解决方案，只有替换成 "v" 和 "w" 不符合题目要求。因为 "ubvvw" 和 "ubvww" 都包含连续重复的字符。
</pre>
<p><strong>示例 3：</strong></p>
<pre><strong>输入：</strong>s = "j?qg??b"
<strong>输出：</strong>"jaqgacb"
</pre>
<p><strong>示例 4：</strong></p>
<pre><strong>输入：</strong>s = "??yw?ipkj?"
<strong>输出：</strong>"acywaipkja"
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li>
	<p><code>1 &lt;= s.length&nbsp;&lt;= 100</code></p>
	</li>
	<li>
	<p><code>s</code> 仅包含小写英文字母和 <code>'?'</code> 字符</p>
	</li>
</ul>
</section>

### My Solution

```csharp
public class Solution {
    public string ModifyString(string s) {
        int n = s.Length;
        char[] arr = s.ToCharArray();
        for (int i = 0; i < n; ++i) {
            if (arr[i] == '?') {
                for (char ch = 'a'; ch <= 'c'; ++ch) {
                    if ((i > 0 && arr[i - 1] == ch) || (i < n - 1 && arr[i + 1] == ch)) {
                        continue;
                    }
                    arr[i] = ch;
                    break;
                }
            }
        }
        return new String(arr);
    }
}

```

# 数学

## [390. 消除游戏](https://leetcode-cn.com/problems/elimination-game/)

---

难度 `中等` | 标签 `数学` 

---

### Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>列表 <code>arr</code> 由在范围 <code>[1, n]</code> 中的所有整数组成，并按严格递增排序。请你对 <code>arr</code> 应用下述算法：</p>
<div class="original__bRMd">
<div>
<ul>
	<li>从左到右，删除第一个数字，然后每隔一个数字删除一个，直到到达列表末尾。</li>
	<li>重复上面的步骤，但这次是从右到左。也就是，删除最右侧的数字，然后剩下的数字每隔一个删除一个。</li>
	<li>不断重复这两步，从左到右和从右到左交替进行，直到只剩下一个数字。</li>
</ul>
<p>给你整数 <code>n</code> ，返回 <code>arr</code> 最后剩下的数字。</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<pre><strong>输入：</strong>n = 9
<strong>输出：</strong>6
<strong>解释：</strong>
arr = [<strong><em>1</em></strong>, 2, <em><strong>3</strong></em>, 4, <em><strong>5</strong></em>, 6, <em><strong>7</strong></em>, 8, <em><strong>9</strong></em>]
arr = [2, <em><strong>4</strong></em>, 6, <em><strong>8</strong></em>]
arr = [<em><strong>2</strong></em>, 6]
arr = [6]
</pre>
<p><strong>示例 2：</strong></p>
<pre><strong>输入：</strong>n = 1
<strong>输出：</strong>1
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>1 &lt;= n &lt;= 10<sup>9</sup></code></li>
</ul>
</div>
</div>
</section>

### My Solution

```csharp
public class Solution {
    public int LastRemaining(int n) {
        int a1 = 1;
        int k = 0, cnt = n, step = 1;
        while(cnt > 1)
        {
            if(k % 2 == 0)
            {
                a1 = a1 + step;
            }
            else
            {
                a1 = (cnt % 2 == 0) ? a1 : a1 + step;
            }
            k++;
            cnt = cnt >> 1;
            step = step << 1;
        }
        return a1;
    }
}
```

## [1185. 一周中的第几天](https://leetcode-cn.com/problems/day-of-the-week/)

---

难度 `简单` | 标签 `数学` 

---

### Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给你一个日期，请你设计一个算法来判断它是对应一周中的哪一天。</p>
<p>输入为三个整数：<code>day</code>、<code>month</code> 和&nbsp;<code>year</code>，分别表示日、月、年。</p>
<p>您返回的结果必须是这几个值中的一个&nbsp;<code>{"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"}</code>。</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<pre><strong>输入：</strong>day = 31, month = 8, year = 2019
<strong>输出：</strong>"Saturday"
</pre>
<p><strong>示例 2：</strong></p>
<pre><strong>输入：</strong>day = 18, month = 7, year = 1999
<strong>输出：</strong>"Sunday"
</pre>
<p><strong>示例 3：</strong></p>
<pre><strong>输入：</strong>day = 15, month = 8, year = 1993
<strong>输出：</strong>"Sunday"
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li>给出的日期一定是在&nbsp;<code>1971</code> 到&nbsp;<code>2100</code>&nbsp;年之间的有效日期。</li>
</ul>
</section>

### My Solution

```csharp
public class Solution {
    public string DayOfTheWeek(int day, int month, int year) {
        string[] week = {"Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", 
"Sunday"};
        int[] monthDays = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};

        int ans = 3;
        for(int i = 1971; i < year; i++)
        {
            bool isLeap = (i % 4 == 0 && i % 100 != 0) || i % 400 == 0;
            ans += isLeap ? 366 : 365;
        }

        for(int i = 0; i < month - 1; i++)
        {
            ans += monthDays[i];
            if(i == 1 && ((year % 4 == 0 && year % 100 != 0) || year % 400 == 0)) ans += 1;
        }
        ans += day;
        return week[ans % 7];
    }
}
```

# 我也不知道是什么鬼题型

## [1995. 统计特殊四元组](https://leetcode-cn.com/problems/count-special-quadruplets/)

---

难度 `简单` | 标签 `数组` `枚举` 

---

### Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给你一个 <strong>下标从 0 开始</strong> 的整数数组 <code>nums</code> ，返回满足下述条件的 <strong>不同</strong> 四元组 <code>(a, b, c, d)</code> 的 <strong>数目</strong> ：</p>
<ul>
	<li><code>nums[a] + nums[b] + nums[c] == nums[d]</code> ，且</li>
	<li><code>a &lt; b &lt; c &lt; d</code></li>
</ul>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<pre><strong>输入：</strong>nums = [1,2,3,6]
<strong>输出：</strong>1
<strong>解释：</strong>满足要求的唯一一个四元组是 (0, 1, 2, 3) 因为 1 + 2 + 3 == 6 。
</pre>
<p><strong>示例 2：</strong></p>
<pre><strong>输入：</strong>nums = [3,3,6,4,5]
<strong>输出：</strong>0
<strong>解释：</strong>[3,3,6,4,5] 中不存在满足要求的四元组。
</pre>
<p><strong>示例 3：</strong></p>
<pre><strong>输入：</strong>nums = [1,1,1,3,5]
<strong>输出：</strong>4
<strong>解释：</strong>满足要求的 4 个四元组如下：
- (0, 1, 2, 3): 1 + 1 + 1 == 3
- (0, 1, 3, 4): 1 + 1 + 3 == 5
- (0, 2, 3, 4): 1 + 1 + 3 == 5
- (1, 2, 3, 4): 1 + 1 + 3 == 5
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>4 &lt;= nums.length &lt;= 50</code></li>
	<li><code>1 &lt;= nums[i] &lt;= 100</code></li>
</ul>
</section>

### My Solution

```csharp
public class Solution {
    public int CountQuadruplets(int[] nums) {

    }
}apublic class Solution {
    public int CountQuadruplets(int[] nums) {
        int n = nums.Length;
        int ans = 0;
        Dictionary<int, int> cnt = new Dictionary<int, int>();
        for (int c = n - 2; c >= 2; --c) {
            if (!cnt.ContainsKey(nums[c + 1])) {
                cnt.Add(nums[c + 1], 1);
            } else {
                ++cnt[nums[c + 1]];
            }
            for (int a = 0; a < c; ++a) {
                for (int b = a + 1; b < c; ++b) {
                    int sum = nums[a] + nums[b] + nums[c];
                    if (cnt.ContainsKey(sum)) {
                        ans += cnt[sum];
                    }
                }
            }
        }
        return ans;
    }
}

```

# 模拟

## [2022. 将一维数组转变成二维数组](https://leetcode-cn.com/problems/convert-1d-array-into-2d-array/)

---

难度 `简单` | 标签 `数组` `矩阵` `模拟` 

---

### Description

<style>
section pre{
    background-color: #eee;
    border: 1px solid #ddd;
    padding:10px;
    border-radius: 5px;
}
</style>
<section>
<p>给你一个下标从 <strong>0</strong>&nbsp;开始的一维整数数组&nbsp;<code>original</code>&nbsp;和两个整数&nbsp;<code>m</code>&nbsp;和&nbsp;&nbsp;<code>n</code>&nbsp;。你需要使用&nbsp;<code>original</code>&nbsp;中&nbsp;<strong>所有</strong>&nbsp;元素创建一个&nbsp;<code>m</code>&nbsp;行&nbsp;<code>n</code>&nbsp;列的二维数组。</p>
<p><code>original</code>&nbsp;中下标从 <code>0</code>&nbsp;到 <code>n - 1</code>&nbsp;（都 <strong>包含</strong> ）的元素构成二维数组的第一行，下标从 <code>n</code>&nbsp;到 <code>2 * n - 1</code>&nbsp;（都 <strong>包含</strong>&nbsp;）的元素构成二维数组的第二行，依此类推。</p>
<p>请你根据上述过程返回一个<em>&nbsp;</em><code>m x n</code>&nbsp;的二维数组。如果无法构成这样的二维数组，请你返回一个空的二维数组。</p>
<p>&nbsp;</p>
<p><strong>示例 1：</strong></p>
<img style="width: 500px; height: 174px;" src="https://assets.leetcode.com/uploads/2021/08/26/image-20210826114243-1.png">
<pre><b>输入：</b>original = [1,2,3,4], m = 2, n = 2
<b>输出：</b>[[1,2],[3,4]]
<strong>解释：
</strong>构造出的二维数组应该包含 2 行 2 列。
original 中第一个 n=2 的部分为 [1,2] ，构成二维数组的第一行。
original 中第二个 n=2 的部分为 [3,4] ，构成二维数组的第二行。
</pre>
<p><strong>示例 2：</strong></p>
<pre><b>输入：</b>original = [1,2,3], m = 1, n = 3
<b>输出：</b>[[1,2,3]]
<b>解释：</b>
构造出的二维数组应该包含 1 行 3 列。
将 original 中所有三个元素放入第一行中，构成要求的二维数组。
</pre>
<p><strong>示例 3：</strong></p>
<pre><b>输入：</b>original = [1,2], m = 1, n = 1
<b>输出：</b>[]
<strong>解释：
</strong>original 中有 2 个元素。
无法将 2 个元素放入到一个 1x1 的二维数组中，所以返回一个空的二维数组。
</pre>
<p><strong>示例 4：</strong></p>
<pre><b>输入：</b>original = [3], m = 1, n = 2
<b>输出：</b>[]
<strong>解释：</strong>
original 中只有 1 个元素。
无法将 1 个元素放满一个 1x2 的二维数组，所以返回一个空的二维数组。
</pre>
<p>&nbsp;</p>
<p><strong>提示：</strong></p>
<ul>
	<li><code>1 &lt;= original.length &lt;= 5 * 10<sup>4</sup></code></li>
	<li><code>1 &lt;= original[i] &lt;= 10<sup>5</sup></code></li>
	<li><code>1 &lt;= m, n &lt;= 4 * 10<sup>4</sup></code></li>
</ul>
</section>

### My Solution

```csharp
public class Solution {
    public int[][] Construct2DArray(int[] original, int m, int n) {
        if (original.Length != m * n) {
            return new int[0][];
        }
        int[][] ans = new int[m][];
        for (int i = 0; i < m; ++i) {
            ans[i] = new int[n];
        }
        for (int i = 0; i < original.Length; i += n) {
            Array.Copy(original, i, ans[i / n], 0, n);
        }
        return ans;
    }
}
```

