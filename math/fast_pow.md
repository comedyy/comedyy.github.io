# 快速幂

## 计算x的y次方
- x循环乘y-1次，复杂度O(y)
- 每次循环x = x * x, 那么复杂度就只要 O(log2(y))

代码实现：
```
int FastPow(int x, int y)
{
    int ret = 1;
    int b = y;
    int m = x;

    while (b > 0)
    {
        if((b & 1) != 0)
        {
            ret *= m;
        }

        m = m * m;
        b /= 2;
    }

    return ret;
}
```

## 扩展到矩阵
```
fn(1) = 1;
fn(2) = 1;
f(n) = f(n-1) + fn(n-2)
求 f(1000) = ？
```

构造矩阵 
$$
M=\left[
 \begin{matrix}
   1 & 1\\\\
   1 & 0\\\\
  \end{matrix} 
  \right]
$$

$$
\left[
 \begin{matrix}
   fn(n+2)\\\\
   fn(n+1)\\\\
  \end{matrix} 
  \right] = M^N * \left[
 \begin{matrix}
   fn(2)\\\\
   fn(1)\\\\
  \end{matrix} 
  \right]
$$

$M^N$可以通过快速幂乘快速算出。就可以快速计算斐波那契数列。

