# Generics

## 为何需要范性

先创建一个包 generics，然后创建一个类 List，它的实现如下：

```java
public class List {
    private int[] items = new int[10];
    private int count;

    public void add(int item) {
        items[count++] = item;
    }

    public int get(int index) {
        return items[index];
    }
}
```

这里我们忽略很多细节，比如要检查 index 的范围。

现在我们可以去主函数中为刚才创建的类构造对象，并对其进行相关操作。

但可能之后我们又会需要一个 list 去记录用户，因此我们又得创建一个类。

因为之前创建的 List 类只能用于存储整型数。

所以我们需要范性来解决这个问题。

## A Poor Solution

简单的方法如下：

```java
public class List {
    private Object[] items = new Object[10];
    private int count;

    public void add(Object item) {
        items[count++] = item;
    }

    public Object get(int index) {
        return items[index];
    }
}
```

由于 Object 是所有类的父类，所以把 int 改为 Object 就可以解决问题，从而我们可以有如下的代码。

```java
public class Main {
    public static void main(String[] args) {
        var list = new List();
        list.add(1);
        list.add("2");
        
        int number = (int)list.get(0);
    }
}
```

上面的 `list.add(1);` 中的 1 是一个原始值，不是一个 Object，但在编译时，Java 编译器会把这段代码转化为类似的代码：`list.add(Integer.valueOF(1));`。

但这个方法不好，原因有很多 。

比如上面在取出第一个元素时，就需要进行转换。

并且，有时我们可能并不清楚取出的元素会是什么类型的。

## Generic Classes

我们新建一个类叫做 GenericList。

我们希望这个类是 generic 的，从而我们可以重复使用它来存储不同类型的对象。

因此就在类名的后面我们加上一对尖括号，并在括号内写上大写字母 T，意味着 Type。当然其他字母也行，例如用 E 表示 element，但习惯用 T。

这个 T 是这个类的类型参数，因此就像是方法可以有参数，类也可以有参数，这里的 T 表示要存储在此列表中的对象。

当我们创建这个类的实例时，我们必须给这个参数指定一个值。

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

## Generics and Primitive Types

在创建 generic 类型的实例时，我们只能使用 reference 类型作为 generic 的类型参数，比如 Object 或者 String。对于类似 int，short，boolean 等的 primitive 类型，则不能使用。

如果想在一个 generic list 中存储这些 primitive 类型的值，我们必须使用他们的 wrapper class。

在 Java 中，每一个 primitive type 都有一个对应的 wrapper class，例如：

- int $\to$ Integer
- float $\to$ Float
- boolean $\to$ Boolean

我们可以有如下代码：

```java
public class Main {
    public static void main(String[] args) {
        GenericList<Integer> numbers = new GenericList<>();
        numbers.add(1);
        int number = numbers.get(0);
    }
}
```

在上述代码中，在创建 numbers 时，为了简洁，不用重复声明 Integer。

在 add 方法中，我们可以直接传入一个 primitive value 比如说 1，Java编译器会自动地把这个值包装进一个 Integer 实例内。这个过程叫做 Boxing（封装）。

相应的，在 get 方法中，Java 编译器会自动 Unboxing（解封装）。

## Constraints

有时想加一个约束或者是一个对类型参数的限制。例如，假设我们只想在这个 list 中存储数字。可能这个 list 将支持一些只对数字有意义的操作。

为了实现这个目的，我们在 T 后面输入 `extends Number`，这样，T 就只能是 Number 类或者它的子类。有了这个约束条件，如果我们想创建一个新的 String list，就会出现编译错误。

这个约束不一定是一个类，也可以是一个接口（interface）。例如，在 Java 中我们有一个流行的接口叫做 Comparable，我们用这个接口来实现那些可以互相比较的类。

可以使用 `&` 来并联多个类或者接口。

```java
public class GenericList<T extends Comparable & Cloneable> {
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

## Type Erasure

这一节要讲 generic 是如何工作的。

将上一节的代码中的 extends 部分的代码删除，即取出 constraints，重新编译代码，即在上方工具栏中的 Build 菜单中点击 build project，然后在左侧的项目窗口中单击选中 GenericList 类，然后在上方工具栏中的 View 菜单中选择 Show Bytecode，然后就可以看到编译的结果。

具体内容看视频 15 - 20 分钟。

## The Comparable Interface

这个接口可用于排序算法。

Comparable 是一个 generic 接口。

这个接口只有一个方法叫做 compareTo

应用实例如下：

```java
public class User implements Comparable<User>{
    private int points;

    public User(int points) {
        this.points = points;
    }

