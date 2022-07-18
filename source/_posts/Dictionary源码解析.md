---
title: Dictionary源码解析
date: 2022-02-23 16:26:01
tags: c#
categories: 语言基础
---

# Dictionary源码解析

## 前置知识

### Hash算法

**散列函数**（英语：Hash function）又称**散列算法**、**哈希函数**，是一种从任何一种数据中创建小的数字“指纹”的方法。散列函数把消息或数据压缩成摘要，使得数据量变小，将数据的格式固定下来。该函数将数据打乱混合，重新创建一个叫做**散列值**（hash values，hash codes，hash sums，或hashes）的指纹。散列值通常用一个短的随机字母和数字组成的字符串来代表。

所有的Hash函数都有如下一个基本特性：如果两个散列值是不相同的（根据同一函数），那么这两个散列值的原始输入也是不相同的。这个特性是散列函数具有确定性的结果，具有这种性质的散列函数称为单向散列函数。但另一方面，散列函数的输入和输出不是唯一对应关系的，如果两个散列值相同，两个输入值很可能是相同的，但也可能不同，这种情况称为“Hash碰撞”，这通常是两个不同长度的输入值，刻意计算出相同的输出值。输入一些数据计算出散列值，然后部分改变输入值，一个具有强混淆特性的散列函数会产生一个完全不同的散列值。

常见构造Hash函数的算法：

1. 直接寻址法：取key的某个线性函数值为哈希地址。即$H(key) = a*key+b，(a,b为常量)$
2. 数字分析法：比方有一组数据，其中每个数据都由十位数字组成。通过观察可得，每一组数据中前五位有大量重复数据，这样后五位就可以看作是随机的，即可选后五位作为哈希地址。
3. 平方取中法：取key平方后的中间几位作为哈希地址。
4. 折叠法：将key分成位数相同的几部分(最后一部分位数可以不同)，然后叠加和作为哈希地址。比如$12345$即可拆成$12+34+5$。
5. 除留余数法：选定一个统一的基数p，对所有键取余，从而得到对应的哈希地址。即$H(key) = key\mod {p}$。p的选择一般为素数或为表长。

> 对于为什么取素数可以看下面链接，我们的目标是让key和p的最大公约数都为1。
>
> [Hash时取模一定要模质数吗？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/20806796)

### Hash桶

将生成的HashCode分段拆开，每一段称为一个桶，常见的桶便是对结果取余。比如源码中这一段`return ref buckets[hashCode % (uint)buckets.Length]`。

### Hash碰撞

一个哈希函数能够将键转化为数组索引。算法的第二步是**碰撞处理**。也就是处理两个或者多个键的散列值相同的情况。

#### 拉链法

将大小为$M$的数组中的每一个元素指向一条链表，链表中每个节点都存储了散列值为该元素的索引的键值对。这种方法被称为拉链法，因为发生冲突的元素都被存储在链表中。

这个方法的基本思想就是选择足够大的$M$，使所有链表都尽可能保证高效的查找。

> 当然，这里`M`并不是越大越好，选择一个足够大的即可。

查找分为两步：

1. 根据散列值找到对应的链表
2. 沿着链表顺序查找相应的键

#### 线性探测法

用大小为$M$的数组保存$N$个键值对，其中$M > N$。我们需要依靠数组中的**空位**解决碰撞冲突。基于这种策略的方法被统称为开放地址散列表。

其中最简单的方法叫做线性探测法：

当发生碰撞时，我们直接检查散列表的下一个位置。这样的线性探测可能会产生三种结果：

- 命中，该位置的键和被查找的键相同
- 未命中，键为空（该位置没有键）
- 继续查找，该位置的键和被查找的键不同

## Dictionary构造器

```csharp
private struct Entry
{
    public uint hashCode; 
    /// <summary>
    /// 0-based index of next entry in chain: -1 means end of chain
    /// also encodes whether this entry _itself_ is part of the free list by changing sign and subtracting 3,
    /// so -2 means end of free list, -3 means index 0 but on free list, -4 means index 1 but on free list, etc.
    /// </summary>
    public int next; //下一个元素的下标索引
    public TKey key;     // Key of entry
    public TValue value; // Value of entry
}
```

首先引入了`Entry`结构体，这是`Dictionary`中存放数据的最小单位。

