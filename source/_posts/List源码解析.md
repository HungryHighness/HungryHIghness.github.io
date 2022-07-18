---
title: List源码解析
date: 2022-02-23 13:34:21
tags: c#
categories: 语言基础
---

# List源码解析

## List构造器

```csharp
public class List<T> : IList<T>, IList, IReadOnlyList<T>
{
	private const int DefaultCapacity = 4;

	internal T[] _items; // Do not rename (binary serialization)
 	internal int _size; // Do not rename (binary serialization)
	private int _version; // Do not rename (binary serialization)
 	private static readonly T[] s_emptyArray = new T[0];
	public List()
     {
  		_items = s_emptyArray;
     }

  	// Constructs a List with a given initial capacity. The list is
  	// initially empty, but will have room for the given number of elements
	// before any reallocations are required.
  	//
 	public List(int capacity)
  	{
		if (capacity < 0)
         	ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.capacity, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);

     	if (capacity == 0)
        	_items = s_emptyArray;
    	else
           	_items = new T[capacity];
 	}
}
```

从构造函数部分可以看出List的底层是由数组构造的，部分人或许以为是链表，但链表在C#中是LinkedList

## List部分API

### Add

```csharp
public void Add(T item)
{
    _version++;
    T[] array = _items;
    int size = _size;
    if ((uint)size < (uint)array.Length)
    {
        _size = size + 1;
        array[size] = item;
    }
    else
    {
        AddWithResize(item);
    }
}
```

**Add**函数每次会判断当前容量是否足够，如果足够放入即可，不够的话调用`AddWithResize(item)`

```csharp
private void AddWithResize(T item)
{
    Debug.Assert(_size == _items.Length);
    int size = _size;
    Grow(size + 1);
    _size = size + 1;
    _items[size] = item;
}
```

从名字也可以看出，**AddWithResize**的作用是添加并且调整大小。其中调整大小的部分由**Grow**实现

```csharp
private void Grow(int capacity)
{
    Debug.Assert(_items.Length < capacity);

    int newcapacity = _items.Length == 0 ? DefaultCapacity : 2 * _items.Length;

    // Allow the list to grow to maximum possible capacity (~2G elements) before encountering overflow.
    // Note that this check works even when _items.Length overflowed thanks to the (uint) cast
    if ((uint)newcapacity > Array.MaxLength) newcapacity = Array.MaxLength;

    // If the computed capacity is still less than specified, set to the original argument.
    // Capacities exceeding Array.MaxLength will be surfaced as OutOfMemoryException by Array.Resize.
    if (newcapacity < capacity) newcapacity = capacity;

    Capacity = newcapacity;
}
```

```csharp
public int Capacity
{
    get => _items.Length;
    set
    {
        if (value < _size)
        {
            ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.value, ExceptionResource.ArgumentOutOfRange_SmallCapacity);
        }

        if (value != _items.Length)
        {
            if (value > 0)
            {
                T[] newItems = new T[value];
                if (_size > 0)
                {
                    Array.Copy(_items, newItems, _size);
                }
                _items = newItems;
            }
            else
            {
                _items = s_emptyArray;
            }
        }
    }
}
```

从这里也可以看出，如果初始不给List赋值大小的话默认大小为4，如果再频繁调用`Add`，每次new都会产生内存垃圾，给GC带来了负担。这里需要注意每次扩容都是两倍，并且扩容之后是通过调用` Array.Copy(_items, newItems, _size)`从原数组拷贝生成到新数组。

### Remove

```csharp
public bool Remove(T item)
{
    int index = IndexOf(item);
    if (index >= 0)
    {
        RemoveAt(index);
        return true;
    }

    return false;
}
```

```csharp
public int IndexOf(T item)
    => Array.IndexOf(_items, item, 0, _size);
```

```csharp
public void RemoveAt(int index)
{
    if ((uint)index >= (uint)_size)
    {
        ThrowHelper.ThrowArgumentOutOfRange_IndexException();
    }
    _size--;
    if (index < _size)
    {
        Array.Copy(_items, index + 1, _items, index, _size - index);
    }
    if (RuntimeHelpers.IsReferenceOrContainsReferences<T>())
    {
        _items[_size] = default!;
    }
    _version++;
}
```

