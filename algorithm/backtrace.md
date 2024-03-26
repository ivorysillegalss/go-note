# 子集型

### [78. 子集](https://leetcode.cn/problems/subsets/)

给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。

解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。

**示例 1：**

> 输入：nums = [1,2,3]
> 输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]

**示例 2：**

> 输入：nums = [0]
> 输出：[[],[0]]



此题是**子集型回溯**的经典模板题，通过观察可以发现。枚举一个数组的子集，本质上就是在**枚举每一个元素加入此子集与否**的情况。因此只需要分别对**加入**、**非加入**的情况进行深度优先遍历即可得到答案。我们dfs时维护一个**索引i**，记录此时遍历到的位置。

此处借用以[1,2]为例。

**写法1：**

![image-20240324145026329](./images\image-20240324145026329.png)

当索引为0(`dfs(0)`)时，可选择是否将`nums[0]`计入子集中，并将两种情况分开进行讨论。

- 若无需加入，可直接遍历下一元素`dfs(1)`
- 若需要加入，加入当前元素`nums[0]`，然后进行下一次遍历`dfs(1)`

然后以下一个元素继续进行**选or不选**的讨论

当每一次遍历之后，需要恢复现场，即`path.remove(path.size() - 1);`

> **为什么要恢复现场？** 
>
> 因为在对每一种情况回溯处理完之后，需要将当前的元素吐出来，才可以进行下一种情况的处理。即递归中的**归**

下为代码：

```java
class Solution {
    List<List<Integer>> ret;
    int n;
    List<Integer> path;
    int[] nums;
    public List<List<Integer>> subsets(int[] nums) {
        this.nums = nums;
        ret = new ArrayList<>();
        n = nums.length;
        path = new ArrayList<>();
        dfs(0);
        return ret;
    }
    public void dfs(int i){
        if(i == n){
            ret.add(new ArrayList<>(path));
            return;
        }
        dfs(i + 1);

        path.add(nums[i]);
        dfs(i + 1);

        path.remove(path.size() - 1);
    }
}
```

**写法2：**

写法1中，对每一个点都进行考虑是否加入选择。并对其进行分类讨论

与其对应，写法 2站在答案的角度来进行思考。每一次遍历的时候都通过for来遍历实现，并将每一次遍历中得到的结果都计入子集当中。

> 或许读者对他如何达到**不选**这一情况还存在疑问
>
> 此处我们仍以[1,2]为例子 
>
> 写法1中要获得`[2]`这一子集 按照遍历逻辑 有 
>
> 1. 不选择1
> 2. 选择2
>
> 通过选or不选的分类情况得到对应子集
>
> 写法2中通过for实现这一筛选的过程
>
> 1. `dfs(0)`中进入for循环
> 2. `j = 0`，此时需要先将`nums[j]`加入集合当中，即`path == [1]`，递归进行`dfs[1]`，加入`nums[1]`，得到`[1,2]`.
> 3. `j = 1`，此时加入的`nums[j] == 2`,直接单子集为2的情况加入答案中
>
> 也就是 ： 通过for的遍历 来实现选or不选效果

![image-20240324162559503](.\images\image-20240324162559503.png)

```java
class Solution {
    List<List<Integer>> ret;
    int n;
    List<Integer> path;
    int[] nums;
    public List<List<Integer>> subsets(int[] nums) {
        this.nums = nums;
        ret = new ArrayList<>();
        n = nums.length;
        path = new ArrayList<>();
        dfs(0);
        return ret;
    }
    public void dfs(int i){
        ret.add(new ArrayList<>(path));

        if(i == n) return;

        for(int j = i;j < n;j ++){
            path.add(nums[j]);
            dfs(j + 1);
            path.remove(path.size() - 1);
        }
    }
}
```







### [131. 分割回文串](https://leetcode.cn/problems/palindrome-partitioning/)

给你一个字符串 `s`，请你将 `s` 分割成一些子串，使每个子串都是 **回文串** 。返回 `s` 所有可能的分割方案。

**示例 1：**

> 输入：s = "aab"
> 输出：[["a","a","b"],["aa","b"]]

**示例 2：**

> 输入：s = "a"
> 输出：[["a"]]

这一题本质上和上一题是一样的，题目要求求出每一个回文串的方案。即每一个是回文串的字串，而上一题要求求出目标数组的所有子集。这么归类的话读者就能分析出其中共同点。

本题只是在上一题的基础上，加多一个判断，判断遍历出来的字符串是否回文，若回文则加入答案

以下代码：

```java
class Solution {
    String s;
    List<List<String>> ret;
    int n;
    List<String> path;
    public List<List<String>> partition(String s) {
        this.n = s.length();
        this.s = s;
        path = new ArrayList<>();
        ret = new ArrayList<>();
        dfs(0);
        return ret;
    }
    public boolean isP(int l,int r){
        while(l < r){
            if(s.charAt(l ++) != s.charAt(r --)){
                return false;
            }
        }
        return true;

    }
    public void dfs(int i){
        if(i == n){
            ret.add(new ArrayList(path));
            return;
        }
        for(int j = i;j < n;j++){
            if(isP(i,j)){
               path.add(s.substring(i, j + 1));
                dfs(j + 1);
                path.remove(path.size() - 1);
            }
        }
    }
}
```







# 组合型

### [216. 组合总和 III](https://leetcode.cn/problems/combination-sum-iii/)

