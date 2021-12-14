# 7. Java 中的 13 个原子操作类【重点】

Java 从 JDK 1.5 开始提供了 `java.util.concurrent.atomic` 包（以下简称 Atomic 包），这个包中的原子操作类提供了一种用法简单、性能高效、线程安全地更新一个变量的方式。因为变量的类型有很多种，所以在 Atomic 包里一共提供了 13 个类，属于 4 种类型的原子更新方式，分别是**原子更新基本类型**、**原子更新数组**、**原子更新引用**和**原子更新属性（字段）**。Atomic 包里的类基本都是使用 Unsafe 实现的包装类。



## 7.1 原子更新基本类型类

使用原子的方式更新**基本类型**，Atomic 包提供了以下 3 个类：

- **AtomicBoolean**：原子更新布尔类型。 

- **AtomicInteger**：原子更新整型。 

- **AtomicLong**：原子更新长整型。 

以上 3 个类提供的方法几乎一模一样，所以这里仅以 AtomicInteger 为例进行讲解，AtomicInteger 的常用方法如下：

- `int addAndGet(int delta)`：以原子方式将输入的数值与实例中的值（AtomicInteger 里的 value）相加，并返回结果。 

- `boolean compareAndSet(int expect，int update)`：如果输入的数值等于预期值，则以原子方式将该值设置为输入的值。

- `int getAndIncrement()`：以原子方式将当前值加 1，注意，这里返回的是自增前的值。 

- `void lazySet(int newValue)`：最终会设置成 newValue，使用 lazySet 设置值后，可能导致其他线程在之后的一小段时间内还是可以读到旧的值。

- `int getAndSet(int newValue)`：以原子方式设置为 newValue 的值，并返回旧值。 



AtomicInteger 示例代码如下所示：

> AtomicIntegerTest.java：

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicIntegerTest {
    static AtomicInteger ai = new AtomicInteger(1);
    public static void main(String[] args) {
        System.out.println(ai.getAndIncrement());
        System.out.println(ai.get());
    } 
}
```

输出结果如下：

```
1
2
```



`getAndIncrement` 原子操作的实现原理：

`getAndIncrement` 的源码如下代码清单所示：

> AtomicInteger.java：

```java
public final int getAndIncrement() {
    for (; ; ) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next)) return current;
    } 
}

