# rsa
### 基本原理
1. 命名来自创造它的三个科学家名字。
2. 非对称加密算法，用于数据加密，或者签名验证。
3. 算法组成：
找到两个超级大的素数：p1，p2， 计算它的乘积 n = p1 * p2. 待加密的输入 m， 根据规则拿到的e跟d。 都是数字。c是秘文。
c = (m ** e) % n; /// 加密过程 (n，e) 是公钥
m = (c ** d) % n; /// 解密过程 (n, d) 是私钥， 实际情况私钥塞了更多东西。p1，p2， e，d都塞进去。

d其实是 e跟n计算出来的。但是如果无法获取到p1跟p2， 直接用e跟n拿去计算，需要非常长的时间。因为n，p1，p2都是非常大的整数。

这个算法是根据 
```https://blog.csdn.net/wjiabin/article/details/85228078?spm=1001.2101.3001.6650.5&utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-5.pc_relevant_antiscan&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-5.pc_relevant_antiscan&utm_relevant_index=10```

基本原理是 X(n) 函数，指的是 小于n的数有多少个跟互质。
如果n是质数： X(n) = n - 1。
如果n不是质数,并且 n = p * q; p, q 互质，那么X(n) = X(p) * X(q)
n是一个数的次方： 略

欧拉定理：
m，n互质， 那么(m ** X(n)) % n == 1。不证明。

模反元素：
e， x互质，那么可以找到一个数d，使得（e * d ） % x == 1;  或者是： e * d = x * k + 1; k >= 0

推理：
(m ** X(n)) % n == 1
=> (m ** K*X(n)) % n == 1**K == 1                   // 同时 k次方
=> (m ** K*X(n) + 1) % n == m * 1**K == m           // 同时乘以m
把 K*x(n)+1 替换为ed                                 // 需要满足 X(n),e 互质。 d可以从满足条件的列表中随意选取，似乎不同的d，都可以正常加密解密。计算结果都一样。
=> (m ** ed) % n == m                               // e 为选取的跟x(n) 互质的数
假设 c = (m ** e) % n // 加密过程
=> (c ** d) % n == m // 解密过程

理论上说 只要有n跟e，就可以算出d。为什么不好算出来？
d = (X(n) * K + 1) / e; X(n)太难算了。因为不知道p1 跟p2， 如果知道p1，p2的话, X(n)=X(p)*X(q) = (p - 1)*(q - 1)

### https中rsa的应用。
https客户端先发起请求，告诉服务器ssl版本。
服务器返回 公钥。 客户端使用公钥加密了 一个随机数，发送给服务器，服务器解析出随机数之后，用这个随机数加密http页面。


### openssl文件生成密钥
### 使用rsa来数字签名。
苹果的开发者账号上都有很多证书，证书里面挂着公钥。一般时候都有对应的私钥。
打包的时候，用私钥把 ipa的哈希值 用私钥加密，生成签名文件。 同时把公钥放到ipa包中。
苹果手机安装的时候，用公钥揭开签名文件，对比是否跟包的哈希值一致，来确定包是否有被修改过。


