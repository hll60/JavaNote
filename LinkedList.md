# LinkedList源码分析

##### 基于JDK1.8.0_121

`LinkedList` 就是双向链表，可以被当作堆栈、队列或双端队列进行操作。  
`LinkedList` 是非同步的。

### 1.定义

~~~java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
~~~

`LinkedList` 继承自 `AbstractSequentialList` , 并实现了 `List` , `Deque` , `Cloneable` , `Serializable` 接口。

`AbstractSequentialList` 继承自`AbstractList`  默认实现了 `List` 接口的一些方法。

`Deque` 继承自 `Queue` , 提供了双端队列的接口支持。

其他: 实现了 `Cloneable` 接口, 用于实现对象的浅克隆; 实现了 `Serializable` 用于实现序列化功能。

### 2.属性

~~~java
	transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
~~~

`LinkedList` 有三个属性: `size` 存储元素的数量, `first` 存储第一个节点, `last` 存储最后一个节点。

`transient` 修饰的变量不会被序列化。

### 3.构造方法

~~~java
	/**
     * Constructs an empty list.
     */
    public LinkedList() {
    }

    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param  c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
~~~

`LinkedList` 提供了两个构造方法:

`public LinkedList()` 默认构造方法为空实现。因为使用链表结构来存储, 所以不需要初始化属性。

`public LinkedList(Collection<? extends E> c)` 使用指定的集合初始化。内部直接调用了 `addAll()` 方法。

`Node` 是 `LinkedList` 的一个内部类。实现了节点的数据结构: `item` 存储了当前节点的值; `next` 存储了下一个节点; `prev` 存储了上一个节点。

~~~java
	private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
~~~



### 4.常用方法

`public int size()` 返回数组的大小, 源码实现 `return size` 。

`public boolean isEmpty()` 判断数组大小是否为0, 源码实现 `return size == 0` 。

`public boolean add(E e)` 在集合尾部添加一个新元素。内部调用 `void linkLast(E e)` 方法来实现。该方法是在链表尾部插入一个元素。具体实现如下: 首先临时存储 `last`  为 `l`  , 再新建一个 `Node newNode` , 将 `e` 放入其中, 将 `newNode.prev` 指向 `last` , 将 `newNode` 赋值给 `last` , 判断是否已经存在 `l` (即判断之前的链表长度是否为0), 如果不存在则将 `newNode` 赋值给 `first` , 否则 将 `newNode`  赋值给 `l.next` , 最后改变 `size` 的大小。类似的操作方法还有 `private void linkFirst(E e)` 、`void linkBefore(E e, Node<E> succ)` 、`private E unlinkFirst(Node<E> f)` 、`private E unlinkLast(Node<E> l)` 、`E unlink(Node<E> x)` 。  这些是常用的链表操作, 对链表数据结构和操作不清楚的可以查询相关资料。这里附上一篇博客供参考: [链表各类操作详解](http://blog.csdn.net/hackbuteer1/article/details/6591486)。

~~~java
	/**
     * Appends the specified element to the end of this list.
     *
     * <p>This method is equivalent to {@link #addLast}.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        linkLast(e);
        return true;
    }

	/**
     * Links e as first element.
     */
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }	

	/**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }

	/**
     * Inserts element e before non-null Node succ.
     */
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }

    /**
     * Unlinks non-null first node f.
     */
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }

    /**
     * Unlinks non-null last node l.
     */
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }

    /**
     * Unlinks non-null node x.
     */
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
~~~

`public boolean addAll(Collection<? extends E> c)` 插入指定的集合。内部调用 `public boolean addAll(int index, Collection<? extends E> c)` 在尾部插入指定的集合。

~~~java
/**
     * Appends all of the elements in the specified collection to the end of
     * this list, in the order that they are returned by the specified
     * collection's iterator.  The behavior of this operation is undefined if
     * the specified collection is modified while the operation is in
     * progress.  (Note that this will occur if the specified collection is
     * this list, and it's nonempty.)
     *
     * @param c collection containing elements to be added to this list
     * @return {@code true} if this list changed as a result of the call
     * @throws NullPointerException if the specified collection is null
     */
    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    /**
     * Inserts all of the elements in the specified collection into this
     * list, starting at the specified position.  Shifts the element
     * currently at that position (if any) and any subsequent elements to
     * the right (increases their indices).  The new elements will appear
     * in the list in the order that they are returned by the
     * specified collection's iterator.
     *
     * @param index index at which to insert the first element
     *              from the specified collection
     * @param c collection containing elements to be added to this list
     * @return {@code true} if this list changed as a result of the call
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @throws NullPointerException if the specified collection is null
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
~~~

`public void clear()` 清空所有元素。具体实现是表里所有节点, 清空元素, 最后将 `first` 、`last` 置空。

~~~java
	/**
     * Removes all of the elements from this list.
     * The list will be empty after this call returns.
     */
    public void clear() {
        // Clearing all of the links between nodes is "unnecessary", but:
        // - helps a generational GC if the discarded nodes inhabit
        //   more than one generation
        // - is sure to free memory even if there is a reachable Iterator
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }
~~~

`public E remove(int index)` 移除指定索引的元素, 返回移除的元素。  
`public boolean remove(Object o)` 移除指定的元素。该方法从头节点向后遍历, 移除第一个匹配成功的元素。  
`public E remove()` 移除第一个元素并返回该元素。  
`public E removeFirst()` 移除第一个元素。  
`public E removeLast()` 移除最后一个元素。  
上述的五个方法都是移除元素, 内部实现都是调用了相应的 `E unlinkXXX(Node<E> x)` 方法。

