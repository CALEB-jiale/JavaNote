# The Excutive Framework

在线程的基础上，Java 引入了更高级别的抽象来简化构建并发型应用程序的过程。

通过这些抽象，我们不需要显式地处理线程，只需要专注于我们的任务，让 Java 来处理线程操作。

## Thread Pools

线程是构建并发程序模块的基础，但是直接使用线程有两个问题：

1. Availability：我们可以使用的线程数量有限，如果不小心创建了过多的线程，程序就会崩溃
2. Cost：创建一个独立的线程的开销很大

Java 为了解决开销大的问题，提出了一种称为线程池的解决方案。

线程池本质上就是一池子线程，这个池子里的线程被称为 Worker Thread，这些线程可以被多次使用，从而一个线程就可以执行多次任务。如果一个工作线程的任务完成了，它就会被放回线程池，这样就可以重新用它来执行另一个任务。所以这些线程没有被销毁和重新创建，从而能减小开销。

同时，因为线程池中的线程数量是一定的，所以我们不必担心创建了太多的线程，以至于内存不够用。

比如说，我们可以创建一个有 10 个线程的线程池，再给这个线程池分配 1000 个任务，那线程池就会负责给每个线程分配任务。如果所有的线程都在执行任务，那新的任务就会在队列中保持等待。一旦有线程可用，就会从队列中分配一个任务给这个线程。

有了这个模型，我们就不用再反复创建线程了。我们只需要把任务交给线程池，让线程池来负责线程的管理。

## Executors

在 Java 中，线程池的概念由 ExecutorService 接口和它的实现来表示。

它的实现有：

1. ThreadPoolExecutor：这是一个典型的线程池实现，也是我们最常使用的
2. ScheduledThreadPoolExecutor：它可以安排任务在一定的延迟之后运行或定期运行。例如我们可以安排一个线程从现在起运行 5 个小时，或者每两个小时运行一次。
3. ForkJoinPool：这是一种特殊的线程池，它可以递归地把一个大任务分割成多个小任务，然后再把小任务的结果组合起来，得出大任务的结果。这就像分治算法。

在 java.util.concurrent 中有一个 Executors 类，这个类中有一堆静态工厂方法可以用来构建 ExecutorService，通过这些方法我们可以创建上述实现的实例。

```java
public class ExecutorsDemo {
    public static void show() {
        var executor = Executors.newFixedThreadPool(2);
        System.out.println(executor.getClass().getName());
    }
}
```

为什么我们不直接创建实例，而使用通过一些静态工厂方法来实现？

因为直接创建实例会有一些麻烦。如果我们直接输入 `new ThreadPoolExecutor`，就会看到这个构造函数有一堆参数，比如 corePoolSize，maximumPoolSize，keepAliveTime 等。所以显式地创建实例会有点困难，因此我们需要 Executors 类中的工厂方法。

Executors 类中的工厂方法有：

1. newSingleThreadExecutor：这个方法可以返回一个只有一个线程的执行器，这个我们不常用。
2. newFixedThreadPool：这个方法将根据传入的数字，创建一个有着对应数量的工作线程的线程池，也就是 ThreadPoolExecutor。
3. newScheduledThreadPool：这个方法将返回 ScheduledThreadPoolExecutor。

观察 executor 的类型，它是 java.util.concurrent.ExecutorService 类型的，也就是一个接口。但在运行时，它是 ThreadPoolExecutor，这就是面向接口的编程。

有了 executor，我们就可以通过 submit 方法向线程池提交任务。这个方法被重载了，我们可以传入一个 Runnable 对象，或者一个 Callable 对象，后者表示的是一个会返回结果的任务，稍后会详细解释。

```java
public class ExecutorsDemo {
    public static void show() {
        var executor = Executors.newFixedThreadPool(2);
        executor.submit(() -> System.out.println(Thread.currentThread().getName()));
    }
}
```

运行上述代码，得到的结果是：

> pool-1-thread-1

所以我们不再需要显式地创建线程，即便我们有 1000 个任务，我们也不用担心创建了太多的线程对称内存不够用，我们只需要向线程池提交任务。

举例代码如下：

```java
public class ExecutorsDemo {
    public static void show() {
        var executor = Executors.newFixedThreadPool(2);
        for (var i = 0; i < 10; i++)
            executor.submit(() -> System.out.println(Thread.currentThread().getName()));
    }
}
```

如果我们重新运行我们的代码，编译器会体现我们程序仍在运行当中，这是为何？

因为在我们向执行器提交了一个任务之后，它会认为将来可能有更多任务，所以它不会自动终止，它会留在内存中等待新的任务，所以我们必须明确地关闭执行器来终止我们的程序。

