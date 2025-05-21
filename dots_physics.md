## dots 物理，
终于到了物理了。

代码结构。
```
    1. Base: 数学
    2. Collision： 碰撞检测以及空间查询
    3. Dynamic： 世界step，整合，约束
    4. ECS：Components and Systems.
    5. Extension: 扩展
```

流程
1. Physics World Build： 从component中把数据拿出来。它是无状态的。所以每次都是component都拿出来。创建世界的阶段，有CollisionWorld跟DynamicWorld。CollisionWorld用来碰撞检测以及给DynamicWorld提供信息。DynamicWorld用来计算物理。
2. BroadPhase：粗的碰撞检测。
3. NarrowPhase： 细的碰撞检测
4. Constraint Solver：通过碰撞点，来计算出速度和角速度。
5. Integration: 通过速度来计算出新的位置。
6. Export： 位置，速度更新到component中

CollisionWorld：
有BVH（Bound Volume Hierarchy）来加速。
一共有两个BVH，一个是针对于静态，一个针对于动态的。
静态的之后有一个静态的collider发生变动的时候才会更新，提供了一种incremental的方式来更新。更快。
动态的是每帧都需要更新。

subScene:使用subscene，可以自动转化ecs。

Component:
PhysicsCollider: Collider。
PhysicsWorldIndex: ShareComponent 用来储存当前是哪个世界。
所有的对象都需要有unity.Transform

动态物体：需要LocalTransform，PhysicsVelocity 
静态物体：最好不能有parent（性能问题）LocalTransform 也需要，或者是LocalToWorld。

Custom Physics Body Tag：应该是跟unity的tag一样。

PhysicsInitializeGroup
PhysicsSimulationGroup

PhysicsWorldSingleton 保存着 PhysicsWorld，
PhysicsWorld 保存着构建的物理世界。 可以用来空间查询。


最后通过 ITriggerEventsJob 来获取到最后的结果。

## 总结：
    1. 可以修改内存好评。
    2. 最好可以自己修改成没有物理的更新，同时找到i性能热点以及卡顿。
    3. 性能应该问题不大，毕竟我们都是使用static collider，并且少量修改collider位置。
    4. 

在sample里面可以找到自定义的authoring,
PhysicsBodyAuthoring : rigidbody 的 authoring
PhysicsShapeAuthoring ： collider的authoring
你需要去好好研究一下。

1. 通过Author来创建一整个


