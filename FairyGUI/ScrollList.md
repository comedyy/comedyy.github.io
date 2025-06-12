## scrollList

目前看是个循环列表，调用方式很简单，
```
    _list = _mainView.GetChild("list").asList;
    _list.itemRenderer = RenderListItem;
    _list.SetVirtual();
    _list.numItems = 1000;
    _list.onTouchBegin.Add(OnClickList);

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

