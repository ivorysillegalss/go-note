### [70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/)

假设你正在爬楼梯。需要 `n` 阶你才能到达楼顶。

每次你可以爬 `1` 或 `2` 个台阶。你有多少种不同的方法可以爬到楼顶呢？

**示例 1：**

> 输入：n = 2
> 输出：2
> 解释：有两种方法可以爬到楼顶。
>
> 1. 1 阶 + 1 阶
> 2. 2 阶



下标为0或1开始爬 表示0或1均为初始状态，由于题目求到底楼梯顶部最低花费，设顶部为**n阶**，可将花费分为两种情况：

1、**n-1阶+n阶花费**

2、**n-2阶+n-1阶的花费**

可得状态转移方程

```
dp[i]=min(dp[i−1],dp[i−2])+cost[i]
```

初步代码如下：

```java
    public int minCostClimbingStairs(int[] cost) {
        int[] dp = new int[1001];
        dp[0] = 0;
        dp[1] = 0;
        for (int i = 2; i < cost.length + 1; i++) {
            dp[i] = Math.min(cost[i - 1] + dp[i - 1],cost[i - 2] + dp[i - 2]);
        }
        return dp[cost.length];
    }
}
```

可在此基础上优化空间 ，**只需要使用到前一项或两项的值 不需要分配数组空间**

````java
class Solution {
        public int minCostClimbingStairs(int[] cost) {
            int dp1 = 0,dp2 = 0,dp3 = 0;
            for(int i = 2;i <= cost.length;++i){
                dp3 = Math.min(dp1 + cost[i - 2],dp2 + cost[i - 1]);
                dp1 = dp2;
                dp2 = dp3;
            }
            return dp3;
        }
    }

````







### [121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

给定一个数组 `prices` ，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。

你只能选择 **某一天** 买入这只股票，并选择在 **未来的某一个不同的日子** 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 `0` 。



题目需要按序最大的正差值 假如是在第`i`天  即维护第 `i` 项前最大的差值

dp代码如下 以[7,4,1,3,5,6] 为例

题目要求所维护的正差值 只需最大值的索引大于最小值

此时可将第 i 项分为两种情况：

1. 为最小值 标记最小值入min变量

2. 非最小值 以此时的min为基准 计算差值 与所维护的最大差值比较

   

```java
class Solution {
    public int maxProfit(int[] prices) {
        int maxProfit =0;
        int min=Integer.MAX_VALUE;
        for(int i : prices){
            if(i<min){
                min=i;
            }else if(i-min>maxProfit){
                maxProfit=i-min;
            }
        }
        return maxProfit;
    }
}
```





### [122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

给你一个整数数组 `prices` ，其中 `prices[i]` 表示某支股票第 `i` 天的价格。

在每一天，你可以决定是否购买和/或出售股票。你在任何时候 **最多** 只能持有 **一股** 股票。你也可以先购买，然后在 **同一天** 出售。

返回 *你能获得的 **最大** 利润* 。



`[7, 1, 5, 6]` 第二天买入，第四天卖出，收益最大（6-1），所以一般人可能会想，怎么判断不是第三天就卖出了呢? 这里就把问题复杂化了，根据题目的意思，当天卖出以后，当天还可以买入，所以其实可以第三天卖出，第三天买入，第四天又卖出（（5-1）+ （6-5） === 6 - 1）。所以算法可以直接简化为只要今天比昨天大，就卖出。

简而言之 只要有收益就可以加上 本质上就是简单的加减乘除 感觉更偏向于思维题多一点而不是dp

```java
class Solution {
    public int maxProfit(int[] prices) {
        int res = 0;
        for(int i = 1;i < prices.length;i ++){
            res += Math.max(prices[i] - prices[i - 1],0);
        }
        return res;
    }
}
```







### [62. 不同路径](https://leetcode.cn/problems/unique-paths/)

一个机器人位于一个 `m x n` 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

![image-20240206183928225](.\image\image-20240206183928225.png)



`dp[i][j]`表示从位置(i,j)到终点的路径个数

我们从终点开始，已知终点到终点只有一条路径，因此初始值: `d[m-1][n-1]=1`

- 考虑终点正上方：
  终点的正上方想要到达终点，只能选择向下走，因此路径个数仍然为正下方的路径个数

  `dp[i][n-1] = dp[i+1][n-1]`