public final boolean compareAndSet(int expect, int update) {
	return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

源码中 for 循环体的第一步先取得 AtomicInteger 里存储的数值，第二步对 AtomicInteger 的当前数值进行加 1 操作，关键的第三步调用 `compareAndSet` 方法来进行原子更新操作，该方法**先检查当前数值是否等于 current，等于意味着 AtomicInteger 的值没有被其他线程修改过，则将 AtomicInteger 的当前数值更新成 next 的值，如果不等 compareAndSet 方法会返回 false，程序会进入 for 循环重新进行 compareAndSet 操作。**



Atomic 包提供了 3 种基本类型的原子更新，但是 Java 的基本类型里还有 char、float 和 double 等。那么问题来了，如何原子的更新其他的基本类型呢？Atomic 包里的类基本都是使用 Unsafe 实现的，看一下 Unsafe 的源码，如代码清单所示：

> Unsafe.java：

```java
/**
* 如果当前数值是 expected，则原子的将 Java 变量更新成 x 
*
* @return 如果更新成功则返回 true
*/
public final native boolean compareAndSwapObject(Object o, long offset,
Object expected, Object x);
public final native boolean compareAndSwapInt(Object o, long offset, int expected, int x);
public final native boolean compareAndSwapLong(Object o, long offset, long expected, long x);
```

通过代码，发现 Unsafe 只提供了 3 种 CAS 方法：`compareAndSwapObject`、`compareAndSwapInt` 和 `compareAndSwapLong`，再看 AtomicBoolean 源码，发现它是先把 Boolean 转换成整型，再使用 compareAndSwapInt 进行 CAS，所以原子更新 char、float 和 double 变量也可以用类似的思路来实现。 



## 7.2 原子更新数组

通过原子的方式更新**数组里的某个元素**，Atomic 包提供了以下 3 个类：

- **AtomicIntegerArray**：原子更新整型数组里的元素。 

- **AtomicLongArray**：原子更新长整型数组里的元素。 

- **AtomicReferenceArray**：原子更新引用类型数组里的元素。



**AtomicIntegerArray** 类主要是提供原子的方式更新数组里的整型，其常用方法如下：

- `int addAndGet(int i, int delta)`：以原子方式将输入值与数组中索引 i 的元素相加。 

- `boolean compareAndSet(int i, int expect, int update)`：如果当前值等于预期值，则以原子方式将数组位置 i 的元素设置成 update 值。 



以上几个类提供的方法几乎一样，所以这里仅以 AtomicIntegerArray 为例进行讲解， AtomicIntegerArray 的使用实例代码如下代码清单所示：

> AtomicIntegerArrayTest.java：

```java
public static void main(String[] args) {
    static int[] value = new int[]{1，2};
    static AtomicIntegerArray ai = new AtomicIntegerArray(value);
    ai.getAndSet(0， 3);
    System.out.println(ai.get(0));
    System.out.println(value[0]);
}
```

以下是输出的结果：

```
3
1
```

>需要注意的是，数组 value 通过构造方法传递进去，然后 AtomicIntegerArray 会将当前数组复制一份，所以当 AtomicIntegerArray 对内部的数组元素进行修改时，不会影响传入的数组。



## 7.3 原子更新引用类型

原子更新基本类型的 AtomicInteger，只能更新一个变量，如果要**原子更新多个变量**，就需要使用这个原子更新**引用类型**提供的类。Atomic 包提供了以下 3 个类：

- **AtomicReference**：原子更新引用类型。 

- **AtomicReferenceFieldUpdater**：原子更新引用类型里的字段。

- **AtomicMarkableReference**：原子更新带有标记位的引用类型。可以原子更新一个布尔类型的标记位和引用类型。构造方法是 `AtomicMarkableReference(V initialRef, boolean initialMark)`

以上几个类提供的方法几乎一样，所以这里仅以 AtomicReference 为例进行讲解，AtomicReference 的使用示例代码如代码清单所示：

> AtomicReferenceTest.java：

```java
public class AtomicReferenceTest {
    public static AtomicReference<user> atomicUserRef = new
    AtomicReference<user>();
    
    public static void main(String[] args) {
        User user = new User("conan"， 15);
        atomicUserRef.set(user);
        User updateUser = new User("Shinichi"， 17); 
        atomicUserRef.compareAndSet(user， updateUser);
        System.out.println(atomicUserRef.get().getName());
        System.out.println(atomicUserRef.get().getOld());
    }
    
    static class User {
        private String name;
        private int old;
        
        public User(String name， int old) {
            this.name = name;
            this.old = old;
        }
        
        public String getName() {
        	return name;
        }
        
        public int getOld() {
        	return old;
        }
    } 
}
```

代码中首先构建一个 user 对象，然后把 user 对象设置进 AtomicReferenc 中，最后调用 `compareAndSet` 方法进行原子更新操作，实现原理同 AtomicInteger 里的 `compareAndSet` 方法。代码执行后输出结果如下：

```
Shinichi
17
```



## 7.4 原子更新字段类

如果需原子地更新**某个类里的某个字段**时，就需要使用原子更新字段类，Atomic 包提供了以下 3 个类进行原子字段更新：

- **AtomicIntegerFieldUpdater**：原子更新整型的字段的更新器。 

- **AtomicLongFieldUpdater**：原子更新长整型字段的更新器。 

- **AtomicStampedReference**：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。 



要想原子地更新字段类需要两步：

1. 第一步，因为原子更新字段类都是抽象类，每次使用的时候必须使用静态方法 `newUpdater()`创建一个更新器，并且需要设置想要更新的类和属性。

2. 第二步，更新类的字段（属性）必须使用 `public volatile` 修饰符。



以上 3 个类提供的方法几乎一样，所以这里仅以 AtomicIntegerFieldUpdater 为例进行讲解， AtomicIntegerFieldUpdater 的示例代码如代码清单所示：

> AtomicIntegerFieldUpdaterTest.java：

```java
public class AtomicIntegerFieldUpdaterTest {
    // 创建原子更新器，并设置需要更新的对象类和对象的属性
    private static AtomicIntegerFieldUpdater<User> a = AtomicIntegerFieldUpdater.
    newUpdater(User.class， "old");
    
    public static void main(String[] args) { // 设置柯南的年龄是 10 岁
        User conan = new User("conan"， 10);
        // 柯南长了一岁，但是仍然会输出旧的年龄
        System.out.println(a.getAndIncrement(conan));
        // 输出柯南现在的年龄
        System.out.println(a.get(conan));
    }
    
    public static class User {
        private String name;
        public volatile int old;
        
        public User(String name， int old) {
            this.name = name;
            this.old = old;
        }
        
        public String getName() {
        	return name;
        }
        
        public int getOld() {
        	return old;
        }
    } 
}
```

代码执行后输出如下：

```
10
11
```

