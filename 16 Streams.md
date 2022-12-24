# Streams

Stream 允许我们以声明式编程的方式来处理数据集。

## Imperative VS Functional Programming

假设我们有一个电影列表，我们想知道有多少个电影超过十个赞。

先有 Movie 类：

```java
public class Movie {
    private String title;

    private int likes;

    public Movie(String title, int likes) {
        this.title = title;
        this.likes = likes;
    }

    public int getLikes() {
        return likes;
    }
}
```

然后进行如下操作：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 15),
                new Movie("c", 20)
                );

        // Imperative Programming
        int count = 0;
        for (var movie : movies)
            if (movie.getLikes() > 10)
                count++;
    }
}
```



这种编程方式叫做命令式编程。我们有不同的编程方式：

1. Declarative：what should be done
2. Imperative：how something should be done
3. Functional
4. Object-oriented
5. Event-driven
6. ……

在一个应用中，我们可能有多种模式。

我们可能为了实现某种特性，使用 event-driven 模式。但我们也可能在程序的其他部分使用 functional 模式。

在 imperative 编程中，我们有特定的陈述来定义 how something should be done。

SQL 是一种 declarative 编程的语句，它的代码都是在说什么需要被做，而不涉及具体该怎么做。

Stream 的作用是：process a collection of data in a declarative way。更准确地说是一种 function 的方式。

Functional 编程是 declarative 编程的一种特殊类型。

Java 中的每个 collection 都有一个叫做 stream 的方法，这个方法会返回一个对象流（steam of objects）。

Stream 是对象的序列（a sequence of objects），但它不像是一个集合，它不存储数据，它只是从集合中获取数据的一种方法。

比如说一个水箱，实际的水在水箱里面，但我们有很多的管道来在水箱外面取水。所以集合就像是一个水箱，这是我们春初数据的地方，而 stream 就像是一个管子，然后我们就可以一个接着一个地接上管子。我们可以建立一个管道来传输数据，把它从集合中取出来。

每个 stream 对象有很多有用的方法，其中一个是 filter，用这个我们可以过滤数据。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 15),
                new Movie("c", 20)
                );

        // Declarative (Functional) Programming
        var count = movies.stream()
                .filter(movie -> movie.getLikes() > 10)
                .count();
    }
}
```

上面的代码中，filter 方法要求传入的数据类型是 `Predicate<? super Movie>`，上一章我们提过 Predicate 接口接受一个对象，返回一个布尔值，因此通过 Predicate 我们可以检测数据是否满足要求。

这是 declarative 编程，或者更准确地说是 functional 编程。因为我们用了一堆函数，比如说 filter 和 count，把它们组合起来表达一个复杂的逻辑。在这些代码中，我们没有具体的命令来告诉我们如何去做某一件事，只是单纯地表达什么需要被做。这就是命令式编程与声明式编程的区别。

通过使用 stream，我们可以通过 functional 的方式来处理一个集合的数据，这使得我们的代码更清晰，更易于阅读。

## Creating a Stream

我们有几种不同的方法来创建 stram：

1. From collections
2. From arrays
3. From an arbitrary number of objects
4. Infinite/finite streams

## From collections

Java 中的 Collection 接口有一个方法叫做 stream，因此每一个实现了 Collection 接口的类都可以使用这个方法。

### From arrays

数组（Array）因为不是 Collecion 的实现，所以它没有 stream 方法。

但是我们有在 java.util 中声明的 Arrays 类，它有 stream 方法。我们可以在这个方法中传入一个数组，然后就可以得到这个数组的 stream。

每一个 stream 都有一个方法叫做 forEach，它要求传入一个 Consumer 接口的对象。

```java
public class CreatingStreamsDemo {
    public static void show(){
        int [] numbers = {1, 2, 3};
        Arrays.stream(numbers)
                .forEach(num -> System.out.println(num));
    }
}
```

### From an arbitrary number of objects

Stream 中有一个静态的构造方法叫做 of，我们可以在这个方法中传入任意数量的参数，这个方法将返回对应的 stream。

