# Vector源码解析

<img src="https://cdn.nlark.com/yuque/0/2019/png/650477/1577083448563-a6b8724e-60f5-41e9-86c3-1c08f718533d.png#align=left&amp;display=inline&amp;height=768&amp;originHeight=768&amp;originWidth=1594&amp;size=0&amp;status=done&amp;style=none&amp;width=1594"  />

# 1. 概述

自Java1.2版本起，Vector类添加了List接口的实现，使其成为了Java集合体系中的一员

Vector是一个底层使用数组实现，线程安全的List集合

> 我觉得这就是一个线程安全的ArrayList😊


# 2. 成员变量

```java
/**
     * 存储元素的数组
     */
    protected Object[] elementData;

    /**
     * 数组上有效元素的个数
     */
    protected int elementCount;

    /**
     * Vector容量增长的数量
     * [1] 如果capacityIncrement小于或等于零，那么Vector容量每次以双倍的形式增长
     */
    protected int capacityIncrement;
    
    /**
     * 最大容量
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

[1]处，`Vector`的扩容策略跟`ArrayList`的稍微不一样，允许调用者主动指定数组扩容时的扩容数量，扩容的详细过程在`java.util.Vector#grow`方法，下文会详细解释

# 3. 构造函数

```java
public Vector() {
        // 调用重载构造函数，初始容量为10
        this(10);
    }
    
    public Vector(int initialCapacity) {
        // 调用重载构造函数，指定初始的容量，已经容量增长数量为0
        // 也就是说在数组扩容的时候，扩容为原来的两倍
        this(initialCapacity, 0);
    }
    
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }
    
    public Vector(Collection<? extends E> c) {
        elementData = c.toArray();
        elementCount = elementData.length;
        // [1] c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
    }
```

[1]处，关于toArray方法不一定返回Object数组，可以看看这篇文章
[c.toArray might not return Object[]?](https://www.cnblogs.com/liqing-weikeyuan/p/7922306.html)

# 4. 常用方法

`Vector`的线程同步的方式是对外提供的增删改方法中添加`synchronized`关键字，加锁的粒度比较粗，注定并发性能不高

`Vector`添加元素的实现跟`ArrayList`差不多，流程基本是检查数据空间，空间不够则进行扩容，然后再添加元素

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083448561-7775c37c-dee7-420d-b6d1-421f63fd8066.png#align=left&display=inline&height=753&originHeight=753&originWidth=1017&size=0&status=done&style=none&width=1017)

- `add*`包括了`java.util.Vector#add(E)`、`java.util.Vector#addElement`、`java.util.Vector#addAll(java.util.Collection<? extends E>)`

接下来直接看看常用的方法的源码实现

## java.util.Vector#add(E)

```java
public synchronized boolean add(E e) {
        // 修改次数加1
        modCount++;
        // 确保数组空间足够
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }
    
    private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            // 容量不够，则进行扩容
            grow(minCapacity);
    }
    
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 当capacityIncrement = 0的时候，扩容后的newCapacity是原来oldCapacity的两倍
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        // 如果扩容以后，newCapacity还是小于最小需要的容量的话，newCapacity = minCapacity
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 如果扩容后的newCapacity大于数组长度最大值，则newCapacity取数组长度最大值
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 将元素复制到扩容后的新数组中
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

从`grow`方法可以看到Vector的扩容策略，如果新建Vector对象的时候没有指定`capacityIncrement`（默认为0），那么Vector的扩容策略为**将数组扩容为原来的两倍**，如果指定了`capacityIncrement`的大小，则**将数组扩大capacityIncrement的长度**，然后再比较扩容后`newCapacity`与数组最小需要的`minCapacity`的大小，如果`newCapacity`还是小于`minCapacity`，则`newCapacity = minCapacity`

## java.util.Vector#add(int, E)

```java
public void add(int index, E element) {
        // 在index位置插入元素
        insertElementAt(element, index);
    }
    
    public synchronized void insertElementAt(E obj, int index) {
        // 修改次数加一
        modCount++;
        // 检查边界
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index
                                                     + " > " + elementCount);
        }
        // 确保数组的容量足够
        ensureCapacityHelper(elementCount + 1);
        // 将elementData[index:]的元素移动到elementData[index+1:]
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
        // 插入元素
        elementData[index] = obj;
        elementCount++;
    }
