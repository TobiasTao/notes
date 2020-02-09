# 介绍

Java里有一个叫做*Stack*的类，却没有叫做*Queue*的类（它是个接口名字）。当需要使用栈时，Java已不推荐使用*Stack*，而是推荐使用更高效的*ArrayDeque*；既然*Queue*只是一个接口，当需要使用队列时也就首选*ArrayDeque*了（次选是*LinkedList*）。

要讲栈和队列，首先要讲*Deque*接口。*Deque*的含义是“double ended queue”，即双端队列，它既可以当作栈使用，也可以当作队列使用。下表列出了*Deque*与*Queue*相对应的接口：

| Queue Method | Equivalent Deque Method | 说明                     |
| ------------ | ----------------------- | ---------------------- |
| `add(e)`     | `addLast(e)`            | 向队尾插入元素，失败则抛出异常        |
| `offer(e)`   | `offerLast(e)`          | 向队尾插入元素，失败则返回`false`   |
| `remove()`   | `removeFirst()`         | 获取并删除队首元素，失败则抛出异常      |
| `poll()`     | `pollFirst()`           | 获取并删除队首元素，失败则返回`null`  |
| `element()`  | `getFirst()`            | 获取但不删除队首元素，失败则抛出异常     |
| `peek()`     | `peekFirst()`           | 获取但不删除队首元素，失败则返回`null` |

下表列出了*Deque*与*Stack*对应的接口：

| Stack Method | Equivalent Deque Method | 说明                     |
| ------------ | ----------------------- | ---------------------- |
| `push(e)`    | `addFirst(e)`           | 向栈顶插入元素，失败则抛出异常        |
| 无            | `offerFirst(e)`         | 向栈顶插入元素，失败则返回`false`   |
| `pop()`      | `removeFirst()`         | 获取并删除栈顶元素，失败则抛出异常      |
| 无            | `pollFirst()`           | 获取并删除栈顶元素，失败则返回`null`  |
| `peek()`     | `peekFirst()`           | 获取但不删除栈顶元素，失败则抛出异常     |
| 无            | `peekFirst()`           | 获取但不删除栈顶元素，失败则返回`null` |

上面两个表共定义了*Deque*的12个接口。添加，删除，取值都有两套接口，它们功能相同，区别是对失败情况的处理不同。**一套接口遇到失败就会抛出异常，另一套遇到失败会返回特殊值（`false`或`null`）**。除非某种实现对容量有限制，大多数情况下，添加操作是不会失败的。**虽然*Deque*的接口有12个之多，但无非就是对容器的两端进行操作，或添加，或删除，或查看**。

# ArrayDeque

从名字可以看出*ArrayDeque*底层通过数组实现，为了满足可以同时在数组两端插入或删除元素的需求，该数组还必须是循环的，即**循环数组（circular array）**，也就是说数组的任何一点都可能被看作起点或者终点。*ArrayDeque*是非线程安全的（not thread-safe），当多个线程同时使用的时候，需要程序员手动同步；另外，该容器不允许放入`null`元素。

## 属性

```java
 /**
     * The index of the element at the head of the deque (which is the
     * element that would be removed by remove() or pop()); or an
     * arbitrary number equal to tail if the deque is empty.
     */
    transient int head;

    /**
     * The index at which the next element would be added to the tail
     * of the deque (via addLast(E), add(E), or push(E)).
     */
    transient int tail;

    /**
     * The minimum capacity that we'll use for a newly created deque.
     * Must be a power of 2.
     */
    private static final int MIN_INITIAL_CAPACITY = 8;
```

## 创建

```java
public ArrayDeque() {

        // 默认容量为16
        elements = new Object[16];
    }

    public ArrayDeque(int numElements) {
        allocateElements(numElements);
    }

    private void allocateElements(int numElements) {
        int initialCapacity = MIN_INITIAL_CAPACITY; //  MIN_INITIAL_CAPACITY = 8

        // Find the best power of two to hold elements.
        // Tests "<=" because arrays aren't kept full.
        if (numElements >= initialCapacity) {
            initialCapacity = numElements;
            initialCapacity |= (initialCapacity >>>  1);
            initialCapacity |= (initialCapacity >>>  2);
            initialCapacity |= (initialCapacity >>>  4);
            initialCapacity |= (initialCapacity >>>  8);
            initialCapacity |= (initialCapacity >>> 16);
            initialCapacity++;

            if (initialCapacity < 0)   // Too many elements, must back off
                initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
        }
        elements = new Object[initialCapacity];
    }
```

## addFirst(E e)

```java
public void addFirst(E e) {
    if (e == null)//不允许放入null
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;//2.下标是否越界
    if (head == tail)//空间是否够用

        doubleCapacity();//扩容
}
```

**空间问题是在插入之后解决的**，因为`tail`总是指向下一个可插入的空位，也就意味着`elements`数组至少有一个空位，所以插入元素的时候不用考虑空间问题。





