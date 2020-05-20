# ArrayList源码解析

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083324299-e90f55ff-f9b8-4687-bd07-2b14cc98dfe8.png#align=left&display=inline&height=907&originHeight=907&originWidth=2335&size=0&status=done&style=none&width=2335)

# 1. 概述

ArrayList底层通过数组实现，是线程不安全的，具有随机访问快（根据下标），随机增删慢的特点

ArrayList继承`AbstractList`抽象类，实现`List`接口

# 2. 成员变量

```java
/**
     * 默认初始容量
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 用于空对象的共享空数组
     * 用于创建指定长度为0的ArrayList的构造方法
     * java.util.ArrayList#ArrayList(int)
     * java.util.ArrayList#ArrayList(java.util.Collection)
     *
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * 用于默认大小的空对象的共享空数组（跟上面的区分开来）
     * 用于创建默认长度为10的ArrayList的构造方法
     * java.util.ArrayList#ArrayList()
     *
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 存储内容的数组
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * ArrayList的长度
     */
    private int size;
    
    /**
     * 数组的最大长度
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

这里有一点要特别注意:
`elementData.length`是指数组的长度
`size`是指List的长度
这两个长度不是一样的

# 3. 构造函数

```java
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            // 初始容量大于0时，新建Object数组指向elementData
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            // 初始容量等于0时，EMPTY_ELEMENTDATA数组指向elementData
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            // 初始容量小于0，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    public ArrayList() {
        // 无参构造，elementData指向默认的空数组
        // 这也表明，elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA的话
        // 表示ArrayList还没有添加过任何元素
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    public ArrayList(Collection<? extends E> c) {
        // collection转数组
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // 数组长度不等于0
            // [2] c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                // [1]
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

[1] 处ArrayList的源码中，涉及到数组复制的，都是使用Arrays提供的工具类

[2] 处关于toArray方法不一定返回Object数组，可以看看这篇文章
[c.toArray might not return Object[]?](https://www.cnblogs.com/liqing-weikeyuan/p/7922306.html)

> 如果是涉及到数组的移动（有添加、删除引发），会使用`java.lang.System#arraycopy`方法


# 4. 常用的方法

## 4.1 添加

**方法执行流程**

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083288122-6951d90b-f428-4179-88b7-dd1ebaaa339e.png#align=left&display=inline&height=901&originHeight=901&originWidth=661&size=0&status=done&style=none&width=661)

可以看到`add`和`addAll`方法都会使用到`ensureCapacityInternal`，该方法中的核心代码是对`ensureExplicitCapacity`方法的调用

`ensureExplicitCapacity`有两点作用：

1. 增加ArrayList的修改次数`modCount`
1. 容量不够的情况下，执行`grow`方法，进行扩容操作

> `modCount`记录了对象被修改的次数


### java.util.ArrayList#add(E)

```java
public boolean add(E e) {
        /*
        1. 确保elementData数组拥有足够的容量
        2. 增加修改次数
         */
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 数组添加元素
        elementData[size++] = e;
        return true;
    }
    /**
     * 确保数组拥有足够的容量
     * @param minCapacity 最小需要的容量
     */
    private void ensureCapacityInternal(int minCapacity) {

        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            // 表示elementDat还没有添加过元素
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        // 增加修改次数
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            // 当最小需要的容量超出了数组的长度，则需要扩容
            grow(minCapacity);
    }
    
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 扩容1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 扩容后还是小于需要的最小容量，则使用minCapacity作为newCapacity
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 扩容后发现newCapacity大于最大的数组长度
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 通过Arrays工具类进行数组扩容
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

### java.util.ArrayList#add(int, E)

```java
public void add(int index, E element) {
        // 检查index有没有越界
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 将elementData[index:]的元素移动到elementData[index+1:]
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        // 在index位置添加元素
        elementData[index] = element;
        size++;
    }
    
    private void rangeCheckForAdd(int index) {
        // [1]
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

- [1]处判断是否越界不是根据`elementData`的长度，而是根据ArrayList的长度`size`去判断

### java.util.ArrayList#addAll(java.util.Collection<? extends E>)

```java
public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
```

> 方法很简单，自己look look就行了


## 4.2 删除

### java.util.ArrayList#remove(int)

```java
public E remove(int index) {
        // 检查是否越界
        rangeCheck(index);

        modCount++;
        // 根据index找到对应的value
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            // 将elementData[index+1:]移动到elementData[index:]
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        
        // 清空最后一个元素
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
    
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    
    E elementData(int index) {
        return (E) elementData[index];
    }
```

### java.util.ArrayList#remove(java.lang.Object)

```java
public boolean remove(Object o) {
        if (o == null) {
        // 如果o为null对象
            for (int index = 0; index < size; index++)
            // 正序遍历，将第一个null对象移除
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
    
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

其他删除的方法的实现都不负责，有兴趣可以自己挑来看看

> 这里可以看到，很多方法里面，关于数据的移动，都是通过`System.arraycopy`实现


## 4.3 查找

```java
public E get(int index) {
        // 检查数组越界
        rangeCheck(index);

        return elementData(index);
    }
```

方法很简单，自己look look

# 5. 内部类

## SubList

ArrayList有个`subList`的方法，这个方法返回的是ArrayList的一个视图

```java
public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }
```

方法返回的是SubList类的对象

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083288565-c8a61e0f-1a05-4185-86c9-6d6398521c7c.png#align=left&display=inline&height=1132&originHeight=1132&originWidth=864&size=0&status=done&style=none&width=864)

可以看到这个类把ArrayList的增删改查方法基本都自己实现了一遍

先来看看构造方法

```java
SubList(AbstractList<E> parent,
                int offset, int fromIndex, int toIndex) {
            this.parent = parent;
            this.parentOffset = fromIndex;
            this.offset = offset + fromIndex;
            this.size = toIndex - fromIndex;
            // [1]
            this.modCount = ArrayList.this.modCount;
        }
```

可以看到[1]处，在创建SubList对象的时候，会把当时ArrayList.modCount的值赋值给SubList.modCount，Sublist的modCount的用处体现在`checkForComodification`方法中（下文会提及）

再来看看增删改查的流程

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083288173-855efa13-ccd9-467e-9e6c-6c7642ce3e8f.png#align=left&display=inline&height=907&originHeight=907&originWidth=989&size=0&status=done&style=none&width=989)

可以看到增删改查的方法都会先调用检查边界的方法`rangeCheck`、`rangeCheckForAdd`，然后调用`checkForComodification`

`checkForComodification`方法的作用是检查SubList对象的modCount跟当前ArrayList的modCount是否相等

```java
private void rangeCheck(int index) {
            if (index < 0 || index >= this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

        private void rangeCheckForAdd(int index) {
            if (index < 0 || index > this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }
        
        private void checkForComodification() {
        // [1]
            if (ArrayList.this.modCount != this.modCount)
                throw new ConcurrentModificationException();
        }
```

可以看到[1]处，当检查modCount不相等的时候，表示这时还有其他对象在修改ArrayList，就会抛出`ConcurrentModificationException`

> `modCount`在`Itr`内部类中也起着相同的作用


现在来看看SubList增删改查的具体是现实

```java
public void add(int index, E e) {
            rangeCheckForAdd(index);
            checkForComodification();
            // 实际调用的是ArrayList的add方法
            parent.add(parentOffset + index, e);
            // 同步一下modCount的信息
            this.modCount = parent.modCount;
            this.size++;
        }
        
        public E remove(int index) {
            rangeCheck(index);
            checkForComodification();
            // 实际调用的是ArrayList的remove方法
            E result = parent.remove(parentOffset + index);
            // 同步一下modCount
            this.modCount = parent.modCount;
            this.size--;
            return result;
        }
        
        public E set(int index, E e) {
            rangeCheck(index);
            checkForComodification();
            // 调用ArrayList的elementData方法
            E oldValue = ArrayList.this.elementData(offset + index);
            ArrayList.this.elementData[offset + index] = e;
            return oldValue;
        }
        
        public E get(int index) {
            rangeCheck(index);
            checkForComodification();
            // 调用ArrayList的elementData方法
            return ArrayList.this.elementData(offset + index);
        }
```

正如上面所提及的，SubList对增删改查的实现就是`检查边界` + `比较modCount` + `调用ArrayList本身对应的方法`

## Itr

在ArrayList中会有`iterator`方法，方法具体实现如下

```java
public Iterator<E> iterator() {
        return new Itr();
    }
```

可以看到方法是直接返回一个新建的Iter类对象

![](https://cdn.nlark.com/yuque/0/2019/png/650477/1577083288238-e9bc788f-8c4a-460d-bdec-a1822e5d6280.png#align=left&display=inline&height=644&originHeight=644&originWidth=868&size=0&status=done&style=none&width=868)

Itr内部类实现了Iterator接口，来看看它的成员变量

```java
int cursor;       // 下一个元素的下标
        int lastRet = -1; // 最后一次返回的元素的下标; -1 if no such
        int expectedModCount = modCount; // [1]期望的修改次数
```

可以从[1]处看到，Itr对象也会使用modCount，实际作用也是检查当前有没有别的对象在修改ArrayList，如果有就抛`ConcurrentModificationException`异常

接下来看看常用方法的实现

```java
public boolean hasNext() {
            return cursor != size;
        }
        
        public E next() {
            // 检查modCount
            checkForComodification();
            int i = cursor;
            // 检查有没有超出size
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            // 检查有没有超出数组长度
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
        
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```

方法还是比较容易读懂的，可以自己看看

# 6. 总结

ArrayList是底层基于数组实现的常用的集合类，相较于数组，ArrayList内部实现了关于数组越界、扩容相关的检查，使用起来比数据要简单，当然，数组的效率肯定会比ArrayList要高

# 7. 推荐阅读

[为什么阿里巴巴要求谨慎使用ArrayList中的subList方法](https://blog.csdn.net/hollis_chuang/article/details/93558499)

[为什么阿里巴巴建议集合初始化时，指定集合容量大小](https://blog.csdn.net/hollis_chuang/article/details/89349735)

[为什么阿里巴巴禁止在 foreach 循环里进行元素的 remove/add 操作](https://blog.csdn.net/hollis_chuang/article/details/88292661)