~~~java
	/**
     * Removes the element at the specified position in this list.  Shifts any
     * subsequent elements to the left (subtracts one from their indices).
     * Returns the element that was removed from the list.
     *
     * @param index the index of the element to be removed
     * @return the element previously at the specified position
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
	
	/**
     * Removes the first occurrence of the specified element from this list,
     * if it is present.  If this list does not contain the element, it is
     * unchanged.  More formally, removes the element with the lowest index
     * {@code i} such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>
     * (if such an element exists).  Returns {@code true} if this list
     * contained the specified element (or equivalently, if this list
     * changed as a result of the call).
     *
     * @param o element to be removed from this list, if present
     * @return {@code true} if this list contained the specified element
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
    
    /**
     * Retrieves and removes the head (first element) of this list.
     *
     * @return the head of this list
     * @throws NoSuchElementException if this list is empty
     * @since 1.5
     */
    public E remove() {
        return removeFirst();
    }
~~~

`public E set(int index, E element)` 替换指定索引的元素。具体实现, 通过 `Node<E> node(int index)` 方法找到 `index` 对应的节点, 替换该节点上的内容 `item` 。

~~~java
	/**
     * Replaces the element at the specified position in this list with the
     * specified element.
     *
     * @param index index of the element to replace
     * @param element element to be stored at the specified position
     * @return the element previously at the specified position
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }
    
    /**
     * Returns the (non-null) Node at the specified element index.
     */
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
~~~

`public void addFirst(E e)` 在第一个节点前插入元素。  
`public void addLast(E e)` 在最后一个节点后插入元素。  
上述两个方法内部使用`linFirst(e)` 、`linkLast(e)` 实现。

`public int indexOf(Object o)`  查询元素的索引。从头节点遍历, 返回第一个匹配的索引。

~~~java
	/**
     * Returns the index of the first occurrence of the specified element
     * in this list, or -1 if this list does not contain the element.
     * More formally, returns the lowest index {@code i} such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
     * or -1 if there is no such index.
     *
     * @param o element to search for
     * @return the index of the first occurrence of the specified element in
     *         this list, or -1 if this list does not contain the element
     */
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
~~~

` public int lastIndexOf(Object o)`  查询元素的索引。从尾节点遍历, 返回第一个匹配的索引。

~~~java
	/**
     * Returns the index of the last occurrence of the specified element
     * in this list, or -1 if this list does not contain the element.
     * More formally, returns the highest index {@code i} such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
     * or -1 if there is no such index.
     *
     * @param o element to search for
     * @return the index of the last occurrence of the specified element in
     *         this list, or -1 if this list does not contain the element
     */
    public int lastIndexOf(Object o) {
        int index = size;
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (x.item == null)
                    return index;
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (o.equals(x.item))
                    return index;
            }
        }
        return -1;
    }
~~~

`public E get(int index)` 返回指定索引的元素。内部直接调用 `Node<E> node(int index)` 方法实现。

`public boolean contains(Object o)` 查询是否包含指定的元素。直接调用`indexOf()` 方法获取索引, 索引不等于0即代表包含。

`public Object[] toArray()` 返回对象数组。遍历所有节点, 生成新的数组。

~~~java
	/**
     * Returns an array containing all of the elements in this list
     * in proper sequence (from first to last element).
     *
     * <p>The returned array will be "safe" in that no references to it are
     * maintained by this list.  (In other words, this method must allocate
     * a new array).  The caller is thus free to modify the returned array.
     *
     * <p>This method acts as bridge between array-based and collection-based
     * APIs.
     *
     * @return an array containing all of the elements in this list
     *         in proper sequence
     */
    public Object[] toArray() {
        Object[] result = new Object[size];
        int i = 0;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;
        return result;
    }
~~~

`public <T> T[] toArray(T[] a)` 将集合元素填充到指定数组并返回。该接口支持泛型。

~~~java
	public <T> T[] toArray(T[] a) {
        if (a.length < size)
            a = (T[])java.lang.reflect.Array.newInstance(
                                a.getClass().getComponentType(), size);
        int i = 0;
        Object[] result = a;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;

        if (a.length > size)
            a[size] = null;

        return a;
    }
~~~

### 5.其他方法

`public Object clone()` 克隆该集合，返回一个新的 `LinkedList`。注意: 该方法为浅克隆, 元素本身不会本克隆。

~~~java
	/**
     * Returns a shallow copy of this {@code LinkedList}. (The elements
     * themselves are not cloned.)
     *
     * @return a shallow copy of this {@code LinkedList} instance
     */
    public Object clone() {
        LinkedList<E> clone = superClone();

        // Put clone into "virgin" state
        clone.first = clone.last = null;
        clone.size = 0;
        clone.modCount = 0;

        // Initialize clone with our elements
        for (Node<E> x = first; x != null; x = x.next)
            clone.add(x.item);

        return clone;
    }	
~~~

`public void push(E e)` 入栈。使用 `linkFirst(e)` 实现。

`public E pop()` 出栈。使用 `removeFirst(e)` 实现。

`public E offer()` 入列。使用 `removeFirst(e)` 实现。

`public E poll()` 出列。使用 `removeFirst(e)` 实现。

还有很多栈、队列相关方法, 不在一一列举。

