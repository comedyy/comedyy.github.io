## linq
集合操作，类似与sql语句。

1. Filter 筛选器， Where, 有两个重载。有个下标模式。
2. Projection： Select， SelectMany (selectMany 可以把多个list select到一个list中)
select 有还有一个下标模式。
```
List<List<int>> lists = new List<List<int>>()
{
    new List<int>(){1, 2, 3, 4},
    new List<int>(){1, 2, 3, 4},
    new List<int>(){1, 2, 3, 4},
    new List<int>(){1, 2, 3, 4},
};
var list = lists.SelectMany(m=>m).ToList();
Debug.Log(string.Join(" ", list));  // 1 2 3 4 1 2 3 4 1 2 3 4 1 2 3 4
```
3. Partitioning :   选取数据， 从第一个元素开始，Skip就跳过， Take的就选取。
    Take(n) 取前N个.  1，2，3，4，5， Take(2) = 1 , 2
    TakeWhile(predicate) 取数据，停止在predicate 失败的地方 TakeWhile(m=>m<4) = 1, 2, 3
    Skip(n) 跳过N个   Skip(2) = 3, 4, 5
    SkipWhile(predicate) 跳过数据，停止在predicate 失败的地方 SkipWhile(m=>m<4) = 4

4. Order: 排序
OrderBy, OrderByDescending
ThenBy, ThenByDescending, 排序完如果相同，再更有这个排序，二级或者三级排序
Reverse

5. Group
GroupBy ： 会把一个列表拆分成很多列表，每个列表有一个共同key， 这个key由传入GroupBy的函数决定的
返回一个 IEnumable<IGrouping<key, value>>
```
var group = list.GroupBy(m=>m);
foreach(var x in group)
{
    Debug.Log(x.Key + "=> " + string.Join(" ", x));
}
```
ToLookup, 同样的，返回一个ILookUp对象，ILookUp继承IEnumable<IGrouping<key, value>>。 跟groupby一样

6. Set
Distinct:
Union： 会自动Distinct
Interect： 取叉集，两个集合的共同对象。
Except： 取除了新集合的所有。

7. Conversion
ToArray()
ToDictionary()
ToList()
ToLookUp()

8. Element
First
FirstOrDefault
ElementAt 第几个。

9.Generation
Enumerable.Range()  // 制作从N开始的M个对象
Enumerable.Repeat() // 重复N个
Enumerable.Empty() // 

10. Quantifiers
All  所有
Any  任意

11. Aggregate  计数
Count
Sum
Min
Max
Avg
Aggregate(s, n) // 累加器。 s是最终结果，n是每个结果。 跟sum类似，但是能实现更多东西。比如你有灵活的操作。


12. Miscellaneous
Concat(IEnumerable) 
SequenceEqual(IEnumerable)

Join: 可以把两张表格通过相同字段合并，似乎只有两个字段值相同才行。
```
var result =  persons.Join(cities, p => p.CityID, c => c.ID, (p, c) => new { PersonName = p.Name, CityName = c.Name}); // 创建一个新类型出来。
foreach(var item in result)
{
    Debug.Log($"{item.PersonName},{item.CityName}");
}
```

GroupJoin 不仅join，还给你group好，按id 来group，生成一个新对象，下面是按城市筛选出所有在城市里的人。
```
var result1 =  cities.GroupJoin(persons, c => c.ID, p => p.CityID, (c, ps) => new { cityName = c.Name, p = ps});
        foreach(var item in result1)
        {
            Debug.Log($"{item.cityName}"+ string.Join(" ", item.p.Select(m=>$"[{m.CityID} { m.Name}]")));
        }
```