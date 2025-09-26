# c# 中关键字， 修饰class, struct

### readonly
public readonly struct XXXX
{
    public readonly int a;
    public readonly int b;

    public XXXX(int v1, int v2) : this()
    {
        this.a = v1;
        this.b = v2;
    }
}

在创建XXX之后，就不能修改。所有的字段都需要是readonly。

### ref 关键字  c# 7.2引入的。
ref struct XXX
{
}
限制了XXX必须分配在stack上，它无法被存在堆中，
比如：
class A
{
    XXX B; // 编译报错。    
}
用法比如 Span<int> x = stackalloc int[5]