- 考虑终点正左方：

  同理，终点的正左方想要到达终点，只能选择向右走，因此路径个数仍然为正右方的路径个数
  `dp[m-1][i] = dp[m-1][i+1]`

- 对于其他位置，当前路径数量等于 右方位置路径数 + 下方位置路径数 之和，即
  `dp[i][j] = dp[i+1][j]+ dp[i][j+1];`

```java
// 这个是反过来计算的代码
class Solution {
    public int uniquePaths(int m, int n) {
        int[][] f = new int[m][n];
        for(int i = 0;i < m;i ++){
            f[i][0] = 1;
        }
        for(int i = 0;i < n;i ++){
            f[0][i] = 1;
        }
        for(int i = 1;i < m;i ++){
            for(int j = 1;j < n;j ++){
                f[i][j] = f[i-1][j] + f[i][j - 1];
            }
        }
        return f[m - 1][n - 1];
    }
}
```









### [63. 不同路径 II](https://leetcode.cn/problems/unique-paths-ii/)

一个机器人位于一个 `m x n` 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish”）。

现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？

网格中的障碍物和空位置分别用 `1` 和 `0` 来表示。

![image-20240213141610706](.\image\image-20240213141610706.png)

> 输入：obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]]
> 输出：2
> 解释：3x3 网格的正中间有一个障碍物。
> 从左上角到右下角一共有 2 条不同的路径：
>
> 1. 向右 -> 向右 -> 向下 -> 向下
> 2. 向下 -> 向下 -> 向右 -> 向右



这一题和上一题相比只多了一个"障碍"的条件，基本的思路还是一样的，详细思路见上一题，终点正上方和左方的方法数只有一种，且此种方法是向右或下走，但是一旦之中存在障碍，就拦住了去路。

以上图为例

- 若终点正上方存在阻碍，其点和其上的点，均不存在能到达终点的路径 即若`obstacleGrid[1][2]`为1 则`dp[0][2]`和`dp[1][2]`均为0
- 若终点正左方存在阻碍，其点和其左的点，均不存在能到达终点的路径 即若`obstacleGrid[2][1]`为1 则`dp[1][1]`和`dp[0][1]`也均为0
- 对应其他的点，在使用状态转移方程赋值前，先对他进行判断`obstacleGrid[i][j]`是否为1(是否存在阻碍)就好，若为1则赋值其的方法数为0(没有能去到终点的方法)，除此之外，一般情况下的状态转移推导都是一样的 见代码

```java
class Solution {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int y = obstacleGrid.length;
        int x = obstacleGrid[0].length;
        int[][] dp = new int[y][x];
//        flag为标志 判断是否存在障碍
        boolean flag = false;
//        打表终点正上方和左方
        for (int i = x - 1; i >= 0; i--) {
            if (obstacleGrid[y - 1][i] == 1) {
                flag = true;
            }
            if (flag) {
                dp[y - 1][i] = 0;
            } else {
                dp[y - 1][i] = 1;
            }
        }
        flag = false;
        for (int i = y - 1; i >= 0; i--) {
            if (obstacleGrid[i][x - 1] == 1) {
                flag = true;
            }
            if (flag) {
                dp[i][x - 1] = 0;
            } else {
                dp[i][x - 1] = 1;
            }
        }
//        从终点开始往回进行dp推导 
        for (int i = x - 2; i >= 0; i--) {
            for (int j = y - 2; j >= 0; j--) {
                if (obstacleGrid[j][i] == 1) {
                    dp[j][i] = 0;
                } else {
                    dp[j][i] = dp[j][i + 1] + dp[j + 1][i];
                }
            }
        }
//        打完表之后 可以得到每一个点到终点的路径方法个数
        return dp[0][0];
    }
}
```







### [740. 删除并获得点数](https://leetcode.cn/problems/delete-and-earn/)

给你一个整数数组 `nums` ，你可以对它进行一些操作。

每次操作中，选择任意一个 `nums[i]` ，删除它并获得 `nums[i]` 的点数。之后，你必须删除 **所有** 等于 `nums[i] - 1` 和 `nums[i] + 1` 的元素。

开始你拥有 `0` 个点数。返回你能通过这些操作获得的最大点数。

**示例 1：**

