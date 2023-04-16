### 题目描述

不降数，这种数字必须满足从左到右各位数字呈非下降关系，如 123，446。

指定一个整数闭区间 $[a,b]$，问这个区间内有多少个不降数。


#### 样例
输入样例
```
1 9
1 19
```
输出样例
```
9
18
```
样例解释
```
1~9全是一位数，所以都满足条件
1~19除了10其他都可以
```
----------
#### C++ 代码
```
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;
const int N = 32;
int f[N][11];
int init()
{
    for (int i = 0; i <= 9; i ++ ) f[1][i] = 1;
    for (int i = 0; i < N; i ++ ) f[i][9] = 1;
    for (int i = 2; i < N; i ++ )
        for (int j = 8; j >= 0; j -- )
            f[i][j] = f[i][j + 1] + f[i - 1][j];
}

int dp(int n)
{
    if (!n) return 1;
    vector<int> nums;
    while (n) nums.push_back(n % 10), n /= 10;
    int res = 0, last = 0; //res为结果，last为上一个位置留下来的条件
    for (int i = nums.size() - 1; i >= 0; i -- )
    {
        int x = nums[i];
        for (int j = last; j < x; j ++ ) res += f[i + 1][j];
        
        if (x < last) break;
        last = x;
        if (!i) res ++;
    }
    return res;
}

int main()
{
    init();
    int a, b;
    while (cin >> a >> b)
    {
        cout << dp(b) - dp(a - 1) << endl;
    }
    return 0;
}
```
#### 思路解释
和上一道题的思路是一样的，从高位开始我们是要一直增长的数，这个可以通过动态规划求出来，但是同样因为有范围限制在每一个位置上我们都需要讨论不同的分布：
1. 如果这一位我们放一个比当前数大的数字，那么后续无论怎么放置（指新的一段递增数列）都无所谓，可以通过我们预处理的结果直接计算。
2. 如果我们就放当下这个数字，那么在下一个位置时，我们依然需要保持这个最低的界限（也可以认为起点没有变）。

我们全程无非是在讨论如果现在数列开始上升会有几种情况，如果下一位开始上升会有几种情况，在下下位开始...所以理解难度比上一题小很多。

预处理：利用动态规划求解,很容易得到以下的递推关系：
$$f[i, j] = \sum^{9}_{k = j}f[i - 1][k]$$

做一个用处没有特别大的优化：

$$f[i, j] = \sum^{9}_{k = j}f[i - 1][k]$$

$$f[i, j + 1] = \sum^{9}_{k = j + 1}f[i - 1][k]$$

所以
$$f[i, j] = f[i][j + 1] + f[i - 1][j]$$

这里的话不可以拿$f[i, j - 1]$处理，因为这需要$f[i][0]$的初值，而这个显然是很难求的，我们只能知道$f[i][9] = 1$,所以要反着往回推。

```
int dp(int n)
{
    if (!n) return 1;
    vector<int> nums;
    while (n) nums.push_back(n % 10), n /= 10;
    int res = 0, last = 0; //res为结果，last为当前情况的下界
    for (int i = nums.size() - 1; i >= 0; i -- )
    {
        int x = nums[i];
        for (int j = last; j < x; j ++ ) res += f[i + 1][j]; //把哪些严格小于的直接一次性求完
        
        if (x < last) break; //这种情况说明在结束之前的步骤后在第i个数满足大于之前的数字和小于n发生矛盾
        last = x; //更新界限
        if (!i) res ++; //当我们循环到最后一个数时，已经没有下一个位置让我们考虑了，只有相等一种情况
    }
    return res;
}
```