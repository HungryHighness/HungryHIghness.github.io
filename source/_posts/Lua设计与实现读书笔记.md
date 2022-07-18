---
title: Lua设计与实现读书笔记
date: 2022-05-02 20:48:59
tags: Lua
categories: 语言基础
---

# 简介

lua设计与实现的摸鱼笔记。主要是为了记录那些我觉得比较重要的东西。

本书的实现由为Lua5.1，后续可能会存在区别。

> 比如说Lua5.1的gc方式是三色标记，而后续是引用。

入门的话我推荐《Lua程序设计》搭配[简介 · GitBook (shenjun-coder.github.io)](https://shenjun-coder.github.io/LuaBook/)进行学习。

同时后续也可以看一下[云风的 BLOG (codingnow.com)](https://blog.codingnow.com/)。

# 基础数据类型

## 字符串

### 性能优化

因字符串拼接会产生新的字符串，所以会对性能上有所影响。

比如说：

```lua
a = os.clock()
local s = ''
for i = 1, 300000 do
    s = s .. 'a'
end
b = os.clock()
print(b - a)   -- 2.301
```

这段代码使用字符串拼接来生成新的字符串。

```lua
a = os.clock()
local s = ''
local t = {}
for i = 1, 300000 do
    t[#t + 1] = 'a'
end
s = table.concat(t, '')
b = os.clock()
print(b - a) --0.019
```

这段代码使用table来模拟字符串，避免使用拼接，很明显性能得到了很大的提升。

## 表

### 操作算法

#### 查找

查找的伪代码大概如下：

1. 输入的key为正整数，并且它的值大于0并且小于等于数组大小，那么他会尝试在数组部分查找。
2. 否则会尝试在散列表部分进行查找，首先会计算出该key具体的散列值，再根据散列值得到桶所在的位置，遍历该散列桶下所有的链表元素，直到找到该key为止。

可以从上发现，即使是一个正整数的key，他也不一定会存储在数组部分，这完全取决于当前数组大小。

比如说，下面代码便会发现只有1作为数组部分存储下来了，而100存储在散列表部分。

```lua
local t = {}
t[1] = 0
t[100] = 10
for i, v in ipairs(t) do
    print(i)
end
```

既然涉及到这种数据是如何进行存储的，那最佳的方案就是去看元素是如何增加的。

#### 新增元素

#### 取长度

```lua
--如果表存在数组部分：
--    初始化i = 0,j = table的sizearray
--    满足(j - i) > 1的条件下，循环：
--        m = (j + i) / 2
--        如果array[m - 1]为nil值: j = m   右边界收缩
--        否则 i = m                      左边界收缩
--    返回 i
--否则进入散列表中进行查找，具体规则同上
```

可以看出，如果表中同时拥有数组和散列表的数据，那么会优先取数组部分的长度。

# Lua 虚拟机

## 指令格式

[![OUn5UP.png](https://s1.ax1x.com/2022/05/11/OUn5UP.png)](https://imgtu.com/i/OUn5UP)

可以看出Lua的指令是32位的，从低位到高位开始解析。

首先，最低位是**OpCode**，称为操作数，接下来就是A，B，C参数。

列出目前所有的指令。这个枚举可以直接从`lopcodes.h`从寻找。

```c
typedef enum {
/*----------------------------------------------------------------------
  name		args	description
------------------------------------------------------------------------*/
OP_MOVE,/*	A B	R[A] := R[B]					*/
OP_LOADI,/*	A sBx	R[A] := sBx					*/
OP_LOADF,/*	A sBx	R[A] := (lua_Number)sBx				*/
OP_LOADK,/*	A Bx	R[A] := K[Bx]					*/
OP_LOADKX,/*	A	R[A] := K[extra arg]				*/
OP_LOADFALSE,/*	A	R[A] := false					*/
OP_LFALSESKIP,/*A	R[A] := false; pc++	(*)			*/
OP_LOADTRUE,/*	A	R[A] := true					*/
OP_LOADNIL,/*	A B	R[A], R[A+1], ..., R[A+B] := nil		*/
OP_GETUPVAL,/*	A B	R[A] := UpValue[B]				*/
OP_SETUPVAL,/*	A B	UpValue[B] := R[A]				*/

OP_GETTABUP,/*	A B C	R[A] := UpValue[B][K[C]:string]			*/
OP_GETTABLE,/*	A B C	R[A] := R[B][R[C]]				*/
OP_GETI,/*	A B C	R[A] := R[B][C]					*/
OP_GETFIELD,/*	A B C	R[A] := R[B][K[C]:string]			*/

OP_SETTABUP,/*	A B C	UpValue[A][K[B]:string] := RK(C)		*/
OP_SETTABLE,/*	A B C	R[A][R[B]] := RK(C)				*/
OP_SETI,/*	A B C	R[A][B] := RK(C)				*/
OP_SETFIELD,/*	A B C	R[A][K[B]:string] := RK(C)			*/

OP_NEWTABLE,/*	A B C k	R[A] := {}					*/

OP_SELF,/*	A B C	R[A+1] := R[B]; R[A] := R[B][RK(C):string]	*/

OP_ADDI,/*	A B sC	R[A] := R[B] + sC				*/

OP_ADDK,/*	A B C	R[A] := R[B] + K[C]:number			*/
OP_SUBK,/*	A B C	R[A] := R[B] - K[C]:number			*/
OP_MULK,/*	A B C	R[A] := R[B] * K[C]:number			*/
OP_MODK,/*	A B C	R[A] := R[B] % K[C]:number			*/
OP_POWK,/*	A B C	R[A] := R[B] ^ K[C]:number			*/
OP_DIVK,/*	A B C	R[A] := R[B] / K[C]:number			*/
OP_IDIVK,/*	A B C	R[A] := R[B] // K[C]:number			*/

OP_BANDK,/*	A B C	R[A] := R[B] & K[C]:integer			*/
OP_BORK,/*	A B C	R[A] := R[B] | K[C]:integer			*/
OP_BXORK,/*	A B C	R[A] := R[B] ~ K[C]:integer			*/

OP_SHRI,/*	A B sC	R[A] := R[B] >> sC				*/
OP_SHLI,/*	A B sC	R[A] := sC << R[B]				*/

OP_ADD,/*	A B C	R[A] := R[B] + R[C]				*/
OP_SUB,/*	A B C	R[A] := R[B] - R[C]				*/
OP_MUL,/*	A B C	R[A] := R[B] * R[C]				*/
OP_MOD,/*	A B C	R[A] := R[B] % R[C]				*/
OP_POW,/*	A B C	R[A] := R[B] ^ R[C]				*/
OP_DIV,/*	A B C	R[A] := R[B] / R[C]				*/
OP_IDIV,/*	A B C	R[A] := R[B] // R[C]				*/

OP_BAND,/*	A B C	R[A] := R[B] & R[C]				*/
OP_BOR,/*	A B C	R[A] := R[B] | R[C]				*/
OP_BXOR,/*	A B C	R[A] := R[B] ~ R[C]				*/
OP_SHL,/*	A B C	R[A] := R[B] << R[C]				*/
OP_SHR,/*	A B C	R[A] := R[B] >> R[C]				*/

OP_MMBIN,/*	A B C	call C metamethod over R[A] and R[B]	(*)	*/
OP_MMBINI,/*	A sB C k	call C metamethod over R[A] and sB	*/
OP_MMBINK,/*	A B C k		call C metamethod over R[A] and K[B]	*/

OP_UNM,/*	A B	R[A] := -R[B]					*/
OP_BNOT,/*	A B	R[A] := ~R[B]					*/
OP_NOT,/*	A B	R[A] := not R[B]				*/
OP_LEN,/*	A B	R[A] := #R[B] (length operator)			*/

OP_CONCAT,/*	A B	R[A] := R[A].. ... ..R[A + B - 1]		*/

OP_CLOSE,/*	A	close all upvalues >= R[A]			*/
OP_TBC,/*	A	mark variable A "to be closed"			*/
OP_JMP,/*	sJ	pc += sJ					*/
OP_EQ,/*	A B k	if ((R[A] == R[B]) ~= k) then pc++		*/
OP_LT,/*	A B k	if ((R[A] <  R[B]) ~= k) then pc++		*/
OP_LE,/*	A B k	if ((R[A] <= R[B]) ~= k) then pc++		*/

OP_EQK,/*	A B k	if ((R[A] == K[B]) ~= k) then pc++		*/
OP_EQI,/*	A sB k	if ((R[A] == sB) ~= k) then pc++		*/
OP_LTI,/*	A sB k	if ((R[A] < sB) ~= k) then pc++			*/
OP_LEI,/*	A sB k	if ((R[A] <= sB) ~= k) then pc++		*/
OP_GTI,/*	A sB k	if ((R[A] > sB) ~= k) then pc++			*/
OP_GEI,/*	A sB k	if ((R[A] >= sB) ~= k) then pc++		*/

OP_TEST,/*	A k	if (not R[A] == k) then pc++			*/
OP_TESTSET,/*	A B k	if (not R[B] == k) then pc++ else R[A] := R[B] (*) */

OP_CALL,/*	A B C	R[A], ... ,R[A+C-2] := R[A](R[A+1], ... ,R[A+B-1]) */
OP_TAILCALL,/*	A B C k	return R[A](R[A+1], ... ,R[A+B-1])		*/

OP_RETURN,/*	A B C k	return R[A], ... ,R[A+B-2]	(see note)	*/
OP_RETURN0,/*		return						*/
OP_RETURN1,/*	A	return R[A]					*/

OP_FORLOOP,/*	A Bx	update counters; if loop continues then pc-=Bx; */
OP_FORPREP,/*	A Bx	<check values and prepare counters>;
                        if not to run then pc+=Bx+1;			*/

OP_TFORPREP,/*	A Bx	create upvalue for R[A + 3]; pc+=Bx		*/
OP_TFORCALL,/*	A C	R[A+4], ... ,R[A+3+C] := R[A](R[A+1], R[A+2]);	*/
OP_TFORLOOP,/*	A Bx	if R[A+2] ~= nil then { R[A]=R[A+2]; pc -= Bx }	*/

OP_SETLIST,/*	A B C k	R[A][C+i] := R[A+i], 1 <= i <= B		*/

OP_CLOSURE,/*	A Bx	R[A] := closure(KPROTO[Bx])			*/

OP_VARARG,/*	A C	R[A], R[A+1], ..., R[A+C-2] = vararg		*/

OP_VARARGPREP,/*A	(adjust vararg parameters)			*/

OP_EXTRAARG/*	Ax	extra (larger) argument for previous opcode	*/
} OpCode;
```

再这些指令中，有不同的取值方式，具体可以看下图。

[![OUu0MQ.png](https://s1.ax1x.com/2022/05/11/OUu0MQ.png)](https://imgtu.com/i/OUu0MQ)

# 附录
