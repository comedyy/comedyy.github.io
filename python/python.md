# python 语法相关

## 获取当前环境数据
```
sys.path[0] 当前脚本的目录
sys.argv[0] 当前的python文件路径
sys.argv[1] 当前的第一个参数，以此类推
```

## glob使用
相当于shell中的find命名

```
patten = "/**/*.ipa"
file_list = [f for f in glob.glob(patten, recursive=True)]
```


## 获取文件时间
```
create_time = os.stat(x)[stat.ST_CTIME]  #获取文件创建的秒数
current_time = int(time.time()) #当前时间
```

## 执行
python -u xxx.py 可以在jenkins上立即输出日志
pythonw xxx.py 可以不弹出框，在后台运行


## 乱码问题
python使用uf8作为字符串的编码。但是print()函数并不一定是接收utf8的字符串来输出。
print(sys.stdout.encoding) # 使用来输出当前的stdout的接收的编码。
如果编码不是utf8，那么输出会乱码。
我们需要把stdout的编码方式修改成utf8，这样它接收到输出的buffer对象的时候，使用utf8的方式来解析收到的输出buffer。
sys.stdout.reconfigure(encoding='utf-8')