### Infinite/finite streams

我们有两种方法可以生成 infinite stream。

```java
public class CreatingStreamsDemo {
    public static void show() {
        var stream = Stream.generate(() -> Math.random());
    }
}
```

上面的代码将会产生一个随机数的无尽流，这不是我们用集合能做到的，因为如果你想在一个集合中储存无限数量的对象，我们会耗尽内存。但是通过 stream，每次我们想从这个 stream 中读取一个数字的时候，random 函数就会被执行，而不会提前执行，这也是我们之前提到过的 lazy evaluation。

执行上面的代码，什么都不会发生，没有生成数字，也没有消耗任何东西，内存中也没有无尽的数字。

但如果我们此时调用 forEach 方法，它将会不断地从 stream 中请求一个新的数字，并对它进行对应的操作。

为了避免这种情况，我们可以使用 limit 方法：

```java
public class CreatingStreamsDemo {
    public static void show() {
        var stream = Stream.generate(() -> Math.random());
        stream.limit(3).forEach(num -> System.out.println(num));
    }
}
```

我们还可以用迭代的方法来生成 infinite stream 或 finite stream：

```java
public class CreatingStreamsDemo {
    public static void show() {
        Stream.iterate(1, n -> n + 1)
                .limit(10)
                .forEach(n -> System.out.println(n));
    }
}
```

iterate 方法有两个参数，首先要传递一个种子或者一个初始值，然后传递一个 UnaryOperator 用来改变前面的那个值，也就是一个函数。

## Mapping Elements

我们经常需要转换字符串中的值，为此我们需要使用 map 或 flatMap 方法。

### map

假设我们有一个电影列表，而我们只对这些电影的名字感兴趣。因此我们想要的是一个 String 类型的 stream，而不是 Movie 类型的。

我们在 Movie 类中为 title 添加 Getter 后，编辑如下代码：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 15),
                new Movie("c", 20)
                );

        movies.stream()
                .map(movie -> movie.getTitle())
                .forEach(name -> System.out.println(name));
    }
}
```

我们使用 map 方法来把每个 Movie 对象转换成另一个类型。map 方法要输入的参数类型是 `Function<? super Movie, ?>`。

stream 的强大之处就在于可以构建 processing pipelines。

我们先用一条管道从数据源中取出数据，然后用另一条管道提取出每个 Movie 对象的标题，然后我们可以继续用其他管道对之前得到的数据进行处理。

### flatMap

有如下代码：

```java
public class StreamDemo {
    public static void show() {
        var stream = Stream.of(List.of(1, 2, 3), List.of(4, 5, 6));
        // Stream<List<x>> -> Stream<x>
        stream.flatMap(list -> list.stream())
                .forEach(n -> System.out.println(n));
    }
}
```

上面代码中的 stream 中的每个对象，都是一个 Integer 的 List。

如果没有 flatMap 方法，直接利用 forEach 方法把 stream 中的对象打印出来，我们会得到：

> [1, 2, 3]
> [4, 5, 6]

但有时我们想跨过 List 直接对每个单独的数字进行操作，这就需要 flatMap 方法。

flatMap 方法需要传入一个 `Function<? super List<Integer>, ? extends Stream<?>>` 类型的函数，它以一个整数列表为输入，并返回一个 stream。

## Filtering Elements

之前谈过如何对数据进行筛选，这里作出进一步解释。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 15),
                new Movie("c", 20)
                );

        // Declarative (Functional) Programming
        var count = movies.stream()
                .filter(movie -> movie.getLikes() > 10)
                .count();
    }
}
```

还是之前的代码。

我们要看的是 filter 方法的返回类型，他会返回 `Stream<Movie>`，这也是为何我们可以在 filter 方法后继续调用其它属于 stream 的方法。

而 forEach 方法没有返回，所以它是终端操作。

因此，stream 的方法分为两类，有些是中间操作（Intermediate），有些是终端操作（Terminal）：

- Intermediate：map，filter
- Terminal：forEach

## Slicing a Stream

