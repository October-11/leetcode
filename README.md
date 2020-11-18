# leetcode

 记录刷题过程中的一些值得注意的知识点和经典题

[TOC]



***

## 2020/11/06	根据数字二进制下1的数目排序

给你一个整数数组 arr 。请你将数组中的元素按照其二进制表示中数字 1 的数目升序排序。如果存在多个数字二进制中 1 的数目相同，则必须将它们按照数值大小升序排列。请你返回排序后的数组。

```c++
示例1：

输入：arr = [0,1,2,3,4,5,6,7,8]
输出：[0,1,2,4,8,3,5,6,7]
解释：[0] 是唯一一个有 0 个 1 的数。
[1,2,4,8] 都有 1 个 1 。
[3,5,6] 有 2 个 1 。
[7] 有 3 个 1 。
按照 1 的个数排序得到的结果数组为 [0,1,2,4,8,3,5,6,7]

示例2：

输入：arr = [1024,512,256,128,64,32,16,8,4,2,1]
输出：[1,2,4,8,16,32,64,128,256,512,1024]
解释：数组中所有整数二进制下都只有 1 个 1 ，所以你需要按照数值大小将它们排序。
```

***

代码实现：

```c++
class Solution {
public:
    static int cal(int a) {
        int count = 0;
        while (a) {
            count += (a % 2);
            a /= 2;
        }
        return count;
    }
    
    static bool compare(int a, int b) {
        if (cal(a) > cal(b)) {
            return false;
        }
        else if (cal(a) == cal(b)) {
            return a > b ? false : true;
        }
        else {
            return true;
        }
    }
    
    vector<int> sortBybits(vector<int>& arr) {
 		sort(arr.begin(), arr.end(), conmpare);
        return arr;
    }
}
```

***

这道题本身很简单，只需要改写sort函数的比较函数重新排序就好。但是在写的过程中遇到了报错的问题`invalid use of non-static member function`

经过查询可知，在类中写比较函数时，compare需要被定义成static类型。原因在于其实我们写参数compare时，是把函数名作为实参传递给了sort函数，而sort函数内部是用一个函数指针去调用这个函数的，我们知道class普通类成员函数需要通过`对象名.compare()`来调用，而sort()函数早就定义好了，那个时候哪知道你定义的是什么对象，所以内部是直接compare()`的，那你不加static时，去让sort()直接用``compare()`当然会报错。

**顺便复习一下几个基础知识点：**

### compare和static的用法

- `compare(int a, int b)`是一个布尔类型的函数，当`a > b`，返回true时，表示数据降序排列。当`a < b`，返回true时，表示数据升序排列

- `static`的用法
  - 在全局变量前加上关键字static，全局变量就定义成一个全局静态变量。存储在静态存储区中。全局静态变量在声明他的文件之外是不可见的，准确地说是从定义之处开始，到文件结尾。
  - 在局部变量之前加上关键字static，局部变量就成为一个局部静态变量。存储在静态存储区中。表示当局部静态变量离开作用域后，并没有销毁，而是仍然驻留在内存当中，只不过我们不能再对它进行访问，直到该函数再次被调用，并且值不变；
  - 在函数返回类型前加static，函数就定义为静态函数。函数的定义和声明在默认情况下都是extern的，但静态函数只是在声明他的文件当中可见，不能被其他文件所用。
  - 在类中，静态成员可以实现多个对象之间的数据共享，并且使用静态数据成员还不会破坏隐藏的原则，即保证了安全性。因此，静态成员是类的所有对象中共享的成员，而不是某个对象的成员。对多个对象来说，静态数据成员只存储一处，供所有对象共用。
  - 静态成员函数和静态数据成员一样，它们都属于类的静态成员，它们都不是对象成员。因此，对静态成员的引用不需要用对象名。在静态成员函数的实现中不能直接引用类中说明的非静态成员，可以引用类中说明的静态成员（这点非常重要）。如果静态成员函数中要引用非静态成员时，可通过对象来引用。从中可看出，调用静态成员函数使用如下格式：<类名>::<静态成员函数名>(<参数表>);

***

## **2020/11/11**	股票问题

前两天做题做到了经典的股票问题，股票问题有好几个变种题，正好股票题都是用动态规划做，我发现自己动态规划用的还是不好，因此既当作总结，也当作练手了。