```csharp
public class Dictionary<TKey, TValue> : IDictionary<TKey, TValue>, IDictionary, IReadOnlyDictionary<TKey, TValue>,
    ISerializable, IDeserializationCallback where TKey : notnull
{
    private int[]? _buckets; //Hash桶
    private Entry[]? _entries; //Entry数组，用于存放元素
    private int _count; //当前Entries的index位置
    private int _freeList; // 被删除Entry在Entries中的下标index
    private int _freeCount; // 有多少被删除的Entry
    private int _version; // 当前版本，防止迭代过程中集合被更改
    private IEqualityComparer<TKey>? _comparer; // 比较器
    private KeyCollection? _keys; // 存放key的集合
    private ValueCollection? _values; // 存放Value的集合
    private const int StartOfFreeList = -3;
}
```

```csharp
public Dictionary(int capacity, IEqualityComparer<TKey>? comparer)
{
    if (capacity < 0)
    {
        ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.capacity);
    }

    if (capacity > 0)
    {
        Initialize(capacity);
    }

    if (comparer is not null && comparer != EqualityComparer<TKey>.Default) // first check for null to avoid forcing default comparer instantiation unnecessarily
    {
        _comparer = comparer;
    }

    // Special-case EqualityComparer<string>.Default, StringComparer.Ordinal, and StringComparer.OrdinalIgnoreCase.
    // We use a non-randomized comparer for improved perf, falling back to a randomized comparer if the
    // hash buckets become unbalanced.
    if (typeof(TKey) == typeof(string))
    {
        IEqualityComparer<string>? stringComparer = NonRandomizedStringEqualityComparer.GetStringComparer(_comparer);
        if (stringComparer is not null)
        {
            _comparer = (IEqualityComparer<TKey>?)stringComparer;
        }
    }
}
```

```csharp
private int Initialize(int capacity)
{
    int size = HashHelpers.GetPrime(capacity);
    int[] buckets = new int[size];
    Entry[] entries = new Entry[size];

    // Assign member variables after both arrays allocated to guard against corruption from OOM if second fails
    _freeList = -1;
    _buckets = buckets;
    _entries = entries;

    return size;
}
```

可以看到，`Dictionary`在构造的时候做了以下几件事

1. 设置`size`为大于容量的一个最小质数（见目录中HashHelpers源码）。
2. 初始化`int[] buckets`，大小为`size`,用来进行Hash碰撞。
3. 初始化`Entry[] entries`，大小为`size`，用来存储字典的内容，并且标识下一个元素的位置。

## 部分API

### Add

```csharp
public void Add(TKey key, TValue value)
{
    bool modified = TryInsert(key, value, InsertionBehavior.ThrowOnExisting);
    Debug.Assert(modified); // If there was an existing key and the Add failed, an exception will already have been thrown.
}
```

Add这里的英文注释写的很清晰，如果有一个现有的key，并且添加失败了，那么将会抛出一个异常，接下来去看`TryInsert()`方法。

因`TryInsert()`过长，这里选择拆开讲解，完整版见目录。

首先，先看输入参数`TryInsert(key, value, InsertionBehavior.ThrowOnExisting)`。

这里面`key`和`value`都没什么可说的，但是第三个参数可以拉出来看看。

```csharp
internal enum InsertionBehavior : byte
{
    None,
    OverwriteExisting, // 如果存在重复Key重新赋值
    ThrowOnExisting, // 如果存在重复Key抛出异常
}
```

这个参数将会在一会源码中看到用途，现在有个印象即可。

```csharp
IEqualityComparer<TKey>? comparer = _comparer;
    uint hashCode = (uint)((comparer == null) ? key.GetHashCode() : comparer.GetHashCode(key));
```

根据`key`的值或者`comparer`来计算`hashCode`。

```csharp
uint collisionCount = 0;
ref int bucket = ref GetBucket(hashCode);
int i = bucket - 1; // Value in _buckets is 1-based

private ref int GetBucket(uint hashCode)
{
    int[] buckets = _buckets!;
#if TARGET_64BIT
            return ref buckets[HashHelpers.FastMod(hashCode, (uint)buckets.Length, _fastModMultiplier)];
#else
    return ref buckets[hashCode % (uint)buckets.Length];
#endif
}
```

根据`hashCode`计算出应该选择哪个桶。

接卸来大致逻辑可以简单拆成

```csharp
if(comparer == null) //是否自己实现了比较器
{
	if (typeof(TKey).IsValueType){} //Key是否是值类型
	else{}
}
else{} //下面看这个else的内容
```

在具体代码内部中其实都差不多，所以这里选择看实现比较器的部分(也就是注释中说的哪个`else`)。