有两个方法可以关闭执行器，shutdown 和 shutdownNow。区别在于 shutdown 方法不会停止当前的任务，所以它会等待任务的完成，但不会接受新的任务。而 shutdownNow 会强迫停止现有的任务。

```java
public class ExecutorsDemo {
    public static void show() {
        var executor = Executors.newFixedThreadPool(2);
        executor.submit(() -> System.out.println(Thread.currentThread().getName()));
        executor.shutdown();
    }
}
```

但如果在执行任务时，或其它在关闭执行器前的时刻，程序出现了异常，就不能正常地关闭执行器，所以，作为一个好习惯，我们应该把这一部分放入 try/finally 模块中。

```java
public class ExecutorsDemo {
    public static void show() {
        var executor = Executors.newFixedThreadPool(2);
        
        try {
            executor.submit(() -> System.out.println(Thread.currentThread().getName()));
        }
        finally {
            executor.shutdown();
        }
    }
}
```

通过上述格式，我们就能确保无论发生什么，总能正确关闭执行器，这就是使用执行框架（Executive Framework）的好处。

但要注意的是，即便是使用了线程池，仍要担心两个线程共享资源时会遇到的问题，它只是简化了线程管理的问题。

## Callables and Futures

有时候我们的任务需要返回一个结果，我们就要用到 Callable 接口。

Callable 接口与 Runnable 接口类似，但它表示的是一个会返回结果的任务，它只有一个抽象方法叫做 call，这雨 Runnable 接口的 run 方法类似，但 call 返回的不是 void，而是 V，V 表示的是范型类型的参数。

所以如果我们在之前的 lambda 表达式中 return 一个结果，那这个 lambda 表达式就不再是 Runnable，而是 Callable 了。

```java
public class ExecutorsDemo {
    public static void show() {
        var executor = Executors.newFixedThreadPool(2);

        try {
            var future = executor.submit(() -> {
                LongTask.simulate();
                return 1;
            });

            System.out.println("Do more work");

            try {
                var result = future.get();
                System.out.println(result);
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        }
        finally {
            executor.shutdown();
        }
    }
}
```

其中 LongTask 的实现如下：

```java
public class LongTask {
    public static void simulate() {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}
```

上面代码中，future 被用来存储 Callable 返回的值，它是一个 Future<Integer> 类型的值。Future 是一个接口，它表示的是一个将在未来完成的操作的结果，所以这个操作的结果并不能立即在 CPU 中被算出来。

所以当我们使用 submit 方法，它会立即返回 Future 对象，而不会先等 3 秒，但这个对象中没有对应的值。

之后，我们需要通过 get 方法取出 future 中的值。

get 方法被重载了，其中一个版本可以输入 timeout 参数，如果操作时间太长，我们不想继续等待，就可以使用 timeout 参数来定义多久后超时。如果没有 timeout 参数，get 方法会阻塞当前线程，直到它得到了操作的结果。

使用 get 方法还要注意两种异常，一种是 InterruptedException，另一种是 ExecutionException，前者在线程被打断时抛出，后者在任务发生异常是被抛出。

future 中还有许多其他方法：cancel 方法可以用于取消操作，isCancelled 方法用来检查操作是否被取消了，isDone 方法告诉我们操作是否已经完成。

执行上面的代码，将看到 “Do more work” 会立即出现在屏幕上，因为 submit 方法不会等待它提交的任务的完成，它会在一个单独的线程上启动那个任务。然后当运行到 get 方法的时候，就必须去等待任务的完成，也就是 3 秒后，才能得到对应的结果，才能看到 1 被打印在屏幕上。

## Asynchronous Programming

上一节提过，get 方法会阻塞当前线程，直到它得到了操作的结果。所以即便我们把这个操作放入了一个线程中去完成，我们还是得等待，这是对线程的浪费。如果我们让负责 UI 的主线程去等待另一个线程的完成，主线程就无法对 UI 事件给出响应。

为了更大限度地利用线程，我们应该以非阻塞的方式来编写代码，这就是所谓的异步编程（Asynchronous Programming）。Asynchronous 的含义等价于 Non-blocking。

打个比方，假设我们去餐厅吃饭，服务员在我们点单后会把我们的菜单交给厨房，让厨房来制作食物。如果是阻塞编程，服务员在把菜单交给厨房后就会坐着不动等厨房把菜做好，然后给我们上菜。如果是非阻塞编程，服务员会在把菜单交给厨房后先去服务别的客人。服务员就像是应用程序中的线程。

