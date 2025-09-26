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

# C# 7.2
## 8. in 关键字。 防止大结构体拷贝， 并且只读。
void Print(in Point p) // p 是只读引用
{
    Console.WriteLine(p.X);
}

## 9 泛型约束增强
可以限制泛型为 unmanaged、Enum 等：
void M<T>() where T : unmanaged // 在指针中使用
{
    // T 是非托管类型
}


# C# 8.0
## 10.1 readonly 的函数修饰， 无法修改成员变量。只能修饰struct的函数
struct Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y) => (X, Y) = (x, y);

    public readonly double Distance => Math.Sqrt(X * X + Y * Y);  //  x,y 不能在这里修改
}

      
## 10. using 可以不使用花括号
void ReadFile(string path)
{
    using var reader = new StreamReader(path);
    Console.WriteLine(reader.ReadToEnd());
} // 离开方法自动 Dispose

## 11. 默认接口实现
interface ILogger
{
    void Log(string message);
    void LogError(string message) => Log("[Error] " + message);
}

class ConsoleLogger : ILogger
{
    public void Log(string message) => Console.WriteLine(message);
}

LogError 可以默认实现。

## 12. static 本地函数

void Process()
{
    static int Add(int a, int b) => a + b;
    Console.WriteLine(Add(2, 3));
}
无法构成闭包(引用外部的变量)

## 13. 索引和范围

var arr = new[] { 1, 2, 3, 4, 5 };
Console.WriteLine(arr[^1]); // 5，最后一个元素
var slice = arr[1..4];      // { 2, 3, 4 }

## 14. 属性模式

