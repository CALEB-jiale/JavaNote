# Collections

<img src="https://github.com/CALEB-jiale/JavaNote/raw/main/14.1%20集合框架的概述.png" alt="14.1 集合框架的概述" style="zoom:50%;" />

上图是集合框架的概述，其中绿色的是接口，蓝色的是类。

顶部的 Iterable 接口，实现这个接口的类都可以被用于 for 循环。

下面的 Collection 接口继承了上面的 Iterable 接口。实现这个接口的类都可以充当容器，或表示一些对象的集合。它实现的功能有添加（add），移除（remove），检查对象是否存在（contains），清空（clear）等等。

接下来是 Collection 的三个子接口，列表（List）、队列（Queue）和集合（Set）。

List 允许我们使用有序集合，并使用对象的索引访问对象，就像我们在数组（Array）中做的那样。

ArrayList 是动态数组。它在内部用数组来存储对象，如果数组已满，它会自动扩充（resize）。

LinkedList 是基于链表的数据结构。

Queue 使用的情况通常是分配共享资源。比如说一个打印机一次只能打印一张纸，我们就需要把任务放入队列，然后打印机就可以一个接着一个地完成任务。

Set 表示一个没有重复元素的集合。

## The Need for Iterables

准确地说，Iterable 接口不是集合的框架中的一部分。

但在集合中要加入 Iterable 是因为如果没有这个，我们就不能实现在类的外部循环访问集合中的内容。

```java
public class GenericList<T> {
    private T[] items = (T[]) new Object[10];
    private int count;

    public void add(T item) {
        items[count++] = item;
    }

    public T get(int index) {
        return items[index];
    }
}
```

一种方法是把上面代码的第二行由 private 改成 public，但很明显这会导致很多问题。

这就说明 Iterable 的重要性。

## The Iterable Interface

Iterable 接口有三个方法，其中 forEach 和 spliterator 都有默认实现，因此在实现 Iterable 接口的时候只需要实现 iterator 方法即可。

iterator 返回一个迭代器对象，它的具体功能下一节再讲。

实现以下代码：

```java
public class GenericList<T> implements Iterable<T> {
    private T[] items = (T[]) new Object[10];
    private int count;

    public void add(T item) {
        items[count++] = item;
    }

    public T get(int index) {
        return items[index];
    }

    @Override
    public Iterator<T> iterator() {
        return null;
    }
}
```

其中重写的快捷键是 option + enter。

方法的具体的实现内容见下一节。

回到 main 函数，可以实现以下代码：

```java
public class Main {
    public static void main(String[] args) {
        var list = new GenericList<String>();
        var iterator = list.iterator();
        list.add("a");
        list.add("b");
        while (iterator.hasNext()) {
            var current = iterator.next();
            System.out.println(current);
        }
    }
}
```

假设我们的列表如下：

`[a, b, c]`

迭代器就会类似一个指针，它一开始指向第一个对象，即 `a`。

每一次 while 循环，如果还有更多的对象，迭代器就会读取当前的对象然后向后移动指针。

到最后指针指向空，hasNext 就会返回 false，while 循环结束。

上面的代码实际上可以通过下面的方式实现：

```java
public class Main {
    public static void main(String[] args) {
        var list = new GenericList<String>();
        var iterator = list.iterator();
        list.add("a");
        list.add("b");
        for (var item : list)
            System.out.println(item);
    }
}
```

上面的两个代码都不能运行，因为我们还没有实现 iterator 方法。

但即便我们把 iterator 方法正确实现了，如果我们把 `implements Iterable<T>` 移除，只保留 iterator 方法，使用 for 循环的代码会报错，while 循环则不会。

因为通过观察 Bytecode 我们可以知道，for 循环实际上不仅使用了 iterator 方法，还使用了 forEach 方法。

## The Iterator Interface

这一节要实现迭代器。

从声明中可以看出，iterator 方法会返回一个迭代器对象，这个迭代器实际上是一个被声明过的接口。

接口中共声明了四个方法，其中 forEachRemaining 和 remove 两个方法都已经有默认实现，所以我们只需要实现 hasNext 和 next 两个方法。

在 iterator 的声明之后，我们要先创建一个私有的嵌套类 ListIterator。

```java
public class GenericList<T> implements Iterable<T> {
    private T[] items = (T[]) new Object[10];
    private int count;

    public void add(T item) {
        items[count++] = item;
    }

    public T get(int index) {
        return items[index];
    }

    @Override
    public Iterator<T> iterator() {
        return new ListIterator(this);
    }

    private class ListIterator implements Iterator<T> {
        private GenericList<T> list;
        private int index;

        public ListIterator(GenericList<T> list) {
            this.list = list;
        }

        @Override
        public boolean hasNext() {
            return (index < list.count);
        }

        @Override
        public T next() {
            return list.items[index++];
        }
    }
}
```

