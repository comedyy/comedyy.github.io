## UniTask学习

用于替代Task，避免GC，同时加入了更多的Unity的异步操作的等待功能。是针对Unity优化的Task。


主要功能：

UniTask.DelayFrame(100)
UniTask.Delay(TimeSpan.FromSeconds(10), ignoreTime)

// replacement of yield return null
await UniTask.Yield();
await UniTask.NextFrame();

// replacement of WaitForEndOfFrame(requires MonoBehaviour(CoroutineRunner))
await UniTask.WaitForEndOfFrame(this); // this is MonoBehaviour

// replacement of yield return new WaitForFixedUpdate(same as UniTask.Yield(PlayerLoopTiming.FixedUpdate))
await UniTask.WaitForFixedUpdate();

// You can await IEnumerator coroutines
await FooCoroutineEnumerator();

// replacement of yield return WaitUntil
await UniTask.WaitUntil(() => isActive == false);


// You can await a standard task
await Task.Run(() => 100);

// Multithreading, run on ThreadPool under this code
await UniTask.SwitchToThreadPool();

// return to MainThread(same as `ObserveOnMainThread` in UniRx)
await UniTask.SwitchToMainThread();

// concurrent async-wait and get results easily by tuple syntax
var (google, bing, yahoo) = await UniTask.WhenAll(task1, task2, task3);

// shorthand of WhenAll, tuple can await directly
var (google2, bing2, yahoo2) = await (task1, task2, task3);

// any of them
var (google2, bing2, yahoo2) = await Task.WhenAny(task1, task2, task3);

await (asset as TextAsset)?.text ?? throw new InvalidOperationException("Asset not found");



Unity的异步：
await Resources.LoadAsync 
await Resources.LoadAsync().ToUniTask(IProgress<float>, PlayerLoopTiming, CancellationToken)  // 把它一个ResourcesRequest转成一个UniTask,并且接受IProcess
await Resources.LoadAsync().WithCancellation(CancellationToken) 


// ...unity中的所有异步操作


```
public static AsyncOperationAwaiter GetAwaiter(this AsyncOperation asyncOperation) // 2023.0.1unity自己实现了
{
    Error.ThrowArgumentNullException(asyncOperation, nameof(asyncOperation));
    return new AsyncOperationAwaiter(asyncOperation);       // 实现也很简单，直接把回调赋给了 complete函数。
}
```

## cancel操作，
WithCancellation(CancellationToken) 
this.GetCancellationTokenOnDestroy() // 获取monobehavior的cancel操作

```
try
{
    var x = await FooAsync();
    return x * 2;
}
catch (Exception ex) when (!(ex is OperationCanceledException)) // when (ex is not OperationCanceledException) //这样捕获异常
{
    return -1;
}
```

不过使用 .SuppressCancellationThrow(); 可以轻量化异常捕获。
```
var (isCanceled, _) = await UniTask.DelayFrame(10, cancellationToken: cts.Token).SuppressCancellationThrow();
if (isCanceled)
{
    // ...
}

```
```
var cts = new CancellationTokenSource();    // 倒计时取消
cts.CancelAfterSlim(TimeSpan.FromSeconds(5)); // 5sec timeout.
try
{
    await UnityWebRequest.Get("http://foo").SendWebRequest().WithCancellation(cts.Token);
}
catch (OperationCanceledException ex)
{
    if (ex.CancellationToken == cts.Token)
    {
        UnityEngine.Debug.Log("Timeout");
    }
}
或者使用  TimeoutController 
.WithCancellation(timeoutController.Timeout(TimeSpan.FromSeconds(5)));

this.clickCancelSource = new CancellationTokenSource();
this.timeoutController = new TimeoutController(clickCancelSource); // 和其他的取消一起。

var linkedTokenSource = CancellationTokenSource.CreateLinkedTokenSource(cancelToken.Token, timeoutToken.Token); // 合并一个cancel Token source
```


## Progress





