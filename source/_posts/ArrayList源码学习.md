---
title: ArrayList源码学习
copyright: true
date: 2019-05-29 09:40:06
tags: JAVA
---


# ArrayList简介
```JAVA
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable
```
- ArrayList 底层是数组队列， 属于动态数组， 可以根据实际的需要去动态地调整数组的大小
- ArrayList 继承了AbstractList类，说明它提供了增删改查等功能
- ArrayList 实现了RandomAccess接口， 说明实现这个接口的List集合是支持随机访问的
- ArrayList 实现了Cloneable接口， 说明它是可克隆的
- ArrayList 实现了Serialiable接口， 说明它是可以被序列化的
<!--more-->

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable {
    private static final long serialVersionUID = 8683452581122892189L;

    /**
    * 默认初始化大小， 为10
    */
    private static final int DEFAULT_CAPACITY = 10;

    /**
    * empty_element_data 默认空数组，对应下面var1==0时的构造方法
    */
    private static final Object[] EMPTY_ELEMENTDATA = new Object[0];

    /**
    *
    */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = new Object[0];

    /**
    * 保存ArrayList数据的数组
    */
    transient Object[] elementData;

    /**
    * 数组所包含的数据的个数，值得注意的是，此处并不是指数组的大小
    */
    private int size;

    /**
    * 数组容量的最大值
    */
    private static final int MAX_ARRAY_SIZE = 2147483639;


    /**
    * 带参构造函数，自定义容量
    */
    public ArrayList(int var1) {
        if (var1 > 0) {
            this.elementData = new Object[var1];
        } else {
            if (var1 != 0) {
                throw new IllegalArgumentException("Illegal Capacity: " + var1);
            }

            this.elementData = EMPTY_ELEMENTDATA;
        }

    }

    /**
    * 默认空数组，，
    */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
    * 构造一个包含指定集合的数组
    */
    public ArrayList(Collection<? extends E> var1) {
        this.elementData = var1.toArray();
        if ((this.size = this.elementData.length) != 0) {
            // 判断是否为Objectp[]类型
            if (this.elementData.getClass() != Object[].class) {
                this.elementData = Arrays.copyOf(this.elementData, this.size, Object[].class);
            }
        } else {
            this.elementData = EMPTY_ELEMENTDATA;
        }

    }

    /**
    * 修改当前数组的大小，调整为与当前的数据大小保持一致
    */
    public void trimToSize() {
        ++this.modCount;
        if (this.size < this.elementData.length) {
            this.elementData = this.size == 0 ? EMPTY_ELEMENTDATA : Arrays.copyOf(this.elementData, this.size);
        }

    }

    /**
    * ArrayList的扩容机制， 算法为int var3 = var2 + (var2 >> 1);每次扩容约为原来的1.5倍
    */
    public void ensureCapacity(int var1) {
        int var2 = this.elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA ? 0 : 10;
        if (var1 > var2) {
            this.ensureExplicitCapacity(var1);
        }

    }

    /**
    * 计算最小容量
    */
    private static int calculateCapacity(Object[] var0, int var1) {
        return var0 == DEFAULTCAPACITY_EMPTY_ELEMENTDATA ? Math.max(10, var1) : var1;
    }

    /**
    * 判断是否需要扩容
    */
    private void ensureCapacityInternal(int var1) {
        this.ensureExplicitCapacity(calculateCapacity(this.elementData, var1));
    }

    private void ensureExplicitCapacity(int var1) {
        ++this.modCount;
        if (var1 - this.elementData.length > 0) {
            this.grow(var1);
        }

    }

    /**
    * 扩容的核心算法
    */
    private void grow(int var1) {
        int var2 = this.elementData.length;
        int var3 = var2 + (var2 >> 1);
        if (var3 - var1 < 0) {
            var3 = var1;
        }

        if (var3 - 2147483639 > 0) {
            var3 = hugeCapacity(var1);
        }

        this.elementData = Arrays.copyOf(this.elementData, var3);
    }

    private static int hugeCapacity(int var0) {
        if (var0 < 0) {
            throw new OutOfMemoryError();
        } else {
            return var0 > 2147483639 ? 2147483647 : 2147483639;
        }
    }

    public int size() {
        return this.size;
    }

    public boolean isEmpty() {
        return this.size == 0;
    }

    public boolean contains(Object var1) {
        return this.indexOf(var1) >= 0;
    }

    /**
    * 这里分了两张情况，空或非空
    */
    public int indexOf(Object var1) {
        int var2;
        if (var1 == null) {
            for(var2 = 0; var2 < this.size; ++var2) {
                if (this.elementData[var2] == null) {
                    return var2;
                }
            }
        } else {
            for(var2 = 0; var2 < this.size; ++var2) {
                if (var1.equals(this.elementData[var2])) {
                    return var2;
                }
            }
        }

        return -1;
    }

    public int lastIndexOf(Object var1) {
        int var2;
        if (var1 == null) {
            for(var2 = this.size - 1; var2 >= 0; --var2) {
                if (this.elementData[var2] == null) {
                    return var2;
                }
            }
        } else {
            for(var2 = this.size - 1; var2 >= 0; --var2) {
                if (var1.equals(this.elementData[var2])) {
                    return var2;
                }
            }
        }

        return -1;
    }

    /**
    * 浅拷贝？？？
    */
    public Object clone() {
        try {
            ArrayList var1 = (ArrayList)super.clone();
            //Arrays.copyOf功能是实现数组的复制，返回复制后的数组。参数是被复制的数组和复制的长度
            var1.elementData = Arrays.copyOf(this.elementData, this.size);
            var1.modCount = 0;
            return var1;
        } catch (CloneNotSupportedException var2) {
            throw new InternalError(var2);
        }
    }

    /**
    * 此处返回的数组是安全的，因为ArrayList不会保留对它的引用
    */
    public Object[] toArray() {
        return Arrays.copyOf(this.elementData, this.size);
    }

    public <T> T[] toArray(T[] var1) {
        if (var1.length < this.size) {
            return (Object[])Arrays.copyOf(this.elementData, this.size, var1.getClass());
        } else {
          //调用System提供的arraycopy()方法实现数组之间的复制
          // var1是目的数组
            System.arraycopy(this.elementData, 0, var1, 0, this.size);
            if (var1.length > this.size) {
                var1[this.size] = null;
            }

            return var1;
        }
    }

    E elementData(int var1) {
        return this.elementData[var1];
    }

    public E get(int var1) {
        this.rangeCheck(var1);
        return this.elementData(var1);
    }

    public E set(int var1, E var2) {
        this.rangeCheck(var1);
        Object var3 = this.elementData(var1);
        this.elementData[var1] = var2;
        return var3;
    }

    public boolean add(E var1) {
        this.ensureCapacityInternal(this.size + 1);
        this.elementData[this.size++] = var1;
        return true;
    }

    public void add(int var1, E var2) {
        this.rangeCheckForAdd(var1);
        this.ensureCapacityInternal(this.size + 1);
        System.arraycopy(this.elementData, var1, this.elementData, var1 + 1, this.size - var1);
        this.elementData[var1] = var2;
        ++this.size;
    }

    /**
    * 通过数组间的复制， 使用System.arraycopy去删除指定索引的元素
    */
    public E remove(int var1) {
        this.rangeCheck(var1);
        ++this.modCount;
        Object var2 = this.elementData(var1);
        int var3 = this.size - var1 - 1;
        if (var3 > 0) {
            System.arraycopy(this.elementData, var1 + 1, this.elementData, var1, var3);
        }

        this.elementData[--this.size] = null;
        return var2;
    }

    public boolean remove(Object var1) {
        int var2;
        if (var1 == null) {
            for(var2 = 0; var2 < this.size; ++var2) {
                if (this.elementData[var2] == null) {
                    this.fastRemove(var2);
                    return true;
                }
            }
        } else {
            for(var2 = 0; var2 < this.size; ++var2) {
                if (var1.equals(this.elementData[var2])) {
                    this.fastRemove(var2);
                    return true;
                }
            }
        }

        return false;
    }

    /*
     * Private remove method that skips bounds checking and does not
     * return the value removed.
     */
    private void fastRemove(int var1) {
        ++this.modCount;
        int var2 = this.size - var1 - 1;
        if (var2 > 0) {
            System.arraycopy(this.elementData, var1 + 1, this.elementData, var1, var2);
        }

        this.elementData[--this.size] = null;
    }

    /*
    * 删除所有元素
    */
    public void clear() {
        ++this.modCount;

        for(int var1 = 0; var1 < this.size; ++var1) {
            this.elementData[var1] = null;
        }

        this.size = 0;
    }

    /**
     * 按指定集合的Iterator返回的顺序将指定集合中的所有元素追加到此列表的末尾。
     */
    public boolean addAll(Collection<? extends E> var1) {
        Object[] var2 = var1.toArray();
        int var3 = var2.length;
        this.ensureCapacityInternal(this.size + var3);
        System.arraycopy(var2, 0, this.elementData, this.size, var3);
        this.size += var3;
        return var3 != 0;
    }

    /**
     * 将指定集合中的所有元素插入到此列表中，从指定的位置开始。
     */
    public boolean addAll(int var1, Collection<? extends E> var2) {
        this.rangeCheckForAdd(var1);
        Object[] var3 = var2.toArray();
        int var4 = var3.length;
        this.ensureCapacityInternal(this.size + var4);
        int var5 = this.size - var1;
        if (var5 > 0) {
            System.arraycopy(this.elementData, var1, this.elementData, var1 + var4, var5);
        }

        System.arraycopy(var3, 0, this.elementData, var1, var4);
        this.size += var4;
        return var4 != 0;
    }

    protected void removeRange(int var1, int var2) {
        ++this.modCount;
        int var3 = this.size - var2;
        System.arraycopy(this.elementData, var2, this.elementData, var1, var3);
        int var4 = this.size - (var2 - var1);

        for(int var5 = var4; var5 < this.size; ++var5) {
            this.elementData[var5] = null;
        }

        this.size = var4;
    }

    /*
    * 检查索引是否越界
    */
    private void rangeCheck(int var1) {
        if (var1 >= this.size) {
            throw new IndexOutOfBoundsException(this.outOfBoundsMsg(var1));
        }
    }

    private void rangeCheckForAdd(int var1) {
        if (var1 > this.size || var1 < 0) {
            throw new IndexOutOfBoundsException(this.outOfBoundsMsg(var1));
        }
    }

    /**
     * 返回越界的具体信息
     */
    private String outOfBoundsMsg(int var1) {
        return "Index: " + var1 + ", Size: " + this.size;
    }

    /**
     * 删除集合内指定元素
     */
    public boolean removeAll(Collection<?> var1) {
        Objects.requireNonNull(var1);
        return this.batchRemove(var1, false);
    }

    public boolean retainAll(Collection<?> var1) {
        Objects.requireNonNull(var1);
        return this.batchRemove(var1, true);
    }

    private boolean batchRemove(Collection<?> var1, boolean var2) {
        Object[] var3 = this.elementData;
        int var4 = 0;
        int var5 = 0;
        boolean var6 = false;

        while(true) {
            boolean var11 = false;

            try {
                var11 = true;
                if (var4 >= this.size) {
                    var11 = false;
                    break;
                }

                if (var1.contains(var3[var4]) == var2) {
                    var3[var5++] = var3[var4];
                }

                ++var4;
            } finally {
                if (var11) {
                    if (var4 != this.size) {
                        System.arraycopy(var3, var4, var3, var5, this.size - var4);
                        var5 += this.size - var4;
                    }

                    if (var5 != this.size) {
                        for(int var9 = var5; var9 < this.size; ++var9) {
                            var3[var9] = null;
                        }

                        this.modCount += this.size - var5;
                        this.size = var5;
                        var6 = true;
                    }

                }
            }
        }

        if (var4 != this.size) {
            System.arraycopy(var3, var4, var3, var5, this.size - var4);
            var5 += this.size - var4;
        }

        if (var5 != this.size) {
            for(int var7 = var5; var7 < this.size; ++var7) {
                var3[var7] = null;
            }

            this.modCount += this.size - var5;
            this.size = var5;
            var6 = true;
        }

        return var6;
    }

    private void writeObject(ObjectOutputStream var1) throws IOException {
        int var2 = this.modCount;
        var1.defaultWriteObject();
        var1.writeInt(this.size);

        for(int var3 = 0; var3 < this.size; ++var3) {
            var1.writeObject(this.elementData[var3]);
        }

        if (this.modCount != var2) {
            throw new ConcurrentModificationException();
        }
    }

    private void readObject(ObjectInputStream var1) throws IOException, ClassNotFoundException {
        this.elementData = EMPTY_ELEMENTDATA;
        var1.defaultReadObject();
        var1.readInt();
        if (this.size > 0) {
            int var2 = calculateCapacity(this.elementData, this.size);
            SharedSecrets.getJavaOISAccess().checkArray(var1, Object[].class, var2);
            this.ensureCapacityInternal(this.size);
            Object[] var3 = this.elementData;

            for(int var4 = 0; var4 < this.size; ++var4) {
                var3[var4] = var1.readObject();
            }
        }

    }

    /**
     * 从列表中的指定位置开始，返回列表中的元素（按正确顺序）的列表迭代器。
     *指定的索引表示初始调用将返回的第一个元素为next 。 初始调用previous将返回指定索引减1的元素。
     *返回的列表迭代器是fail-fast 。
     */
    public ListIterator<E> listIterator(int var1) {
        if (var1 >= 0 && var1 <= this.size) {
            return new ArrayList.ListItr(var1);
        } else {
            throw new IndexOutOfBoundsException("Index: " + var1);
        }
    }

    /**
     *返回列表中的列表迭代器（按适当的顺序）。
     *返回的列表迭代器是fail-fast 。
     */
    public ListIterator<E> listIterator() {
        return new ArrayList.ListItr(0);
    }

    /**
     *以正确的顺序返回该列表中的元素的迭代器。
     *返回的迭代器是fail-fast 。
     */
    public Iterator<E> iterator() {
        return new ArrayList.Itr();
    }

    public List<E> subList(int var1, int var2) {
        subListRangeCheck(var1, var2, this.size);
        return new ArrayList.SubList(this, 0, var1, var2);
    }

    static void subListRangeCheck(int var0, int var1, int var2) {
        if (var0 < 0) {
            throw new IndexOutOfBoundsException("fromIndex = " + var0);
        } else if (var1 > var2) {
            throw new IndexOutOfBoundsException("toIndex = " + var1);
        } else if (var0 > var1) {
            throw new IllegalArgumentException("fromIndex(" + var0 + ") > toIndex(" + var1 + ")");
        }
    }

    public void forEach(Consumer<? super E> var1) {
        Objects.requireNonNull(var1);
        int var2 = this.modCount;
        Object[] var3 = (Object[])this.elementData;
        int var4 = this.size;

        for(int var5 = 0; this.modCount == var2 && var5 < var4; ++var5) {
            var1.accept(var3[var5]);
        }

        if (this.modCount != var2) {
            throw new ConcurrentModificationException();
        }
    }

    public Spliterator<E> spliterator() {
        return new ArrayList.ArrayListSpliterator(this, 0, -1, 0);
    }

    public boolean removeIf(Predicate<? super E> var1) {
        Objects.requireNonNull(var1);
        int var2 = 0;
        BitSet var3 = new BitSet(this.size);
        int var4 = this.modCount;
        int var5 = this.size;

        for(int var6 = 0; this.modCount == var4 && var6 < var5; ++var6) {
            Object var7 = this.elementData[var6];
            if (var1.test(var7)) {
                var3.set(var6);
                ++var2;
            }
        }

        if (this.modCount != var4) {
            throw new ConcurrentModificationException();
        } else {
            boolean var10 = var2 > 0;
            if (var10) {
                int var11 = var5 - var2;
                int var8 = 0;

                for(int var9 = 0; var8 < var5 && var9 < var11; ++var9) {
                    var8 = var3.nextClearBit(var8);
                    this.elementData[var9] = this.elementData[var8];
                    ++var8;
                }

                for(var8 = var11; var8 < var5; ++var8) {
                    this.elementData[var8] = null;
                }

                this.size = var11;
                if (this.modCount != var4) {
                    throw new ConcurrentModificationException();
                }

                ++this.modCount;
            }

            return var10;
        }
    }

    public void replaceAll(UnaryOperator<E> var1) {
        Objects.requireNonNull(var1);
        int var2 = this.modCount;
        int var3 = this.size;

        for(int var4 = 0; this.modCount == var2 && var4 < var3; ++var4) {
            this.elementData[var4] = var1.apply(this.elementData[var4]);
        }

        if (this.modCount != var2) {
            throw new ConcurrentModificationException();
        } else {
            ++this.modCount;
        }
    }

    public void sort(Comparator<? super E> var1) {
        int var2 = this.modCount;
        Arrays.sort((Object[])this.elementData, 0, this.size, var1);
        if (this.modCount != var2) {
            throw new ConcurrentModificationException();
        } else {
            ++this.modCount;
        }
    }

    static final class ArrayListSpliterator<E> implements Spliterator<E> {
        private final ArrayList<E> list;
        private int index;
        private int fence;
        private int expectedModCount;

        ArrayListSpliterator(ArrayList<E> var1, int var2, int var3, int var4) {
            this.list = var1;
            this.index = var2;
            this.fence = var3;
            this.expectedModCount = var4;
        }

        private int getFence() {
            int var1;
            if ((var1 = this.fence) < 0) {
                ArrayList var2;
                if ((var2 = this.list) == null) {
                    var1 = this.fence = 0;
                } else {
                    this.expectedModCount = var2.modCount;
                    var1 = this.fence = var2.size;
                }
            }

            return var1;
        }

        public ArrayList.ArrayListSpliterator<E> trySplit() {
            int var1 = this.getFence();
            int var2 = this.index;
            int var3 = var2 + var1 >>> 1;
            return var2 >= var3 ? null : new ArrayList.ArrayListSpliterator(this.list, var2, this.index = var3, this.expectedModCount);
        }

        public boolean tryAdvance(Consumer<? super E> var1) {
            if (var1 == null) {
                throw new NullPointerException();
            } else {
                int var2 = this.getFence();
                int var3 = this.index;
                if (var3 < var2) {
                    this.index = var3 + 1;
                    Object var4 = this.list.elementData[var3];
                    var1.accept(var4);
                    if (this.list.modCount != this.expectedModCount) {
                        throw new ConcurrentModificationException();
                    } else {
                        return true;
                    }
                } else {
                    return false;
                }
            }
        }

        public void forEachRemaining(Consumer<? super E> var1) {
            if (var1 == null) {
                throw new NullPointerException();
            } else {
                ArrayList var5;
                Object[] var6;
                if ((var5 = this.list) != null && (var6 = var5.elementData) != null) {
                    int var3;
                    int var4;
                    if ((var3 = this.fence) < 0) {
                        var4 = var5.modCount;
                        var3 = var5.size;
                    } else {
                        var4 = this.expectedModCount;
                    }

                    int var2;
                    if ((var2 = this.index) >= 0 && (this.index = var3) <= var6.length) {
                        while(var2 < var3) {
                            Object var7 = var6[var2];
                            var1.accept(var7);
                            ++var2;
                        }

                        if (var5.modCount == var4) {
                            return;
                        }
                    }
                }

                throw new ConcurrentModificationException();
            }
        }

        public long estimateSize() {
            return (long)(this.getFence() - this.index);
        }

        public int characteristics() {
            return 16464;
        }
    }

    private class SubList extends AbstractList<E> implements RandomAccess {
        private final AbstractList<E> parent;
        private final int parentOffset;
        private final int offset;
        int size;

        SubList(AbstractList<E> var2, int var3, int var4, int var5) {
            this.parent = var2;
            this.parentOffset = var4;
            this.offset = var3 + var4;
            this.size = var5 - var4;
            this.modCount = ArrayList.this.modCount;
        }

        public E set(int var1, E var2) {
            this.rangeCheck(var1);
            this.checkForComodification();
            Object var3 = ArrayList.this.elementData(this.offset + var1);
            ArrayList.this.elementData[this.offset + var1] = var2;
            return var3;
        }

        public E get(int var1) {
            this.rangeCheck(var1);
            this.checkForComodification();
            return ArrayList.this.elementData(this.offset + var1);
        }

        public int size() {
            this.checkForComodification();
            return this.size;
        }

        public void add(int var1, E var2) {
            this.rangeCheckForAdd(var1);
            this.checkForComodification();
            this.parent.add(this.parentOffset + var1, var2);
            this.modCount = this.parent.modCount;
            ++this.size;
        }

        public E remove(int var1) {
            this.rangeCheck(var1);
            this.checkForComodification();
            Object var2 = this.parent.remove(this.parentOffset + var1);
            this.modCount = this.parent.modCount;
            --this.size;
            return var2;
        }

        protected void removeRange(int var1, int var2) {
            this.checkForComodification();
            this.parent.removeRange(this.parentOffset + var1, this.parentOffset + var2);
            this.modCount = this.parent.modCount;
            this.size -= var2 - var1;
        }

        public boolean addAll(Collection<? extends E> var1) {
            return this.addAll(this.size, var1);
        }

        public boolean addAll(int var1, Collection<? extends E> var2) {
            this.rangeCheckForAdd(var1);
            int var3 = var2.size();
            if (var3 == 0) {
                return false;
            } else {
                this.checkForComodification();
                this.parent.addAll(this.parentOffset + var1, var2);
                this.modCount = this.parent.modCount;
                this.size += var3;
                return true;
            }
        }

        public Iterator<E> iterator() {
            return this.listIterator();
        }

        public ListIterator<E> listIterator(final int var1) {
            this.checkForComodification();
            this.rangeCheckForAdd(var1);
            final int var2 = this.offset;
            return new ListIterator<E>() {
                int cursor = var1;
                int lastRet = -1;
                int expectedModCount;

                {
                    this.expectedModCount = ArrayList.this.modCount;
                }

                public boolean hasNext() {
                    return this.cursor != SubList.this.size;
                }

                public E next() {
                    this.checkForComodification();
                    int var1x = this.cursor;
                    if (var1x >= SubList.this.size) {
                        throw new NoSuchElementException();
                    } else {
                        Object[] var2x = ArrayList.this.elementData;
                        if (var2 + var1x >= var2x.length) {
                            throw new ConcurrentModificationException();
                        } else {
                            this.cursor = var1x + 1;
                            return var2x[var2 + (this.lastRet = var1x)];
                        }
                    }
                }

                public boolean hasPrevious() {
                    return this.cursor != 0;
                }

                public E previous() {
                    this.checkForComodification();
                    int var1x = this.cursor - 1;
                    if (var1x < 0) {
                        throw new NoSuchElementException();
                    } else {
                        Object[] var2x = ArrayList.this.elementData;
                        if (var2 + var1x >= var2x.length) {
                            throw new ConcurrentModificationException();
                        } else {
                            this.cursor = var1x;
                            return var2x[var2 + (this.lastRet = var1x)];
                        }
                    }
                }

                public void forEachRemaining(Consumer<? super E> var1x) {
                    Objects.requireNonNull(var1x);
                    int var2x = SubList.this.size;
                    int var3 = this.cursor;
                    if (var3 < var2x) {
                        Object[] var4 = ArrayList.this.elementData;
                        if (var2 + var3 >= var4.length) {
                            throw new ConcurrentModificationException();
                        } else {
                            while(var3 != var2x && SubList.this.modCount == this.expectedModCount) {
                                var1x.accept(var4[var2 + var3++]);
                            }

                            this.lastRet = this.cursor = var3;
                            this.checkForComodification();
                        }
                    }
                }

                public int nextIndex() {
                    return this.cursor;
                }

                public int previousIndex() {
                    return this.cursor - 1;
                }

                public void remove() {
                    if (this.lastRet < 0) {
                        throw new IllegalStateException();
                    } else {
                        this.checkForComodification();

                        try {
                            SubList.this.remove(this.lastRet);
                            this.cursor = this.lastRet;
                            this.lastRet = -1;
                            this.expectedModCount = ArrayList.this.modCount;
                        } catch (IndexOutOfBoundsException var2x) {
                            throw new ConcurrentModificationException();
                        }
                    }
                }

                public void set(E var1x) {
                    if (this.lastRet < 0) {
                        throw new IllegalStateException();
                    } else {
                        this.checkForComodification();

                        try {
                            ArrayList.this.set(var2 + this.lastRet, var1x);
                        } catch (IndexOutOfBoundsException var3) {
                            throw new ConcurrentModificationException();
                        }
                    }
                }

                public void add(E var1x) {
                    this.checkForComodification();

                    try {
                        int var2x = this.cursor;
                        SubList.this.add(var2x, var1x);
                        this.cursor = var2x + 1;
                        this.lastRet = -1;
                        this.expectedModCount = ArrayList.this.modCount;
                    } catch (IndexOutOfBoundsException var3) {
                        throw new ConcurrentModificationException();
                    }
                }

                final void checkForComodification() {
                    if (this.expectedModCount != ArrayList.this.modCount) {
                        throw new ConcurrentModificationException();
                    }
                }
            };
        }

        public List<E> subList(int var1, int var2) {
            ArrayList.subListRangeCheck(var1, var2, this.size);
            return ArrayList.this.new SubList(this, this.offset, var1, var2);
        }

        private void rangeCheck(int var1) {
            if (var1 < 0 || var1 >= this.size) {
                throw new IndexOutOfBoundsException(this.outOfBoundsMsg(var1));
            }
        }

        private void rangeCheckForAdd(int var1) {
            if (var1 < 0 || var1 > this.size) {
                throw new IndexOutOfBoundsException(this.outOfBoundsMsg(var1));
            }
        }

        private String outOfBoundsMsg(int var1) {
            return "Index: " + var1 + ", Size: " + this.size;
        }

        private void checkForComodification() {
            if (ArrayList.this.modCount != this.modCount) {
                throw new ConcurrentModificationException();
            }
        }

        public Spliterator<E> spliterator() {
            this.checkForComodification();
            return new ArrayList.ArrayListSpliterator(ArrayList.this, this.offset, this.offset + this.size, this.modCount);
        }
    }

    private class ListItr extends ArrayList<E>.Itr implements ListIterator<E> {
        ListItr(int var2) {
            super();
            this.cursor = var2;
        }

        public boolean hasPrevious() {
            return this.cursor != 0;
        }

        public int nextIndex() {
            return this.cursor;
        }

        public int previousIndex() {
            return this.cursor - 1;
        }

        public E previous() {
            this.checkForComodification();
            int var1 = this.cursor - 1;
            if (var1 < 0) {
                throw new NoSuchElementException();
            } else {
                Object[] var2 = ArrayList.this.elementData;
                if (var1 >= var2.length) {
                    throw new ConcurrentModificationException();
                } else {
                    this.cursor = var1;
                    return var2[this.lastRet = var1];
                }
            }
        }

        public void set(E var1) {
            if (this.lastRet < 0) {
                throw new IllegalStateException();
            } else {
                this.checkForComodification();

                try {
                    ArrayList.this.set(this.lastRet, var1);
                } catch (IndexOutOfBoundsException var3) {
                    throw new ConcurrentModificationException();
                }
            }
        }

        public void add(E var1) {
            this.checkForComodification();

            try {
                int var2 = this.cursor;
                ArrayList.this.add(var2, var1);
                this.cursor = var2 + 1;
                this.lastRet = -1;
                this.expectedModCount = ArrayList.this.modCount;
            } catch (IndexOutOfBoundsException var3) {
                throw new ConcurrentModificationException();
            }
        }
    }

    private class Itr implements Iterator<E> {
        int cursor;
        int lastRet = -1;
        int expectedModCount;

        Itr() {
            this.expectedModCount = ArrayList.this.modCount;
        }

        public boolean hasNext() {
            return this.cursor != ArrayList.this.size;
        }

        public E next() {
            this.checkForComodification();
            int var1 = this.cursor;
            if (var1 >= ArrayList.this.size) {
                throw new NoSuchElementException();
            } else {
                Object[] var2 = ArrayList.this.elementData;
                if (var1 >= var2.length) {
                    throw new ConcurrentModificationException();
                } else {
                    this.cursor = var1 + 1;
                    return var2[this.lastRet = var1];
                }
            }
        }

        public void remove() {
            if (this.lastRet < 0) {
                throw new IllegalStateException();
            } else {
                this.checkForComodification();

                try {
                    ArrayList.this.remove(this.lastRet);
                    this.cursor = this.lastRet;
                    this.lastRet = -1;
                    this.expectedModCount = ArrayList.this.modCount;
                } catch (IndexOutOfBoundsException var2) {
                    throw new ConcurrentModificationException();
                }
            }
        }

        public void forEachRemaining(Consumer<? super E> var1) {
            Objects.requireNonNull(var1);
            int var2 = ArrayList.this.size;
            int var3 = this.cursor;
            if (var3 < var2) {
                Object[] var4 = ArrayList.this.elementData;
                if (var3 >= var4.length) {
                    throw new ConcurrentModificationException();
                } else {
                    while(var3 != var2 && ArrayList.this.modCount == this.expectedModCount) {
                        var1.accept(var4[var3++]);
                    }

                    this.cursor = var3;
                    this.lastRet = var3 - 1;
                    this.checkForComodification();
                }
            }
        }

        final void checkForComodification() {
            if (ArrayList.this.modCount != this.expectedModCount) {
                throw new ConcurrentModificationException();
            }
        }
    }
}

