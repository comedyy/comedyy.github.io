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