完成上述代码后，就可以运行上一节的 main 函数了。

 ## The Collection Interface

Collection 接口也是 generic 的，但我们可以发现它的 <T> 被改成了 <E>，这是声明具有集合意义的类或接口通用约定，因为 E 代表 element。

我们创建一个新包 collections，并在其中创建新的类 CollectionsDemo。

```java
public class CollectionDemo {
    public static void show() {
        Collection<String> collection = new ArrayList<>();
        collection.add("a");
        collection.add("b");
        collection.add("c");
        System.out.println(collection);
        for (var item : collection)
            System.out.println(item);
    }
}
```

在 main 函数中调用 show 方法，即可以看到结果。

注意代码第三行的 ` Collection<String>` 不能用 `var` 来代替，不然的话这个变量的类型就是 ArrayList 类，而不是 Collection 接口，所以它将包括 ArrayList 中声明的其它方法。之前说过我们应该针对接口进行编程，这将使我们的应用程序更加灵活和松耦合。

如果我们想一次性添加多个元素，而不是每次都调用 add 方法，就可以用 Utils 中的 Collections.addAll 方法。

通过快捷键 command + P 可以查看函数的参数信息。

Collections.addAll 方法的第一个参数是要修改的集合，第二个参数有“...”说明可以输入多个参数。

```java
public class CollectionDemo {
    public static void show() {
        Collection<String> collection = new ArrayList<>();
        Collections.addAll(collection, "a", "b", "c");
        System.out.println(collection);
        for (var item : collection)
            System.out.println(item);
    }
}
```

Collection 类中的 size 方法可以返回集合中元素的个数。

此外常用的方法还有 remove，clear，isEmpty, contains 等等。

有时我们希望将集合转换为常规数组，因此我们调用 Collection 类中的 toArray 方法。

```java
public class CollectionDemo {
    public static void show() {
        Collection<String> collection = new ArrayList<>();
        Collections.addAll(collection, "a", "b", "c");
        var objectArray = collection.toArray();
        var stringArray = collection.toArray(new String[0]);
    }
}
```

从上面的代码可以看到，这个方法有两种重载（实际有三种，只讲两种）：

1. `toArray()` 如果不穿入任何参数，将会返回一个对象数组，这个数组中的每一项都是 Object 类的对象。这意味着如果我们拿出其中的某一项，然后使用 `.` 操作符，编辑器的体式中不会有任何与 String 有关的方法，尽管我们的集合本来是一个 String 的集合。
2. `toArray(T[] a)` 这个形式的重载的使用方法如上面的代码所示，其中 `String[0]` 不是说就创建一个空数组，而只是一种约定，编译器会自己根据需要去创建数组。这么做是因为我们不想每次使用这个方法，都要去查一下集合中的元素个数。

有时我们会想要比较两个集合是否相等：

```java
public class CollectionDemo {
    public static void show() {
        Collection<String> collection = new ArrayList<>();
        Collections.addAll(collection, "a", "b", "c");

        Collection<String> other = new ArrayList<>();
        other.addAll(collection);

        System.out.println(collection == other);
        System.out.println(collection.equals(other));
    }
}
```

第九行的代码我们会得到 false，第十行的代码我们会得到 true。

## The List Interface

List 接口是一个有序的 Collection 接口，也称为序列。所以在 List 中我们可以通过索引来访问对象。

在集合中我们并不关心对象的索引，我们只关心从集合中添加或者删除它们。

我们在包 collections 中新创建一个类 ListDemo。

List 接口中，除了 Collection 接口中的方法外，还有如下常用方法：

```java
public class ListDemo {
    public static void show() {
        List<String> list = new ArrayList<>();
        Collections.addAll(list, "a", "b", "c", "d", "e", "f");
        System.out.println(list.get(0));
        list.add(0, "1");
        System.out.println(list.subList(0, 2));

        list.set(0, "a+");
        System.out.println(list);

        list.remove(0);
        System.out.println(list);
    }
}
```

上述代码的输出结果为：

> a
> [1, a]
> [a+, a, b, c, d, e, f]
> [a, b, c, d, e, f]

List 的常用方法还有 indexOf 和 lastIndexOf。

## The Comparable Interface

这一节要讲的是如何通过 Comparable 接口来对数据排序。

在 collections 包中新增 Customer 类。

```java
public class Customer {
    private String name;

    public Customer(String name) {
        this.name = name;
    }
}
```

在 main 函数中进行如下操作：