```

#### 小结
在阅读源码的过程中， 我发现， 源码中使用System.arraycopy这个方法居多， 其次便是Arrays.copyOf  
System.arraycopy与Arrays.copyOf的**区别**是：
- arraycopy()需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置
- copyOf()是系统自动在内部新建一个数组，并返回该数组。

##### ArrayList 核心扩容
```java
/**
* 判断是否需要扩容
*/
private void ensureCapacityInternal(int var1) {
    this.ensureExplicitCapacity(calculateCapacity(this.elementData, var1));
}

private void ensureExplicitCapacity(int var1) {
    ++this.modCount;
    if (var1 - this.elementData.length > 0) {
        this.grow(var1);
    }

}
/**
* 扩容的核心算法
*/
private void grow(int var1) {
    int var2 = this.elementData.length;
    // 对于大数据的2进制运算,位移运算符比那些普通运算符的运算要快很多,因为程序仅仅移动一下而已,不去计算,这样提高了效率,节省了资源
    // 右移一位相当于除2，右移n位相当于除以 2 的 n 次方。这里 oldCapacity 明显右移了1位所以相当于oldCapacity /2。
    int var3 = var2 + (var2 >> 1);
    if (var3 - var1 < 0) {
        var3 = var1;
    }

    if (var3 - 2147483639 > 0) {
        var3 = hugeCapacity(var1);
    }

    this.elementData = Arrays.copyOf(this.elementData, var3);
}

