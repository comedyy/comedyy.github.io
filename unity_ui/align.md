# 内存对齐。
    计算机对对齐的内存比较好优化！
    正常情况下 一个struct的对齐方式 = struct中最大的对齐方式。
    每个成员对象的对齐字节数 = 它的大小。
    int = 4， double = 8, byte = 1;

1. 对齐的数据都在cacheline中。
    cacheline正常一次内存读写都会多读写一些字节，通过如果是1，2，4，8这种方式的对齐的话，一般一个数字不会在cacheline中。因为cacheline的对齐方式可能是64字节。