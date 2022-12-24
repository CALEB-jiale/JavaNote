# Lambda Expressions and Functional Interfaces

## Functional Interfaces

A functional interface is a interface with a single abstract method.

之前提过的 Comparable 接口就只有一个 compareTo 方法，所以它就是一个 functional interface。

Comparator 接口虽然有很多方法，但只有 compare 方法没有默认实现，所以它也是一个 functional interface。

创建包 lambdas，类 LambdasDemo 以及接口 Printer。

```java
public interface Printer {
    void print(String message);
}
```

作为一个接口，Printer 可能有不同的实现。

```java
public class ConsolePrinter implements Printer{
    @Override
    public void print(String message) {
        System.out.println(message);
    }
}
```

LambdasDemo 的代码如下：

```java
public class LambdasDemo {
    public static void show() {
        greet(new ConsolePrinter());
    }

    public static void greet(Printer printer) {
        printer.print("Hello World");
    }
}
```

运行后可以看到“Hello World”信息。

有时我们不想显式地创造一个类去实现某个接口，因为这需要写很多代码，而这个类我们只想用一次。

下一节将介绍匿名内部类来解决上述问题。

## Anonymous Inner Classes

我们可以将 LambdasDemo 的代码更改如下：

```java
public class LambdasDemo {
    public static void show() {
        greet(new Printer() {
            @Override
            public void print(String message) {
                System.out.println(message);
            }
        });
    }

    public static void greet(Printer printer) {
        printer.print("Hello World");
    }
}
```

在 new 后我们不再创建一个 ConsolePrinter 实例，而是输入想要利用的接口的名称，然后按 enter 键，编译器就会为我们创建一个匿名内部类。

匿名是指这个类没有名字，它只是一个实现。

内部是指这个类包含在一个方法里。

匿名内部类允许我们通过使用更少的代码来达到同样的效果。

但 Java 中还有另一种方法更简洁，那就是 lambda 表达式。

## Lambda Expressions

lambda 表达式在代码中的形式就是 `->`。

利用 lambda 表达式，上一节的代码可以按如下方式重写：

```java
public class LambdasDemo {
    public static void show() {
        greet((String message) -> {
            System.out.println(message);
        });
    }

    public static void greet(Printer printer) {
        printer.print("Hello World");
    }
}
```

上述代码还可以更简洁。

首先我们可以删除 message 的类型名 String。编译器会知道 message 是 String 类型的，因为 lambda 表达式所表达的内容应该是 Printer 接口的一个匿名实现，而 Printer 接口中的 print 方法是以 String 类型的参数为输入的，所以 message 也应该是 String 类型的。

如果只有一个参数要传入，我们还可以去掉对应的括号。我们在无参数或者有多个参数的情况下需要使用括号，多个参数之间要用逗号隔开。

如果函数体只有一行代码，我们还可以去掉对应的花括号。在花括号边上按下 option + enter，选择 Replace with expression lambda 即可以去掉花括号。

```java
public class LambdasDemo {
    public static void show() {
        greet(message -> System.out.println(message));
    }

    public static void greet(Printer printer) {
        printer.print("Hello World");
    }
}
```

我们还可以把 lambda 表达式存储在一个变量中：

```java
public class LambdasDemo {
    public static void show() {
        Printer printer = message -> System.out.println(message);
        greet(printer);
    }

    public static void greet(Printer printer) {
        printer.print("Hello World");
    }
}
```

## Variable Capture

在 lambda 表达式中，我们使用了我们在表达式内声明的变量 message，但我们也可以使用在封闭方法中声明的局部变量：

```java
public class LambdasDemo {
    public static void show() {
        String prefix = "-";
        Printer printer = message -> System.out.println(prefix + message);
        greet(printer);
    }

    public static void greet(Printer printer) {
        printer.print("Hello World");
    }
}
```

上面代码中 prefix 就是我们在封闭方法 show 中声明的变量。

我们也可以访问封闭类中的静态方法或静态变量：

