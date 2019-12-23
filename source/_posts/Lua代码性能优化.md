---
title: Lua中的string连接
date: 2019-12-23 22:09:02
tags: [Lua]
categories: 技术文章
---
# Lua性能优化

### 1 Local总是比global快

Local变量更快，因为机制上决定了它们是通过index来索引的，而Global变量由于存在table里的原因，使用之前要进行一次hash查找。

### 2 尽量减少可GC对象的生成

会产生GC对象的操作：

1 循环中使用`..`连接字符串

2 新建一个table 

3 函数闭包

### 3 table预分配

local t = {}; for i=1,N do t[i] = 0 end

这种方法可导致 O(log(N))次rehash，还要涉及内存重新分配和数据的拷贝。

最实用的方法就是，在table比较小的情况下，预先塞入初始值：

```
local t = {0,0,0,0,0}
```

```
local t = {nil,nil,nil,nil,nil}
```

可以避免多次rehash。



也有一劳永逸的方法，能够绑定C函数的情况下使用`lua_createtable` 方法，

```
> Here is a helper function that creates an empty table where you can supply
> the parameters you want to create the table with:
 
```

### 4 多使用table.concat  /  减少使用string.format

Lua的string机制是这样的，当我们创建一个字符串的时候，Lua首先会检查内部是否已经有相同的字符串了，如果有就会直接返回其引用，没有的话才会真正创建。这样在字符串拼接的时候，Lua先要遍历查找是否有相同的字符串，然后会把后者整个拷贝然后拼接，产生一个新的buffer。我们可以使用`table.concat`来代替`..`，在table.concat的内部实现中，会用table来模拟buffer，来建少不必要的buffer创建。










