## jenkins

### 1. jenkins 的api列表

各种的python的api，包括队列的查询

```
http://<Your Jenkins>/api/
http://<Your Jenkins>/api/xml
http://<Your Jenkins>/queue/api/
http://<Your Jenkins>/queue/api/xml
```



python方式触发jenkins打包，需要用户名密码验证。

```python
requests.post("http://10.0.17.10:8080/job/android_build/buildWithParameters?test_param=BB", auth=HTTPBasicAuth('jenkins', '1103bdf422cb5ee47e87e79ea4a7ff506e'))
```



## 2. jenkins gitlab webhook

jenkins安装gitlab插件，在项目配置里面就可以找到一个`Build when a change is pushed to GitLab`选项，勾选之后，会显示webhook的URL。在`高级`选项中，点击 `Generate`生成Token。

在gitlab中，点击项目->设置->webHook，填入URL跟Token，再选择事件。之后可以通过点击已经生成的webhook的测试按钮，进行测试。



