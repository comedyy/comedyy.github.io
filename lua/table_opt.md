# lua 表格优化

为了开发方便，我们使用lua文件作为配置表方式。但是lua因为自身数据结构的问题，内存和文件大小都非常的大。

## 1.Lua的内存结构
```
// 基础类型
typedef union Value {
　　　  GCObject *gc;    /* collectable objects */
　　　  void *p;        /* light userdata */
　　　  int b;          /* booleans */
　　　  lua_CFunction f; /* light C functions */
　　　  long long i;    /* integer numbers */
　　　  double n;      /* float numbers */
　　　} Value;
　　　
　　　struct lua_TValue {
　　　  Value value_;
      int tt_;
　　　} TValue;
```
差不多一个int 16个字节，一个bool也是16个字节。
每个table有一个数组跟一个哈希表组成，如果放入的key是数字，而且这个数字正好在数组的长度附近的话，那么就插入到数组中。否者就插入到哈希表。

## 2.优化哈希表到数组

效果：节省差不多1/3的大小

因为哈希表每个数据都需要两份内存，一个key一个value，所以占用两倍左右的内存。所以把它转成数组。

实现方式： 每个数据表新加一个 索引表，记录着key跟 col的对应。

```
ID Name Score
1   李磊    100
```

之前的做法是 
```
{
    ["ID"] = 1，
    ["Name"] = 李磊,
    ["score"] = 100
}
```
现在改成
```
IndexTable = {
    "ID" =  1,
    "Name" = 2,
    "Score" = 3,
}

数据部分：

local tb = {1, "李磊", 100};
```

这时候会发现 `tb["Name"]` == 空，无法满足兼容，我们就加入元表。

```
function(tb, key)
    return rawget(tb, IndexTable[key])
end
```

## 3.优化重复字段（添加默认字段）
效果：可以明显改善有重复字段的表格的大小，根据重复性效果不一。项目中文件大小从25M-》23M

```
ID Name Score
1   李磊    100
2   韩梅梅    100
3   露西   100
4   乔治    96
```

会发现score字段都一样，我们建立一个新的表格， 重复字段表格。

```
DefautlTable = {
"Score" = 100
}
```

这时候会发现 `tb["Name"] == nil`，无法满足兼容，我们就加入元表。

```
function(tb, key)
    local index = IndexTable[key]
    if(index) then
        return rawget(tb, index)
    else
        return defautlTable[key]
    end
end
```

这个时候你会发现如果 乔治考了96分。。那我们的 表格
```
local tb = {1， “李磊”}， {2， 韩梅梅}， {3， 露西}， {4， 乔治， ”Score” = 96}
```

## 4.优化子表对象

效果：可以让有很多字表的表格的内存和文件大小显著变小，根据字表的重复性，效果不一。 项目文件大小中 从 23M-》16M

```
ID Name Scores (vector<chinese int; english int>)
1   李磊    {chinese:100 , English :100}{chinese:100 , English :100}
2   韩梅梅    {chinese:100 , English :100}{chinese:100 , English :10}
3   露西  {chinese:100 , English :100}{chinese:100 , English :10}
4   乔治    {chinese:100 , English :100}{chinese:100 , English :10}
```

这个时候发现chinese大家都是100分，

```
local tb = {1， 李磊，  {chinese = 100 ,English =100}, {chinese = 100 ,English =100}} , 2 . .....
```
我们定义
```
tbC = {"chinese"= 100, English = 100}
local tb = {{1, 李磊， {tbC, tbC}} , {2,  韩梅梅 {tbC，{"chinese" = 100 ,English =10}}}...
```

就我们把公共部分的字表拆出来，用一个已经存在的表格来替代。减少内存的使用量

## 5.存在问题
目前重复字段需要超过 80%的重复性才会提取。 重复子对象只有超过5个才会提取。