> 输入：nums = [3,4,2]
> 输出：6
> 解释：
> 删除 4 获得 4 个点数，因此 3 也被删除。
> 之后，删除 2 获得 2 个点数。总共获得 6 个点数。

在打家劫舍中我们通过动态规划比较，是`i-1`家的最大数，还是`i-2`家 + 当前第i家的和的数量关系，也就得状态转移方程：

`dp[i] = Math.max(dp[i - 1],dp[i  -2] + nums[i])` 

这一题本质上也是一样的，我们需计算不同情况下的累加和。但是我们还没有**去掉某个点时的点数合集**，所以先算出去掉每一个点时所累加的值，就可以按照相同思路写出。而这个值，只需要遍历一次即可记录在一个辅助数组当中

整个方法的时间复杂度为`O(n²)` 空间复杂度为`O(n)`

```java
class Solution {
    public int deleteAndEarn(int[] nums) {
        int[] count = new int[10001]; // 由于nums[i]的范围是1到10000，所以创建大小为10001的数组
        int max = 0;
        for (int num : nums) {
            count[num] += num; // 直接累加值，而不是次数
            max = Math.max(max, num);
        }

        // 动态规划数组，dp[i]代表考虑到数字i时能获得的最大点数
        int[] dp = new int[max + 1];
        dp[1] = count[1];
        if (max > 1) {
            dp[2] = Math.max(count[1], count[2]);
        }

        for (int i = 3; i <= max; i++) {
            dp[i] = Math.max(dp[i - 1], dp[i - 2] + count[i]);
        }

        return dp[max]; // 返回考虑所有数字时的最大点数
    }
}
```







### [2369. 检查数组是否存在有效划分](https://leetcode.cn/problems/check-if-there-is-a-valid-partition-for-the-array/)

给你一个下标从 **0** 开始的整数数组 `nums` ，你必须将数组划分为一个或多个 **连续** 子数组。

如果获得的这些子数组中每个都能满足下述条件 **之一** ，则可以称其为数组的一种 **有效** 划分：

1. 子数组 **恰** 由 `2` 个相等元素组成，例如，子数组 `[2,2]` 。
2. 子数组 **恰** 由 `3` 个相等元素组成，例如，子数组 `[4,4,4]` 。
3. 子数组 **恰** 由 `3` 个连续递增元素组成，并且相邻元素之间的差值为 `1` 。例如，子数组 `[3,4,5]` ，但是子数组 `[1,3,5]` 不符合要求。

如果数组 **至少** 存在一种有效划分，返回 `true` ，否则，返回 `false` 。

**示例 1：**

> 输入：nums = [4,4,4,5,6]
> 输出：true
> 解释：数组可以划分成子数组 [4,4] 和 [4,5,6] 。
> 这是一种有效划分，所以返回 true 。

**示例 2：**

> 输入：nums = [1,1,1,2]
> 输出：false
> 解释：该数组不存在有效划分。



设数组长度为**n**  
标记前n项数组满足要求或否的布尔数组**f[]**

题目要求计算数组是否划分成合法子数组，按照动态规划的思想，可以将问题转变成**数组的前n项是否可划分成子数组**。讨论这个话题的时候，可以将**符合要求的情况**转化成三种子情况，三种情况满足其一即可：

- `nums[n] == nums[n - 1]` 末2项相等 且 `f[n - 1]`前n-2项 可有效划分
- `nums[n] == nums[n - 1] == nums[n - 2]` 末三项相等 且 `f[n - 2]`前n-3项可有效划分

- `nums[n] == nums[n - 1] + 1 == nums[n - 2] + 2` 末三项相等 且 `f[n - 2]` 前三项可有效划分

> 关于f[n]:
>
> 这里设置f[]的长度为**n + 1** 并且**f[0] = true**
> 其含义是第n项前的数组可否划分为子数组 eg:
>
> - f[0] = true 前0项默认为有效 便于后期dp
>
> - f[1] = false 前1项不可划分为有效数组
>
> - f[2] = true 前2项可划分成有效数组

代码如下