```java
public class LambdasDemo {
    public static String prefix = "-";

    public static void show() {
        Printer printer = message -> System.out.println(prefix + message);
        greet(printer);
    }

    public static void greet(Printer printer) {
        printer.print("Hello World");
    }
}
```

上面代码中 prefix 是在封闭类 LambdasDemo 中声明的静态变量。

我们还可以访问实例字段，例如可以把上面代码中 show 方法和变量 prefix 的 static 声明全部去除，代码仍是可以运行的。

在 lambda 表达式中，还可以使用 this 关键字来引用表达式所在的方法所在的类。如上面的代码中，lambda 表达式中如果有 this 关键字，它就表示的是类 LambdasDemo 在当时语境下创造的对象。

但在匿名内部类中，this 引用的是这个匿名内部类。

这两种情况下的另一个不同点是，匿名内部类可以有状态，它可以用一些字段来存储一些数据。而在 lambda 表达式中，我们不能有字段。

## Method References

有时我们在 lambda 表达式中所做的一切不过就是把参数传入一个已经存在的方法中去，就像上面我们就是把 message 传入到 sout 中去。在这种情况下，直接使用方法引用可能更简单。

```java
public class LambdasDemo {
    public static void show() {
        greet(message -> System.out.println(message));
        greet(System.out::println);
    }

    public static void greet(Printer printer) {
        printer.print("Hello World");
    }
}
```

方法引用的格式如上所示，两个 greet 得到的结果相同。我们可以在第一个 greet 后面的 println 方法上按 option + enter，然后选择 Replace lambda with method reference 即可。

通过使用方法引用，我们可以引用一个类中的静态或实例方法，以及构造函数。

引用静态方法：

```java
public class LambdasDemo {
    public static void print(String message) {}

    public static void show() {
        greet(message -> print(message));
        greet(LambdasDemo::print);
    }

    public static void greet(Printer printer) {
        printer.print("Hello World");
    }
}
```

引用实例方法：

```java
public class LambdasDemo {
    public void print(String message) {}

    public static void show() {
        var demo = new LambdasDemo();
        greet(message -> demo.print(message));
        greet(demo::print);
    }

    public static void greet(Printer printer) {
        printer.print("Hello World");
    }
}
```

引用构造函数：

```java
public class LambdasDemo {
    public LambdasDemo(String message) {
    }

    public static void show() {
        greet(message -> new LambdasDemo(message));
        greet(LambdasDemo::new);
    }

    public static void greet(Printer printer) {
        printer.print("Hello World");
    }
}
```

## Built-in Functional Interfaces

Java 提供了许多可以用来执行普通任务的预定义功能接口，这些接口被定义在了 java.util.function 包中。

功能接口大致可以被分为四类：

1. Consumer：这类接口需要且只需要一个参数，并且没有返回结果。它消耗了一个参数，因此被称为消费者。我们之前的 Printer 接口就是个例子。
2. Supplier：这类接口与消费者相反，它不接受输入，但会返回一个值。
3. Function：函数接口可以将一个值映射到另一个值。
4. Predicate：这类接口需要一个输入，并检查这个输入是否满足某些要求。我们可以用它来过滤数据。

## The Consumer Interface

Consumer 接口需要一个输入，但没有输出。

它有两个方法 accept 和 andThen，其中 andThen 有默认实现。

这个接口有一些变体，如：

- BiConsumer：它需要两个输入
- IntConsumer：它只接受 int 类型的输入，这可以免去自动封装的过程。类似地，我们还有 LongConsumer 和 DoubleConsumer

接下来介绍如何使用 Consumer 接口。

```java
public class LambdasDemo {
    public static void show() {
        List<Integer> list = List.of(1, 2, 3);

        // Imperative Programming (for, if/else, switch/case)
        for (var item : list)
            System.out.println(item);

        // Declarative Programming
        list.forEach(item -> System.out.println(item));
    }
}
```

上述代码中，利用 for 循环或利用 forEach 方法都可以实现对列表的遍历。

其中 forEach 方法期待的是一个 Consumer 对象作为输入。

由于 Consumer 接口是一个功能性接口，所以我们可以用 lambda 表达式来表示。