```

看了上面这两个方法，其实不难看出，`Vector`在对数组进行增删改和扩容的方面，跟`ArrayList`是一样的，涉及数组复制的，会使用`Arrays`工具类，
涉及到数组移动的，会使用`System.arraycopy`的一系列重载方法

## java.util.Vector#addAll(int, java.util.Collection<? extends E>)

```java
public synchronized boolean addAll(int index, Collection<? extends E> c) {
        // 修改次数加一
        modCount++;
        // 检查数组边界问题
        if (index < 0 || index > elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityHelper(elementCount + numNew);

        int numMoved = elementCount - index;
        if (numMoved > 0)
            // 移动数组元素
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);
        // 添加集合中的元素到数组
        System.arraycopy(a, 0, elementData, index, numNew);
        elementCount += numNew;
        return numNew != 0;
    }
```

## java.util.Vector#contains

```java
public boolean contains(Object o) {
        // 指定从索引下标为0的地方开始搜索对象
        return indexOf(o, 0) >= 0;
    }
    
    public synchronized int indexOf(Object o, int index) {
        if (o == null) {
            // 从index开始，正序遍历数组，找到null的位置
            for (int i = index ; i < elementCount ; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            // 从index开始，正序遍历数组，找到object的位置
            for (int i = index ; i < elementCount ; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        // 找不到返回-1
        return -1;
    }
```

连`contains`方法都跟`ArrayList`的实现那么相似😊

## java.util.Vector#containsAll

```java
public synchronized boolean containsAll(Collection<?> c) {
        return super.containsAll(c);
    }
    
    // java.util.AbstractCollection#containsAll
    public boolean containsAll(Collection<?> c) {
        for (Object e : c)
        // 调用AbstractCollection本身的contains方法
            if (!contains(e))
                return false;
        return true;
    }
    
    // java.util.AbstractCollection#contains
    public boolean contains(Object o) {
    // 获取迭代器
        Iterator<E> it = iterator();
        if (o==null) {
        // 迭代器遍历寻找null元素
            while (it.hasNext())
                if (it.next()==null)
                    return true;
        } else {
        // 迭代器遍历，寻找o对象
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;
        }
        return false;
    }
```

[1]处，`containsAll`的实现实际上是通过调用父类的`java.util.AbstractCollection#containsAll`方法实现

在`AbstractCollection`的`containsAll`方法又调用了`AbstractCollection`的`contains`方法

在`contains`方法里面通过获取Vector的迭代器对象，遍历Vector，寻找元素

## java.util.Vector#copyInto

```java
public synchronized void copyInto(Object[] anArray) {
        // 将数组元素复制到指定的数组中
        System.arraycopy(elementData, 0, anArray, 0, elementCount);
    }
```

很简单，自己look look

## java.util.Vector#ensureCapacity

```java
public synchronized void ensureCapacity(int minCapacity) {
    // 确保数组的空间满足指定的大小
        if (minCapacity > 0) {
            modCount++;
            ensureCapacityHelper(minCapacity);
        }
    }
```

很简单，自己look look

## java.util.Vector#indexOf(java.lang.Object, int)

```java
public synchronized int indexOf(Object o, int index) {
        if (o == null) {
            // 从index开始，正序遍历数组，找到null的位置
            for (int i = index ; i < elementCount ; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            // 从index开始，正序遍历数组，找到object的位置
            for (int i = index ; i < elementCount ; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        // 找不到返回-1
        return -1;
    }
```

如果看过ArrayList的源码，对这段代码应该也会感到很熟悉

## java.util.Vector#setSize

```java
public synchronized void setSize(int newSize) {
        // 修改次数加一
        modCount++;
        if (newSize > elementCount) {
            // 确保数组容量大于等于newSize
            ensureCapacityHelper(newSize);
        } else {
            // 如果指定的newSize要小于当前数组的有效元素个数
            // 那么从newSize开始，到数组的最后一个元素，都要设置为null
            for (int i = newSize ; i < elementCount ; i++) {
                elementData[i] = null;
            }
        }
        elementCount = newSize;
    }
```

`setSize`方法很好理解，如果指定的size大于数组中有效元素的个数，检查数组容量（可能会扩容），否则，把把多出来的那部分有效元素干掉

> 理解是很好理解，但是不明白为啥要对外提供这样一个方法😂


小结一下，可以看到`Vector`类对外提供的方法都会加上`synchronized`关键字修饰

# 5. 内部类

## java.util.Vector.Itr

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083448621-188f4a80-82f6-4a5c-9220-f2e258d44501.png#align=left&display=inline&height=152&originHeight=152&originWidth=171&size=0&status=done&style=none&width=171)

`Itr`实现了`Iterator`接口，是`Vector`的内部类

先来看看其成员变量

```java
// 下一个返回的元素的下标
        int cursor;    
        // 最近返回的元素的下标
        int lastRet = -1; 
        // 期望修改次数
        int expectedModCount = modCount;
```

比较简单的三个成员变量，`Itr`实现的方法也是同步的，是线程安全的

```java
public boolean hasNext() {
            // Racy but within spec（优雅且符合规范）
            return cursor != elementCount;
        }
        
        public E next() {
            // 加锁
            synchronized (Vector.this) {
                // 检查修改次数
                checkForComodification();
                int i = cursor;
                if (i >= elementCount)
                    throw new NoSuchElementException();
                cursor = i + 1;
                return elementData(lastRet = i);
            }
        }
        
        public void remove() {
            if (lastRet == -1)
                throw new IllegalStateException();
            // 加锁
            synchronized (Vector.this) {
                // 检查修改次数
                checkForComodification();
                // 调用Vector的remove方法
                Vector.this.remove(lastRet);
                expectedModCount = modCount;
            }
            cursor = lastRet;
            // 清空lastReturn
            lastRet = -1;
        }
```

`Itr`加锁实现比较简单，自己look look 就行了😁

## java.util.Vector.ListItr

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083448569-e2e09a62-073e-45d4-bbd8-4656a03ffcff.png#align=left&display=inline&height=247&originHeight=247&originWidth=322&size=0&status=done&style=none&width=322)

`ListItr`可以在指定位置开始，以任意方向遍历`Vector`

同样的，`ListItr`的实现也是加锁的

```java
public boolean hasPrevious() {
            return cursor != 0;
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor - 1;
        }

        public E previous() {
            // 加锁
            synchronized (Vector.this) {
                // 检查修改次数
                checkForComodification();
                int i = cursor - 1;
                if (i < 0)
                    throw new NoSuchElementException();
                cursor = i;
                // 返回cursor前一个元素
                return elementData(lastRet = i);
            }
        }

        public void set(E e) {
        // 修改最近返回的元素的值
            if (lastRet == -1)
                throw new IllegalStateException();
            synchronized (Vector.this) {
                checkForComodification();
                Vector.this.set(lastRet, e);
            }
        }

        public void add(E e) {
            int i = cursor;
            synchronized (Vector.this) {
                checkForComodification();
                Vector.this.add(i, e);
                expectedModCount = modCount;
            }
            cursor = i + 1;
            // 添加元素后，lastReturn重置
            lastRet = -1;
        }
```

代码实现不复杂，中心思想还是通过`synchronized`关键字加锁

## 6. 总结

`Vector`跟`ArrayList`有很多相似的地方，比如继承共同的父类，实现相同的接口，底层都是通过数组实现，它们连源码的实现也有很多相似之处

不过`Vector`比`ArrayList`多出一个有点，那就是`Vector`是线程安全的，虽然加锁粒度是方法维度，并发性能不高

如果看过`ArrayList`的源码，那么`Vector`的源码阅读基本不会遇到阻碍
