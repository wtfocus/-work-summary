[toc]

## ArrayList 源码分析 - 扩容机制

1.  ArrayList 可以实现容量的自动扩容，下面我们基于 JDK1.8 中 ArrayList 的源码来分析。

### 源代码

1.  构造方法

    -   ```java
        
            /**
             * 初始容量
             */
            private static final int DEFAULT_CAPACITY = 10;
        
            /**
             * 空实例
             */
            private static final Object[] EMPTY_ELEMENTDATA = {};
        
            private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
        
            /**
             * 保存 ArrayList 数据的数组
             */
            transient Object[] elementData; // non-private to simplify nested class access
        
            /**
             * ArrayList 数组长度
             */
            private int size;
        
            /**
             * 指定初始容量的构造函数
             */
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
        
            /**
             * 无参构造
             * 默认空数组，当赋值第一个元素时，会扩容为默认大小为 10 的数组 
             */
            public ArrayList() {
                this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
            }
        
            /**
             * 指定包含一个集合元素的 ArrayList
             */
            public ArrayList(Collection<? extends E> c) {
                // 集合转数组
                elementData = c.toArray();
                // 集合非空
                if ((size = elementData.length) != 0) {
                    // 将原来非 Object 数组类型转为 Object 数组类型
                    if (elementData.getClass() != Object[].class)
                        elementData = Arrays.copyOf(elementData, size, Object[].class);
                } else {
                    // 创建空数组
                    this.elementData = EMPTY_ELEMENTDATA;
                }
            }
        
        ```
    
2.  add(E e) 方法

    -   ```java
            /**
            * 添加指定元素到数组尾部
            */
        	public boolean add(E e) {
                // 赋值前先执行 ensureCapacityInternal
                ensureCapacityInternal(size + 1);  // Increments modCount!!
                // 数组赋值
                elementData[size++] = e;
                return true;
            }
        ```

3.  ensureCapacityInternal 方法

    -   ```java
            private void ensureCapacityInternal(int minCapacity) {
                ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
            }
        ```

4.  先看 calculateCapacity 方法

    -   ```java
            /**
            * 计算扩容量
            */
        	private static int calculateCapacity(Object[] elementData, int minCapacity) {
                // 空数组情况
                // 数组首次赋值时，会返回默认的容量大小
                if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
                    return Math.max(DEFAULT_CAPACITY, minCapacity);
                }
                // 数组非空情况
                return minCapacity;
            }
        ```

5.  再看 ensureExplicitCapacity 方法

    -   ```java
        	/**
            * 判断是否需要扩容
            */
        	private void ensureExplicitCapacity(int minCapacity) {
                modCount++;
        
                // overflow-conscious code
                if (minCapacity - elementData.length > 0)
                    // 扩容
                    grow(minCapacity);
            }
        ```

6.  grow 方法

    -   ```java
        
            /**
             * 要分配数组最大大小
             */
            private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
        
            /**
             * 扩容核心方法
             */
            private void grow(int minCapacity) {
                // 记录老容量
                int oldCapacity = elementData.length;
                // 计算新容量（老容量的 1.5 倍）
                // 对于大数据的 2 进制运算,位移运算符比那些普通运算符的运算要快很多,因为程序仅仅移动一下而已,不去计算,这样提高了效率,节省了资源
                int newCapacity = oldCapacity + (oldCapacity >> 1);
                // 新容量如果还是小于最小容量要求，则把最小容量赋值给新容量。
                if (newCapacity - minCapacity < 0)
                    newCapacity = minCapacity;
                // 新容量如果大于数组最大容量，执行 hugeCapacity
                if (newCapacity - MAX_ARRAY_SIZE > 0)
                    newCapacity = hugeCapacity(minCapacity);
                // minCapacity is usually close to size, so this is a win:
                elementData = Arrays.copyOf(elementData, newCapacity);
            }
        
        ```

7.  hugeCapacity 方法

    -   ```java
                private static int hugeCapacity(int minCapacity) {
                    if (minCapacity < 0) // overflow
                        throw new OutOfMemoryError();
                    // 最小容量大于 MAX_ARRAY_SIZE 返回 Integer.MAX_VALUE，否则，返回 MAX_ARRAY_SIZE （Integer.MAX_VALUE - 8）
                    return (minCapacity > MAX_ARRAY_SIZE) ?
                        Integer.MAX_VALUE :
                        MAX_ARRAY_SIZE;
                }
            
        ```

8.  add(int index, E element) 方法

    -   ```java
            /**
             * 在此列表中的指定位置插入指定的元素。
             */
        	public void add(int index, E element) {
                // 先调用 rangeCheckForAdd 对index进行界限检查；
                rangeCheckForAdd(index);
                
        		// 然后调用 ensureCapacityInternal 方法保证capacity足够大；
                ensureCapacityInternal(size + 1);  // Increments modCount!!
                // 再将从index开始之后的所有成员后移一个位置；
                System.arraycopy(elementData, index, elementData, index + 1,
                                 size - index);
                // 将element插入index位置；最后size加 1。
                elementData[index] = element;
                size++;
            }
        ```

9.  rangeCheckForAdd 方法

    -   ```java
            private void rangeCheckForAdd(int index) {
                if (index > size || index < 0)
                    throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
            }
        ```

    -   

### 扩展

1.  System.arraycopy

    -   ```java
        	/**
        	* 在此列表中的指定位置插入指定的元素。
        	*/
        	public static native void arraycopy(Object src,  int  srcPos,
                                                Object dest, int destPos,
                                                int length);
        ```

    -   

2.  Arrays.copyOf

    -   ```java
            /**
             * 复制指定数组、指定长度
             * original：要复制的数组；
             * newLength：要复制的长度
             */
        	@SuppressWarnings("unchecked")
            public static <T> T[] copyOf(T[] original, int newLength) {
                return (T[]) copyOf(original, newLength, original.getClass());
            }
        
            /**
             * 
             */
            public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
                @SuppressWarnings("unchecked")
                T[] copy = ((Object)newType == (Object)Object[].class)
                    ? (T[]) new Object[newLength]
                    : (T[]) Array.newInstance(newType.getComponentType(), newLength);
                // copyOf 也是调用的 System.arraycopy
                System.arraycopy(original, 0, copy, 0,
                                 Math.min(original.length, newLength));
                return copy;
            }
        ```

3.  System.arraycopy VS Arrays.copyOf
    -   联系：
        -   `copyOf()`内部实际调用了 `System.arraycopy()` 方法
    -   区别
        -   `arraycopy()` 需要目标数组，将原数组拷贝到你自己定义的数组里或者原数组，而且可以选择拷贝的起点和长度以及放入新数组中的位置
        -   `copyOf()` 是系统自动在内部新建一个数组，并返回该数组。