    @Override
    public int compareTo(User o) {
        // this < o -> -1
        // this == o -> 0
        // this > o -> 1
        return points - o.points;
    }
}
```

Main 函数中的代码如下：

```java
public class Main {
    public static void main(String[] args) {
        var user1 = new User(10);
        var user2 = new User(20);
        if (user1.compareTo(user2) < 0)
            System.out.println("user1 < user2");
    }
}
```

## Generic Methods

前几节中的 GenericList 类中的 add 和 get 方法都属于 generic 方法，因为它们都会根据类型 T 的变化发生变化。

我们也可以在一个非 generic 的类中声明 generic 方法。

添加一个新类 Utils：

```java
public class Utils {
    public static <T extends Comparable<T>> T max(T first, T second) {
        return (first.compareTo(second) > 0) ? first : second;
    }
}
```

Utils 中的方法一般都是 static 的，这样在调用这些方法的时候就不需要创建 Utils 的实例了。

因为上例中我们想比较两个元素的大小，所以我们给 T 增加了 constraint，让它是 comparable 的。

对应的 main 方法如下：

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utils.max(1, 2));
        System.out.println(Utils.max(new User(5), new User(3)));
    }
}
```

上例中的第二个输出是一段 hash 码，要想得到更明确的结果，应该重写 User 方法的 toString 方法。

## Multiple Type Parameters

有时我们想声明多个类型的参数。

假设我们想实现一个方法，来打印键和值，键和值可能是不同的类型。

```java
public class Utils {
    public static <T extends Comparable<T>> T max(T first, T second) {
        return (first.compareTo(second) > 0) ? first : second;
    }
    
    public static <K, V> void print(K key, V value) {
        System.out.println(key + "=" + value);
    }
}
```

我们也可以声明一个具有多给类型的参数的类：

```java
public class KeyValuePair<K, V> {
    private K key;
    private V value;

    public KeyValuePair(K key, V value) {
        this.key = key;
        this.value = value;
    }
}
```

## Generic Classes and Inheritance

这一节要介绍一个与 generic 有关的常见的问题。

添加一个新类 Instructor 如下：

```java
public class Instructor extends User {
    public Instructor(int points) {
        super(points);
    }
}
```

然后在 main 类中，我们可以实现以下两种操作：

```java
public class Main {
    public static void main(String[] args) {
        User user = new User(1);
        User instructor = new Instructor(3);
    }
}
```

这是因为 Instructor 继承了 User。

我们回到 Utils 类中，添加一个新的方法 printUser，如下：

```java
public class Utils {
    public static <T extends Comparable<T>> T max(T first, T second) {
        return (first.compareTo(second) > 0) ? first : second;
    }

    public static <K, V> void print(K key, V value) {
        System.out.println(key + "=" + value);
    }

    public static void printUser(User user) {
        System.out.println(user);
    }
}
```

此时 main 类中可以实现以下操作：

```java
public class Main {
    public static void main(String[] args) {
        User user = new User(1);
        User instructor = new Instructor(3);
        Utils.printUser(new Instructor(10));
    }
}
```

如果我们这里要的是一个 user 的 list 呢？

此时我们应该回到 Utils 类中，进行如下修改：

```java
public class Utils {
    public static <T extends Comparable<T>> T max(T first, T second) {
        return (first.compareTo(second) > 0) ? first : second;
    }

    public static <K, V> void print(K key, V value) {
        System.out.println(key + "=" + value);
    }

    public static void printUser(User user) {
        System.out.println(user);
    }

    public static void printUsers(GenericList<User> users) {
    }
}
```

其中的具体实现不重要。

然后我们回到 main 函数，希望进行以下两种操作：

```java
public class Main {
    public static void main(String[] args) {
        var users = new GenericList<User>();
        Utils.printUsers(users);
      
        var insturctors = new GenericList<Instructor>();
        Utils.printUsers(insturctors);
    }
}
```

此时发现第一个操作没问题，但第二个操作编译器会提示错误。

因为 `GenericList<Instructor>` 不是 `GenericList<User>` 的子类。

原因见 Type Erasure 一节。

解决方法见下一节。

## Wildcards

通配符就是一个 `?`，它表示未知的类型。

要解决上一节的问题，只需要回到 Utils 类中，把 `printUsers(GenericList<User> users)` 改为 `printUsers(GenericList<?> users)` 就可以了。

在这时我们无论传入一个什么类型的 list 都可以。

要想让它只接收 User 类，只需加一个限制，改为 `printUsers(GenericList<? extends User> users)` 即可。

extends 关键词会让编译器把 `?` 所代表的匿名类当作是 User 的子类。

但 extends 关键词只能允许我们从 list 中取出一个对象。

如果我们想用 add 方法在这个 list 中添加一个对象，要用到 super 关键词。

super 关键词可以让编译器把 `?` 所代表的匿名类当作是 User 的父类。

此处，User 的父类就是 Object 类。

```java
public class Utils {
    public static <T extends Comparable<T>> T max(T first, T second) {
        return (first.compareTo(second) > 0) ? first : second;
    }

    public static <K, V> void print(K key, V value) {
        System.out.println(key + "=" + value);
    }

    public static void printUser(User user) {
        System.out.println(user);
    }

    public static void printUsers(GenericList<? super User> users) {
//        User x = users.get(0);
        users.add(new User(1));
        users.add(new Instructor(1));
    }
}
```

 综上，如果想要从 list 中读取数据，就要用 extends 关键词。

如果想要往 list 中写入数据，就要用 supper 关键词。