> `Array.IndexOf` 方法在这里需要注意的是返回值
>
> 如果找到，则为 `array` 中 `value` 的第一个匹配项的索引；否则为该数组的下限减 1。
>
> 具体可查看[Array.IndexOf 方法 (System) | Microsoft Docs](https://docs.microsoft.com/zh-cn/dotnet/api/system.array.indexof?view=net-6.0)
>
> `RuntimeHelpers.IsReferenceOrContainsReferences<T>`方法在这里需要注意的是返回值
>
> 如果给定类型是引用类型或包含引用的值类型，则为 `true`；否则为 `false`。
>
> 具体可查看[RuntimeHelpers.IsReferenceOrContainsReferences 方法 (System.Runtime.CompilerServices) | Microsoft Docs](https://docs.microsoft.com/zh-cn/dotnet/api/system.runtime.compilerservices.runtimehelpers.isreferenceorcontainsreferences?view=net-6.0)

从源码中即可看出，删除的原理其实就是元素覆盖，通过`Array.IndexOf`找到元素索引，之后调用`Array.Copy`覆盖被删除的元素。

### Insert

```csharp
public void Insert(int index, T item)
{
    // Note that insertions at the end are legal.
    if ((uint)index > (uint)_size)
    {
        ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.index, ExceptionResource.ArgumentOutOfRange_ListInsert);
    }
    if (_size == _items.Length) Grow(_size + 1);
    if (index < _size)
    {
        Array.Copy(_items, index, _items, index + 1, _size - index);
    }
    _items[index] = item;
    _size++;
    _version++;
}
```

同样，先检查容量是否足够，之后向后覆盖，接下来将元素放入即可。

### Clear

```csharp
public void Clear()
{
    _version++;
    if (RuntimeHelpers.IsReferenceOrContainsReferences<T>())
    {
        int size = _size;
        _size = 0;
        if (size > 0)
        {
            Array.Clear(_items, 0, size); // Clear the elements so that the gc can reclaim the references.
        }
    }
    else
    {
        _size = 0;
    }
}
```

> `Array.Clear`需要注意这不是删除方法，他是将数组中每个元素重置为元素的默认值。 它将引用类型的元素设置 (包括string) 到的元素 `null` ，并将值类型的元素设置为下表中显示的默认值。
>
> [Array.Clear(Array, Int32, Int32) 方法 (System) | Microsoft Docs](https://docs.microsoft.com/zh-cn/dotnet/api/system.array.clear?view=net-6.0)中注解也提到了
>
> **此方法仅清除元素的值;它不会删除元素本身。 数组具有固定的大小;因此，无法添加或删除元素。**

### Contains

```csharp
public bool Contains(T item)
{
    // PERF: IndexOf calls Array.IndexOf, which internally
    // calls EqualityComparer<T>.Default.IndexOf, which
    // is specialized for different types. This
    // boosts performance since instead of making a
    // virtual method call each iteration of the loop,
    // via EqualityComparer<T>.Default.Equals, we
    // only make one virtual call to EqualityComparer.IndexOf.

    return _size != 0 && IndexOf(item) != -1;
}
```

同样这里是通过`IndexOf`来寻找元素

### ToArray

```csharp
public T[] ToArray()
{
    if (_size == 0)
    {
        return s_emptyArray;
    }

    T[] array = new T[_size];
    Array.Copy(_items, array, _size);
    return array;
}
```

new一个新的数组，拷贝一份返回

### Sort

```csharp
public void Sort(int index, int count, IComparer<T>? comparer)
{
    if (index < 0)
    {
        ThrowHelper.ThrowIndexArgumentOutOfRange_NeedNonNegNumException();
    }

    if (count < 0)
    {
        ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.count, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
    }

    if (_size - index < count)
        ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_InvalidOffLen);

    if (count > 1)
    {
        Array.Sort<T>(_items, index, count, comparer);
    }
    _version++;
}
```

调用`Array.Sort<T>(_items, index, count, comparer)`进行排序

> 这里需要注意的是Sort函数不是单纯的快速排序，官网中同样也提到了，这里我将内容贴出来。[Array.Sort 方法 (System) | Microsoft Docs](https://docs.microsoft.com/zh-cn/dotnet/api/system.array.sort?view=net-6.0#system-array-sort(system-array))
>
> 此方法使用的是反省sort(introsort)算法，如下所示：
>
> - 如果分区大小小于或等于16个元素，则它将使用 [插入排序](https://en.wikipedia.org/wiki/Insertion_sort) 算法
> - 如果分区超过$2*log^n$,其中`n`为输入数组的范围，则使用则使用 [Heapsort](https://en.wikipedia.org/wiki/Heapsort) 算法(堆排序)。
> - 否则，它使用的是[快速排序](https://en.wikipedia.org/wiki/Quicksort) 算法
>
> 此实现执行不稳定的排序;也就是说，如果两个元素相等，则可能不会保留它们的顺序。 相反，稳定排序会保留相等元素的顺序。
>
> 此方法是一个 $O(nlog^n)$ 操作，其中 `n` 是 `length`。
>
> PS：当然如果你点进去并且看的是中文的话，你会发现很明显的翻译错误(狗头)

## 索引器

```csharp
public T this[int index]
{
	get
	{
    	// Following trick can reduce the range check by one
   		 if ((uint)index >= (uint)_size)
   		 {
     	  	 ThrowHelper.ThrowArgumentOutOfRange_IndexException();
   		 }
   		return _items[index];
	}

	set
	{
   	 	if ((uint)index >= (uint)_size)
    	{
     	   ThrowHelper.ThrowArgumentOutOfRange_IndexException();
    	}
    	_items[index] = value;
    	_version++;
	}
}
```

使用数组索引获取元素

## 迭代器

先分开讲解源码中比较重要的两个元素**IEnumerable**和**IEnumerator**。

### IEnumerable

```csharp
public interface IEnumerable
{
  IEnumerator GetEnumerator();
}
```

公开枚举数，该枚举数支持在非泛型集合上进行简单迭代。(可枚举类型)。

当`foreach`遍历可枚举类型的时候，比如`List`，他会调用`GetEnumerator`方法来获取Enumerator(枚举数)，接下来从枚举数中请求每一项并且作为迭代变量。

[IEnumerable 接口 (System.Collections) | Microsoft Docs](https://docs.microsoft.com/zh-cn/dotnet/api/system.collections.ienumerable?view=net-6.0)

### IEnumerator

```csharp
public interface IEnumerator
{
    bool MoveNext();

    object Current { get; }

    void Reset();
}
```

支持对非泛型集合的简单迭代。(枚举数)

该接口包含三个成员。

1. `bool MoveNext()`从集合的一个元素移动到下个元素，同时检查是否遍历完毕
2. 只读属性`Current`用于返回当前元素
3. `Reset`重置为初始态，一般用于报错(bushi)，最好永远不要调用它，如果要重新开始枚举，重新创建一个新的即可。

[IEnumerator 接口 (System.Collections) | Microsoft Docs](https://docs.microsoft.com/zh-cn/dotnet/api/system.collections.ienumerator?view=net-6.0)

### 源码

```csharp
public Enumerator GetEnumerator() => new Enumerator(this);
IEnumerator<T> IEnumerable<T>.GetEnumerator() => new Enumerator(this);
IEnumerator IEnumerable.GetEnumerator() => new Enumerator(this);
public struct Enumerator : IEnumerator<T>, IEnumerator
{
    private readonly List<T> _list;
    private int _index;
    private readonly int _version;
    private T? _current;
    internal Enumerator(List<T> list)
    {
        _list = list;
        _index = 0;
        _version = list._version;
        _current = default;
    }

    public void Dispose()
    {
    }

    public bool MoveNext()
    {
        List<T> localList = _list;

        if (_version == localList._version && ((uint)_index < (uint)localList._size))
        {
            _current = localList._items[_index];
            _index++;
            return true;
        }

        return MoveNextRare();
    }

    private bool MoveNextRare()
    {
        if (_version != _list._version)
        {
            ThrowHelper.ThrowInvalidOperationException_InvalidOperation_EnumFailedVersion();
        }

        _index = _list._size + 1;
        _current = default;
        return false;
    }

    public T Current => _current!;

    object? IEnumerator.Current
    {
        get
        {
            if (_index == 0 || _index == _list._size + 1)
            {
                ThrowHelper.ThrowInvalidOperationException_InvalidOperation_EnumOpCantHappen();
            }

            return Current;
        }
    }

    void IEnumerator.Reset()
    {
        if (_version != _list._version)
        {
            ThrowHelper.ThrowInvalidOperationException_InvalidOperation_EnumFailedVersion();
        }

        _index = 0;
        _current = default;
    }
}
```
上述模式大致可以简单描述为

```csharp
//当然具体不是这样的
int number = 0;
var enumerator = list.GetEnumerator();
while(enumerator.MoveNext())
{
	number = enumerator.Current;
}
```

或许会有人觉得`GetEnumerator()`有点多此一举,觉得可以像下面这么写也能做到一样的效果

```csharp
int number = 0;
while(list.MoveNext())
{
	number = list.Current;
}
```

但这样，如果两个循环交错遍历同一个集合，或者是多线程循环，交错的循环将会产生干扰。所以不使用该方法，而是用`IEnumerator`来支持`IEnumerator`并且负责维护循环遍历的状态。这样遍历集合便不会产生影响。

Ps:不是非要实现IEnumerable才能对类型进行遍历。遍历器使用了一个名为"Duck typing"的概念，他会查找一个返回“包含Current属性和MoveNext()方法的一个类型”的GetEnumerator()方法。Duck typing按照名称查找方法，而不依赖接口或者显式方法调用。只有当找不到实现时，才会检查集合是否实现了接口。