我们希望尽可能地减少线程的等待时间，这就需要我们协调我们的任务，这样在一个任务完成后，另一个任务可以异步执行。

## Completable Futures

接下来要讲的是一个非常重要的类 CompletableFuture，以及如何使用它来构建复杂的异步操作。

它实现了 Future 接口，所以每一个 CompletableFuture 都是一个 Future 对象。它之所以叫做“可完整的未来”，是因为我们可以显式地完成这个未来的对象。

它实现的另一个接口是 CompletionStage，它表示的是一个可能存在的异步阶段。在现实世界中，异步操作经常涉及很多步骤。例如我们可以调用远程 API 来获取一些数据，这是第一步。然后我们可能想要把这些数据转换成一种不同的结构，这是第二个步骤。最后我们分想把这些数据写到数据库中，这是第三个步骤。这个接口表示的就是一个步骤，或者说是异步操作过程中的一个阶段，它给了我们一堆用声明的方式来组合这些步骤的方法。这有点像我们使用流（Stream）API，以声明的方式构建复杂的查询过程。之前我们用的是 map，filter 和 reduce 等方法，现在可以用 CompletionStage 接口完成同样的操作。

## Creating a Completable Future

在 CompletableFuture 类中，有一堆可以来构建 CompletableFuture 对象的方法。

如果想要运行一个返回为 void 的任务，就可以使用 runAsync 方法，这是在告诉 Java 以异步或者说非阻塞的方式来执行这个任务。这个方法需要传入一个 Runnable 对象作为参数，需要的话可以再传入一个 Executor 对象。如果没有传入 Executor，这个方法将在一个公共的线程池中执行任务。ForkJoinPool 是 ExecutorService 接口的实现的一种，它的静态方法 commonPool 会返回一个被 CompletableFuture 类使用的线程池。所以如果我们不在 runAsync 方法中提供 Executor 对象，代码运行时底层使用的就是这个 commonPool。这个公共的线程池知道可用的线程的数量，可以通过 `Runtime.getRuntime().availableProcessors()` 来获得。根据实际中要搭建的应用程序的需求，我们可以用这个 commonPool 或者自己提供线程池。

```java
public class CompletableFutureDemo {
    public static void show() {
        Runnable task = () -> System.out.println("a");
        var future = CompletableFuture.runAsync(task);
    }
}
```

上面代码中 future 的类型是 CompletableFuture<void>，这是因为 Runnable 对象没有返回值。

通过 runAsync 方法，我们可以用异步的形式来执行任务，我们不需要创建一个 Executor，然后向它提交任务，然后关闭它。

如果任务会返回一个值，就不应该用 runAsync 方法，而应该用 supplyAsync 方法，这个方法需要传入一个 Supplier 对象，以及一个可选的 Executor 对象。

```java
public class CompletableFutureDemo {
    public static void show() {
        Supplier task = () -> 1;
        var future = CompletableFuture.supplyAsync(task);

        try {
            var result = future.get();
            System.out.println(result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

上面代码中 future 的类型是 CompletableFuture<Integer>。

## Implementing an Asynchronous API

这一节要展示如何创建一个异步 API，这是一个非常强大的技术，我们应该在我们的应用程序中用到它。

在 executors 包中创建新的类 MailService，我们将使用这个类来给我们的用户发送邮件。

```java
public class MailService {
    public void send() {
        LongTask.simulate();
        System.out.println("Mail was sent.");
    }
}
```

在实际应用中，send 方法应该需要一个 Mail 对象作为输入，但这里被简化了。

发送邮件是一个运行时间比较长的操作，因为我们会用到互联网。实际上，任何需要接触文件系统或者网络的操作，都属于运行时间比较长的操作，因此我们不应该在主线程中运行它们，而应该给它们分配一个单独的线程。

主函数的代码如下：

```java
public class Main {
    public static void main(String[] args) {
        var service = new MailService();
        service.send();
        System.out.println("Hello World");
    }
}
```

由于这是一个同步的或者说是阻塞的操作，所以我们会在 send 方法返回后才能看到 “Hello World” 被打印出来，输出结果如下：

> Mail was sent.
> Hello World

如今，每当我们有一个长运行操作，比如说访问数据库、呼叫远程服务器等，我们应该以异步的方式来执行这些操作。

```java
public class MailService {
    public void send() {
        LongTask.simulate();
        System.out.println("Mail was sent.");
    }

