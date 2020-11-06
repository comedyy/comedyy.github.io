在jenkins的乱码处理中，遇到了一些关于乱码的问题。

1. request.post 的对象r.content是一个bytes对象，里面的内容是

   `x =b'{"data":"","error":{"code":-1,"message":"hrm\\u63a5\\u53e3\\u67e5\\u8be2\\u7528\\u6237\\u4e0d\\u5b58\\u5728:jenkins"}}'`

   我们需要怎么把它解码出来呢。

   首先这个b开头的意思就是当前是个bytes对象，并且每个字符都对应与内存上的ascii。就是它是一个由 `len(x)`个字符(内存中对应byte)组成的一个byte列表。

   你可以在python中直接decode输出。

   ```python
   unicode_escape = b'{"data":"","error":{"code":-1,"message":"hrm\\u63a5\\u53e3\\u67e5\\u8be2\\u7528\\u6237\\u4e0d\\u5b58\\u5728:jenkins"}}' # \u 后面接着四个字符，代表一个unicode，在内存中使用utf-8进行编码。
   utf8_encode = b'\xe6\x8e\xa5'  # \x 后面接两个字符，代表一个byte。
   
   print(unicode_escape.decode("unicode-escape")) # 直接只用unicode-escape解码成python的字符串。
   print(utf8_encode.decode("utf-8")) # 还原成之前的格式
   ```

    这个字总结：使用`unicode-escape`可以把`\uxxxx`类型的bytes类型对象解析成字符串,`utf-8`把`\x`类型的bytes解析成字符串。

   现在的python的字符串默认都是unicode字符串，所以`str=\\u63a5`跟`str=接`一样

2. git 命令行乱码

   git log 乱码=>

   	1. 同时设置 .bash_profile配置export LESSCHARSET="windows"，让Less指令是使用系统的编码，因为`git log`其实就是调用less来输出的。
    	2. 设置.gitconfig配置 [i18n] logoutputencoding = GBK, 这样让输出日志为输出为GBK
    	3. git窗口的语言设置为 zh,GBK

 	3. unity的日志文件乱码： 打开文件的时候，使用`utf-8`编码打开
 	4. python执行git的返回值编码错误
      	1. 在第2点设置完了之后，在调用git命令的结束之后，使用locale.getdefaultlocale()[1] 这个系统默认的编码方式来解码，可以保证不会出现解码错误。
	5. 如果decode失败怎么办：decode(xxx, errors="ignore")，可以不触发异常