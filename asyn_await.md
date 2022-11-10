# async/await

## 1. async 用来修饰函数，标明它是一个异步的函数
```
async void Listen(int port){}
```
当然你的返回值不一定是void，可以是个Task，或者Task<T> 如果你想异步返回一个返回值。

## 2. await 用来执行一个异步函数，标示等待异步函数执行结果
跟unity协程中的yield语句类似
```
async void Listen(int port){
    _server = new TcpListener(IPAddress.Parse("0.0.0.0"), port);
    _server.Start();
    client = await _server.AcceptTcpClientAsync();
    _client = new NetWorkClient(client);
    await _client.StartReceiving();
}
```

上面例子中等待用户连接跟等待用户消息过程都是异步的，使用await方法可以阻塞住它。

## 3. 函数调用
调用一个async函数也很简单，直接在普通函数中调用就可以。

但是中间涉及到一个执行线程的问题。
它是基于Task的，而Task是基于线程池的。
它是从线程池中拿出线程执行的，所以它可以是在主线程上跑，也可以在其他线程上。

## 4.代码同步编写代替异步回调
```
static void Main(string[] args)
{
    string content = GetContentAsync(Environment.CurrentDirectory + @"/test.txt").Result;
    //调用同步方法
    //string content = GetContent(Environment.CurrentDirectory + @"/test.txt");
    Console.WriteLine(content);
    Console.ReadKey();
}
//异步读取文件内容
async static Task<string> GetContentAsync(string filename)
{
    
    FileStream fs = new FileStream(filename, FileMode.Open);
    var bytes = new byte[fs.Length];
    //ReadAync方法异步读取内容，不阻塞线程
    Console.WriteLine("开始读取文件");
    int len = await fs.ReadAsync(bytes, 0, bytes.Length);
    string result = Encoding.UTF8.GetString(bytes);
    return result;
}
```
.Result方法可以在普通函数里面等待 异步操作的完成。

不过它似乎并不能解决卡住主线程的问题。

```
static void Main(string[] args)
{
    asyncRun();   
    Console.ReadKey();
}
static async void asyncRun()
{
    string content = await GetContentAsync(Environment.CurrentDirectory + @"/test.txt");
    Console.WriteLine(content);
}
```

这样写应该会更好，至少它不会阻塞主线程了。

 

# 内部原理

先上代码：

```c#

class JJ : ICriticalNotifyCompletion
// 或者是ICriticalNotifyCompletion， 它的回回调是 UnsafeOnCompleted
{
    int i = 0;
    // 如果返回true，不会回调 UnsafeOnCompleted，
    // 如果返回false，会调用UnsafeOnCompleted，把回调传给你， 让你自己爱怎么做怎么做，然后做完记得回调函数就是了
    public bool IsCompleted         
    {
        get{
            Debug.LogError("isComplete");
            i++;
            return i > 10;
        }
    }

    // 返回给外部的返回值，await JJ()的返回值。这个返回值类型随意定义，你定义什么外面就获得什么。
    public string GetResult()
    {
        Debug.LogError("getResult");
        return "KKk";
    }

    // IsCompelte == false的时候，并且无ICriticalNotifyCompletion接口的时候调用
    public void OnCompleted(Action continuation)
    {
        continuation();// 继续往下执行， 更正确的做法是保存起来，等任务完成的时候再回调
        Debug.Log("oncomplete");
    }

    // IsCompelte == false的时候，并且是ICriticalNotifyCompletion接口的时候调用， 
    public void UnsafeOnCompleted(Action continuation)
    {
        continuation();	// 继续往下执行， 更正确的做法是保存起来，等任务完成的时候再回调
        Debug.Log("unsafeOnCompleted");
    }
}

class KK
{
    public JJ GetAwaiter()
    {
        return new JJ();
    }
}

public class NewBehaviourScript : MonoBehaviour
{
    // Start is called before the first frame update
    async void Start()
    {
        // await跟着一个对象，这个对象需要有一个GetAwaiter的函数。返回一个Awaiter对象，这个Awaiter对象需要继承ICriticalNotifyCompletion, 或者是INotifyCompletion，并且还需要bool IsCompleted {get} 跟     public string GetResult(){} 函数。
        // 这awater 对象是给编译器用的，因为async跟await这个关键字其实就是个语法糖，在编译代码的时候会生成新的代码。然后调用awaiter里面的函数。外部不建议调用awaiter。
        var x = await new KK();		
        Debug.Log(x);
    }
}

可以查看Unitask的实现。

```

编译器生成之后的代码：

