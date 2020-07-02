# c# 6.0 语法糖

## 1. ?.表达式非常强大，可以不需要一直使用if来判断空。
```
string a = null;
Debug.Log(a?.ToString());
Console.WriteLine(Employee?.Name ?? "No Employee name provided");
```

## 2.自动format
不需要再指定{0} {1}了， 直接把参数写进去。
```
string name = "AAA";
DateTime date = DateTime.Now;
Debug.LogFormat($"Hello, {name}! Today is {date.DayOfWeek}, it's {date:HH:mm} now.");
```
前提是要在字符串前面加一个$美元符号。
直接把0,1,2替换成对应变量名就可以。其他都不变。

## 3.out参数简写。
```
string a = "1";
int.TryParse(a, out int a_int);
Debug.Log(a_int);
```

## 4.nameof
直接把变量，类型的名字变成字符串，在debug，或者异常返回的时候特别有用。
它在编译时完成，并没有消耗。

```
Debug.Log(nameof(String));
```

## 5.简化函数表达式：
```
member => expression;
public void Function=>aaa();
public string Name => locationName;
public void DisplayName() => Console.WriteLine(ToString());
```

## 6.自动属性初始化
```
public string Name { get; set; } = "Joe";
```

## 7.静态using
```
using static UnityEngine.Debug;
Log("ABC");
```