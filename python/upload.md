## 局域网实现多机器上传文件的实现

碍于公司的每人带宽限制，做了一个可以通过不同人的机器，把文件上传到服务器的demo。
其实可以叫IT帮忙加带宽，不过开两天又给关了。

### 基本原理
大文件切块-> sftp传输分块到局域网节点->统一上传到服务器上->组装分块

### 切块
使用linux命名`split`, windows上可以在`git shell`的`/usr/bin`目录中找到。或者直接下载GNU工具。
```
file_count = peer_count + 1 # 确定分块数量
peer_file_size = file_size / (peer_count + 1) + 1000 # 确定分块大小
Excute.execCom("split -b {0} {1} {2}".format(int(peer_file_size), base_name, base_name1))  # 执行分块操作
file_list = Excute.FindFiles(splitpackage_folder +"/"+ base_name1+ "??")  ## 获得分块结果

# split -b 100M xxxx.zip kkkk 
这个xxxx.zip就是你要分块的文件，kkkk是分块后的前缀

```

### sftp 传输到局域网节点
[使用paramiko库传输](./paramiko.md)

### 同时上传文件到服务器
这个时候我们需要开启进程来上传，每个进程代表一个上传任务，需要监听每个进程，等进程全部结束，并且没有报错，就说明文件已经成功上传。

```
# 创建一个进程：
def exeAsyncTask(command):
	return Popen(command, stdout=PIPE, stderr=STDOUT)
# 检查进程是否结束
def IsAsyncTaskFinish(proc):
	return GetAsyncTaskRetCode(proc) is not None
# 获得进程的输出
def GetAsyncTaskOutPut(proc): # 返回stdout 跟 stderr
	ret = proc.communicate()
	out = ""
	error = ""
	if ret[0] != None:
		out = ret[0].decode("utf-8")
	if ret[1] != None:
		error = ret[1].decode("utf-8")
	return out, error

# 循环遍历进程的状态
	error_occur = False
	while len(proc_list) > 0:
		for proc in proc_list:

			if error_occur:
				proc.terminate()			

			if Excute.IsAsyncTaskFinish(proc):
				proc_list.remove(proc)

				out,error = Excute.GetAsyncTaskOutPut(proc)
				print(out)
				print(error)

				if Excute.GetAsyncTaskRetCode(proc) != 0:
					error_occur = True
					break
			else:
				time.sleep(.5)
```

```
# 执行上传，本地跟局域网节点运行的脚本不一样
# 本地直接新建一个进程，调用SendToFtp.py
# 局域网节点的时候的新建一个进程，调用ExcuteRemote.py,再在ExcuteRemote.py中，同步调用ssh命令，执行局域网节点的SendToFtp.
def SendToServer(local_path, server_folder, peer_info, server_info):
	if peer_info == None:
		return Excute.exeAsyncTask(['python3', 'SendToFtp.py', local_path, server_folder, server_info["host"], str(server_info["port"]), server_info["username"], server_info["password"]])
	else:
		return Excute.exeAsyncTask(['python3', 'ExcuteRemote.py', peer_info["host"], str(peer_info["port"]), peer_info["username"], peer_info["password"], peer_info["python_path"], 'SendToFtp.py', local_path, server_folder, server_info["host"], str(server_info["port"]), server_info["username"], server_info["password"]])

```
### 合并文件

使用cat命令来完成, 把分文件合并成一个，最后执行`md5sum`来判断文件是否正确
```
cat xxx/* > b.zip 

```