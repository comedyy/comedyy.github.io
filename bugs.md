# 记录查比较久的bug

1. 
```
    TeamData FindSnakeIndex(Snake snakeHit)
    {
        for (int i = 0; i < _allSnakes.Count; i++)
        {
            if(_allSnakes[i].snake = snakeHit) return _allSnakes[i];
        }

        return null;
    }
```
* 这里 `_allSnakes[i].snake = snakeHit` 写错了，需要是`==`而不是`=`。
* c#里面，正常情况下你不需要像c++那样考虑if里面的`=`问题，因为if只接受 bool作为参数，而其它的类型都无法隐式转化成bool（有精度丢失），_allSnakes[i].snake = snakeHit 这个表达式，最后它的返回值就是 一个Snake类型。 
* 那为啥这个可以正常运行。原因就是MonoBehavior重写了==，它可以隐式转化成bool。if(snake) 是可以正常编译通过的。导致了这个bug。
* 所以不好说这个设计是好的还是坏的。
