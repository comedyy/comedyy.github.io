## Job 用来利用多线程减少主线程压力。
IJob只能跑在单一线程上
IJobFor可以跑在多线程上。
IJob.Run() 在主线程上跑，IJob.Schedule()在线程上跑，IJob.ScheduleParallel() 在多个线程上跑。

JobHandle.Compelete（） 是完成一个job。


## Thread Local
做一个功能，用到IJobFor,但是需要访问Random类，怎么办。
Unity提交类struct类型的Random在数学库中。就算是struct类型的，也无法直接放到IJobFor中去，因为不同的Job都会修改Random的内部。
所以需要一个NativeArray<Random> 每个thread一个Random。同时需要IJobFor定义一个private int m_ThreadIdx;

```
m_Randoms = new NativeArray<Random>(JobsUtility.MaxJobThreadCount, Allocator.Persistent);
for (int i = 0; i < m_Randoms.Length; i++)
{
    m_Randoms[i] = Random.CreateFromIndex((uint)i);
}

struct RandomVelocityJob : IJobFor
{
    //input
    [ReadOnly]
    public NativeArray<float> speeds;
    [ReadOnly] 
    public NativeArray<Random> randoms;
    public float deltaTime;

    [NativeSetThreadIndex]
    private int m_ThreadIdx;

    //output
    public NativeArray<float3> positions;

    public void Execute(int i)
    {
        positions[i] += randoms[m_ThreadIdx].NextFloat3Direction() * speeds[i] * deltaTime;
    }
}
```


##  ScheduleParallel 的batch
调用IJob.ScheduleParallel()的时候需要填入innerloopBatchCount，但是在64的时候，性能会更好。这个值代表的是每个IJobFor一次性拿走多少数据去执行。如果这个数据可以是cpu的cacheline的整数倍，那么就可以提高效率。
cpu加载数据的时候，会有一个cacheline，通常是32或者64.

## Custom batch
我们可以实现自己的batch。
public void Execute(int i)
{
    var parentPos = rootPositions[i];
    var parentRot = rootRotations[i];

    for (int j = 0; j < 5; j++)
    {
        var index = i * 5 + j;

        parentPos += math.mul(parentPos, childLocalPositions[index]);
        parentRot = math.mul(parentRot, childLocalRotations[index]);

        childLocalPositions[index] = parentPos;
        childLocalRotations[index] = parentRot;
    }
}

调用JobHandle.ScheduleBatchedJobs();可以立即开启job执行，否者unity之后把job缓存，到来系统执行调用JobHandle.ScheduleBatchedJobs();的时候才会执行。

## 数据保存方式：
最好使用
NativeArray<int> a;
NativeArray<int> b;
来替代
struct P{
    public int a;
    public int b;
}
NativeArray<P> lst;

说是执行的更快。