## ComponetFixture

![RUNOOB 图标](../img/unity_ui/componentfixture.png)

### 用处
为了应对ui的更新，在xlua的框架下，我们需要有一个方式可以在uiprefab的inspect面板上显示当前ui程序用到的对象（image，text等）。
最普通的做法是直接新建一个Monobehavior，同时暴露给inspect。但是这个做法无法做到热更新。
鲁大师做了一个componentfixture可以做到既可以在uiprefab的inspect面板上显示对象，又可以实现热更新。

### 实现
使用一个通用的脚本Componetfixture，配合一个Editor，从ui控制器中拿取ui控件的类型跟名字，把它绘制到inspect面板上。
用户通过拖拽控件到inspect面板来实现引用关系的对应，同时序列化到prefab中。
在ui加载的时候，把componetfixture中的控件传递给ui控制脚本就可以了。

#### 序列化到Prefab
componetfixure中存储了几个数组，
```
[Serializable]
private struct ArrayInfo
{
    public int Next;
    public int Count;
}

[SerializeField]private string[] _field_names;                // 存放字段名字
[SerializeField]private Object[] _field_values;               // 存放Unity对象，包括在数组中的对象，数量不一定跟_filed_names一样
[SerializeField]private ArrayInfo[] _field_arrays;               // 存放数组信息
[SerializeField]private bool[] _field_references;           // 存放_filed_values的当前位置是否存放数据（为了显示Missing等信息）
```
具体序列化规则：
_field_names存放着所有的ui控制器中的变量的名字。
如果当前对象是一个正常的控件，那么直接在_filed_values里面加一个值，如果是数据，就往_filed_values里面插入数据中所有的数据。同时添加一个_filed_arrays对象，标明当前是数组。最后如果当前_filed_values有赋值，就设置成true，否者false。

1.从ui控制脚本中获取控件的变量名跟数据类型。
从lua获取对象，是直接加载lua脚本，并且通过调用lua脚本的方法返回对应的ui控件信息,然后解析出来。
```
function UISplitPackageDownloadInfo.GetFieldsInfo()
	local fields = {
		"CloseBtn|UnityEngine.GameObject|false",
		"PauseBtn|UnityEngine.GameObject|false",
		"ResumeBtn|UnityEngine.GameObject|false",
		"_img_process|UnityEngine.UI.Image|true",
	}
	return connect(fields, Cs2Lua.Wartune.UI.UIBase.GetFieldsInfo())
end
```

2.对比当前序列化的数据，检查修改的兼容性。
代码就不贴，有点复杂，涉及到数据结构的变动的话，需要remap，remap过程比较复杂。（remap需要一个位置一个位置对比看看是否有改动）
我想如果可以的话，用一个中间对象来进行交互。在editor模式下，可以生成数据结构：
```
struct FieldInfo
{
    public Object[] values;
    public bool is_array;
    public Type type;
}

Dictionary<string, FiledInfo> _content; // 可以通过生成这样一个临时对象来交互，而不是当前代码中直接使用序列化数据进行交互。（_field_names之类）

```

2.绘制到inspect面板上
直接遍历当前的lua里面的控件的key，然后用componentfixture中取数据填充。
```
EditorGUI.ObjectField(position, property, value_type, label);
```

#### 传递给UI控制器
调用LoadFixture方法来进行传递
我们加载完ui之后，需要把componetfixture中的ui控件信息传给ui控制器
```
G_ComponentFixtureLoader.LoadFixture(script_obj, obj:GetComponent(typeof(G_ComponentFixture)));
```
这样，componentfixture就可以往lua脚本里面直接塞入 keyvalue形式的ui控件。

同时因为它支持把Componentfixture作为控件传入，所以塞入lua脚本的之前，需要把它子componentfixture构造完成。

### 序列化实例
我们通过阅读prefab，可以看到prefab中有很多保存componentfixture的信息。
```
  _field_names:
  - CloseBtn
  - PauseBtn
  - ResumeBtn
  - PrizeRoot
  - _lab_progress
  - _lab_progress1
  - _lab_speed
  - _lab_netstate
  - _img_process
  - _content
  _field_values:
  - {fileID: 8438908265597126324}
  - {fileID: 3521886830639981239}
  - {fileID: 8015256500171995675}
  - {fileID: 336652095499426565}
  - {fileID: 7393377172280483632}
  - {fileID: 4029446761648927266}
  - {fileID: 3969175316313098121}
  - {fileID: 7235229811203991523}
  - {fileID: 6241419249655851358}
  - {fileID: 3232975874774776105}
  _field_arrays: []
  _field_references: 01010101010101010101

```