private static int hugeCapacity(int var0) {
    if (var0 < 0) {
        throw new OutOfMemoryError();
    } else {
        return var0 > 2147483639 ? 2147483647 : 2147483639;
    }
}
```

#### [来自JAVA Guid的demo](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/ArrayList.md)
```JAVA
public class ArrayListDemo {

    public static void main(String[] srgs){
         ArrayList<Integer> arrayList = new ArrayList<Integer>();

         System.out.printf("Before add:arrayList.size() = %d\n",arrayList.size());

         arrayList.add(1);
         arrayList.add(3);
         arrayList.add(5);
         arrayList.add(7);
         arrayList.add(9);
         System.out.printf("After add:arrayList.size() = %d\n",arrayList.size());

         System.out.println("Printing elements of arrayList");
         // 三种遍历方式打印元素
         // 第一种：通过迭代器遍历
         System.out.print("通过迭代器遍历:");
         Iterator<Integer> it = arrayList.iterator();
         while(it.hasNext()){
             System.out.print(it.next() + " ");
         }
         System.out.println();

         // 第二种：通过索引值遍历
         System.out.print("通过索引值遍历:");
         for(int i = 0; i < arrayList.size(); i++){
             System.out.print(arrayList.get(i) + " ");
         }
         System.out.println();

         // 第三种：for循环遍历
         System.out.print("for循环遍历:");
         for(Integer number : arrayList){
             System.out.print(number + " ");
         }

         // toArray用法
         // 第一种方式(最常用)
         Integer[] integer = arrayList.toArray(new Integer[0]);

         // 第二种方式(容易理解)
         Integer[] integer1 = new Integer[arrayList.size()];
         arrayList.toArray(integer1);

         // 抛出异常，java不支持向下转型
         //Integer[] integer2 = new Integer[arrayList.size()];
         //integer2 = arrayList.toArray();
         System.out.println();

         // 在指定位置添加元素
         arrayList.add(2,2);
         // 删除指定位置上的元素
         arrayList.remove(2);    
         // 删除指定元素
         arrayList.remove((Object)3);
         // 判断arrayList是否包含5
         System.out.println("ArrayList contains 5 is: " + arrayList.contains(5));

         // 清空ArrayList
         arrayList.clear();
         // 判断ArrayList是否为空
         System.out.println("ArrayList is empty: " + arrayList.isEmpty());
    }
}
```
