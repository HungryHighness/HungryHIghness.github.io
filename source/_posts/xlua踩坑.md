---
title: xlua踩坑
date: 2022-03-03 23:04:50
tags: 
- Unity
- Lua
categories: Unity
---

## 打包AB包时XLua报错

和下面这个链接雀氏是同一个问题，但是我clear了五次莫名其妙又打包好了，我不能理解，我大受震撼。尤其是刚好在我搜出这个问题的时候，他打包好了？？

[解决XLua运行正常，但是打包出错问题_CoffeMilk的博客-CSDN博客_xlua 打包报错](https://blog.csdn.net/xiaochenXIHUA/article/details/96910189)

## emmylua使用xlua

因为emmylua的原理是：lua代码中通过`local dbg = require("emmy_core")` 主动加载 emmy_core.dll到宿主程序中，并启动调试内核代码。调试内核通过socket与IDEA/VSCode侧连接通讯。

一般情况下，都会重新自定义`loader`，这里emmy_core并不是一个lua文件，在加载的时候会报错，需要特判。

```csharp
private byte[] MyCustomLoaderFormAB(ref string filepath)
{
	if (filepath.Equals("emmy_core"))
    {
        return null;
    }
}
```

调试的时候需要在lua代码入口处插入(可以直接打开插件看具体要插入的信息)

```lua
dbg.tcpListen('localhost', 9966)
```

## emmylua智能提示突然失效

解决方案：将xlua从项目移除后重新导入解决。。我不能理解