我们有四种方法来获取流的片段：

- limit(n) 方法可以限制流中的元素数量
- skip(n) 方法可以跳过流中的一些元素
- takeWhile(predicate) 方法可以筛选出满足条件的元素
- dropWhile(predicate) 方法可以筛选掉满足条件的元素

以下代码是 limit 和 skip 的示例：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 15),
                new Movie("c", 20)
                );

        movies.stream()
                .limit(2)
                .forEach(m -> System.out.println(m.getTitle()));

        movies.stream()
                .skip(2)
                .forEach(m -> System.out.println(m.getTitle()));

        // 1000 movies
        // 10 movies per page
        // 3rd page
        // skip(20) = skip((page - 1) * pageSize)
        // limit(10) = limit(pageSize)
    }
}
```

通过结合 limit 和 skip，可以实现页面展示的功能，如注视中所示。

然后看后两种方法：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 30),
                new Movie("c", 20)
                );

        movies.stream()
                .takeWhile(movie -> movie.getLikes() < 30)
                .forEach(m -> System.out.println(m.getTitle()));

        System.out.println();

        movies.stream()
                .dropWhile(movie -> movie.getLikes() < 30)
                .forEach(m -> System.out.println(m.getTitle()));
    }
}
```

takeWhile 方法和 filter 方法的区别在于，filter 方法会迭代整个数据源，来找到符合标准的对象，但 takeWhile 方法会在返回 false 后就停止，哪怕后面还有数据可能满足要求，它也不会被筛选出来。

dropWhile 与 takeWhile 相反，他们输出的结果如下：

> a
>
> b
> c

## Sorting Streams

一般情况下，流中的元素顺序与数据源中的元素顺序相同，但我们可以用 sort 方法来改变这个顺序。

sort 方法被重载了，一种不需要参数，但需要被排序的对象实现了 Comparable 接口，另一种需要一个 Comparator 对象。

通过 Comparable 接口中的 compareTo 方法，我们可以决定排序的方式。但这个方法有一个局限，就是我们有时可能想换一种参数去排序。这就需要用 Comparator 接口。

Comparator 接口只有一个 compare 方法没被实现，所以它是一个功能性接口，也就是说它可以用 lambda 表达式来代替。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("b", 10),
                new Movie("a", 20),
                new Movie("c", 30)
                );

        movies.stream()
                .sorted((a, b) -> a.getTitle().compareTo(b.getTitle()))
                .forEach(movie -> System.out.println(movie.getTitle()));
    }
}
```

Comparator 接口中还有一个叫做 comparing 的静态方法，通过这个方法我们可以轻松决定要进行比较的属性。因此上述代码可以用以下代码等价替换：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("b", 10),
                new Movie("a", 20),
                new Movie("c", 30)
                );

        movies.stream()
                .sorted(Comparator.comparing(movie -> movie.getTitle()))
                .forEach(movie -> System.out.println(movie.getTitle()));
    }
}
```

但是还可以利用 option + enter 进一步化简：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("b", 10),
                new Movie("a", 20),
                new Movie("c", 30)
                );

        movies.stream()
                .sorted(Comparator.comparing(Movie::getTitle))
                .forEach(movie -> System.out.println(movie.getTitle()));
    }
}
```

comparing 方法会返回一个 Comparator 接口，因此在调用 comparing 方法后，我们可以在 Comparator 接口后调用另一个方法，通过 reversed 方法可以把前面的排序翻转。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("b", 10),
                new Movie("a", 20),
                new Movie("c", 30)
                );

        movies.stream()
                .sorted(Comparator.comparing(Movie::getTitle).reversed())
                .forEach(movie -> System.out.println(movie.getTitle()));
    }
}
```

## Getting Distinct Elements

有时我们想得到字符串中唯一的值。

现在假设之前的 likes 参数成了电影的售价，而我们现在想打印电影的售价表，我们有两个 10 元的电影，但只想展示一次 10 这个数字。

