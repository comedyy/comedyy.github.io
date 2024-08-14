DOTS

unity实现的ECS，并结合它的Job System，可以数量众多的游戏对象。适合做一些对象众多的游戏，比如rts等。



ECS中有一个World对象。

1. 我们自己创建一个world。
2. 可以新建很多的Systems，同时定义这些system的执行顺序。
3. 可以高效的保存Component信息，快速遍历system所需要的数据。
4. 利用jobsystem多线程实现提升吞吐量。



还有其他很多的ECS的库。

1. [Entitas](https://github.com/sschmid/Entitas-CSharp)
2. [Struct Base Ecs](https://github.com/Leopotam/ecs)



主要有如下功能

1. 快速的筛选system需要的component。

   Entitas和Unity的ECS都可以根据`ALL`, `ANY`, `None`方式来筛选，StructBaseECS去掉了ANY选项。

   筛选器在Entity对象的component变动的时候，自动变动。

   Unity不仅提供EntityQuery方式，还可以在System中使用Entitys.Foreach方式来方便查找component。

2. 能捕获component的变动。

   Entitas 通过 `ReactiveSystem`来实现，它会监听一个组件的`ADD`, `Remove`, `AddOrRemove`事件，从而对这些组件的对象进行处理。

   StructBaseECS 通过OneFrame的方式处理。就是创建一个新的事件组件对象，然后在下一帧把它删除。这个事件组件将在它对应的system中被处理。

   Unity的ECS使用EntityQuery.AddChangedVersionFilter。不过并不是真正数据改变的时候触发change，而是由访问到的时候。

3. 数据存储方式不一样。

   Entitas因为是class base，所以它的内存分布是非连续的，它仅仅是实现了ECS的编程思想。

   StructBaseECS在内存分配上是连续的，至少每种component的内存是连续的。

   你首先先给它一份配置，你需要多大的缓存，比如Entity的数量，component的数量，它把他们都创建好。

   UnityEcs的内存根据官网的介绍也是相对连续的，以chunk的形式存储。

4. SharedComponent

   unity的ECS支持。主要是应对大量的Entity有很多相同的Component。比如角色的RenderMesh，一群一样的小兵公用。

   它需要时很少变动的对象。EntityManager.GetAllUniqueSharedComponents() 这个api来查询。内存开销较小。

   
   
   这里重点讲一下StructBaseEcs。
   
   
   
   ```
   EcsWorld _world;		// world，里面包含Entity跟Component
   EcsSystems _systems;
   
   // 初始化
   _world = new EcsWorld();
   _systems = new EcsSystems(_world);
   _systems
   .Add(new MoveSystem())
   .OneFrame<ScoreChangeEvent> () // ScoreChangeEvent 的这个component是一个一帧的对象，添加之后下一帧移除。
   .Init(); 	// MoveSystem就是一个移动相关的system。
   
   // 每帧update
   _systems?.Run();
   
   // 析构
   if (_systems != null)
   {
       _systems.Destroy();
       _systems = null;
       _world.Destroy();
       _world = null;
   }
   
   // 定义component
   public struct Location	// 位置
   {
       public Vector2I position;
       public Vector2I rotation;
   }
   public struct InputDir	// 方向
   {
       public Vector2I dir;
   	public List<EcsComponentRef<SnakeSegment>> Body; // EcsComponentRef 是一个struct，包含着这个Componnet的引用。我为了减少拷贝才这么做的吧。毕竟EcsComponentRef很小， 通过Body[i].UnRef() 获取数据。
   }
   
   // 定义system
   // 还有IEcsDestroySystem，IEcsPostDestroySystem， IEcsInitSystem， IEcsPreInitSystem。
   public class MoveSystem : IEcsRunSystem							
   {
       readonly EcsFilter<Location, InputDir> _filter = null;		// 筛选器，动态注入进去的。
   	readonly EcsFilter<Score>.Exclude<ScoreChangeEvent> _scoreUiFilter = null; // 包含Score组件，不包含ScroeChangeEvent。
   
       public void Run()
       {
           foreach (var idx in _filter)				// 遍历所有包含Location跟InputDir的对象
           {
               ref var entity = ref _filter.GetEntity(idx);	// 拿取Entity
               ref var location = ref _filter.Get1(idx);		// 拿去Entity的位置信息
               ref var input = ref _filter.Get2(idx);			// 拿去Entitty的移动方向
   
               if (input.dir.x != 0 || input.dir.y != 0)
               {
                   location.rotation = input.dir;
               }
               location.position += input.dir * Define.DELTATIME;	// 修改component数据
           }
       }
   }
   
   // 是否存在组件
   bool exist = ref _world.NewEntity ().Has<ScoreChangeEvent> ();
   // 添加一个component
   ref var changeScore = ref _world.NewEntity ().Get<ScoreChangeEvent> ();
   
   
   ```
   
   

   #### 系统化学习dots。
   
   
   使用TransformAccessArray来提高多线程设置transform的能力。
   1. TransformAccessArray, TransformAccess, IJobParallelForTransform
   https://docs.unity3d.com/ScriptReference/Jobs.IJobParallelForTransform.html。



   unity 的内存管理都是走Native那块的内存。
1. 申请一份64块chunk，每隔chunk = 16k，那么一次性生成 1M的内存。
2. 每个chunk给每个archtype使用，chunk里面已经计算好了 可以几个entity，以及每个component的起始位置，大小。 还是跟其它的ecs一样，相同的entity还是并排存放。
3. Entity同样是增长的，一个list存放了Entity的version， 一个list存放了entity所在的chunk已经chunk中的位置。 我估计chunk中的内存都是并排的，没有缝隙，删除一个之后，不会留空。提高效率。但是删除的效率就不是很高。 allocate效率高，尤其是allocate多个的时候。
4. 里面大量使用了strcut指针，申请的内存是native内存。是个native地址，强转城struct*，这样操作都是针对native内存操作。 没有gc，同样避免了struct大量拷贝。
5. chunk里面保存着一个archtype对象，archType对象中保存着所有的chunk。
6. enitycomponetStore中保存了 archeType列表，
7. filter是通过archeType来实现的，只要知道跟哪些archeType 匹配，就遍历它的chunk就是了。
