## IEnumerator && IEnumerable

IEnumerable ： 可迭代的，有迭代器
IEnumerator ： 迭代器

public interface IEnumerable<out T> : IEnumerable
{
    IEnumerator<T> GetEnumerator();
}

public interface IEnumerator<out T> : IEnumerator, IDisposable
{
    T Current { get; }
}

public interface IEnumerator
{
    object Current { get; }

    bool MoveNext();
    void Reset();
}

举例： List<int> : IEnumerable<int>

这个类就是可迭代的。可迭代有什么好处呢，就是可以放在foreach语法糖里面。
foreach(var x in list) 
====>
var interator = list.GetInterator();
while(interator.MoveNext())
{
    var x = interator.Current;
}
interator.Dispose();