```java
class Solution {
    public boolean validPartition(int[] nums) {
        int n = nums.length;
        boolean[] f = new boolean[n + 1];
        f[0] = true;
        
        for(int i = 1;i < n;i ++){
            // 站在中间 实则是根据第n - 1项 判断第n + 1项此时的状态
            if(f[i - 1] && nums[i] == nums[i - 1]){
                f[i + 1] = true;
                continue;
            }
            if(i > 1 && f[i - 2] && 
               ((nums[i] == nums[i - 1] && nums[i] == nums[i - 2]) 
             || (nums[i] == nums[i - 1] + 1 && nums[i] == nums[i - 2] + 2))){
                f[i + 1] = true;
            }
        }
        return f[n];
    }
}
```

相似的，因为题目中最多只需要n-2时的状态，所以还可以在空间上进行优化，使用变量替换`f[]`

```java
class Solution {
    public boolean validPartition(int[] nums) {
        int n = nums.length;
        boolean a = true;
        boolean b = false;
//        直接计算当只有两个值时候的情况
        boolean c = nums.length > 1 && nums[0] == nums[1];
        boolean d = false;
//        从第三项开始计算dp
        for(int i = 2;i < n;i ++){
            d = false;
            // 站在中间 实则是根据第n - 1项 判断第n + 1项此时的状态
            if(b && nums[i] == nums[i - 1]){
                d = true;
//                这里不能continue了 会跳过状态刷新的语句
            }
            if(i > 1 && a && ((nums[i] == nums[i - 1] && nums[i] == nums[i - 2]) || (nums[i] == nums[i - 1] + 1 && nums[i] == nums[i - 2] + 2))){
                d = true;
            }
            a = b;
            b = c;
            c = d;
        }
        return c;
    }
}
```







### [834. 树中距离之和](https://leetcode.cn/problems/sum-of-distances-in-tree/)

给定一个无向、连通的树。树中有 `n` 个标记为 `0...n-1` 的节点以及 `n-1` 条边 。

给定整数 `n` 和数组 `edges` ， `edges[i] = [ai, bi]`表示树中的节点 `ai` 和 `bi` 之间有一条边。

返回长度为 `n` 的数组 `answer` ，其中 `answer[i]` 是树中第 `i` 个节点与所有其他节点之间的距离之和。



**示例 1:**