找出所有相加之和为 `n` 的 `k` 个数的组合，且满足下列条件：

- 只使用数字1到9
- 每个数字 **最多使用一次** 

返回 *所有可能的有效组合的列表* 。该列表不能包含相同的组合两次，组合可以以任何顺序返回。

**示例 1:**

> 输入: k = 3, n = 7
> 输出: [[1,2,4]]
> 解释:
> 1 + 2 + 4 = 7
> 没有其他符合的组合了。

**示例 2:**

> 输入: k = 3, n = 9
> 输出: [[1,2,6], [1,3,5], [2,3,4]]
> 解释:
> 1 + 2 + 6 = 9
> 1 + 3 + 5 = 9
> 2 + 3 + 4 = 9
> 没有其他符合的组合了。

这一道题其实也可以理解成**子集型回溯 + 剪枝**

分析题目可得，合法的组合需要以下两个条件限定：

- 数量为`k`
- 和为`n`

同时要求数字由`1 ~ 9`中挑选，并且**不允许重复**

按照上方dfs的思路，我们可以由9往前回溯计算，每一次**需凑成的和与当前值相减**

转换一下思路，也就是说当出现以下条件时不符合要求：**（剪枝）**

设一共需凑成`k`个数，已凑成`a`个数，也就还需加入`k - a`个数

- 由于从9往前回溯计算，每次递归中的索引`i`即代表着还剩`i`个数可进行遍历，若此时`i < k - a`，从数量上不符合要求。
- 递归时，相减后的值<0，即**递归时计算和 > n**，从大小上不符合要求。
- 递归时，**剩余的待回溯的值总和 < 相减后的值**，即`0 ~ a`中的**所有值加起来仍 < n**，从大小上不符合要求。



处理好这四个递归时限定条件，可以按照`0~1`背包的思路写出代码如下：

写法1：通过答案

```java
class Solution {
    int k;
    List<List<Integer>> ret = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    public List<List<Integer>> combinationSum3(int k, int n) {
        this.k = k;
        dfs(9,n);
        return ret;
    }

    public void dfs(int i,int t){
        int d = k - path.size();
        // 表示促成k个数还需要d个数
        if(i < d) return;
        // 如果剩下的数的数量 小于 需要的数量
        if(t < 0 && t > (i * 2 - d + 1) * d / 2) return;
        // 如果剪掉之后剩的小于0 即代表减多了
        // 或 如果剩下的数的和小于所需的 代表怎么加都加不到  

        if(t == 0 && d == 0){
            // 如果满足和为n 且 数为k
            ret.add(new ArrayList<>(path));
        }

        // 回溯
        for(int j = i;j > 0;--j){
            path.add(j);
            // 类似0~1背包回溯思路
            dfs(j - 1,t - j);
            path.remove(path.size() - 1);
        }
    }

}
```

写法2：通过输入

```java
class Solution {
    int k;
    List<List<Integer>> ret = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    public List<List<Integer>> combinationSum3(int k, int n) {
        this.k = k;
        dfs(9,n);
        return ret;
    }

    public void dfs(int i,int t){
        int d = k - path.size();
        if(i < d) return;
        if(t < 0 || t > (i * 2 - d + 1) * d / 2) return;
        if(t == 0 && d == 0){
            ret.add(new ArrayList<>(path));
            return;
        }
        // 不选这个数
        if(i > d) dfs(i - 1,t);

        path.add(i);
        dfs(i - 1,t - i);

        path.remove(path.size() - 1);
    }

}
```

不难看出，两种写法虽然回溯的思路不一样，但是他们在写法上都是相似的。

1. 判断各种剪枝条件
2. 判断是否为合法所求元素，即`归`的边界条件
3. 根据题意，思考每一次`递`时需要变化的条件。





### [22. 括号生成](https://leetcode.cn/problems/generate-parentheses/)

数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。

**示例 1：**

> 输入：n = 3
> 输出：["((()))","(()())","(())()","()(())","()()()"]

**示例 2：**

> 输入：n = 1
> 输出：["()"]

这一题本质上也是回溯的问题，如果使用**选或不选**的思路来看，这一题就非常易懂了。

由题意，我们可以知道符合要求的要求有：

- 左括号与右括号相配对
- 递归时，左括号的个数一定大于等于右括号

相反的，我们可知剪枝条件 & 限定条件有：

- 右括号 >  左括号，不可能组成合法序列
- 左括号和右括号数量大于n （保证数量范围的情况下，不可能会对应相同数量的相反括号）

明确了这一条件之后，我们即可写出如下代码：

这里直接把左右括号的数量初始化为n，并且在dfs中动态改变括号个数

```java
class Solution {
    List<String> ret = new ArrayList<>();
    StringBuilder sb = new StringBuilder();

    public List<String> generateParenthesis(int n) {
        dfs(n, n);
        return ret;
    }

    public void dfs(int l, int r) {
        if (l > r) { // 保证每时每刻左括号数量不少于右括号
            return;
        }
        if (l == 0 && r == 0) { // 当所有括号都正确添加
            ret.add(sb.toString());
            return;
        }
        if (l > 0) {
            sb.append('('); // 尝试添加左括号
            dfs(l - 1, r);
            sb.setLength(sb.length() - 1); // 回溯
        }
        if (r > 0) {
            sb.append(')'); // 尝试添加右括号
            dfs(l, r - 1);
            sb.setLength(sb.length() - 1); // 回溯
        }
    }
}
```