    public CompletableFuture<Void> sendAsync() {
        return CompletableFuture.runAsync(() -> send());    }
}
```

因为 send 方法返回的是 void，所以 sendAsync 返回的是 CompletableFuture<Void>，对应的，如果 前者返回的是 int，后者就是 CompletableFuture<Integer>。

根据惯例，对于异步操作，我们在添加 Async 后缀。

由于我们不需要返回值，所以返回时用的方法是 runAsync，相反，如果需要返回值，就要用 supplyAsync。

按照上面代码的模式，我们可以轻松地啊一个以同步方式执行的代码转换为以异步方式执行的代码。

```java
public class Main {
    public static void main(String[] args) {
        var service = new MailService();
        service.sendAsync();
        System.out.println("Hello World");
    }
}
```

运行上面的代码，可以看到 “Hello World” 被立即打印出来。

但是，不像之前，“Mail was sent.” 却不见了。这是因为在终端上我们有一个命令行程序，因为这个程序结束的太快了，以至于我们没能看到异步操作的结果，因为这个异步操作在一个独立的线程上运行着。但是在移动或者桌面程序上不会出现这个问题，因为这些程序是持续运行的，直到用户将它关闭。

为了证明邮件被成功发送了，我们可以把主函数所在的线程暂停 5 秒。

```java
public class Main {
    public static void main(String[] args) {
        var service = new MailService();
        service.sendAsync();
        System.out.println("Hello World");

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

运行后，我们会先看到 “Hello World”，3 秒后看到 “Mail was sent.”，再过两秒后看到程序结束。

异步编程 API 的好处就是不会阻塞当前线程，允许我们更好地利用并行硬件。

## Running Code on Completion

我们经常会需要在一些异步操作完成后执行一些代码。

比如我们可能会想把异步操作的结果打印出来，或者存入数据库中。

这一节将讲解如何进行这种操作。

```java
public class CompletableFutureDemo {
    public static void show() {
        Supplier task = () -> 1;
        var future = CompletableFuture.supplyAsync(task);
        future.thenRun(() -> {
            System.out.println(Thread.currentThread().getName());
            System.out.println("Done");
        });
    }
}
```

thenRun 方法是由 CompletionStage 接口提供的，前面提到过，它表示的是一个可能存在的异步阶段。在这个接口中还有很多方法可以用来构建复杂的异步操作，它们中大多数都是以 then 开头的，这表示这个操作会接在前一个操作后面。

同理还有 thenRunAsync，这会以异步的方式来执行任务。

```java
public class CompletableFutureDemo {
    public static void show() {
        Supplier task = () -> 1;
        var future = CompletableFuture.supplyAsync(task);
        future.thenRunAsync(() -> {
            System.out.println(Thread.currentThread().getName());
            System.out.println("Done");
        });
    }
}
```

观察两次代码的运行结果，会发现线程是不一样的，前者是在主线程上运行，后者在公共线程池中运行。

如果我们想要获取 CompletableFuture 的结果，我们可以使用 thenAccept 方法。这个方法需要传入一个 Consumer 对象，Consumer 接口只有一个未初始化的方法 accept，它以一个对象作为输入，不返回任何值。

```java
public class CompletableFutureDemo {
    public static void show() {
        Supplier task = () -> 1;
        var future = CompletableFuture.supplyAsync(task);
        future.thenAccept(result -> {
            System.out.println(Thread.currentThread().getName());
            System.out.println(result);
        });
    }
}
```

输出的结果是：

> main
> 1

同理也有 thenAcceptAsync 方法：

```java
public class CompletableFutureDemo {
    public static void show() {
        Supplier task = () -> 1;
        var future = CompletableFuture.supplyAsync(task);
        future.thenAcceptAsync(result -> {
            System.out.println(Thread.currentThread().getName());
            System.out.println(result);
        });
    }
}
```

输出结果如下：

> ForkJoinPool.commonPool-worker-1
> 1

有时候我们并不能看到任务返回的结果，也就是上面的数字 1，这是因为主线程结束的太快了，我们只需要让主线程等待一会儿就好：

```java
public class CompletableFutureDemo {
    public static void show() {
        Supplier task = () -> 1;
        var future = CompletableFuture.supplyAsync(task);
        future.thenAcceptAsync(result -> {
            System.out.println(Thread.currentThread().getName());
            System.out.println(result);
        });

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## Handling Exceptions

在异步操作中，如果出现了异常，该怎么办？

假设我们试图呼叫一个远程服务器，这个服务器可以给我们返回给定城市的当前天气，但由于各种原因，这个远程呼叫可能会失败，接下来就看看如何处理异常，以及如何通过把异常转化为值来修复异常。

```java
public class CompletableFutureDemo {
    public static void show() {
        Supplier task = () -> {
            System.out.println("Getting the current weather");
            throw new IllegalStateException();
        };
        var future = CompletableFuture.supplyAsync(task);
    }
}
```

我们运行以上代码，会发现终端上什么也没有出现，这是因为这个线程是在另一个线程上抛出的，要得到这个线程，我们必须使用 Future 接口中的 get 方法。

如果我们看 Future 接口的文档，会发现 get 方法会返回一个 V 类型的值，会抛出 InterruptedException 或者 ExecutionException。所以如果一个异常在线程上被抛出，get 方法能够把它带到主线程中。

```java
public class CompletableFutureDemo {
    public static void show() {
        Supplier task = () -> {
            System.out.println("Getting the current weather");
            throw new IllegalStateException();
        };
        
        var future = CompletableFuture.supplyAsync(task);

        try {
            future.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

这样我们就可以看到异常了。

输出结果为：

> Getting the current weather
> java.util.concurrent.ExecutionException: java.lang.IllegalStateException
>
> ……

结果表明，我们在试图获取天气情况的过程中发生了执行异常，而这个执行异常是由非法状态异常引起的。

所以 lambda 中抛出的异常被包裹进了执行异常里。

在我们调用 get 方法的时候，如果当时线程仍处于被暂停的状态中，而我们尝试去打扰它，就会返回一个 InterruptedException。如果是在执行任务期间出了异常，就会返回一个 ExecutionException。

我们也可以使用 getCause 方法来获取异常的产生原因。

如果我们不想让应用程序崩溃，我们可以修复并尝试提供一个预设值，比如我们之前成功读取的最后一个温度，这需要用到 exceptionally 方法，这个方法需要的参数的类型为 `Function<Throwable, ?>`，这个 Function 对象会把一个 Throwable 对象转换成另一种类型的对象，其中 Throwable 类是 Java 中所有的 Error 或者 Exception 类的父类。

```java
public class CompletableFutureDemo {
    public static void show() {
        Supplier task = () -> {
            System.out.println("Getting the current weather");
            throw new IllegalStateException();
        };

        var future = CompletableFuture.supplyAsync(task);

        try {
            var temperature = future.exceptionally(exception -> 1).get();
            System.out.println(temperature);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.getCause();
            e.printStackTrace();
        }
    }
}
```

我们需要理解的是，exceptionally 方法会返回一个新的 CompletableFuture 对象，这个对象与我们前面的 future 变量所代表的不同，而后面的 get 方法也不再属于 future，而属于最新返回的那个对象。

在 CompletableFuture 中还有很多方法与 exceptionally 类似，会返回一个新的 CompletableFuture 对象，这帮助我们可以实现复杂的操作。

## Transforming Results

有时候我们需要把异步操作的结果转化，比如说我们的天气服务需要把返回的温度转化为一个复杂的数据结构，或者把摄氏度转换为华氏度，所以我们需要把结果映射或转换成另一种类型。

thenApply 方法可以将结果映射为另一种类型。

```java
public class CompletableFutureDemo {
    public static void show() {
        var future = CompletableFuture.supplyAsync(() -> 20);
        try {
            var result = future
                    .thenApply(celsius -> (celsius * 1.8) + 32)
                    .get();
            System.out.println(result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

执行上述代码，可以得到 20 摄氏度等于 68 华氏度。

上述代码可以改进，我们可以把 lambda 表达式封装到一个方法中：

```java
public class CompletableFutureDemo {
    public static int toFahrenheit(int celsius) {
        return (int) (celsius * 1.8) + 32;
    }

    public static void show() {
        var future = CompletableFuture.supplyAsync(() -> 20);
        future.thenApply(CompletableFutureDemo::toFahrenheit).thenAccept(System.out::println);
    }
}
```

这是 CompletableFuture 的好处之一，通过使用它，我们可以以声明的方式完成复杂的异步操作。

## Composing Completable Futures

我们经常会想要在完成一个任务之后再开始另一个任务，例如我们有一个用户的 ID，我们想要得到他的电子邮箱，所以我们要去数据库中读取用户数据，从而就可以找到他的电子邮箱。之后，我们想要把他的邮箱发给一个音乐流媒体网站，在这个网站上，人们都有自己的播放列表，所以我们传递邮箱，并获取了播放列表。从而我们就有了两个异步操作：

> ID $\to$ email
>
> email $\to$ playlist

我们想要在第一个操作完成后就开始第二个操作，CompletableFuture 能够帮助我们用声明的方法高效简单地完成这项工作。

```java
public class CompletableFutureDemo {
    public static void show() {
        // ID -> email
        var future = CompletableFuture.supplyAsync(() -> "email");
        // email -> playlist
        future
            .thenCompose(email -> CompletableFuture.supplyAsync(() -> "playlist"))
            .thenAccept(System.out::println);
    }
}
```

代码中大部分过程都被简化了。

上面的代码还可以简化，比如说去掉 future 变量，以及一下变化：

```java
public class CompletableFutureDemo {
    public static String getUserEmail() {
        return "email";
    }

    public static CompletableFuture<String> getUserEmailAsync() {
        return CompletableFuture.supplyAsync(CompletableFutureDemo::getUserEmail);
    }

    public static CompletableFuture<String> getPlaylistAsync(String email) {
        return CompletableFuture.supplyAsync(() -> "playlist");
    }

    public static void show() {
        getUserEmailAsync()
            .thenCompose(CompletableFutureDemo::getPlaylistAsync)
            .thenAccept(System.out::println);
    }
}
```

又一次，我们通过使用 CompletableFuture，以声明的方式完成了复杂的异步操作。

这里我们只写了 getPlaylistAsync，没有写 getPlaylist，这是因为这个方法需要一个参数，但在 CompletableFuture 中只有类似 run 和 supply 类型的方法，前者需要传入 Runnable 对象，后者需要传入 Callable 对象，但没有可以传入一个功能类似 Function 接口的方法，这个问题的求解看下一节内容。

## Combining Completable Futures

CompletableFuture 最强大的功能是它可以以异步的方式同时开始两个操作，然后把两个操作的结果组合起来。

例如我们通过远程服务器获取产品的价格，服务器以美元返回价格（假设为 20 美元），同时我们要通过另一个服务器获取美元与欧元的汇率（假设为 0.9）。我们希望上述两个操作能同时开始，等它们都完成了以后，再计算结果。

通过 thenCombine 方法，我们可以在两个异步操作都完成后，把它们的结果整合到一起。但要注意的是，调用这个方法不会阻塞当前线程。

我们基本上可以说是在构建一个处理管道，我们在告诉 Excutive Framework：“同时开始这两个任务，当它们都完成的时候，开始一个新的任务去做其他的事情。”所有的事情都是以异步的方式展开的。

```java
public class CompletableFutureDemo {
    public static void show() {
        var first = CompletableFuture.supplyAsync(() -> 20);
        var second = CompletableFuture.supplyAsync(() -> 0.9);

        first
            .thenCombine(second, (price, exchangeRate) -> price * exchangeRate)
            .thenAccept(System.out::println);
    }
}
```

thenCombine 方法有两个参数，第一个参数是 `CompletionStage<?>` 类型的，也就是我们上面代码中的 second，第二个参数是 `BiFunction<? super Integer, ? super Double, ?>` 类型的，这里的 Integer 和 Double 是因为我们的 first 返回的是 Integer，second 返回的是 Double。

thenCombine 会返回一个新的 CompletableFuture 对象。

现在我们假设第一个操作返回的是一个 String，那我们就需要把它进行转换：

```java
public class CompletableFutureDemo {
    public static void show() {
        var first = CompletableFuture
                .supplyAsync(() -> "20USD")
                .thenApply(str -> {
                    var price = str.replace("USD", "");
                    return Integer.parseInt(price);
                });
        var second = CompletableFuture.supplyAsync(() -> 0.9);

        first
            .thenCombine(second, (price, exchangeRate) -> price * exchangeRate)
            .thenAccept(System.out::println);
    }
}
```

以上所有操作都是异步发生的，它们不会阻塞当前进程。

## Waiting for Many Tasks

有时候我们在去做其他事情之前，需要等待很多任务先完成，而不像上一节只需要等待两个任务的完成，这是就要用到 allOf 方法。

allOf 方法需要的参数类型为 `CompletableFuture<?>...`，三个点表示参数数量不定，当我们把所有任务都传入后，这个方法会返回一个新的 CompletableFuture 对象，这个对象表示一个任务，这个任务会在前面传入的所有任务都完成后完成。但这个返回的对象是 `CompletableFuture<Void>` 类型的，因为那些任务返回的参数的类型是不确定的。

```java
public class CompletableFutureDemo {
    public static void show() {
        var first = CompletableFuture.supplyAsync(() -> 1);
        var second = CompletableFuture.supplyAsync(() -> 2);
        var third = CompletableFuture.supplyAsync(() -> 3);

        var all = CompletableFuture.allOf(first, second, third);
        all.thenRun(() -> {
            System.out.println("All tasks completed successfully!");
        });
    }
}
```

运行上述代码即可得到结果。

但如果我们希望得到那些任务的结果，我们可以在 thenRun 方法中调用每个任务的 get 方法来得到对应任务的结果，而且这里的 get 方法不会阻塞当前线程，因为 thenRun 模块内的代码只有在前面的任务全部完成后才会开始执行。

```java
public class CompletableFutureDemo {
    public static void show() {
        var first = CompletableFuture.supplyAsync(() -> 1);
        var second = CompletableFuture.supplyAsync(() -> 2);
        var third = CompletableFuture.supplyAsync(() -> 3);

        var all = CompletableFuture.allOf(first, second, third);
        all.thenRun(() -> {

            try {
                var firstResult = first.get();
                System.out.println(firstResult);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }

            System.out.println("All tasks completed successfully!");
        });
    }
}
```

## Waiting for the First Task

假设我们有两种不同的方法来得到天气信息，也许我们有两个不同的远程服务器，但有时其中一个服务器反应会比较慢，所以我们想要并发地访问这两个服务器，并且只要我们可以得到一个响应，我们就把结果显示给用户。

此时我们要用到的方法是 anyOf，这个方法与 allOf 方法类似，但它会在所有任务中的任意一个任务完成后完成。

```java
public class CompletableFutureDemo {
    public static void show() {
        var first = CompletableFuture.supplyAsync(() -> {
            LongTask.simulate();
            return 20;
        });

        var second = CompletableFuture.supplyAsync(() -> 20);

        CompletableFuture
                .anyOf(first, second)
                .thenAccept(System.out::println);
    }
}
```

运行上述代码，我们可以立即得到结果，而不用等待 3 秒。

## Handling Timeouts

在调用远程服务时，我们会想要设定一个最长等待时间，毕竟我们不希望一直等下去。

这是我们就要用到 orTimeout 方法，这个方法需要两个参数，前一个是数值，属于 long 类型，后一个是时间单位，属于 TimeUnit 类型。这个方法会在超时后返回一个新的 CompletableFuture 对象，此时如果我们用 get 方法来获得对应的结果，会得到一个异常。

```java
public class CompletableFutureDemo {
    public static void show() {
        var future = CompletableFuture.supplyAsync(() -> {
            LongTask.simulate();
            return 1;
        });

        try {
            var result = future.orTimeout(1, TimeUnit.SECONDS).get();
            System.out.println(result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

    }
}
```

运行上述代码，我们将在 1 秒后得到一个异常，而这个异常的起因是 TimeoutException。

这对于用户来说不会是一个好的体验，更好的方法是使用默认值进行修复，这就不再需要 orTimeout 方法，而要用到 completeOnTimeout 方法。

这个方法需要传入三个值，第一个是我们预设的默认值，这个值会在任务超时后被返回，后两个与 orTimeout 方法类似，前者是数值，后者是时间单位。

```java
public class CompletableFutureDemo {
    public static void show() {
        var future = CompletableFuture.supplyAsync(() -> {
            LongTask.simulate();
            return 1;
        });

        try {
            var result = future.completeOnTimeout(1, 1, TimeUnit.SECONDS).get();
            System.out.println(result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        
    }
}
```

运行上述代码，我们将在 1 秒后得到我们的预设值 1。

## Project: Best Price Finder

这一节，我们要综合之前学到的知识来写一个程序，这个程序要负责找到最便宜的航班。

我们有不同的飞行中介，我们需要对他们进行询问才能得到给定航班的报价。

```java
public class Quote {
    private final String site;
    private final int price;

    public Quote(String site, int price) {
        this.site = site;
        this.price = price;
    }

    public String getSite() {
        return site;
    }

    public int getPrice() {
        return price;
    }

    @Override
    public String toString() {
        return "Quote{" +
                "site='" + site + '\'' +
                ", price=" + price +
                '}';
    }
}
```

先构建以上类，用于记录网站信息与航班价格。其中 toString 方法可以通过 command + N 来构建。

```java
public class FlightService {
    public CompletableFuture<Quote> getQuoteAsync(String site) {
        return CompletableFuture.supplyAsync(() -> {
            System.out.println("Getting a quote from" + site);

            LongTask.simulate();

            var random = new Random();
            var price = 100 + random.nextInt(10);

            return new Quote(site, price);
        });
    }
}
```

再构建以上类，用于模拟访问服务器的过程。

然后回到 CompletableFutureDemo 中进行更改：

```java
public class CompletableFutureDemo {
    public static void show() {
        var service = new FlightService();

        service.getQuoteAsync("site1")
                .thenAccept(System.out::println);

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

以上只是一个小的模块，我们可以继续扩展。

```java
public class FlightService {
    public List<CompletableFuture<Quote>> getQuotes() {
        var sites = List.of("site1", "site2", "site3");

        return sites.stream()
                .map(this::getQuoteAsync)
                .collect(Collectors.toList());

    }

    public CompletableFuture<Quote> getQuoteAsync(String site) {
        return CompletableFuture.supplyAsync(() -> {
            System.out.println("Getting a quote from" + site);

            LongTask.simulate();

            var random = new Random();
            var price = 100 + random.nextInt(10);

            return new Quote(site, price);
        });
    }
}
```

在 FlightService 中构建新的方法如上。

```java
public class CompletableFutureDemo {
    public static void show() {
        var service = new FlightService();

        service.getQuotes()
                .map(future -> future.thenAccept(System.out::println))
                .collect(Collectors.toList());

        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

然后将 CompletableFutureDemo 进行如上修改。

其中 collect 方法在这里只是用于结束流的操作，此处实际意义不大，如果用 forEach 方法将内容打印出来，会得到以下结果：

> java.util.concurrent.CompletableFuture@2a84aee7[Not completed]
> java.util.concurrent.CompletableFuture@a09ee92[Not completed]
> java.util.concurrent.CompletableFuture@30f39991[Not completed]
> Getting a quote fromsite2
> Getting a quote fromsite1
> Getting a quote fromsite3
> Quote{site='site1', price=109}
> Quote{site='site2', price=106}
> Quote{site='site3', price=103}

这是因为上面的操作被分配到了不同的线程上，在主线程运行到 `forEach(System.out::println)` 的时候，CompletableFuture 都还没被返回。但在之后其他线程都结束以后，CompletableFuture 的内容会被填入到 List 中，这也就是 CompletableFuture 存在的意义。

而打印操作在 map 模块就已经完成，但如果我们不想要把结果收集起来，也可以不用 map，直接进行 forEach：

```java
public class CompletableFutureDemo {
    public static void show() {
        var service = new FlightService();

        service.getQuotes()
                .forEach(future -> future.thenAccept(System.out::println));

        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

上述代码同样可以得到理想结果。这告诉我们，如果我们既想对每个结果进行操作，又不想立即就结束流，就可以使用 map 方法。

但我们之所以使用 map 方法而不是用 forEach 方法，是因为我们希望能收集结果，这个在之后会有用。

接下来，我们要给每个网站设定一个随机延迟。

```java
public class LongTask {
    public static void simulate() {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void simulate(int delay) {
        try {
            Thread.sleep(delay);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

先按上面的方法重写 LongTask 类中的 simulate 方法。

```java
public class FlightService {

    public Stream<CompletableFuture<Quote>> getQuotes() {
        var sites = List.of("site1", "site2", "site3");

        return sites.stream().map(this::getQuoteAsync);
    }

    public CompletableFuture<Quote> getQuoteAsync(String site) {
        return CompletableFuture.supplyAsync(() -> {
            System.out.println("Getting a quote from" + site);

            var random = new Random();

            LongTask.simulate(1_000 + random.nextInt(2000));

            var price = 100 + random.nextInt(10);

            return new Quote(site, price);
        });
    }
}
```

然后如上修改 FlightService 类，让每次访问服务器都有 1 到 3 秒的延迟。

运行上面的代码，可以看到三次访问同时开始，但不同时结束。

接下来我们希望在查询结束后打印一条信息，这条信息要告诉我们查询所有网站共花了多长时间。

在 allOf 中我们期待 CompletableFuture 能以 Array 的形式呈现，而不是 List，所以用到了 toArray 方法。

而 toArray 方法在默认情况下会返回一个 Object 类型的数组，但 allOf 方法期待的是一个 CompletableFuture 类型的数组，所以我们需要使用 toArray 的重写的版本，传入一个 CompletableFuture 类型的数组作为参数。通过传入一个空数组，我们在告诉这个方法我们希望得到的结果是 CompletableFuture 类型的数组。

```java
public class CompletableFutureDemo {
    public static void show() {
        var start = LocalTime.now();

        var service = new FlightService();
        var futures = service.getQuotes()
                .map(future -> future.thenAccept(System.out::println)).toList();

        CompletableFuture
                .allOf(futures.toArray(new CompletableFuture[0]))
                .thenRun(() -> {
                    var end = LocalTime.now();
                    var duration = Duration.between(start, end);
                    System.out.println("Retrieved all quotes in " + duration.toMillis() + " msec.");
                });

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

上面的代码中，between 方法可以用来计算两个时间的差，toMillis 会把时间单位转换为毫秒。

运行上面的代码就可以得到理想结果。