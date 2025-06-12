## cutScene
``` 动态加载ui
    _cutSceneView = UIPackage.CreateObject("CutScene", "CutScene").asCom;
    _cutSceneView.SetSize(GRoot.inst.width, GRoot.inst.height);
    _cutSceneView.AddRelation(GRoot.inst, RelationType.Size);
```

## Filter
```
    GObject obj = _mainView.GetChildAt(i);
    if (obj.filter is ColorFilter)
    {
        ColorFilter filter = (ColorFilter)obj.filter;
        filter.Reset();
        filter.AdjustBrightness((float)(_s0.value - 100) / 100f);
        filter.AdjustContrast((float)(_s1.value - 100) / 100f);
        filter.AdjustSaturation((float)(_s2.value - 100) / 100f);
        filter.AdjustHue((float)(_s3.value - 100) / 100f);
    }
    else if (obj.filter is BlurFilter)
    {
        BlurFilter filter = (BlurFilter)obj.filter;
        filter.blurSize = (float)_s4.value / 100;
    }
```

## Gesture
```
     SwipeGesture gesture1 = new SwipeGesture(holder);
    gesture1.onMove.Add(OnSwipeMove);
    gesture1.onEnd.Add(OnSwipeEnd);

    LongPressGesture gesture2 = new LongPressGesture(holder);
    gesture2.once = false;
    gesture2.onAction.Add(OnHold);

    PinchGesture gesture3 = new PinchGesture(holder);
    gesture3.onAction.Add(OnPinch);

    RotationGesture gesture4 = new RotationGesture(holder);
    gesture4.onAction.Add(OnRotate);
```


## Guide
```
    通过mask，反向挖洞。来实现。

    GRoot.inst.AddChild(_guideLayer); //!!Before using TransformRect(or GlobalToLocal), the object must be added first
    Rect rect = bagBtn.TransformRect(new Rect(0, 0, bagBtn.width, bagBtn.height), _guideLayer);

    GObject window = _guideLayer.GetChild("window");
    window.size = new Vector2((int)rect.size.x, (int)rect.size.y);
    window.TweenMove(new Vector2((int)rect.position.x, (int)rect.position.y), 0.5f);
```

# HitTestMain
```
    Stage.inst.onTouchBegin.Add(OnTouchBegin); // ui单例
```

## joystick
```
    _touchArea.onTouchBegin.Add(this.OnTouchBegin);
    _touchArea.onTouchMove.Add(this.OnTouchMove);
    _touchArea.onTouchEnd.Add(this.OnTouchEnd);


```


## ModalWaiting
```
    UIConfig.globalModalWaiting = "ui://ModalWaiting/GlobalModalWaiting";
    UIConfig.windowModalWaiting = "ui://ModalWaiting/WindowModalWaiting";

    GRoot.inst.ShowModalWait();     // 显示
    GRoot.inst.CloseModalWait();    // 关闭
```


## Particle
```
    Object prefab = Resources.Load("Flame"); // 粒子
    GameObject go = (GameObject)Object.Instantiate(prefab);

    _mainView.GetChild("holder").asGraph.SetNativeObject(new GoWrapper(go));        // 往这个位置挂上东西。 GoWrapper就是挂一个GameObject (Particle),

    _mainView.GetChild("c0").draggable = true;
    _mainView.GetChild("c1").draggable = true;      // 让它可以拖动。这个还没搞懂 ❓
```

## RenderTexture
```
    同样支持renderTexture。比较麻烦。


```

## Transition
```
    _g1 = UIPackage.CreateObject("Transition", "BOSS").asCom;
    _btnGroup.visible = false;  // 隐藏 一整个group，group没有在unity下渲染是比较麻烦。可能害怕打断合批
    
    GRoot.inst.AddChild(target); // 看了下groot好像是一个单独的节点。 其实就是把ui放到这个下面去渲染。
    
    Transition t = target.GetTransition("t0");
    t.Play(()=>{
        _btnGroup.visible = true;  // 隐藏
        GRoot.inst.RemoveChild(target);
    })
```

## TurnCard ❓目前不知道两个有什么区别。。对了，可以看xml。
```
    _back.displayObject.perspective = true; // 设置成透视的可以看到立体的效果。
```


## typeEffect
``` 需要创建 TypingEffect 对象
    _te1 = new TypingEffect(_mainView.GetChild("n2").asTextField);
    _te1.Start();
    Timers.inst.StartCoroutine(_te1.Print(0.050f));

    _te2 = new TypingEffect(_mainView.GetChild("n3").asTextField);
    _te2.Start();
    Timers.inst.Add(0.050f, 0, PrintText);
```

## scrollList

目前看是个循环列表，调用方式很简单，
```
    _list = _mainView.GetChild("list").asList;
    _list.itemRenderer = RenderListItem;
    _list.SetVirtual();
    _list.numItems = 1000;
    _list.onTouchBegin.Add(OnClickList);


    _list.AddSelection(500, true);
    _list.scrollPane.ScrollTop();
    _list.scrollPane.ScrollBottom();

    void RenderListItem(int index, GObject obj)
    {
        GButton item = obj.asButton;
        item.title = "Item " + index;
        item.scrollPane.posX = 0; //reset scroll pos

        //Be carefull, RenderListItem is calling repeatedly, dont call 'Add' here!
        //请注意，RenderListItem是重复调用的，不要使用Add增加侦听！
        item.GetChild("b0").onClick.Set(OnClickStick);
        item.GetChild("b1").onClick.Set(OnClickDelete);
    }
```