动态规划的实现步骤：

- 状态定义
- 状态转移方程
- 起始条件

***

#### [121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

给定一个数组，它的第 *i* 个元素是一支给定股票第 *i* 天的价格。如果你最多只允许完成一笔交易（即买入和卖出一支股票一次），设计一个算法来计算你所能获取的最大利润。

注意：你不能在买入股票前卖出股票。

 示例 1:

```
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
```

**解题思路：**

- **状态定义**：设dp[i]表示第i天所取得的最大利润，然而当前的持股情况可能分为两种情况：当天持股或者当天不持股，因此可以使用一个二维数组来进行状态定义	`dp[i][0]`表示第i天不持股的情况,`dp[i][1]`表示第i天持股的情况

- **状态转移方程:**

  ```c++
  //第i天不持股可能分为两种情况，第i - 1天就不持股或者第i - 1天持股但是第i天卖出去了
  dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);	
  
  //第i天持股可能分为两种情况，第i - 1天就持股了或者之前一直都没有持股，第i天的时候买入了
  dp[i][1] = max(dp[i - 1][1], 0 - prices[i]);
  ```

- **起始条件：**第一天分为两种情况，第一天不买股票`dp[0][0] `= 0或者第一天买了股票`dp[0][1]= - prices[0]`

**代码实现：**

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if (prices.empty()) {
            return 0;
        }
        int n = prices.size();
        int dp[n + 1][2];
        dp[0][0] = 0;
        dp[0][1] = 0 - prices[0];
        for (int i = 1; i < n; i++) {
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
            dp[i][1] = max(dp[i - 1][1], 0 - prices[i]);
        }
        return dp[n - 1][0];
    }
};
```

***

#### [122. 买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。

注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。



示例 1:

```
输入: [7,1,5,3,6,4]
输出: 7
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。
```

**解题思路：**

- **状态定义**：设dp[i]表示第i天所取得的最大利润，然而当前的持股情况可能分为两种情况：当天持股或者当天不持股，因此可以使用一个二维数组来进行状态定义	`dp[i][0]`表示第i天不持股的情况,`dp[i][1]`表示第i天持股的情况

- **状态转移方程:**

  ```c++
  //第i天不持股可能分为两种情况，第i - 1天就不持股或者第i - 1天持股但是第i天卖出去了
  dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);	
  
  //第i天持股可能分为两种情况，第i - 1天就持股了或者之前卖出了，第i天的时候又买入了
  dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
  ```

- **起始条件：**第一天分为两种情况，第一天不买股票`dp[0][0] `= 0或者第一天买了股票`dp[0][1]= - prices[0]`

**代码实现：**

```c++
class Solution {
public:
    int MaxProfit(vector<int>& prices) {
        if (prices.size() == 0 || prices.size() == 1) {
            return 0;
        }
        int dp[prices.size() + 1][2];
        dp[0][0] = 0;
        dp[0][1] = 0 - prices[0];
        for (int i = 1; i < prices.size(); i++) {
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
        }
        return dp[prices.size() - 1][0];
    }
}
```

***

#### [123. 买卖股票的最佳时机 III](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)

给定一个数组，它的第 i 个元素是一支给定的股票在第 i 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 两笔 交易。

注意: 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

```c++
示例 1:

输入: [3,3,5,0,0,3,1,4]
输出: 6
解释: 在第 4 天（股票价格 = 0）的时候买入，在第 6 天（股票价格 = 3）的时候卖出，这笔交易所能获得利润 = 3-0 = 3 。
     随后，在第 7 天（股票价格 = 1）的时候买入，在第 8 天 （股票价格 = 4）的时候卖出，这笔交易所能获得利润 = 4-1 = 3 。
