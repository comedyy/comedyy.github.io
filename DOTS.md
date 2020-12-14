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

   