## shell

### 参数处理
```
BuildNo=$1    # 等号需要是中间无空格
ShellFile=$0  # 第一个参数是文件名
BASEDIR=$(dirname "$0") # 取当前路径

```


### if 
```
if [ condition ]; then

elif [ condition ]; then

else

fi
```

### 运行获得结果

```
name=`find $PackageDir -type f -name "${BuildNo}_dSYM.zip"`
或者
name=$(find $PackageDir -type f -name "${BuildNo}-*-v1.symbols.zip")
```

### 退出
`exit errorcode`