## UniTask学习

用于替代Task，避免GC，同时加入了更多的Unity的异步操作的等待功能。是针对Unity优化的Task。


主要功能：

UniTask.DelayFrame(100)
UniTask.Delay(TimeSpan.FromSeconds(10), ignoreTime)

// replacement of yield return null
await UniTask.Yield();   // 这个不一定等同于 yield return null, 要看Yield()的参数时机。
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
有些异步操作的ToUniTask()可以传入 IProgress<T>，用来显示进度的回调。
使用UniTask自带的 Progress.Create 和 Progress.CreateOnlyValueChanged 可以提高效率，比较轻量级。

## Timing 
UniTask 定义了很多种的Timing点，ToUniTask 跟 WithCancellation 都需要指定在哪个timing点回调。
不过AsyncOperation UniTask本身就给他们实现了 GetAwaitor，这个awaitor是在这些异步操作在自己的回调Timing点回调的（unity定义的Timing点），不需要指定Timing点。
所以会有一个问题。如果你的LoadSceneAsync使用了ToUniTask或者WithCancellation，并在指定的Timing点回调。那么它的回调可能会玩与场景自己的`Start`。因为场景的start是在EarlyUpdate.ScriptRunDelayedStartupFrame（Unity的timing点）。

存在问题： Unity的ECS框架可能会让它不工作。如果它在Unitask之后初始化。因为它会重新设置 PlayerLoop。所以你可能需要重新初始化。

默认的初始化会把所有的timing点加入，你可以初始化你自己有用的timing点。
```
var loop = PlayerLoop.GetCurrentPlayerLoop();
PlayerLoopHelper.Initialize(ref loop, InjectPlayerLoopTimings.Minimum); // minimum is Update | FixedUpdate | LastPostLateUpdate
```

## UniTaskTracker
用来监控内存泄漏问题。

## 插件支持
TextMeshPro
Addressable
Dotween 
```
// sequential
await transform.DOMoveX(2, 10);
await transform.DOMoveZ(5, 20);

// parallel with cancellation
var ct = this.GetCancellationTokenOnDestroy();

await UniTask.WhenAll(
    transform.DOMoveX(10, 3).WithCancellation(ct),
    transform.DOScale(10, 3).WithCancellation(ct));
```

## 异步linq，看不懂

## Unity Test Runner支持
UniTask.ToCoroutine 桥接