凡是使用 for，if/else，switch/case 语句的编程都属于命令式编程。 

不用命令来指示需要做的事情（how something should be done），而只是指明需要做的事是什么（what should be done），这种编程属于声明式编程。

## Chaining Consumers

Consumer 接口有一个默认实现的方法叫做 andThen，通过这个方法，我们可以把 consumers 链接起来，也就是说我们可以按顺序来执行许多操作。

下面的代码中我们会声明一个 Consumer<String> 类型的变量，我们将它命名为 print。它的等号右边我们设置为一个 lambda 表达式（它可以是其它已经存在的方法，或者是匿名内部类），这个 lambda 表达式应该匹配的是 Consumer 中的 accept 方法。

```java
public class LambdasDemo {
    public static void show() {
        List<String> list = List.of("a", "b", "c");
        Consumer<String> print = item -> System.out.println(item);
        Consumer<String> printUpperCase = item -> System.out.println(item.toUpperCase());

        list.forEach(print.andThen(printUpperCase));
    }
}
```

上面的代码我们运用 andThen 方法，把 print 和 printUpperCase 链接起来，得到的输出结果是：

> a
> A
> b
> B
> c
> C

将光标放到 andThen 上面，然后在上方工具栏打开 Navigate 菜单，选择 Declaration，快捷键为 command + B，然后这就可以看到这个方法的源码。

```java
default Consumer<T> andThen(Consumer<? super T> after) {
    Objects.requireNonNull(after);
    return (T t) -> { accept(t); after.accept(t); };
}
```

这个方法接受一个消费者 after 作为输入，首先它确认这个 after 不是 null，然后依次调用当前消费者的 accept 方法和 after 的 accept 方法。每次调用这个方法，都会得到一个新的消费者，这意味着我们可以在此调用 andThen 方法。

```java
public class LambdasDemo {
    public static void show() {
        List<String> list = List.of("a", "b", "c");
        Consumer<String> print = item -> System.out.println(item);
        Consumer<String> printUpperCase = item -> System.out.println(item.toUpperCase());

        list.forEach(print.andThen(printUpperCase).andThen(print));
    }
}
```

## The Supplier Interface

Supplier 接口与 Consumer 接口相反，它不消耗参数，而是提供一个参数。

Supplier 接口只有一个抽象方法 get。

```java
public class LambdasDemo {
    public static void show() {
        Supplier<Double> getRandom = () -> {return Math.random();};
    }
}
```

由于上面的代码只是返回了一个值，所以可以去掉花括号和 return 关键词：

```java
public class LambdasDemo {
    public static void show() {
        Supplier<Double> getRandom = () -> Math.random();
        var random = getRandom.get();
        System.out.println(random);
    }
}
```

上述代码需要注意的是，Math.random 函数在我们明确地调用它之前并没有被执行，这被叫做 lazy evaluation。直到我们明确提出要求之前，值不会被产出。

类似 Consumer 接口，Supplier 接口也有一些变体比如说 DoubleSupplier，IntSupplier，LongSupplier 以及 BooleanSupplier。通过使用这些 primitive 专业化的接口，可以免去自动封装和自动解封装的消耗。

## The Function Interface

顾名思义，这类接口表示的是一个方程，或者是一个带函数的操作，然后返回一个值。因此接口定义时需要两个 generic 类型的参数，一个用来定义输入参数的类型，另一个表示输出参数的类型。

这个接口中有四个方法，其中只有 apply 方法没有实现。

它的变体有 BiFunction，接受两个参数作为输入，返回一个参数作为输出。

它的变体也有 primitive 专业化的接口，这些接口可以分为三类：

1. 输入参数有特定的类型，输出参数无特定的类型。比如说 IntFunction 会接受一个 int 类型的参数作为输入，但输出的参数类型在声明接口时定义。同理还有 LongFunction 和 DoubleFunction。
2. 输出参数有特定的类型，输入参数无特定的类型。比如说 ToIntFunction 会输出一个 int 类型的参数，但输入的类型在声明接口时定义。
3. 输入与输出都有特定的类型。如 IntToLongFuction 会接受一个 int 类型的输入，返回一个 long 类型的输出。

