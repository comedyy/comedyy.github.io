## csharp的format

Debug.LogError($"{x:00}");  // 整数占位格子
Debug.LogError($"{y:0.00}");  // 浮点数

这个方式最好记忆。所有的数据反正都是0，你想要什么样子的占位格子，只要用0就可以。