---
title: Lua实现Class遇到的一个坑
date: 2019-10-23 10:00:00
tags: [Lua]
categories: 技术文章
---
https://stackoverflow.com/questions/15456456/changing-value-of-table-in-object-changes-the-value-for-all-objects-how-do-i-ge

# Lua实现Class遇到的一个坑

目前最常见的一种实现方式：

```
ClassBase = {
	data = {1, 2, 3, x = 0};
}
function ClassBase:New()
    local obj = {};
    setmetatable(obj, {__index = self});
    return obj;
end
```

这种方法大多数情况下不会有问题，但是有一种情况却会导致未能预料的错误：

```
function ClassBase:SetData()
	--注意这行
    self.data.x = 999;
end

a = ClassBase:New();
b = ClassBase:New();

b:SetData();
print("a.data.x is :" .. tostring(a.data.x));
print("b.data.x is :" .. tostring(b.data.x));
print("ClassBase.data.x is :" .. tostring(ClassBase.data.x));
```

得到的结果是：

![1575291103612](1575291103612.png)

这就产生了意料之外的结果：修改`b`的数据，为什么影响到另一个实例`a`甚至是ClassBase的数据呢？



其实也不难解释，关键就在Class的实现方式上：

在New函数里，实例通过将父类设置为metatable和将父类的__index设置为父类本身来实现继承和简单的多态。

在ClassBase:SetData()函数里，self就是new出来的实例，self.data.x并不是单个的操作，它其实可以分成两部分，对Lua来说，它其实是这样的：

`self ["data"] ["x"]`。在第一部分`self ["data"]`，首先会self的data成员。但是self有`__index`方法，而没有data成员，所以lua会直接通过`__index`方法访问metatable中的data字段，metatable被分享了一次，而这个metatable就是ClassBase。

然后在`["x"]`这一部分，指向的是metatable中的data字段。



而self.data = 999 则不会有这样的问题，因为self.data是一个一次操作，而且后面紧跟着赋值操作，lua不会使用`__index`方法，而是使用`__newindex`方法，所以会触发lua默认的操作：为self（就是实例）创造一个新的成员，然后赋值，完全符合预期。

最简单的规避方法：

```
ClassBase = {
     data = {1,2,3,x = 0}
}
--新增初始化函数 在初始化函数里为成员赋值
function ClassBase:ctor()
    self.data = {1,2,3,x = 0};
end
function ClassBase:New()
     local obj = {};
     setmetatable(obj, {__index = self});
     -- 初始化函数
     obj:ctor()
     return obj;
end
```

运行结果如图：

![1575292046367](1575292046367.png)

符合预期

此外还可以使用deepcopy的方法，但是也有一些缺点，不再赘述。