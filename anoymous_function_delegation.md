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
        lst.ForEach(XX);
        lst.ForEach(XX1);
        lst.ForEach(XX2);
    }
}

```
可以看到一共有4中匿名函数，分别是
A:引用了静态对象 无gc
B:引用了const对象 无gc
C:引用了类对象 gc
D:引用了本地对象 gc

还有使用了非匿名对象，
E：引用了const对象， gc
F：引用了类对象，gc
G：引用了静态对象，gc
H：静态函数引用了静态对象，gc


使用ilspy查看生成之后代码
```c#
void Update()
{
    <>c__DisplayClass3_0 CS$<>8__locals0 = new <>c__DisplayClass3_0();
    CS$<>8__locals0.<>4__this = this;
    CS$<>8__locals0.local_var = 1;
    lst.ForEach((Action<int>)delegate(int m)
    {
        m += static_var;
    });
    lst.ForEach((Action<int>)delegate(int m)
    {
        m++;
    });
    lst.ForEach((Action<int>)delegate(int m)
    {
        m += CS$<>8__locals0.<>4__this.instance_var;
    });
    lst.ForEach((Action<int>)delegate(int m)
    {
        m += CS$<>8__locals0.local_var;
    });
    lst.ForEach((Action<int>)XX);
    lst.ForEach((Action<int>)XX1);
    lst.ForEach((Action<int>)XX2);
    lst.ForEach((Action<int>)static_XX1);
    lst.ForEach((Action<int>)static_XX2);
}
```

会发现其实它是把匿名函数转成Action<int>来实现的，而Action<int>是一个delegation，说到底也是一个对象，它继承自`System.MulticastDelegate`。
那就要new一个这个delegation对象。

而 A跟B 比较特殊，编译器为他们生成了一个内部类，用来存放这些匿名函数。同时在生成delegation的时候，把这个delegation设置为static，之后再新建delegation的时候，就使用这个static的delegation。
而 C跟D就是直接生成了delegation对象，它的匿名函数直接生成在类中，而且每次的delegation都是动态创建的。

E，F，G，H，都有gc

## 结论
匿名函数居然还有提高效率的功能。
尤其是在无引用local对象的时候，它会生成静态delegation，而其他方式的调用则会生成新的delegation。


## 使用委托来优化反射（静态函数跟成员函数）
```c#

public class A
{
    public void TT(int a)
    {
        Debug.Log("TT");
    }
}

public class CreateDelegation : MonoBehaviour
{
    public delegate void PPP(int a);
    public delegate RaycastHit[] RaycastAllCallback(Ray r, float f, int i);
    public RaycastAllCallback raycast3DAll = null;

    // Start is called before the first frame update
    void Start()
    {
        MethodInfo method = typeof(A).GetMethod("TT", new[]{typeof(int)});
        if(method != null)
        {
            A a = new A();
            var action = (PPP) Delegate.CreateDelegate(typeof(PPP), a, method);
            Debug.Log(action);
        }
  
        var raycastAllMethodInfo = typeof(Physics).GetMethod("RaycastAll", new[] {typeof(Ray), typeof(float), typeof(int)});
        if (raycastAllMethodInfo != null)
            raycast3DAll = (RaycastAllCallback)Delegate.CreateDelegate(typeof(RaycastAllCallback), raycastAllMethodInfo);
        Debug.Log(raycast3DAll);

    }
}

```