```java
public class Main {
    public static void main(String[] args) {
        List<Customer> customers = new ArrayList<>();
        customers.add(new Customer("b"));
        customers.add(new Customer("a"));
        customers.add(new Customer("c"));
        Collections.sort(customers);
        System.out.println(customers);
    }
}
```

此时编译器会报错告诉我们 Customer 没有实现 Comparable 接口。

Comparable 接口的实现我们之前做过，这里只看结果。

```java
public class Customer implements Comparable<Customer>{
    private String name;

    public Customer(String name) {
        this.name = name;
    }

    @Override
    public int compareTo(Customer o) {
        return name.compareTo(o.name);
    }

    @Override
    public String toString() {
        return name;
    }
}
```

注意要实现 toString 方法，不然会打印出一串 Hash 码。

## The Comparator Interface

上面实现的比较方法不是很灵活，因为它只是通过比较 name 来排序，我们还可能希望通过顾客的电子邮件或者地址来排序，因此我们就需要用到 Comparator 接口。

更新 Customer 的内容如下：

```java
public class Customer implements Comparable<Customer>{
    private String name;

    private String email;

    public Customer(String name, String email) {
        this.name = name;
        this.email = email;
    }

    @Override
    public int compareTo(Customer o) {
        return name.compareTo(o.name);
    }

    @Override
    public String toString() {
        return name;
    }

    public String getEmail() {
        return email;
    }
}
```

添加 EmailComparator 类如下：

```java
public class EmailComparator implements Comparator<Customer> {

    @Override
    public int compare(Customer o1, Customer o2) {
        return o1.getEmail().compareTo(o2.getEmail());
    }
}
```

其中 Comparator 接口有很多方法，但只有 compare 方法没有默认实现，所以只需要实现 compare 方法。它的逻辑与我们实现 Comparable 接口的 compareTo 方法的逻辑一致。

然后回到主函数，Utils 中的 Collections.sort 方法有两种不同的实现。

一种是像前面那样，直接传入继承了 Comparable 接口的类的对象。

另一种方法是在传入继承了 Comparable 接口的类的对象后，再传入一个实现了 Comparator 接口的类的对象。

更改 main 函数如下：

```java
public class Main {
    public static void main(String[] args) {
        List<Customer> customers = new ArrayList<>();
        customers.add(new Customer("b", "e3"));
        customers.add(new Customer("a", "e2"));
        customers.add(new Customer("c", "e1"));
        Collections.sort(customers, new EmailComparator());
        System.out.println(customers);
    }
}
```

## The Queue Interface

与 List 接口一样，Queue 也继承了 Collection 接口，我们会在我们希望根据接受到的任务的顺序来线性处理每个任务时使用 Queue，

我们比较可能会用到的 Queue 接口的实现有 ArrayDeque 和 PriorityQueue。

Deque 是 Double Ended Queue 的缩写，它是一个有着两个端的队列，所以项目可以从任意一端输入队列。

PriorityQueue 中的每个项目都有优先级，优先级决定了该项目在队列中的位置。操作系统中的流程管理用的就是 PriorityQueue，有些进程的优先级更高，因此他们有更多的 CPU 时间。

在 collections 包中新建 QueueDemo 类。

```java
public class QueueDemo {
    public static void show() {
        Queue<String> queue = new ArrayDeque<>();
        queue.add("c");
        queue.add("a");
        queue.offer("b");
        queue.offer("d");
        // d -> b -> a -> c
        var front = queue.peek();
        System.out.println(front);
        front = queue.element();
        System.out.println(front);
        // c
        front = queue.remove();
        System.out.println(front);
        // c
        System.out.println(queue);
        // d -> b -> a
        front = queue.poll();
        System.out.println(front);
        // a
        System.out.println(queue);
        // d -> b
    }
}
```

从上面代码可以看出，在 Queue 中添加元素的方法有 add 和 offer，他们的效果如注释所示。

add 和 offer 的区别取决于 Queue 的实现，在 ArrayDeque 的情况下，他们没有区别。

但在有些情况下，Queue 的大小可能是有限的，在这种情况下，如果 Queue 满了，add 会抛出异常，offer 会返回 false。

接下来的 peak 和 element 方法输出相同，都是返回队列最前面的元素，在这里输出的是 c。

但如果队列是空的，则 peak 会返回 null，而element 会抛出异常。

remove 方法会返回并删除队首的元素，如果队列为空，抛出异常。

poll 方法与 remove 类似，如果队列为空，返回 null。

## The Set Interface

Set 表示一个没有重复元素的集合。

```java
public class SetDemo {
    public static void show() {
        Set<String> set = new HashSet<>();
        set.add("sky");
        set.add("is");
        set.add("blue");
        set.add("blue");
        System.out.println(set);
    }
}
```

上面代码的输出结果为：