```csharp
else
{
    while (true)
    {
        // Should be a while loop https://github.com/dotnet/runtime/issues/9422
        // Test uint in if rather than loop condition to drop range check for following array access
        if ((uint)i >= (uint)entries.Length)
        {
            break;
        }

        if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key))
        {
            if (behavior == InsertionBehavior.OverwriteExisting)
            {
                entries[i].value = value;
                return true;
            }

            if (behavior == InsertionBehavior.ThrowOnExisting)
            {
                ThrowHelper.ThrowAddingDuplicateWithKeyArgumentException(key);
            }

            return false;
        }

        i = entries[i].next;

        collisionCount++;
        if (collisionCount > (uint)entries.Length)
        {
            // The chain of entries forms a loop; which means a concurrent update has happened.
            // Break out of the loop and throw, rather than looping forever.
            ThrowHelper.ThrowInvalidOperationException_ConcurrentOperationsNotSupported();
        }
    }
}
```

内部一个死循环，这部分循环有个点是比较隐藏的，那就是如何跳出这个碰撞检测。

如果不发生碰撞冲突，`int i = bucket - 1`根据这个结果即可算出$i=-1$。发生碰撞的话就要分类讨论了，具体可以看源码。当然最终跳出的方案不是返回一个值就是将$i$赋值为$-1$。

```csharp
if ((uint)i >= (uint)entries.Length)
{
	break;
}
```

之后看这里的逻辑，在`uint`也就是无符号整数型下，`i`的大小将会是最大的，因为$-1$的底层表示为$11111111$。

> 当然实际肯定不是八位，这里只是为了简单描述。
>
> 具体是为什么可以看一下csapp或者原码、反码、补码的知识。

在这里就出现了上面提到过的参数`InsertionBehavior`枚举。很明显，如果允许覆盖就覆盖，不允许就抛出异常。

```csharp
int index;
if (_freeCount > 0)
{
    index = _freeList;
    Debug.Assert((StartOfFreeList - entries[_freeList].next) >= -1, "shouldn't overflow because `next` cannot underflow");
    _freeList = StartOfFreeList - entries[_freeList].next;
    _freeCount--;
}
else
{
    int count = _count;
    if (count == entries.Length)
    {
        Resize();
        bucket = ref GetBucket(hashCode);
    }
    index = count;
    _count = count + 1;
    entries = _entries;
}
```

选择合适的位置放入数据。

`freeCount和freeList`在后面`Remove`中再说，不过同样都是为了选择位置。

```csharp
ref Entry entry = ref entries![index];
entry.hashCode = hashCode;
entry.next = bucket - 1; // Value in _buckets is 1-based
entry.key = key;
entry.value = value;
bucket = index + 1; // Value in _buckets is 1-based
_version++;
```

最终`Add()`调用结束后的数值记录。

### Remove

```csharp
public bool Remove(TKey key){}
```

```csharp
if (_buckets != null)
{
    uint collisionCount = 0;
    uint hashCode = (uint)(_comparer?.GetHashCode(key) ?? key.GetHashCode());
    ref int bucket = ref GetBucket(hashCode);
    Entry[]? entries = _entries;
    int last = -1;
    int i = bucket - 1; // Value in buckets is 1-based
    while (i >= 0)
    {
        ///下一段
    }
}
return false;
```

同样还是同过key获取HashCode之后找到桶的位置。`last`用于确定最后一个元素的位置。

```csharp
while (i >= 0)
{
    ref Entry entry = ref entries[i];

    if (entry.hashCode == hashCode &&
        (_comparer?.Equals(entry.key, key) ?? EqualityComparer<TKey>.Default.Equals(entry.key, key)))
    {
        //下一段
    }

    last = i;
    i = entry.next;

    collisionCount++;
    if (collisionCount > (uint)entries.Length)
    {
        // The chain of entries forms a loop; which means a concurrent update has happened.
        // Break out of the loop and throw, rather than looping forever.
        ThrowHelper.ThrowInvalidOperationException_ConcurrentOperationsNotSupported();
    }
}
```

遍历在同一个桶的`entry`直到找到目标元素，或者是碰撞次数过多抛出异常。

