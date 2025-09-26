##大小端

目前主流的设备的cpu都是 little endian。
旧架构的存在bit endian。
unity 员工说 unity只支持little endian的设备。

在处理数据的时候，如果endian不对的话，会导致数据转化错误。
比如你把一个int， 在little endian的设备上转成byte[], 传送给 big endian的机器上，那么big endian直接解析就会报错。 比如 BitConverter.GetBytes()。
所以在一些网络库的序列化中（LiteNetLib）， 会有规定使用哪种的endian来序列化跟反序列化数据。

```
  public static void WriteLittleEndian(byte[] buffer, int offset, short data)
        {
#if BIGENDIAN
            buffer[offset + 1] = (byte)(data);
            buffer[offset    ] = (byte)(data >> 8);
#else
            buffer[offset] = (byte)(data);
            buffer[offset + 1] = (byte)(data >> 8);
#endif
        }
```

在 bitArray的库中，也使用little endian来把byte[] 转成int[], 来加速 bit运算。

因为unity不支持little endian设备。所以我们直接使用以下代码来加速。

```
public unsafe void XorByteArrays(byte[] array1, byte[] array2)
    {
        int length = Math.Min(array1.Length, array2.Length);
        int intLength = length / sizeof(int);

        fixed (byte* ptr1 = array1)
        fixed (byte* ptr2 = array2)
        {
            int* intPtr1 = (int*)ptr1;
            int* intPtr2 = (int*)ptr2;

            for (int i = 0; i < intLength; i++)
            {
                intPtr1[i] ^= intPtr2[i];
            }
        }
    }
```


小端： x86， arm都是小端，以前的大型机是大端
大端： 网络传输都是大端，因为网络发明的时候都是大型机。

小端优势：
1. 强行转化方便。
    0x11223344 如果强制转byte的话，直接读取它前面8字节。 强制转short直接前16字节。
2. 乘法计算方便
    从小计算到大。
3. 