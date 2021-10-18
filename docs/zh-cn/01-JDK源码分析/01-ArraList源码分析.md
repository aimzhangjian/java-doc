# ArraList源码分析

## 构造函数
ArrayList提供了多个重载构造函数，既可以指定初始容量也可以指定初始集合，或什么都不指定

### 无参构造函数
通过ArrayList()构造函数，可以构造一个空的ArrayList，使用DEFAULTCAPACITY_EMPTY_ELEMENTDATA空对象数组初始化属性elementData属性，此
属性即用来存储我们add添加进来的元素

```
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

## 添加元素
ArrayList提供多个添加元素的Api，既可以单个添加到末尾，又可以指定位置添加元素，也可以批量添加元素

### 添加单个元素到末尾
add(E e)提供添加单个元素到集合末尾的功能，其实现比较简单，一共分为两步

- 检查数组容量是否充足，否则进行扩容

- 向数组添加新元素

```
public boolean add(E e) {
    // 判断容量及扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 添加元素
    elementData[size++] = e;
    return true;
}
```

#### ensureCapacityInternal容量判断与扩容解析
```
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```

ensureCapacityInternal方法参数minCapacity为需要的最小集合容量，在add方法调用时会将size + 1，其中size即为当前集合元素个数即数组elementData
的实际存储元素个数。在ensureCapacityInternal方法中先调用calculateCapacity(Object[] elementData, int minCapacity)方法计算需要的最小集合容量

```
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}
```
calculateCapacity计算逻辑为如果当前元素集合为空，则取minCapacity与DEFAULT_CAPACITY（10）的较大值；如果不为空则直接返回minCapacity。
当计算完集合所需容量之后ensureCapacityInternal方法继续调用ensureExplicitCapacity方法

```
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

在ensureExplicitCapacity方法中，首先将属性modCount加一，此属性表示列表结构更改次数。如果参数minCapacity大于目前已有元素列表大小，则
执行grow方法传入minCapacity参数值

```
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

在grow方法中，先判断minCapacity是否大于当前元素容量的1.5倍，如果大于，则按minCapacity进行扩容；如果小于，则按1.5倍容量扩容。但如果需要的
容量大于MAX_ARRAY_SIZE（值为Integer.MAX_VALUE - 8），则会调用hugeCapacity方法重新计算容量

```
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

在hugeCapacity方法中，如果传入的需要扩展容量为负数，则抛出OutOfMemoryError异常。如果需要扩展容量大于MAX_ARRAY_SIZE则返回Integer.MAX_VALUE，
否则返回MAX_ARRAY_SIZE

```
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

至此添加单个元素的源码解析便结束