这是就要用到 distinct 方法：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("b", 10),
                new Movie("b", 10),
                new Movie("a", 20),
                new Movie("c", 30)
                );

        movies.stream()
                .map(Movie::getLikes)
                .distinct()
                .forEach(System.out::println);
    } 
}
```

## Peeking Elements

在处理复杂的问题时，我们可能会遇到错误，为了解决这些问题，我们可以使用 peek 方法。

我们有如下代码：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 20),
                new Movie("c", 30)
                );

        movies.stream()
                .filter(movie -> movie.getLikes() > 10)
                .map(Movie::getTitle)
                .forEach(System.out::println);
    }
}
```

现在假设其中某一个操作出现了异常，我们就可以用 peek 方法来获取每个操作的输出。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 20),
                new Movie("c", 30)
                );

        movies.stream()
                .filter(movie -> movie.getLikes() > 10)
                .peek(movie -> System.out.println("filtered: " + movie.getTitle()))
                .map(Movie::getTitle)
                .peek(title -> System.out.println("mapped: " + title))
                .forEach(System.out::println);
    }
}
```

peek 方法和 forEach 方法的不同就在于 peek 方法是中间操作，而 forEach 方法是终端操作。

## Simple Reducers

我们讨论了所有的中间操作：

- map / flatMap
- filter
- limit / skip
- sorted
- distinct
- peek

通过这些操作我们可以定制自己的管道。

接下来我们会看一组操作，它们被称为 Reducers：

- count
- anyMatch(predicate)
- allMatch(predicate)
- noneMatch(predicate)
- findFirst
- findAny
- max(comparator)
- min(comparator)

它们可以把一个对象流减成一个单一的对象，而这个对象就是我们想得到的结果。

比如说我们可以使用 count 方法来计算流中元素的数量，因此 count 方法就把一个流减成了流中的元素数量。

我们还有一组 Matcher，通过他们我们可以查看流中的 any 或 all 或 none 元素满足某些条件。

findFirst 和 findAny 顾名思义，可以返回流中的某个元素。

max 和 min 会基于 comparator 选择出相应的元素。

上述所有操作都是终端操作。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 20),
                new Movie("c", 30)
                );

        var result = movies.stream()
                .anyMatch(movie -> movie.getLikes() > 20);
        System.out.println(result);

        var result1 = movies.stream()
                .findFirst()
                .get();
        System.out.println(result1.getTitle());

        var result2 = movies.stream()
                .max(Comparator.comparing(Movie::getLikes))
                .get();
        System.out.println(result2.getTitle());
    }
}
```

findFirst 方法会返回一个 Optional 类型的参数，这个类是用来防止空指针异常的。之后我们还需要调用 get 方法来获取实际上的 Movie 对象。findAny，max 和 min 都是类似情况。

## Reducing a Stream

上一节介绍了简单的 Reducers，但还有一些进阶的。

利用 reduce 方法可以把一个流变成单个值。

这个方法被重载了，我们常用的版本需要我们传入一个 BinaryOperator。

与 findFirst 方法类似，reduce 方法也会返回一个 Optional 类型的参数。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 20),
                new Movie("c", 30)
                );

        // [10, 20, 30]
        // [30, 30]
        // [60]
        
        var sum = movies.stream()
                .map(Movie::getLikes)
                .reduce((a, b) -> a + b);

        System.out.println(sum.orElse(0));
    }
}
```

上面代码中，由于 sum 是 Optional 类型的，所以直接使用 get 方法可能会抛出异常，因此我们需要提供一个默认值，这就需要用到 orElse 方法。如果 sum 是空的，orElse 就会给出默认值，否则就给出 sum 应有的值。

具体的叠加过程如代码中的注释所示。

其中的 `(a, b) -> a + b` 可以通过快捷键 option + enter 变为 `Integer::sum`。

reduce 的另一个版本可以在 BinaryOperator 前添加一个参数作为初始值。如果我们提供初始值 0，就不需要再考虑 sum 为空的情况。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 20),
                new Movie("c", 30)
                );

        var sum = movies.stream()
                .map(Movie::getLikes)
                .reduce(0, Integer::sum);

        System.out.println(sum);
    }
}
```

