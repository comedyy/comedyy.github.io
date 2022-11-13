## DOTween
LeanTween 好久没维护了，并且我看UniTask支持DoTween，所以了解下。

Tweener tween =  DOMove(new Vector3(1, 2, 3), 2); 
tween.From();       // 从xxx位置移动到当前位置
tween.SetRelative(); // 设置为相对位置。
tween.SetLoops(-1, LoopType.Yoyo); // 设置循环。
tween.SetAutoKill(false) // 不自动结束
tween.Pause()   //  暂停


Text.DoText()       // text慢慢出现


Sequence 序列
Sequence s = DOTween.Sequence();
s.Append(tween) // 往序列的后面插入一个tween, 串型, 不确定会不会插到 insert后面。
s.Insert(t, tween) // 往序列的某个时间点插入一个tween， 并行


Path: 路线支持
Tween t = target.DOPath(waypoints, 4, pathType)
.SetOptions(true)  // close path
.SetLookAt(0.001f);
// Then set the ease to Linear and use infinite loops
t.SetEase(Ease.Linear).SetLoops(-1);

改变目标点
tween.ChangeEndValue(target.position, true) // true是不重新开始似乎，直接从当前点继续移动
tween.Restart();

DOTween.PlayAll();
DOTween.PauseAll();