```
	// NewBehaviourScript脚本
	[AsyncStateMachine(typeof(<Start>d__0))]
	[DebuggerStepThrough]
	private void Start()
	{
		<Start>d__0 stateMachine = new <Start>d__0();
		stateMachine.<>4__this = this;
		stateMachine.<>t__builder = AsyncVoidMethodBuilder.Create();
		stateMachine.<>1__state = -1;
		AsyncVoidMethodBuilder <>t__builder = stateMachine.<>t__builder;
		<>t__builder.Start(ref stateMachine);
	}

	// <Start>d__0.cs	编译器生成的stateMachine对象的代码。
	private void MoveNext()
	{
		int num = <>1__state;
		try
		{
			JJ awaiter;
			if (num != 0)
			{
				awaiter = new KK().GetAwaiter();
				if (!awaiter.IsCompleted)
				{
					num = (<>1__state = 0);
					<>u__1 = awaiter;
					<Start>d__0 stateMachine = this;
					<>t__builder.AwaitOnCompleted(ref awaiter, ref stateMachine);
					return;
				}
			}
			else
			{
				awaiter = (JJ)<>u__1;
				<>u__1 = null;
				num = (<>1__state = -1);
			}
			<>s__2 = awaiter.GetResult();
			<x>5__1 = <>s__2;
			<>s__2 = null;
			Debug.Log((object)<x>5__1);
		}
```





编译器会为async，await生成一份代码，里面包含一个IStateMachine对象，一个对应的builder。然后使用这一份生成的代码去调用awater，来实现异步操作。



## Unity中的实现
async void xxx(){
    await Task.Delay(1);
    Debug.Log(1);
}

你会发现这个Debug.Log(1)会在主线程上执行，因为unity帮我们处理了回调的线程问题。它会在 UnitySynchronizationContext 中回调。
如果你在dotnetcore上运行，那它的回调是直接在线程池上的。

同样UniTask的实现机制是这个日志会在主线程上，因为它在OnCOmplete中缓存了回调，在unity的生命周期回调中，执行这些回调。
Unitask比线程的Task有更好的内存表现。 同时实现了unity中各种异步行为的等待。

UnitySynchronizationContext 继承自 SynchronizationContext, 目前看是从其他线程往主线程调用函数需要通过这个对象。


var current = SynchronizationContext.Current;
ThreadStart start = new ThreadStart(()=>{
    // 进入其他线程
    current.Post((o)=>{
        Debug.Log("111");
        GameObject oo = new GameObject("oo");  // 这个时候已经在主线程了，可以使用unity api了。
    }, null);
});


## T可以返回给Task<T>

为什么 async Task<int> 可以返回一个int，
async Task<int> GetResult()
{
    await Task.Delay(1);
    return 1;
}

完全没有编译错误。 1 可以强转给 Task<int>，非常难理解。通过查看Unitask的源码发现，因为UniTask有一个属性
`[AsyncMethodBuilder(typeof(AsyncUniTaskMethodBuilder))]` 有这个宏的时候，说明这个异步操作有个Builder=>AsyncUniTaskMethodBuilder,这个builder里面有个Property:
```
 public UniTask<T> Task
{
    [DebuggerHidden]
    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    get
    {
        if (runnerPromise != null)
        {
            return runnerPromise.Task;
        }
        else if (ex != null)
        {
            return UniTask.FromException<T>(ex);
        }
        else
        {
            return UniTask.FromResult(result);
        }
    }
}
```
这样就可以把T强转成UniTask<T>。所以你可以把T返回给UniTask<T>。

## 关于异常的问题
为什么有时候可以打印日志，有时候没有。
当触发异常的时候，就会系统就会调用 
```
// 3. SetException
[DebuggerHidden]
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public void SetException(Exception exception)
{
    if (runnerPromise == null)
    {
        ex = exception;
    }
    else
    {
        runnerPromise.SetException(exception);
    }
}
```
会给这个UniTask挂一个Exception。
当我们的task是有await的时候，它会执行GetResult()函数，在GetResult函数中，把它给抛出来。而我们如果没有执行await的时候，它不会执行GetResult(),就不会抛异常。
```
var x = await KK();

async UniTask<int> KK()
{
    await UniTask.Yield();

    throw new System.Exception("2341");
    return 1;
}

```

但是你发现 UniTaskVoid 则不是这个逻辑，它不管有没有await，都会抛出异常。 UniTaskVoid 跟UniTask使用的是不同的Builder，不同builder里面的实现不同。因为UniTaskVoid不需要GetResult(),所以
在SetException的时候，直接抛异常
```
public void SetException(Exception exception)
{
    ..... // some code here
    UniTaskScheduler.PublishUnobservedTaskException(exception);
}
```

UniTask使用的是：AsyncUniTaskMethodBuilder
UniTaskVoid 使用的是： AsyncUniTaskVoidMethodBuilder

总结：
UniTask因为需要获得返回值，所以只会在await的时候才抛出异常。 UniTaskVoid 不需要返回值，所以可以直接抛出异常。
UniTask就是为了返回值而存在的，所有的await都必须有GetResult(),所以我感觉把异常放在GetResult()抛也是合理。
还有一点需要注意的是，虽然UniTask如果没有await，不会抛出异常，但是在gc的时候，其实还是会把异常抛出来的。（析构对象的时候）。

## await跟没有await的区别：
应该只是差一个GetResult()的调用。