## Collectors

目前为止，我们都一直在用 forEach 方法来处理流的结果。

但有时我们想把流的结果收集起来，并转换成类似于 List，Set 或 Map 的数据结构，这就要用到 collect 方法。

collect 方法被重载了，常用的版本需要一个 Collector 作为输入。

Collector 是一个接口，它有多种实现。我们有将 Stream 转换为 List，Set 或 Map 等的 Collector。

要创建 Collector，就要用到 Collectors 类，它被声明在 java.util.stream 中。这个类有许多可以创建 Collector 对象的静态方法。

例如 toList 方法可以创建一个把 Stream 转换为 List 的 Collector。

同理还有 toSet 和 toMap。

总结如下：

- toList
- toSet
- toMap(Function Key, Function Value)
- summingInt
- summarizingInt
- joining
- counting
- ……

toMap 需要两个 Function 作为输入，前一个确定键，或一个确定值。

如果我们想要一个以 title 为键，以 likes 为值的 Map，我们就要按代码中所示。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 20),
                new Movie("c", 30)
                );

        var result = movies.stream()
                .filter(movie -> movie.getLikes() > 10)
                .collect(Collectors.toMap(Movie::getTitle, Movie::getLikes));

        System.out.println(result);
    }
}
```

如果我们想以 Movie 本身作为值，可以修改如下：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 20),
                new Movie("c", 30)
                );

        var result = movies.stream()
                .filter(movie -> movie.getLikes() > 10)
                .collect(Collectors.toMap(Movie::getTitle, Function.identity()));

        System.out.println(result);
    }
}
```

其中 `Function.identity()` 返回对象本身，也可以用类似 `m -> m` 的语句来替代，但前者更美观。

我们也有用于收集平均值或总值的 Collector。

summingInt 方法可以用于整数求和，它需要传入一个 ToIntFunction，在本例中它是一个以 Movie 对象为参数的函数，然后返回一个整数。同理还有 summingDouble 等。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 20),
                new Movie("c", 30)
                );

        var result = movies.stream()
                .filter(movie -> movie.getLikes() > 10)
                .collect(Collectors.summingInt(Movie::getLikes));

        System.out.println(result);
    }
}
```

还有一个 Collector 叫做 SummarizingInt，同理还有 SummarizingLong 和 SummarizingDouble：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 20),
                new Movie("c", 30)
                );

        var result = movies.stream()
                .filter(movie -> movie.getLikes() > 10)
                .collect(Collectors.summarizingInt(Movie::getLikes));

        System.out.println(result);
    }
}
```

它的输出如下：

> IntSummaryStatistics{count=2, sum=50, min=20, average=25.000000, max=30}

还有一个 Collector 叫做 joining 用来连接数据：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10),
                new Movie("b", 20),
                new Movie("c", 30)
                );

        var result = movies.stream()
                .filter(movie -> movie.getLikes() > 10)
                .map(Movie::getTitle)
                .collect(Collectors.joining(","));

        System.out.println(result);
    }
}
```

输出如下：

> b,c

它返回的是 String 类型。

## Grouping Elements

有时我们需要对数据进行分组或分类。

比如我们需要根据电影的类型进行分类。

我们先在项目中引入类型的概念。

在 streams 包中新建一个类，起名为 Genre，Kind 选为 Enum（枚举）。在这个枚举中我们有三个分类：COMEDY，ACTION 和 THRILLER。它们都用大写因为它们都是常量。

```java
public enum Genre {
    COMEDY,
    ACTION,
    THRILLER
}
```

然后回到 Movie 类中，为其添加 genre 属性。

```java
public class Movie{
    private String title;
    private int likes;
    private Genre genre;

    public Movie(String title, int likes, Genre genre) {
        this.title = title;
        this.likes = likes;
        this.genre = genre;
    }

    public Movie(String title, int likes) {
        this.title = title;
        this.likes = likes;
    }

    public int getLikes() {
        return likes;
    }