```csharp
if (entry.hashCode == hashCode && (_comparer?.Equals(entry.key, key) ?? EqualityComparer<TKey>.Default.Equals(entry.key, key)))
{
    if (last < 0)
    {   // 代表当前是桶的最后一个元素，那么直接赋值即可
        bucket = entry.next + 1; // Value in buckets is 1-based
    }
    else
    {	// 代表当前元素处于链表中间，如果直接删掉会导致链表断开，所以让其头尾相连
        entries[last].next = entry.next;
    }
	// 这里去看entry源码部分
    entry.next = StartOfFreeList - _freeList;
	
    // entry内部数据初始化
    if (RuntimeHelpers.IsReferenceOrContainsReferences<TKey>())
    {
        entry.key = default!;
    }

    if (RuntimeHelpers.IsReferenceOrContainsReferences<TValue>())
    {
        entry.value = default!;
    }
	// 让freeList等于当前位置
    _freeList = i;
    _freeCount++;
    return true;
}
```

这里就提到了`freeList`的作用，可以使下一次`Add`选择该位置

### FindValue

因为还是要区分是否存在比较器来分类讨论，依旧选择存在。

```csharp
uint hashCode = (uint)comparer.GetHashCode(key);
int i = GetBucket(hashCode);
Entry[]? entries = _entries;
uint collisionCount = 0;
i--; // Value in _buckets is 1-based; subtract 1 from i. We do it here so it fuses with the following conditional.
do
{
    // Should be a while loop https://github.com/dotnet/runtime/issues/9422
    // Test in if to drop range check for following array access
    if ((uint)i >= (uint)entries.Length)
    {
        goto ReturnNotFound;
    }

    entry = ref entries[i];
    if (entry.hashCode == hashCode && comparer.Equals(entry.key, key))
    {
        goto ReturnFound;
    }

    i = entry.next;

    collisionCount++;
} while (collisionCount <= (uint)entries.Length);

// The chain of entries forms a loop; which means a concurrent update has happened.
// Break out of the loop and throw, rather than looping forever.
goto ConcurrentOperation;
```

经历了前面增删，这个查找其实变得就很清晰。说白了就是在桶里面跑一边找到就返回，找不到就寄。

不过需要注意的是这里面用了很多`goto`建议在目录大致过一遍即可。

```csharp
//外部
ConcurrentOperation:
	ThrowHelper.ThrowInvalidOperationException_ConcurrentOperationsNotSupported();
ReturnFound:
	ref TValue value = ref entry.value;
Return:
	return ref value;
ReturnNotFound:
	value = ref Unsafe.NullRef<TValue>();
goto Return;
```

说实话，这是我第一次见到`goto`。。。

### Resize

```csharp
private void Resize() => Resize(HashHelpers.ExpandPrime(_count), false);
```

`HashHelpers.ExpandPrime(_count)`方法可以在目录中源码寻找，不过这里大致就理解成**大于两倍大小的最小素数**就行了

```csharp
//精简版，完整版看目录
private void Resize(int newSize, bool forceNewHashCodes)
{
    Entry[] entries = new Entry[newSize];
    int count = _count;
    Array.Copy(_entries, entries, count);
    // Assign member variables after both arrays allocated to guard against corruption from OOM if second fails
    _buckets = new int[newSize];
    for (int i = 0; i < count; i++)
    {
        if (entries[i].next >= -1)
        {
            ref int bucket = ref GetBucket(entries[i].hashCode);
            entries[i].next = bucket - 1; // Value in _buckets is 1-based
            bucket = i + 1;
        }
    }
    _entries = entries;
}
```

可以看出扩容操作其实就是，申请新的`Entry`和`buckets`，之后将现有的元素拷贝进去。

## 索引器

```csharp
public TValue this[TKey key]
{
    get
    {
        ref TValue value = ref FindValue(key);
        if (!Unsafe.IsNullRef(ref value))
        {
            return value;
        }

        ThrowHelper.ThrowKeyNotFoundException(key);
        return default;
    }

    set
    {
        bool modified = TryInsert(key, value, InsertionBehavior.OverwriteExisting);
        Debug.Assert(modified);
    }
}
```

`get`既查找是否有`key`。

`set`即插入。注意参数为`InsertionBehavior.OverwriteExisting`，所以可以覆盖元素。

## 迭代器

这里虽然我还没有写！！但是在`foreach`中是可以删除元素的，不能增加元素，看了源码，会发现删除不会修改版本号。！！

## 源码目录

### HashHelpers类部分源码

