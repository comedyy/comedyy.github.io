# 数学 相关

- [快速幂乘](./math/fast_pow.md)


math.atan2 vs math.atan的区别，都是为了求出角度。
var x = new float2(-1, 0);
Debug.LogError(math.atan(x.y / x.x));
Debug.LogError(math.atan2(x.y, x.x));
一个向量如果你已经求出 tan值了，直接用math.atan求出角度。否者你可能更喜欢使用math.atan2来求出具体位置。
atan2可以具体指出当前向量的角度[-pi, pi]。而atan只能指出当前atan值的角度。[-pi/2, pi/2]