```java
public class LambdasDemo {
    public static void show() {
        Function<String, Integer> map = str -> str.length();
        var length = map.apply("sky");
        System.out.println(length);
    }
}
```

## Composing Functions

这一节讨论如何通过组合小的函数来得到更复杂有趣的函数。

```java
public class LambdasDemo {
    public static void show() {
        // "key:value"
        // first: "key=value"
        // second: "{key=value}"
        Function<String, String> replaceColon = str -> str.replace(":", "=");
        Function<String, String> addBraces = str -> "{" + str + "}";

        // Declarative Programming
        var result = replaceColon
                     .andThen(addBraces)
                     .apply("key:value");
        System.out.println(result);

        result = addBraces.compose(replaceColon).apply("key:value");
        System.out.println(result);
    }
}
```

当进行 Declarative Programming 的时候，代码的拍不应该按上面的代码所示，这样可以更清晰。

之前说过，把光标放到想要了解的函数名上，然后在上方工具栏中的 Navigate 菜单中找到 Declaration，快捷键 command + B，就可以查看函数的源码。

## The Predicate Interface

Predicate 接口用于过滤数据。

这个接口唯一的抽象方法是 test，给他一个参数，可以返回一个布尔值。

它的专业化有 BiPredicate，接受两个参数，返回一个布尔值。

它的 primitive 专业化有 IntPredicate，LongPredicate，DoublePredicate 等。

```java
public class LambdasDemo {
    public static void show() {
        Predicate<String> isLongerThan5 = str -> str.length() > 5;
        var result = isLongerThan5.test("sky");
        System.out.println(result);
    }
}
```

## Combining Predicates

与 Function 接口类似，Predicate 接口也可以通过组合实现更复杂更有趣的操作。

```java
public class LambdasDemo {
    public static void show() {
        Predicate<String> hasLeftBrace = str -> str.startsWith("{");
        Predicate<String> hasRightBrace = str -> str.endsWith("}");
        Predicate<String> hasLeftAndRightBraces = hasLeftBrace.and(hasRightBrace);

        var result = hasLeftAndRightBraces.test("{key:value}");
        System.out.println(result);

        result = hasLeftBrace.or(hasRightBrace).test("{key:value}");
        System.out.println(result);

        result = hasLeftBrace.negate().test("{key:value}");
        System.out.println(result);
    }
}
```

与逻辑操作的与或非类似，Predicate 接口也有 and，or 和 negate 方法分别表示与或非。

## The BinaryOperator Interface

之前说过，java.util.function 包中的接口都可以分为 Consumer，Supplier，Function 和 Predicate 四类接口。

但还有一种特殊的接口叫做 BinaryOperator。

这个接口接收两个 T 类型的参数作为输入，然后返回一个 T 类型的输出。

它也有 primitive 专业化，比如 IntBinaryOperator，两个输入和一个输出都是 int 类型的。

```java
public class LambdasDemo {
    public static void show() {
        BinaryOperator<Integer> add = (a, b) -> a + b;
        var result = add.apply(1, 2);
        System.out.println(result);
    }
}
```

上面的代码使用 IntBinaryOperator 会更高效，尤其是在处理较大数据的时候。

```java
public class LambdasDemo {
    public static void show() {
        BinaryOperator<Integer> add = (a, b) -> a + b;
        Function<Integer, Integer> square = a -> a * a;

        var result = add.andThen(square).apply(1, 2);
        System.out.println(result);
    }
}
```

## The UniaryOperator Interface

这个接口继承了 Function<T, T> 接口，所以它会接受一个 T 类型的输入，返回一个 T 类型的输出。

```java
public class LambdasDemo {
    public static void show() {
        UnaryOperator<Integer> square = n -> n * n;
        UnaryOperator<Integer> increment = n -> n + 1;

        var result = increment.andThen(square).apply(1);
        System.out.println(result);
    }
}
```

## Summary

- Lambda expressions
- Functional interfaces
- Consumers
- Suppliers
- Functions
- Predicates
- Composition