```

**解题思路：**

- **状态定义**：

  设dp[i]表示第i天所取得的最大利润，然而当前的持股情况可能分为两种情况：当天持股或者当天不持股，因此可以使用一个二维数组来进行状态定义	`dp[i][0]`表示第i天不持股的情况,`dp[i][1]`表示第i天持股的情况。

  本体与前两题不同得地方在于本题设置了次数限制，即最多只能购买卖出两次，因此，如果需要继续使用动态规划则需要对原本得状态定义添加条件：`dp[i][k][0],dp[i][k][1]`，k表示第i天时已经购买过得次数，注意此时定义购买时算一次。因此，相应得状态转移方程也需要变换。

- **状态转移方程:**

  ```c++
  //第i天不持股可能分为两种情况，第i - 1天就不持股或者第i - 1天持股但是第i天卖出去了,同时需要计算当k = 1和k = 2是的情况
  dp[i][k][0] = max(dp[i - 1][k][0], dp[i - 1][k][1] + prices[i]);	
  
  //第i天持股可能分为两种情况，第i - 1天就持股了或者之前卖出了，第i天的时候又买入了。
  //因为上一次已经卖出了，所以上一次是dp[i - 1][k - 1][0],因为本题按照买入的时候算做一次
  dp[i][k][1] = max(dp[i - 1][k][1], dp[i - 1][k - 1][0] - prices[i]);
  ```

- **起始条件：**第一天分为两种情况，第一天不买股票`dp[0][k][0] `= 0或者第一天买了股票`dp[0][k][1]= - prices[0]`,`k = 0,1,2`;

**代码实现：**

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if (prices.size() == 0 || prices.size() == 1) {
            return 0;
        }
        int n = prices.size();
        int K = 3;                          //存放已经交易了几次，最多两次
        int dp[n + 1][K][2];
        for (int i = 0; i < K; i++) {
            dp[0][i][0] = 0;
            dp[0][i][1] = 0 - prices[0];
        }
        for (int i = 1; i < n; i++) {
            for (int j = 1; j < K; j++) {
                dp[i][j][0] = max(dp[i - 1][j][0], dp[i - 1][j][1] + prices[i]);
                dp[i][j][1] = max(dp[i - 1][j][1], dp[i - 1][j - 1][0] - prices[i]);
            }
        }
        return dp[n - 1][K - 1][0];
    }
};
```



***

## 2020/11/18 [134. 加油站](https://leetcode-cn.com/problems/gas-station/)

难度中等451收藏分享切换为英文接收动态反馈

在一条环路上有 *N* 个加油站，其中第 *i* 个加油站有汽油 `gas[i]` 升。

你有一辆油箱容量无限的的汽车，从第 *i* 个加油站开往第 *i+1* 个加油站需要消耗汽油 `cost[i]` 升。你从其中的一个加油站出发，开始时油箱为空。

如果你可以绕环路行驶一周，则返回出发时加油站的编号，否则返回 -1。

**说明:** 

- 如果题目有解，该答案即为唯一答案。
- 输入数组均为非空数组，且长度相同。
- 输入数组中的元素均为非负数。

**示例 1:**

```
输入: 
gas  = [1,2,3,4,5]
cost = [3,4,5,1,2]

输出: 3

解释:
从 3 号加油站(索引为 3 处)出发，可获得 4 升汽油。此时油箱有 = 0 + 4 = 4 升汽油
开往 4 号加油站，此时油箱有 4 - 1 + 5 = 8 升汽油
开往 0 号加油站，此时油箱有 8 - 2 + 1 = 7 升汽油
开往 1 号加油站，此时油箱有 7 - 3 + 2 = 6 升汽油
开往 2 号加油站，此时油箱有 6 - 4 + 3 = 5 升汽油
开往 3 号加油站，你需要消
```

```c++
/**
 * 用一次遍历，遍历每个点，看是否能够绕环行驶。
 * start用于表示当前出发点
 * total表示需要消耗的总油量
 * cur表示当前加油站到下一个加油站所需要消耗的油量
 * 从第0个加油站开始计算消耗的总油量和当前需要的油量，如果当前需要的油量<0，说明无法到达下一个加油站
 * 则重置当前的cur，并将start + 1 ，重新开始计算。此时total继续叠加每一次所需要的油量。如果有一个点存在
 * 可以绕环行驶回到原点，那么total的值在全部遍历以后一定会 >= 0；
**/

class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int start = 0, total = 0, cur = 0;
        for (int i = 0; i < gas.size(); i++) {
            total += gas[i] - cost[i];
            cur += gas[i] - cost[i];
            if (cur < 0) {
                cur = 0;
                start = i + 1;
            }
        }
        return total >= 0 ? start : -1;
    }
};
```

