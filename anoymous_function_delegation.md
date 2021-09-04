## 匿名函数
一般跟着委托一起走。
```c#
public class NewBehaviourScript : MonoBehaviour
{
    static int static_var =1;
    int instance_var =1;
    List<int> lst = new List<int>(){
        1, 2, 3, 4
    };

    // Update is called once per frame
    void Update()
    {
        int local_var  = 1;
        lst.ForEach(m=>m=m+static_var);
        lst.ForEach(m=>m=m+1);
        lst.ForEach(m=>m=m+instance_var);
        lst.ForEach(m=>m=m+local_var);
    }
}

```
可以看到一共有4中匿名函数，分别是
A:引用了静态对象 无gc
B:引用了const对象 无gc
C:引用了类对象 gc
D:引用了本地对象 gc


使用ilspy查看生成之后代码
```c#
lst.Foreach((Action<int>)delegate(int m){
    m=m+static_var
})
```

会发现其实它是把匿名函数转成Action<int>来实现的，而Action<int>是一个delegation，说到底也是一个对象，它继承自`System.MulticastDelegate`。
那就要new一个这个delegation对象。

而 A跟B 比较特殊，编译器为他们生成了一个内部类，用来存放这些匿名函数。同时在生成delegation的时候，把这个delegation设置为static，之后再新建delegation的时候，就使用这个static的delegation。
而 C跟D就是直接生成了delegation对象，它的匿名函数直接生成在类中，而且每次的delegation都是动态创建的。
