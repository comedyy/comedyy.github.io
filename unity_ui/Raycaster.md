# Ugui Raycaster

## Raycaster种类

其实这个Raycaster是通用的事件系统，不单单是给UGUI使用的。
unity提供了三种Raycaster，供不同的情况使用

1. `GraphicsRaycaster`:   UI
2. `Physics2DRaycaster`： 2D
3. `PhysicsRaycaster`:	 3D


## 运行机制

### 必备组件
1. 需要有EventSystem，同时还需要有个InputModule
2. 需要有Raycaster，挂在场景中，初始化的时候向RaycasterManager注册。

### Raycaster介绍

有一个RaycasterManager静态对象，管理所有的Raycaster对象
所有的Raycaster继承BaseRaycaster， 提供几个方法：
- OnEnable：向RaycasterManager注册自己。
- OnDisable：向RaycasterManager取消注册自己。
- Raycast(PointerEventData eventData, List<RaycastResult> resultAppendList)：点击函数
- eventCamera：相机
- sortOrderPriority, renderOrderPriority: 用来对最后的结果进行排序的。然后拿出优先级最高的那个作为最后的点击事件。

### 点击派发流程

> EventSystem.Update()->InputModule.Update()->InputModule.GetTouchPointerEventData()->eventSystem.RaycastAll()->ProcessTouchXXX()

所以EventSystem的点击事件，会去找出每个Raycaster，然后调用它的Raycast方法，找到所有被点击的对象，然后排序，取出第一个PointEventData。找到点击对象的第一个事件接受器，调用它的函数。
事件接收器有 IPointerClickHandler 等