> [sky, blue, is]

说明 Set 不能保证元素的顺序。

现在假设我们有一个集合，我们希望通过 Set 来移除集合中的重复项，过程如下：

```java
public class SetDemo {
    public static void show() {
        Collection<String> collection = new ArrayList<>();
        Collections.addAll(collection, "a", "b", "c", "c");
        Set<String> set = new HashSet<>(collection);
        System.out.println(set);
    }
}
```

在创建 Set 的对象的时候，按住 command + P 可以查看参数信息，可以发现 HashSet 方法被重载了三次。我们可以传入一个 ` Collection<? extends String>` 类型的参数。

接下来讨论集合运算。

```java
public class SetDemo {
    public static void show() {
        Set<String> set1 = new HashSet<>(Arrays.asList("a", "b", "c"));
        Set<String> set2 = new HashSet<>(Arrays.asList("b", "c", "d"));

//        set1.addAll(set2);
//        set1.retainAll(set2);
        set1.removeAll(set2);
        System.out.println(set1);
    }
}
```

上面代码中的 Arrays 也是 Utils 中的类，这个类中的 asList 方法可以把传入的元素以列表形式返回。

要了解一下对集合的操作：

1. Union 求并集，对应的指令为 addAll；
2. Intersection 求交集，对应的指令为 retainAll；
3. Difference 求差集，对应的指令为 removeAll；

## Hash Tables

这一节要离开 Java，讨论一个计算机科学中重要的数据结构 Hash Table。

现在假设我们有一个列表来记录用户的信息，我们想从这个列表上找到对应邮件的客户，可以通过以下算法来实现：

```java
public class MapDemo {
    public static void show() {
        List<Customer> customers = new ArrayList<>();
        for (var customer : customers)
            if (customer.getEmail() == "e1")
                System.out.println("Found!");
    }
}
```

但上述代码的效率不高，因为随着列表中顾客数量的增加，循环需要的时间就越长。

此时我们就需要用到哈希表。

哈希表是一种特殊的数据结构，可以让我们快速查找对象。

在 Java 中我们有一个接口叫做 Map，它就是我们说的哈希表。

哈希表在不同的语言中有不同的表述：

- Java: Maps 
- Python: Dictionary 
- C#: Dictionary
- JavaScript: Objects

## The Map Interface

Map 不在集合的框架中，它没有继承 Collection 或 Iterable。

Map 接口有两个 generic 参数，K 和 V 分别是 Key 和 Value 的缩写。

所以 map（映射）的本质就是将键映射到值。

```java
public class MapDemo {
    public static void show() {
        var c1 = new Customer("a", "e1");
        var c2 = new Customer("b", "e2");

        Map<String, Customer> map = new HashMap<>();
        map.put(c1.getEmail(), c1);
        map.put(c2.getEmail(), c2);

        var customer = map.get("e1");
        System.out.println(customer);

        var unknown = new Customer("Unknown", "");
        customer = map.getOrDefault("e10", unknown);
        System.out.println(customer);

        var exists = map.containsKey("e10");
        System.out.println(exists);

        map.replace("e1", new Customer("a+", "e1"));
        System.out.println(map);
    }
}
```

如果 get 方法中传入的键在 map 中不存在，就返回 null。

如果 getOrDefault 方法中传入的键在 map 中不存在，就返回键后面设定的默认值。

与 containsKey 方法对应，也存在 containsValue 方法。

```java
public class MapDemo {
    public static void show() {
        var c1 = new Customer("a", "e1");
        var c2 = new Customer("b", "e2");

        Map<String, Customer> map = new HashMap<>();
        map.put(c1.getEmail(), c1);
        map.put(c2.getEmail(), c2);

        for (var key : map.keySet())
            System.out.println(key);

        for (var entry : map.entrySet()) {
            System.out.println(entry);
            System.out.println(entry.getKey());
            System.out.println(entry.getValue());
        }

        for (var customer : map.values())
            System.out.println(customer);
    }
}
```

注意 Map 没有继承 Iterable。但有三种方法找到存储在 map 中的对象：

1. keySet 这个方法会返回的是 `Set<String>`，它是 map 中键的集合，它是有继承 Iterable 的。
2. entrySet 这个方法返回的是 `Set<Entry<String, Customer>>`，所以他会返回一个 Set，它是有继承 Iterable 的。Entry 是类似“e1 = a”的形式的数据结构。
3. values 这个方法返回的是 ` Collection<Customer>`。

## Summary

| Interface  | Implementation |
| :--------: | :------------: |
| Collection |   ArrayList    |
|    List    |   ArrayList    |
|   Queue    |   ArrayDeque   |
|    Set     |    HashSet     |
|    Map     |    HashMap     |