```csharp
// Licensed to the .NET Foundation under one or more agreements.
// The .NET Foundation licenses this file to you under the MIT license.

using System.Diagnostics;
using System.Runtime.CompilerServices;

namespace System.Collections
{
    internal static partial class HashHelpers
    {
        public const uint HashCollisionThreshold = 100;

        // This is the maximum prime smaller than Array.MaxLength.
        public const int MaxPrimeArrayLength = 0x7FFFFFC3;

        public const int HashPrime = 101;

        // Table of prime numbers to use as hash table sizes.
        // A typical resize algorithm would pick the smallest prime number in this array
        // that is larger than twice the previous capacity.
        // Suppose our Hashtable currently has capacity x and enough elements are added
        // such that a resize needs to occur. Resizing first computes 2x then finds the
        // first prime in the table greater than 2x, i.e. if primes are ordered
        // p_1, p_2, ..., p_i, ..., it finds p_n such that p_n-1 < 2x < p_n.
        // Doubling is important for preserving the asymptotic complexity of the
        // hashtable operations such as add.  Having a prime guarantees that double
        // hashing does not lead to infinite loops.  IE, your hash function will be
        // h1(key) + i*h2(key), 0 <= i < size.  h2 and the size must be relatively prime.
        // We prefer the low computation costs of higher prime numbers over the increased
        // memory allocation of a fixed prime number i.e. when right sizing a HashSet.
        private static readonly int[] s_primes =
        {
            3, 7, 11, 17, 23, 29, 37, 47, 59, 71, 89, 107, 131, 163, 197, 239, 293, 353, 431, 521, 631, 761, 919,
            1103, 1327, 1597, 1931, 2333, 2801, 3371, 4049, 4861, 5839, 7013, 8419, 10103, 12143, 14591,
            17519, 21023, 25229, 30293, 36353, 43627, 52361, 62851, 75431, 90523, 108631, 130363, 156437,
            187751, 225307, 270371, 324449, 389357, 467237, 560689, 672827, 807403, 968897, 1162687, 1395263,
            1674319, 2009191, 2411033, 2893249, 3471899, 4166287, 4999559, 5999471, 7199369
        };

        public static bool IsPrime(int candidate)
        {
            if ((candidate & 1) != 0)
            {
                int limit = (int)Math.Sqrt(candidate);
                for (int divisor = 3; divisor <= limit; divisor += 2)
                {
                    if ((candidate % divisor) == 0)
                        return false;
                }
                return true;
            }
            return candidate == 2;
        }

        public static int GetPrime(int min)
        {
            if (min < 0)
                throw new ArgumentException(SR.Arg_HTCapacityOverflow);

            foreach (int prime in s_primes)
            {
                if (prime >= min)
                    return prime;
            }

            // Outside of our predefined table. Compute the hard way.
            for (int i = (min | 1); i < int.MaxValue; i += 2)
            {
                if (IsPrime(i) && ((i - 1) % HashPrime != 0))
                    return i;
            }
            return min;
        }

        // Returns size of hashtable to grow to.
        public static int ExpandPrime(int oldSize)
        {
            int newSize = 2 * oldSize;

            // Allow the hashtables to grow to maximum possible size (~2G elements) before encountering capacity overflow.
            // Note that this check works even when _items.Length overflowed thanks to the (uint) cast
            if ((uint)newSize > MaxPrimeArrayLength && MaxPrimeArrayLength > oldSize)
            {
                Debug.Assert(MaxPrimeArrayLength == GetPrime(MaxPrimeArrayLength), "Invalid MaxPrimeArrayLength");
                return MaxPrimeArrayLength;
            }

            return GetPrime(newSize);
        }

        /// <summary>Returns approximate reciprocal of the divisor: ceil(2**64 / divisor).</summary>
        /// <remarks>This should only be used on 64-bit.</remarks>
        public static ulong GetFastModMultiplier(uint divisor) =>
            ulong.MaxValue / divisor + 1;

        /// <summary>Performs a mod operation using the multiplier pre-computed with <see cref="GetFastModMultiplier"/>.</summary>
        /// <remarks>This should only be used on 64-bit.</remarks>
        [MethodImpl(MethodImplOptions.AggressiveInlining)]
        public static uint FastMod(uint value, uint divisor, ulong multiplier)
        {
            // We use modified Daniel Lemire's fastmod algorithm (https://github.com/dotnet/runtime/pull/406),
            // which allows to avoid the long multiplication if the divisor is less than 2**31.
            Debug.Assert(divisor <= int.MaxValue);

            // This is equivalent of (uint)Math.BigMul(multiplier * value, divisor, out _). This version
            // is faster than BigMul currently because we only need the high bits.
            uint highbits = (uint)(((((multiplier * value) >> 32) + 1) * divisor) >> 32);

            Debug.Assert(highbits == value % divisor);
            return highbits;
        }
    }
}
```

