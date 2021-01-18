### NavMesh相关总结

Navmesh 是unity用来做人物移动使用的插件，通过静态编辑可以行走路径跟不可行走路径，保存到资源中以供运行时使用。

同时通过它的API接口，也可以动态的生成，修改跟销毁navMesh.



#### 静态寻路网格

1. 只支持一个Agent类型，可以配置Agent的大小，高度，最大斜坡，最大的楼梯高以及配置多块NavMesh之间的DropHeight跟StepHeight。
2. 在Navigation页面中，有Agent，Areas，Bake，Object几个页面，
   1. Agents界面配置所有的Agent类型，可以添加多种类型的Agent，高矮胖瘦，默认的是Humanoid.
   2. Areas页面配置所有的Area类型，默认有Walkable, NotWalkable跟Jump， 每个area有个cost，寻路的时候优先选择总cost小的路劲。
   3. Base页面配置烘焙需要的参数，配置当前烘焙的Agent信息，以及OffMeshLink的参数。
   4. Object页面配置场景中物体的参数，包括是否static以及是否生成OffMeshLinks，以及当前的Area。



因为只支持一种Agent大小，所以在NavMesh上运行的Agent，都按烘焙的NavMesh的参数来走，各个Agent上配置的参数无效。



#### 动态寻路网格

unity给出了一个使用例子：https://github.com/Unity-Technologies/NavMeshComponents

	1. 支持多个NavMesh，从而支持多种Agent分别运行在各自的NavMesh中。
 	2. 可以动态生成NavMesh，更新和删除NavMesh，更加灵活。
 	3. 支持异步的方式生成
 	4. 可以在prefab中使用。
 	5. 可以使用NavMesh.onPreUpdate来更新NavMesh数据（比如NavMesh里面的物体改动）
 	6. NavMeshModifier跟NavMeshModifierVolume用来修改一些物件的NavMesh属性，比如是否可以行走。
 	7. 可以使用物理碰撞体来替代MeshRenderer来构建寻网格。



它可以做到静态的Mesh跟动态的Mesh相结合。

使用NavMeshSurface，NavMeshModifier，NavMeshLink来构建一个静态的NavMesh。

在运行时，增删改这些组件来更新NavMesh。



#### OffMeshLink && NavMeshLink

用来连接两个NavMesh 或者同一个NavMesh的跳转，默认情况当agent勾选了AutoTraverseOffMeshLink的时候，Agent

会自动移动过去。

而如果我们不勾选，我们可以为不同的情况定制不同的移动形式：比如从低跳到高处，就可以让玩家运行一个跳跃的轨迹并且播放一个跳跃的动画。

```c#
IEnumerator UpdateMove
{
agent.isOnOffMeshLink // 是否在传送点上
yield return /..  中间执行传送代码，慢慢Transform到目标传送点 ../
agent.CompleteOffMeshLink();  // 完成传送    
}
```



#### 其他组件

NavMeshAgent： 驱动一个玩家

NavMeshAbstacle: 用来模拟一个障碍物，这个障碍物跟不可行走的路径是有区别的，它不在参与寻路的计算，只在局部RVO的时候计算。



#### API

Agent.destination 设置寻路的目标点

Agent.isStopped 停止寻路

NavMeshSurface.UpdateNavMesh 异步更新NavMesh