    public String getTitle() {
        return title;
    }

    public Genre getGenre() {
        return genre;
    }
}
```

然后去 StreamDemo 类中作出如下修改：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10, Genre.THRILLER),
                new Movie("b", 20, Genre.ACTION),
                new Movie("c", 30, Genre.ACTION)
                );

        var result = movies.stream()
                .collect(Collectors.groupingBy(Movie::getGenre));

        System.out.println(result);
    }
}
```

我们使用 Collectors 中的 groupingBy 方法来对电影进行分类。

这个方法被重载了，使用最多的版本是要传入一个 classifier，它是一个需要一个对象作为输入的 Function，它将决定我们如何对数据进行分类或分组。

另一个版本是可以传入一个 classifier 和一个 Collector。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10, Genre.THRILLER),
                new Movie("b", 20, Genre.ACTION),
                new Movie("c", 30, Genre.ACTION)
                );

        var result = movies.stream()
                .collect(Collectors.groupingBy(Movie::getGenre, Collectors.counting()));

        System.out.println(result);
    }
}
```

通过上面的代码可以计算每个类的电影的数量。

也可以用 joining 方法把电影的名字串联起来，但因为 joinging 只对 String 类型的 stream 起作用，所以需要先用 mapping 来把 Movie 转换成 String，然后在 mapping 方法的第二个参数中传入 Collectors.joining 将字符串连接。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10, Genre.THRILLER),
                new Movie("b", 20, Genre.ACTION),
                new Movie("c", 30, Genre.ACTION)
                );

        var result = movies.stream()
                .collect(Collectors.groupingBy(
                        Movie::getGenre,
                        Collectors.mapping(
                                Movie::getTitle,
                                Collectors.joining(", "))));

        System.out.println(result);
    }
}
```

输出结果如下：

> {THRILLER=a, ACTION=b, c}

## Partitioning Elements

有时我们需要基于某些条件对数据进行分区。

例如我们可能需要对电影进行分区，一种是点赞数多于 20 的电影，一种是少于 20 的电影。

partitioningBy 方法需要传入一个 Predicate 类型的参数。

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10, Genre.THRILLER),
                new Movie("b", 20, Genre.ACTION),
                new Movie("c", 30, Genre.ACTION)
                );

        var result = movies.stream()
                .collect(Collectors.partitioningBy(movie -> movie.getLikes() > 20));

        System.out.println(result);
    }
}
```

上面的代码中 result 是一个以 Boolean 为键，以 Movie 为值的 Map。

与上一节同理，可以进行以下优化：

```java
public class StreamDemo {
    public static void show() {
        List<Movie> movies = List.of(
                new Movie("a", 10, Genre.THRILLER),
                new Movie("b", 20, Genre.ACTION),
                new Movie("c", 30, Genre.ACTION)
                );

        var result = movies.stream()
                .collect(Collectors.partitioningBy(
                        movie -> movie.getLikes() > 20,
                        Collectors.mapping(
                                Movie::getTitle,
                                Collectors.joining(","))));

        System.out.println(result);
    }
}
```

## Primitive Type Streams

对于 primitive value，Stream 也有进行特殊化，我们有：

- IntStream
- LongStream
- DoubleStream

对于这些 Stream，我们仍可以像之前介绍的那样使用 generate 或 iterate 的方式来生成 Infinite/finite streams，也可以用 of 方法来从任意数据中生成流。

但我们多了两个方法：

- range
- rangeClosed

它们的区别在于 rangeClosed 的上界是包含在内的。

```java
public class StreamDemo {
    public static void show() {
        IntStream.rangeClosed(1, 5)
                .forEach(System.out::println);
    }
}
```

上面的代码的输出结果为：

> 1
> 2
> 3
> 4
> 5

如果用的是 range 方法，5 是不包括在内的。

之前学过的关于 mapping，filtering，slicing 和 collecting 这些操作都仍旧适用。

## Summary

- Streams
- Mapping
- Filtering
- Slicing
- Sorting
- Reducing
- Collectors