### TryInsert源码

```csharp
private bool TryInsert(TKey key, TValue value, InsertionBehavior behavior)
{
    // NOTE: this method is mirrored in CollectionsMarshal.GetValueRefOrAddDefault below.
    // If you make any changes here, make sure to keep that version in sync as well.

    if (key == null)
    {
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);
    }

    if (_buckets == null)
    {
        Initialize(0);
    }
    Debug.Assert(_buckets != null);

    Entry[]? entries = _entries;
    Debug.Assert(entries != null, "expected entries to be non-null");

    IEqualityComparer<TKey>? comparer = _comparer;
    uint hashCode = (uint)((comparer == null) ? key.GetHashCode() : comparer.GetHashCode(key));

    uint collisionCount = 0;
    ref int bucket = ref GetBucket(hashCode);
    int i = bucket - 1; // Value in _buckets is 1-based

    if (comparer == null)
    {
        if (typeof(TKey).IsValueType)
        {
            // ValueType: Devirtualize with EqualityComparer<TValue>.Default intrinsic
            while (true)
            {
                // Should be a while loop https://github.com/dotnet/runtime/issues/9422
                // Test uint in if rather than loop condition to drop range check for following array access
                if ((uint)i >= (uint)entries.Length)
                {
                    break;
                }

                if (entries[i].hashCode == hashCode && EqualityComparer<TKey>.Default.Equals(entries[i].key, key))
                {
                    if (behavior == InsertionBehavior.OverwriteExisting)
                    {
                        entries[i].value = value;
                        return true;
                    }

                    if (behavior == InsertionBehavior.ThrowOnExisting)
                    {
                        ThrowHelper.ThrowAddingDuplicateWithKeyArgumentException(key);
                    }

                    return false;
                }

                i = entries[i].next;

                collisionCount++;
                if (collisionCount > (uint)entries.Length)
                {
                    // The chain of entries forms a loop; which means a concurrent update has happened.
                    // Break out of the loop and throw, rather than looping forever.
                    ThrowHelper.ThrowInvalidOperationException_ConcurrentOperationsNotSupported();
                }
            }
        }
        else
        {
            // Object type: Shared Generic, EqualityComparer<TValue>.Default won't devirtualize
            // https://github.com/dotnet/runtime/issues/10050
            // So cache in a local rather than get EqualityComparer per loop iteration
            EqualityComparer<TKey> defaultComparer = EqualityComparer<TKey>.Default;
            while (true)
            {
                // Should be a while loop https://github.com/dotnet/runtime/issues/9422
                // Test uint in if rather than loop condition to drop range check for following array access
                if ((uint)i >= (uint)entries.Length)
                {
                    break;
                }

                if (entries[i].hashCode == hashCode && defaultComparer.Equals(entries[i].key, key))
                {
                    if (behavior == InsertionBehavior.OverwriteExisting)
                    {
                        entries[i].value = value;
                        return true;
                    }

                    if (behavior == InsertionBehavior.ThrowOnExisting)
                    {
                        ThrowHelper.ThrowAddingDuplicateWithKeyArgumentException(key);
                    }

                    return false;
                }

                i = entries[i].next;

                collisionCount++;
                if (collisionCount > (uint)entries.Length)
                {
                    // The chain of entries forms a loop; which means a concurrent update has happened.
                    // Break out of the loop and throw, rather than looping forever.
                    ThrowHelper.ThrowInvalidOperationException_ConcurrentOperationsNotSupported();
                }
            }
        }
    }
    else
    {
        while (true)
        {
            // Should be a while loop https://github.com/dotnet/runtime/issues/9422
            // Test uint in if rather than loop condition to drop range check for following array access
            if ((uint)i >= (uint)entries.Length)
            {
                break;
            }

            if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key))
            {
                if (behavior == InsertionBehavior.OverwriteExisting)
                {
                    entries[i].value = value;
                    return true;
                }

                if (behavior == InsertionBehavior.ThrowOnExisting)
                {
                    ThrowHelper.ThrowAddingDuplicateWithKeyArgumentException(key);
                }

                return false;
            }

            i = entries[i].next;

            collisionCount++;
            if (collisionCount > (uint)entries.Length)
            {
                // The chain of entries forms a loop; which means a concurrent update has happened.
                // Break out of the loop and throw, rather than looping forever.
                ThrowHelper.ThrowInvalidOperationException_ConcurrentOperationsNotSupported();
            }
        }
    }

    int index;
    if (_freeCount > 0)
    {
        index = _freeList;
        Debug.Assert((StartOfFreeList - entries[_freeList].next) >= -1, "shouldn't overflow because `next` cannot underflow");
        _freeList = StartOfFreeList - entries[_freeList].next;
        _freeCount--;
    }
    else
    {
        int count = _count;
        if (count == entries.Length)
        {
            Resize();
            bucket = ref GetBucket(hashCode);
        }
        index = count;
        _count = count + 1;
        entries = _entries;
    }

    ref Entry entry = ref entries![index];
    entry.hashCode = hashCode;
    entry.next = bucket - 1; // Value in _buckets is 1-based
    entry.key = key;
    entry.value = value;
    bucket = index + 1; // Value in _buckets is 1-based
    _version++;

    // Value types never rehash
    if (!typeof(TKey).IsValueType && collisionCount > HashHelpers.HashCollisionThreshold && comparer is NonRandomizedStringEqualityComparer)
    {
        // If we hit the collision threshold we'll need to switch to the comparer which is using randomized string hashing
        // i.e. EqualityComparer<string>.Default.
        Resize(entries.Length, true);
    }

    return true;
}
```

