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

