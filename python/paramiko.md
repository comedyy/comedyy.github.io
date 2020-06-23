# 使用paramiko与远程服务器交互

paramiko 提供ssh和sftp的能力，你可以把平常的一些对服务器的操作写成脚本，来跟服务器进行交互。

```
def FetchLog(ftp_host, ftp_port, ftp_user, ftp_pass):
	sf = paramiko.Transport((ftp_host, ftp_port))
	sf.connect(username = ftp_user,password = ftp_pass)
	ssh = paramiko.SSHClient()
	ssh._transport = sf

	stdin, stdout, stderr = ssh.exec_command('find /data/wt_game/app/ -name "BattleServer_*.log" | xargs cat |  grep "LUA_ERROR" -A20 > ~/battle_error.log; zip ~/battle_log.zip ~/battle_error.log ')
	print(stdout.readlines())

	sftp = paramiko.SFTPClient.from_transport(sf)
	sftp.get("battle_log.zip", "c://{0}_battle_log.zip".format(ftp_host))
	sf.close()
```

这个函数作用是连接到远程服务器，并且查找战斗服的错误日志信息，并且压缩下载。