### Resize源码

```csharp
private void Resize() => Resize(HashHelpers.ExpandPrime(_count), false);
private void Resize(int newSize, bool forceNewHashCodes)
{
    // Value types never rehash
    Debug.Assert(!forceNewHashCodes || !typeof(TKey).IsValueType);
    Debug.Assert(_entries != null, "_entries should be non-null");
    Debug.Assert(newSize >= _entries.Length);

    Entry[] entries = new Entry[newSize];

    int count = _count;
    Array.Copy(_entries, entries, count);

    if (!typeof(TKey).IsValueType && forceNewHashCodes)
    {
        Debug.Assert(_comparer is NonRandomizedStringEqualityComparer);
        _comparer = (IEqualityComparer<TKey>)((NonRandomizedStringEqualityComparer)_comparer)
            .GetRandomizedEqualityComparer();

        for (int i = 0; i < count; i++)
        {
            if (entries[i].next >= -1)
            {
                entries[i].hashCode = (uint)_comparer.GetHashCode(entries[i].key);
            }
        }

        if (ReferenceEquals(_comparer, EqualityComparer<TKey>.Default))
        {
            _comparer = null;
        }
    }

    // Assign member variables after both arrays allocated to guard against corruption from OOM if second fails
    _buckets = new int[newSize];
#if TARGET_64BIT
            _fastModMultiplier = HashHelpers.GetFastModMultiplier((uint)newSize);
#endif
    for (int i = 0; i < count; i++)
    {
        if (entries[i].next >= -1)
        {
            ref int bucket = ref GetBucket(entries[i].hashCode);
            entries[i].next = bucket - 1; // Value in _buckets is 1-based
            bucket = i + 1;
        }
    }

    _entries = 0;
}
```

### Remove源码

```csharp
public bool Remove(TKey key)
{
    // The overload Remove(TKey key, out TValue value) is a copy of this method with one additional
    // statement to copy the value for entry being removed into the output parameter.
    // Code has been intentionally duplicated for performance reasons.

    if (key == null)
    {
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);
    }

    if (_buckets != null)
    {
        Debug.Assert(_entries != null, "entries should be non-null");
        uint collisionCount = 0;
        uint hashCode = (uint)(_comparer?.GetHashCode(key) ?? key.GetHashCode());
        ref int bucket = ref GetBucket(hashCode);
        Entry[]? entries = _entries;
        int last = -1;
        int i = bucket - 1; // Value in buckets is 1-based
        while (i >= 0)
        {
            ref Entry entry = ref entries[i];

            if (entry.hashCode == hashCode &&
                (_comparer?.Equals(entry.key, key) ?? EqualityComparer<TKey>.Default.Equals(entry.key, key)))
            {
                if (last < 0)
                {
                    bucket = entry.next + 1; // Value in buckets is 1-based
                }
                else
                {
                    entries[last].next = entry.next;
                }

                Debug.Assert((StartOfFreeList - _freeList) < 0,
                    "shouldn't underflow because max hashtable length is MaxPrimeArrayLength = 0x7FEFFFFD(2146435069) _freelist underflow threshold 2147483646");
                entry.next = StartOfFreeList - _freeList;

                if (RuntimeHelpers.IsReferenceOrContainsReferences<TKey>())
                {
                    entry.key = default!;
                }

                if (RuntimeHelpers.IsReferenceOrContainsReferences<TValue>())
                {
                    entry.value = default!;
                }

                _freeList = i;
                _freeCount++;
                return true;
            }

            last = i;
            i = entry.next;

            collisionCount++;
            if (collisionCount > (uint)entries.Length)
            {
                // The chain of entries forms a loop; which means a concurrent update has happened.
                // Break out of the loop and throw, rather than looping forever.
                ThrowHelper.ThrowInvalidOperationException_ConcurrentOperationsNotSupported();
            }
        }
    }

    return false;
}
```

