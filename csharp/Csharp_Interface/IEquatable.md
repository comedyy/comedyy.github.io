## IEquatable<T>
这个接口是c#内置接口，提供Equals()函数， 主要是用来对象之间的比较。
object 有内置的Equals()函数，但是它只是对内存地址进行了对比。
但是很多情况我们并不是只对比内存地址，我们需要对比对象的内容。比如string或者是int。
同样struct的对比也需要对比内容。struct继承自ValueType， ValueType有一个默认的Equals，但是使用反射，效率不行。
所以我们可以继承这个接口，来实现自己的Equals函数。

同时，在C#的内置容器中，比如Dictionary，在对比key的时候也会使用到Equals函数。如果你继承了IEquatable，那么它会使用IEquatable的对比函数。同时在Dictioanry对比key的时候，也是需要对比HashCode(), 所以如果你的struct没有重写GetHashCode的话，也会产生装箱操作。

注： GetHashCode()的作用：
它为了这个对象生产一个hash值，用于Dictionary跟Hashset的key，这样Dictionary可以用这个hash值来建立key。
对象相同，hash值一定相同，对象不同，hash可能相同，会发生hash冲突。
砍了其它人的项目，
struct KK
{
    int a;
    ushort b;

    int GetHashCode()=> (a * 397) ^ b; /// 测试了一下，在b < 250， a <10000的时候，0 冲突。
}


总结：
IEquatable<T>是给一个对象提供一个对比方法的接口，这个接口在c#的内置对象中很经常使用到。所以你的对象需要放到Dictionary之类的容器中做key的话，最好实现一下这个接口。可以提交效率同时避免装箱操作。同时最好也实现GetHashCode，因为Dictionary在对比key的时候也会调用到它，避免装箱。