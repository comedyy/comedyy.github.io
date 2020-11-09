## c# Reflection

### 1. performance

​	reflection到底慢在哪里。

| 直接调用 | createDelegate | 缓存了MethodInfo | 无缓存MethodInfo |
| -------- | -------------: | :--------------: | ---------------- |
| 6ms      |            7ms |      1000ms      | 2000ms           |
