## encoding 相关

### 问题：
在windows下运行，报错
```
print(subprocess.check_output("echo 第三方", shell=True).decode().strip())

Traceback (most recent call last):
  File "E:\work\test\pythonTool\test.py", line 6, in <module>
    print(subprocess.check_output("echo 第三方", shell=True).decode().strip())
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xb5 in position 0: invalid start byte
```

### 分析
这个`subprocess.check_output("echo 第三方", shell=True)`返回的是一个二进制数组，通过decode转成python的字符串。
默认使用的是`utf-8`的编码方式转的，因为它根本不知道`check_output`是使用什么编码方式编码的。

首先是 byte[] 里面存放的是一些二进制，是序列化过的数据。
decode之后变成string类型，不过这个string类型我感觉还是byte[]序列，只是把底层的序列转成了高级的序列。
最后使用print()输出的时候，把对应的byte[] 序列通过Unicode（python字符串的字符集）的对应关系输出成字符。

#### 字符集
我们假设有一张byte[] 到 字符的映射表，把它叫做字符集。
在unicode中，使用两个字节表示一个字符， `11 62`映射到中文`我`
GBK(cp936)，同样使用两个字节表示一个字符，`ce d2`映射到`我`
(ANSI是一个特殊的字符集，它在不同的语言环境下对应的是不同的字符集，在简体中文下是对应GBK，在繁体中文可能就是BIG5)

#### 编码（序列化）
其实我们可以直接使用字符集直接序列化成二进制的，比如上面的`我`直接变成`11 62`写入到文件。
但是存在一个问题就是英文字符如果这样用两个字节写到文件中，非常浪费。

所以需要在序列化的时候再对数据进行转成更加高效的存储格式（编码）。下面四个是记事本的四种编码方式。
1. `Utf8` -> `Unicode` 使用对位编码方式，从而减少占用（英文1字节，中文三字节）
2. `ANSI` -> `GBK`（简体中文环境，在繁体中文环境中对应BIG5）（英文一字节，中文两字节）
3. `Unicode` -> `Unicode` 完全使用Unicode的编码方式，不做任何编码修改（两字节）
4. `Unicode Big Endian` -> `Unicode` 

所以
ANSI中，`ce d2`映射到`ce d2`从而在GBK中映射到`我`， `31` 映射到 `31 00` 映射到 `1`
UTF8中，`e6 88 91` 映射到 `11 62` 从而在Unicode中映射到 `我`, `31` 映射到 `31 00` 到 `1`
Unicode中，`11 62` 映射到 `11 62` 从而Unicode中映射到 `我`, `31 00` 映射到 `31 00`, 映射到 `1`

回到刚刚那个报错。
首先python的字符串是unicode编码的，byte[] 需要使用decode来转成string，decode的默认编码方式是`utf8`,但是因为byte[] 根本不是`utf8`编码的，所以解析不正确报错了。所以我们必须给他传个正确的编码格式进去。它才能正确的转成`unicode`。

我们如何才能知道这个byte[]的编码方式呢？

### windows cmd的字符集
打开cmd的属性，我们可以看到当前的字符集是`936 ANSI/OEM - 简体中文 GBK`, 所以运行的输出也是这个格式。
使用 `locale.getdefaultlocale()[1]` 可以获取当前cmd的字符编码，从而正确解析。

```
def execCom(command):
    local_cp = locale.getdefaultlocale()[1]
	return subprocess.check_output(command, shell=True).decode(local_cp).strip()
```
### 记事本是如何知道当前文件的编码方式的。
UTF-8 EF BB BF
UTF-16（大端序）FE FF
UTF-16（小端序）FF FE

有这些unicode bom的话，直接使用，没有的话靠猜，比如你往记事本里面写入联通两个字，再打开记事本就猜错了。



[编码错误的例子](encode2.md)