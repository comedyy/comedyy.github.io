# MemoryMarshal 
命名空间是在System.Runtime.InteropService, 用于内存的相互操作。

int[] aaa = new int[] { 1, 2, 3, 4 };
Span<byte> bbb = MemoryMarshal.AsBytes(aaa.AsSpan());
var ccc = MemoryMarshal.Cast<byte, int>(bbb);

foreach(var x in ccc) Debug.LogError(x); // 正常输出1，2，3，4
foreach(var x in bbb) Debug.LogError(x); // 正常输出1， 0， 0， 0， 2， 0， 0， 0 (小端模式)
内存都没变。只是视图变了。

