# 1.前言

国庆有时间，正好可以学习一些优秀的源代码，于是便选择了常用的集合 List。

List 是一个为有序集合设计的接口，全称是 java.util.List，诞生于 JDK1.2，核心能力有 添加、删除、替换、查找、排序、遍历、集合运算。

java.util包里面实现了 List接口的普通类有四个：

​	ArrayList，LinkedList，Stack，Vector

其中除了 Stack（继承自 Vector），其他三个类都是直接实现了 List接口。

开发环境：JDK1.8。

# 2.List核心能力

## 2.1 添加元素

涉及的方法如下：

```java
// 添加单个元素，可以指定位置
boolean add(E e);
void add(int index, E element);

// 批量添加元素，可以指定位置，被添加的集合需要实现 Collection<E>接口
boolean addAll(Collection<? extends E> c);
boolean addAll(int index, Collection<? extends E> c);
```

 “**? extends E**”的含义：

​	E指代的是 list中的元素类型，?指代的是其他未知集合里的元素类型，而 ?需要是 E的子类。比如 Integer类继承自 Number类，E指代 Number，则 ?可以指代 Integer或 Number的其他子类。

上述四个方法均没有默认实现，需要实现类自己去实现。

## 2.2 删除元素

```java
// 删除单个元素
boolean remove(Object o);
E remove(int index);

// 删除所有元素
void clear();
```

上述三个方法均没有默认实现，需要实现类自己去实现。

## 2.3 替换元素

```java
E set(int index, E element);
```

## 2.4 查找元素

```java
boolean contains(Object o);

// 目标 o从左往右第一次出现的下标
int indexOf(Object o);

// 目标 o从左往右最后一次出现的下标
// 或 从右往左第一次出现的下标
int lastIndexOf(Object o);
```

## 2.5 元素排序

```java
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray();
    Arrays.sort(a, (Comparator) c);
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {
        i.next();
        i.set((E) e);
    }
}
```

这里可以看到是先拷贝 List中的元素创建了一个数组，然后使用 Arrays进行排序，并且可以指定排序的规则。

## 2.6 遍历元素

```java
Iterator<E> iterator();
```

这里可以使用迭代器去遍历 List中的所有元素。

## 2.7 集合运算

```java
// 这个方法的效果相当于两个集合 A和 C：A交C
boolean retainAll(Collection<?> c);

// 这个方法的效果相当于两个集合 A和 C：A-A交C
boolean removeAll(Collection<?> c);

// 将 List中的所有元素统一执行某个操作
default void replaceAll(UnaryOperator<E> operator) {
    Objects.requireNonNull(operator);
    final ListIterator<E> li = this.listIterator();
    while (li.hasNext()) {
        li.set(operator.apply(li.next()));
    }
}
```

removeAll和 replaceAll虽然字面意思分别是 移除和替换，但是其实际含义用集合运算来表示更易于理解。

# 3.ArrayList源码分析

ArrayList是 List接口的实现类之一，之所以选择它做为分析对象，是因为实际工作中用得比较多。

这里着重于分析 ArrayList对 List接口的部分核心能力是如何实现的，有什么特点。

## 3.1 具体实现：添加元素

首先，先看最简单的**添加元素**是如何实现的：

```java
transient Object[] elementData;

public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

这里可以看到实际数据是存在于一个对象数组 elementData中，transient用于阻止该变量被序列化。elementData如何被初始化取决于使用的是哪个构造方法，如下：

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};    
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
 
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```

接着再解释下 ensureCapacityInternal(size + 1)，这个操作目的是修改 modCount，使其值加 1，关于 modCount可以简单地理解为**并发场景中用于判断集合是否被别的线程修改**。

## 3.2 具体实现：查找元素

这个方法比较有意思的地方是：支持查找 null元素。

```java
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

具体的查找逻辑跟笔者猜测一致，使用了顺序遍历和 equals方法。

## 3.3 具体实现：集合运算

```java
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}