![img](https://assets.leetcode.com/uploads/2021/07/23/lc-sumdist1.jpg)

> 输入: n = 6, edges = [[0,1],[0,2],[2,3],[2,4],[2,5]]
> 输出: [8,12,6,10,10,10]
> 解释: 树如图所示。
> 我们可以计算出 dist(0,1) + dist(0,2) + dist(0,3) + dist(0,4) + dist(0,5) 
> 也就是 1 + 1 + 2 + 2 + 2 = 8。 因此，answer[0] = 8，以此类推。



初步思路是可以通过dfs + 暴力算出每一个点到根节点之间的距离，自顶向下进行递归，递归时通过记录该点父节点的指，防止回溯

```java
class Solution {

    private List<List<Integer>> graph; // 用于存储树的邻接表表示
    private int[] answer; // 存储每个节点到所有其他节点距离之和的数组

    public int[] sumOfDistancesInTree(int n, int[][] edges) {
        graph = new ArrayList<>();
        answer = new int[n];

        // 初始化图的邻接表
        for (int i = 0; i < n; i++) {
            graph.add(new ArrayList<>());
        }

        // 构建无向图
        for (int[] edge : edges) {
            graph.get(edge[0]).add(edge[1]);
            graph.get(edge[1]).add(edge[0]);
        }

        // 对每个节点执行DFS，计算到其他所有节点的距离之和
        for (int i = 0; i < n; i++) {
            answer[i] = dfs(i, -1, 0);
        }

        return answer;
    }

    // 将每一个节点作为根节点 进行dfs计算总距离
    // 每一个节点利用for循环遍历 对于节点的子节点亦是同理
    public int dfs(int child,int parent,int dis){
        // 这个sum则是当前节点与根据点已计算出的距离值 （父节点与根节点之间的距离综合）
        int sum = dis;
        // 遍历当前子节点的每一个子节点
        for(int n:graph.get(child)){
            // 题目按照索引来标记节点 通过判断子节点和父节点索引是否相同 排除回溯父节点
            if(n != parent){
                // 对子节点 的子节点进行递归 其距离是dis+1
                sum += dfs(n,child,dis + 1);
            }
        }
        return sum;
    }

}
```

这种方法首先需要对所有点(n个点)都进行一次dfs，而一次dfs需要遍历所有点，即遍历n次。所以计算出得时间复杂度为**O(n²)**，遗憾超时

于是可以在此基础上进行改进，以示例为例：

- 从0开始的距离总和是  `ans[0] = 1 + 1 + 2 + 2 + 2 = 8`

- 从2开始的距离总和是  `ans[1] = 2 + 1 + 1 + 1 + 1 = 6`

0~2的距离变化量为1，以其为基准进行分析，可以将所有的点分为两种情况：

- 目标点在2的子树中：距离**减少1**

- 目标点非2的子树中：距离**增加1**

这里的距离指的是**当根变化时，不同点到根的距离变化**

尝试将这个规律推广到一般情况中，可以推出：

若i~j的距离变化量为1，设为j子树点的数量为 `x`, 即非j子树点数量为 `n - x` 

- 非j子树的点到j距离增加1，总路径变化量增加 `(n - x) * 1`

- j子树的点到j距离减少1，总路径变化量减少 `x * 1`

到现在已经可以窥见dp的雏形了，但是现在并不知道单个点是否为对应根的子树，但题目所求的是总路径的数量，与具体节点并无直接关系，所以说我们可以将问题推广到**想要知道根变化时，总路径的变化量。**也就是只需要知道，当根节点变化时，所有节点改变带来的变化量即可。

综合上面所说的一般规律，我们可以得出当**根变化为相邻节点**时，其路径和数量变化为：
$$
j = i + (n - x) - x = i + n - 2 * x
$$
然后我们可以发现在公式中，我们所需要的是**初始路径和**、树的长度以及**对应根节点j子树的节点数量**

而初始路径和我们只需要通过**一次dfs**就可以得出，并且同时求出**每一个点子树中的节点数量**

一次dfs时间复杂度为n，而根据状态转移方程时间复杂度也为n

所以这种方法的时间复杂度为**O(n)**

整个过程中 ，我们使用的是dfs通过递归遍历单节点及其子节点，所以只需要考虑相邻节点的情况就可以了。

```java
class Solution {

    private List<List<Integer>> graph; // 用于存储树的邻接表表示
    private int[] answer; // 存储每个节点到所有其他节点距离之和的数组
    private int[] size;

    public int[] sumOfDistancesInTree(int n, int[][] edges) {
        graph = new ArrayList<>();
        answer = new int[n];
        size = new int[n];

        // 初始化图的邻接表
        for (int i = 0; i < n; i++) {
            graph.add(new ArrayList<>());
        }

        // 构建无向图
        for (int[] edge : edges) {
            graph.get(edge[0]).add(edge[1]);
            graph.get(edge[1]).add(edge[0]);
        }

        // dfs一次以算出从0开始到每一个点的距离 并且算出每一棵子树的大小
        answer[0] = dfs(0, -1, 0);

        // 完了之后 再次进行dfs来dp
        reroot(0,-1);

        return answer;
    }

    public int dfs(int child,int parent,int dis){
        int sum = dis;
        // 初始化自己的高度 作为父节点子树大小为1
        size[child] = 1;
        // 遍历当前子节点的每一个子节点
        for(int n:graph.get(child)){
            if(n != parent){
                sum += dfs(n,child,dis + 1);
                // 在确定了这个是子节点的子树之后 子节点累加它自身的子节点的子树的大小
                size[child] += size[n];
            }
        }
        return sum;
    }
    
    public void reroot(int child,int parent){
        for(int n : graph.get(child)){
//            避开父节点
            if(n != parent){
                // 状态转移方程
                answer[n] = answer[child] + graph.size() - 2 * size[n];
                // 递归到子节点再次计算路径和
                reroot(n,child);
            }
        }
    }
}
```







### [2581. 统计可能的树根数目](https://leetcode.cn/problems/count-number-of-possible-root-nodes/)

Alice 有一棵 `n` 个节点的树，节点编号为 `0` 到 `n - 1` 。树用一个长度为 `n - 1` 的二维整数数组 `edges` 表示，其中 `edges[i] = [ai, bi]` ，表示树中节点 `ai` 和 `bi` 之间有一条边。

Alice 想要 Bob 找到这棵树的根。她允许 Bob 对这棵树进行若干次 **猜测** 。每一次猜测，Bob 做如下事情：

- 选择两个 **不相等** 的整数 `u` 和 `v` ，且树中必须存在边 `[u, v]` 。
- Bob 猜测树中 `u` 是 `v` 的 **父节点** 。

Bob 的猜测用二维整数数组 `guesses` 表示，其中 `guesses[j] = [uj, vj]` 表示 Bob 猜 `uj` 是 `vj` 的父节点。

Alice 非常懒，她不想逐个回答 Bob 的猜测，只告诉 Bob 这些猜测里面 **至少** 有 `k` 个猜测的结果为 `true` 。

给你二维整数数组 `edges` ，Bob 的所有猜测和整数 `k` ，请你返回可能成为树根的 **节点数目** 。如果没有这样的树，则返回 `0`。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2022/12/19/ex-1.png)

