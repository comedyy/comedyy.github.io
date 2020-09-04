## LUA 如何执行

### 问题：lua这类型的解释型的语言是如何执行的？
脚本如何运行成像二进制那样运行？我们知道c#，c++的运行都是把代码编译成二进制，然后操作系统可以通过二进制代码中的指令来执行。
但是脚本语言是如何运行的。

### 分析
为了更好的阅读lua的源码，新建一个[demo工程](https://github.com/comedyy/lua_code_debug)来运行，这样看的不懂的地方可以调试运行。

```
lua_State *state = luaL_newstate();  // 创建一个lua虚拟机
lua_register(state, "howdy", howdy); // 注册一个函数
luaL_openlibs(state);       // 打开lib
result = luaL_loadfile(state, filename); // 加载lua

if (result != LUA_OK) {
    print_error(state);
    return;
}

result = lua_pcall(state, 0, LUA_MULTRET, 0); // 执行lua
```

`lua_register` 往`_G`中插入一个函数，这个操作基本上就是使用lua api操作lua
`luaL_loadfile` 是去加载一个lua脚本到一个lua的函数中。（一个lua文件被抽象成一个函数）
`lua_pcall` 则是去执行这个函数

### lua文件如何生成指令
`luaL_loadfile` 可以加载当前的文件到一个`LClosure`对象，并把它保存到栈顶。
通过对lua文件的词义跟语义分析，生成指令信息。

`LClosure` 这个对象差不多就是函数，闭包。包含一个函数原型跟函数的upvalue列表。
```
typedef struct LClosure {
  ClosureHeader;
  struct Proto *p;
  UpVal *upvals[1];  /* list of upvalues */
} LClosure;
```

`proto`对象是函数原型，里面包含这函数的指令 `Instruction *code` 等信息。

### 执行一个lua文件
当一个lua文件被加载进来的时候，我们在栈顶放了一个`LClosure`对象，这个时候我们就可以执行`luaL_loadfile`来执行栈顶的函数。

lua 全局有个`lua_State` 对象，这个对象有个`CallInfo *ci`对象，这个就是当前调用的调用信息。
`CallInfo` 对象中有 `previous`, `next` 来构成堆栈，同时含有`l`跟`c`是当前的函数。
`l`是lua函数，里面包含这一个`Instruction*`代表指令集合
`c`是c函数，里面一个`lua_KFunction`代表c函数

当调用lua_pcall的时候，就往lua_State中插入一个使用当前`LClosure`生成的`CallInfo`对象，然后开始执行

### 指令如何执行
`Instruction` 对象是一个32位的int，里面包含挺多区域的，
第一个区域是指令区（前七位），第二个区域是寄存器区（8位），寄存器用来存储当前函数中的局部变量，所以如果当前函数的局部变量超过8位存储，会报错。还有其他区域没看到。

在执行函数的时候，遍历这个`Instruction*` 然后取出指令跟当前需要的寄存器，根据执行来做相应的操作
```
vmcase(OP_LOADI) {  // (local a = 1) 加载一个int，这个int也是指令的一部分，同时把它放到了指令里面的寄存器中。
    lua_Integer b = GETARG_sBx(i);
    setivalue(s2v(ra), b);
    vmbreak;
}
vmcase(OP_ADDI) {   // (local c = a + 1) 执行一个加法的操作。（宏晦涩）
    op_arithI(L, l_addi, luai_numadd);
    vmbreak;
}
```

### 延伸
```
typedef union Value { // lua的基本类型 相当于每个对象的大小都一样。内存高效
  struct GCObject *gc;    /* collectable objects */
  void *p;         /* light userdata */
  lua_CFunction f; /* light C functions */
  lua_Integer i;   /* integer numbers */
  lua_Number n;    /* float numbers */
} Value;
```

```
typedef struct Table {      // table:实现，使用链表法实现哈希表，同时包含数组，当你的下标小于limit的时候。
  CommonHeader;
  lu_byte flags;  /* 1<<p means tagmethod(p) is not present */
  lu_byte lsizenode;  /* log2 of size of 'node' array */
  unsigned int alimit;  /* "limit" of 'array' array */
  TValue *array;  /* array part */
  Node *node;
  Node *lastfree;  /* any free position is before this position */
  struct Table *metatable;
  GCObject *gclist;
} Table;

```
