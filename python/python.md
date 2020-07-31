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