### FindValue源码

```csharp
internal ref TValue FindValue(TKey key)
{
    if (key == null)
    {
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);
    }

    ref Entry entry = ref Unsafe.NullRef<Entry>();
    if (_buckets != null)
    {
        Debug.Assert(_entries != null, "expected entries to be != null");
        IEqualityComparer<TKey>? comparer = _comparer;
        if (comparer == null)
        {
            uint hashCode = (uint)key.GetHashCode();
            int i = GetBucket(hashCode);
            Entry[]? entries = _entries;
            uint collisionCount = 0;
            if (typeof(TKey).IsValueType)
            {
                // ValueType: Devirtualize with EqualityComparer<TValue>.Default intrinsic

                i--; // Value in _buckets is 1-based; subtract 1 from i. We do it here so it fuses with the following conditional.
                do
                {
                    // Should be a while loop https://github.com/dotnet/runtime/issues/9422
                    // Test in if to drop range check for following array access
                    if ((uint)i >= (uint)entries.Length)
                    {
                        goto ReturnNotFound;
                    }

                    entry = ref entries[i];
                    if (entry.hashCode == hashCode && EqualityComparer<TKey>.Default.Equals(entry.key, key))
                    {
                        goto ReturnFound;
                    }

                    i = entry.next;

                    collisionCount++;
                } while (collisionCount <= (uint)entries.Length);

                // The chain of entries forms a loop; which means a concurrent update has happened.
                // Break out of the loop and throw, rather than looping forever.
                goto ConcurrentOperation;
            }
            else
            {
                // Object type: Shared Generic, EqualityComparer<TValue>.Default won't devirtualize
                // https://github.com/dotnet/runtime/issues/10050
                // So cache in a local rather than get EqualityComparer per loop iteration
                EqualityComparer<TKey> defaultComparer = EqualityComparer<TKey>.Default;

                i--; // Value in _buckets is 1-based; subtract 1 from i. We do it here so it fuses with the following conditional.
                do
                {
                    // Should be a while loop https://github.com/dotnet/runtime/issues/9422
                    // Test in if to drop range check for following array access
                    if ((uint)i >= (uint)entries.Length)
                    {
                        goto ReturnNotFound;
                    }

                    entry = ref entries[i];
                    if (entry.hashCode == hashCode && defaultComparer.Equals(entry.key, key))
                    {
                        goto ReturnFound;
                    }

                    i = entry.next;

                    collisionCount++;
                } while (collisionCount <= (uint)entries.Length);

                // The chain of entries forms a loop; which means a concurrent update has happened.
                // Break out of the loop and throw, rather than looping forever.
                goto ConcurrentOperation;
            }
        }
        else
        {
            uint hashCode = (uint)comparer.GetHashCode(key);
            int i = GetBucket(hashCode);
            Entry[]? entries = _entries;
            uint collisionCount = 0;
            i--; // Value in _buckets is 1-based; subtract 1 from i. We do it here so it fuses with the following conditional.
            do
            {
                // Should be a while loop https://github.com/dotnet/runtime/issues/9422
                // Test in if to drop range check for following array access
                if ((uint)i >= (uint)entries.Length)
                {
                    goto ReturnNotFound;
                }

                entry = ref entries[i];
                if (entry.hashCode == hashCode && comparer.Equals(entry.key, key))
                {
                    goto ReturnFound;
                }

                i = entry.next;

                collisionCount++;
            } while (collisionCount <= (uint)entries.Length);

            // The chain of entries forms a loop; which means a concurrent update has happened.
            // Break out of the loop and throw, rather than looping forever.
            goto ConcurrentOperation;
        }
    }

    goto ReturnNotFound;

ConcurrentOperation:
    ThrowHelper.ThrowInvalidOperationException_ConcurrentOperationsNotSupported();
ReturnFound:
    ref TValue value = ref entry.value;
Return:
    return ref value;
ReturnNotFound:
    value = ref Unsafe.NullRef<TValue>();
    goto Return;
}
```