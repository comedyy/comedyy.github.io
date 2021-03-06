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

class JJ : 
// 或者是ICriticalNotifyCompletion， 它的回回调是 UnsafeOnCompleted
{
    int i = 0;
    public bool IsCompleted 
    {
        get{
            Debug.LogError("isComplete");
            i++;
            return i > 10;
        }
    }

    public string GetResult()
    {
        Debug.LogError("getResult");
        return "KKk";
    }

    public void OnCompleted(Action continuation)
    {
        continuation();// 继续往下执行
        Debug.Log("oncomplete");
    }

    public void UnsafeOnCompleted(Action continuation)
    {
        continuation();	// 继续往下执行
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