private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;
    boolean modified = false;
    try {
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        if (r != size) {
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        if (w != size) {
            // clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    return modified;
}
```

如上，retainAll是两个集合求交集的一个运算，实现逻辑略长，部分逻辑耐人寻味。

先看一个与核心逻辑无关的代码： Objects.requireNonNull(c)，用它来判断对象是否为空是不是比下面的方式优雅多了？

```java
if(c==null){
    throw new NullPointerException();
}
```

下面是求交集的实现逻辑，利用集合的 contains方法，比较简短：

```java
for (; r < size; r++)
    if (c.contains(elementData[r]) == complement)
        elementData[w++] = elementData[r];
```

这里需要注意的是，该方法大部分逻辑在 finally里面，那么这里是在做些什么呢？

```java
if (r != size) {
    System.arraycopy(elementData, r,
                     elementData, w,
                     size - r);
    w += size - r;
}
if (w != size) {
    // clear to let GC do its work
    for (int i = w; i < size; i++)
        elementData[i] = null;
    modCount += size - w;
    size = w;
    modified = true;
}
```

先想一下什么情况下 r != size？是不是前面的抛异常了就会出现，然后 w += size - r 这一行就会导致 elementData中未与集合 c对比过的元素也会出现在最后的结果里。举个例子如下：

​	elementData= a,b,c,d,e

​	c=b,c,d

当 r=3时，抛出了运行时异常，这时最后求交集的结果将是：

​	b,c,d,e

这个结果有些奇怪，笔者暂时也搞不清楚，不过既然求交集的过程中抛出了异常，那么结果和预期不符也说得过去。

# 4.用法展示

这里继续以 ArrayList作为研究对象，添加、删除、替换、查找和遍历等方法使用起来比较简单就不做研究了，主要研究 自定义排序功能和集合运算。

## 4.1 自定义排序

```java
// 自定义排序：根据字符串长度逆序排列
List<String> sortList=new ArrayList<>();
Collections.addAll(sortList,"R","Java","C++","Python");
System.out.println("origin list is: "+sortList.toString());

sortList.sort((a,b)->b.length()-a.length());
System.out.println("sort list is: "+sortList.toString());
```

运行效果如下：

> origin list is: [R, Java, C++, Python]
> sort list is: [Python, Java, C++, R]

正如第 2节分析的那样，list 提供的 sort方法可以指定排序规则，配合 lambda表达式使用，简单又高效。

## 4.2 集合运算

1. **将集合中的元素全部进行某个转换**

   ```java
   // 把整数集合中的所有元素都加倍
   List<Integer> intList=new ArrayList<>();
   Collections.addAll(intList,2,5,8);
   System.out.println("origin list is: "+intList.toString());
   
   intList.replaceAll(num -> num*2);
   System.out.println("origin list is: "+intList.toString());
   ```

   运行效果如下：

   > origin list is: [2, 5, 8]
   > all integer is double: [4, 10, 16]

2. **求两个集合的交集**

   ```java
   List<Integer> A=new ArrayList<>();
   Collections.addAll(A,2,1,5,8);
   List<Integer> B=new ArrayList<>();
   Collections.addAll(B,2,5,6,9);
   System.out.println("A: "+A.toString()+",B: "+B.toString());
   
   A.retainAll(B);
   System.out.println("A和B的交集: "+A.toString());
   ```

   运行效果如下：

   > A: [2, 1, 5, 8],B: [2, 5, 6, 9]
   > A和B的交集: [2, 5]

3. **求 A-A交B**

   ```java
   A.removeAll(B);
   System.out.println("A-A交B: "+A.toString());
   ```

   运行效果如下：

   > A: [2, 1, 5, 8],B: [2, 5, 6, 9]
   > A-A交B: [1, 8]

list提供的集合运算种类虽少，但胜在实用。

# 5.总结

List接口设计的方法比较全面，除了增删改查，还有部分实用的集合运算。ArrayList作为其实现类之一，方法的实现逻辑较为清晰简洁，在集合运算中还调用了 native方法。在代码风格上也很有借鉴意义，比如 使用 Objects.requireNonNull()方法进行 null判断，使用 'E'来表示本集合元素类型，用 '?'来表示其他未知集合的元素类型。