> 输入：edges = [[0,1],[1,2],[1,3],[4,2]], guesses = [[1,3],[0,1],[1,0],[2,4]], k = 3
> 输出：3
> 解释：
> 根为节点 0 ，正确的猜测为 [1,3], [0,1], [2,4]
> 根为节点 1 ，正确的猜测为 [1,3], [1,0], [2,4]
> 根为节点 2 ，正确的猜测为 [1,3], [1,0], [2,4]
> 根为节点 3 ，正确的猜测为 [1,0], [2,4]
> 根为节点 4 ，正确的猜测为 [1,3], [1,0]
> 节点 0 ，1 或 2 为根时，可以得到 3 个正确的猜测。

这题的大体思路也是一样的，算出所有情况下的猜对次数，并且与k进行比较，筛选合法的根，也可以通过像上一题一样**dfs+暴力**算出，时间复杂度为**O(n²)**，也一样会超时。

可以汲取上一题**换根DP**的思路，首先算出根为0时候的正确次数，寻找规律，思考当根变化时，猜对次数的**如何变化**，又或者是，**处于何情况下的节点的，父子关系会随着根的变化而变化。**

经过观察可以知道，若节点i和j为相邻节点，并且在根节点由`i`变化为`j`的时候。就只有`i`和`j`的父子关系改变，其余父子关系均无变化。所以只有 `[i,j]` 和 `[j,i]` 这两个猜测的正确性变了，其余猜测的正确性不变。因为题目没有获取具体的 变化的父子节点位置的 需求。所以我们只需要在每一次变换根节点的同时，查找猜测中符合 `[i,j]` 和 `[j,i]` 关系的节点组合，并对猜测次数的变化做出改变就好了。

```java
class Solution {
    private List<List<Integer>> graph; // 用于存储树的邻接表表示
    private int[] answer; // 存储每个节点到所有其他节点距离之和的数组
    private HashSet<Long> s = new HashSet<>();
    private int ans;
    private int cnt0;
    private int k;
    public int rootCount(int[][] edges, int[][] guesses, int k) {

        this.k = k;

        graph = new ArrayList<>();

        // 初始化图的邻接表
        for (int i = 0; i < edges.length + 1; i++) {
            graph.add(new ArrayList<>());
        }

        // 构建无向图
        for (int[] edge : edges) {
            graph.get(edge[0]).add(edge[1]);
            graph.get(edge[1]).add(edge[0]);
        }
        
        // 将猜测的数据guess放入哈希表中
        for (int[] guess : guesses){
        //    通过位运算方便查询 相比hashmap优化空间
            s.add((long)guess[0] << 32 | guess[1]);
        }

        // 算出0作为根的时候的合法次数
        dfs(0,-1);

        // 以0为初始 dp算出其他的情况
        reroot(0,-1,cnt0);

        return ans;
    }
    
    // 先对根为0的情况单独dfs一次 算出初始的猜对次数
    private void dfs(int child,int parent){
        for(int c : graph.get(child)){
            if(c != parent){
                if(s.contains((long) child<< 32 | c)){
                    cnt0 ++;
                }
                dfs(c,child);
            }
        }
    }

    private void reroot(int child,int parent,int cnt){
        // 满足要求情况
        if(cnt >= k){
            ans ++;
        }
        // 换根dp主场 当切换根的时候 观察有没有[child,parent]或[parent,child]的情况 递归下去
        for(int c : graph.get(child)){
            if(c != parent){
                int cnt1 = cnt;
                if(s.contains((long) child<< 32 | c)) cnt1 --;
                if(s.contains((long) c << 32 | child)) cnt1 ++;
                reroot(c,child,cnt1);
            }
        }
    }
}
```

