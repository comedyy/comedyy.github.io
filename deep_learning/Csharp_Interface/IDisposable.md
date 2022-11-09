## IDisposable

IDisposable
{
    void Dispose();
}

 IDisposable 有一个dispose函数，用于施放资源的。

 有一个配套的语法糖，是Using

 using(var x = xxx())
 {

 }

 这个xxx()就是返回的是一个IDisposable对象， 当using退出或者结束的时候，调用dispose().
 var x = xxx();
 try
 {
     ...
 }
 catch
 {
     throw;
 }
 finally{
     x.Dispose();
 }

 ## Dispose设计模式
 
 有一个设计模式是dispose设计模式，用来施放managed跟native的资源， 跟IDisposable配合着使用。
 dispose设计模式需要定义一个Dispose(bool disposing)的函数，用来施放资源。
 disposing == true的时候，就是 程序手动 IDispose.Dispose() 调用的。 
 disposing == false的时候 是 C#的gC程序调用 object.Finize(),（就是对象的析构函数），来施放的。（gc线程）， 专门用来施放native内存。因为managed内存肯定已经被销毁，不然不会被回收。

这个防止native内存被一直hold住导致内存泄漏。

class DisposableObj : IDisposable
{
    bool _disposed;

    ~DisposableObj()       // 析构函数，由Finaze线程调用
    {
        Dispose(false);
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);      // 叫gc不用调用析构函数了。
    }

    protected void Dispose(bool disposing)
    {
        if(_disposed)
        {
            return;
        }

        if(disposing)
        {
            // managed cleanup    
            Debug.Log("managed cleanup"+ GetHashCode() + "   " + Thread.CurrentThread.ManagedThreadId);
        }

        // native cleanup
        Debug.Log("native cleanup"+ GetHashCode() + "  " + Thread.CurrentThread.ManagedThreadId);

        _disposed = true;
    }
}

public class NewBehaviourScript : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        DisposableObj obj = new DisposableObj();
        Debug.Log("Create" + obj.GetHashCode());
        obj.Dispose();

        obj = new DisposableObj();
        Debug.Log("Create" + obj.GetHashCode());

        using(obj = new DisposableObj())
        {
            Debug.Log("Create" + obj.GetHashCode());
        }
    }