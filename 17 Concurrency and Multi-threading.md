# Concurrency and Multi-threading

现在大多数计算机都有多核处理器，可以并行执行多个任务。这一章要讲的就是如何在代码中利用多核处理器，这能提升程序的响应速度。

## Processes and Threads

进程（process）是一个程序（program）或一个应用（application）的实例。当启动一个应用程序（如代码编辑器或音乐播放器）时，操作系统就会把这个应用加载到一个进程中，因此一个进程包含着这这个应用的代码的映像，它有一些内存和一些其他资源。

你的操作系统可以同时执行多个进程。例如，它可以在播放音乐的同时运行杀毒软件，这就是进程级的并发。但也可以在进程内或使用线程的应用中使用并发。

技术上来说，线程是指令序列（a sequence of instructions），它就像一条指令线。现实点说，线程就是执行代码的东西。

每个进程至少调用一个线程，称为主线程。但我们也可以创造额外的线程去同时执行多个任务。例如我们可以创建一个可以一同时服务多个用户的 web 服务器，我们将使用一个单独的线程为每个客户端服务。或者我们可以构建一个可以同时下载多个文件的应用程序，让每个线程下载一个单独的文件。这就是多线程。

如今大多数处理器都有多个核心，这些核心可以用于运行许多进程或线程。如果一个程序不使用线程，它实际上只是用了一个处理器核心，所以它并没有充分利用 CPU 的能量，这就导致了硬件资源的浪费。

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Thread.activeCount());
        System.out.println(Runtime.getRuntime().availableProcessors());
    }
}
```

Thread 类在 java.lang 中，它有一个静态方法 activeCount，这个方法会返回当前进程中的线程的数量。

Runtime 类也在 java.lang 中，在调用它的静态方法 getRunTime 后可以调用 availableProcessors 方法，来获取可用的线程数。

上面的代码返回的结果是 2 和 12。

数字 2 说明这个程序使用了两个线程，其中一个是主线程，用来运行 main 函数，另一个是后台线程，用来运行 garbage collector，垃圾收集器是用来从内存中删除不再使用的对象的。

数字 12 说明这台电脑上有 12 个线程可供使用，这个数字每台电脑上的可能不同，我的电脑有六个核心，每个核心上面有两个线程，所以我有 12 个线程可以用来运行程序。

## Starting a Thread

我们使用在 java.lang 中声明的 Thread 类来创建线程，它的构造函数 Thread 被重载了，我们最常用的版本要一个实现了 Runnable 接口的对象作为输入。

Runnable 接口表示一个要在线程中运行的任务，它只有一个方法叫做 run，无参数，并返回 void。

现在假设我们想同时下载许多文件，那我们要执行的任务就是下载一个文件。

创建新的包 concurrency，在包内创建类 ThreadDemo 和 DownloadFileTask。

```java
public class DownloadFileTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Downloading a file: " + Thread.currentThread().getName());
    }
}
```

DownloadFileTask 简化如上。

在 ThreadDemo 创建线程。

```java
public class ThreadDemo {
    public static void show() {
        System.out.println(Thread.currentThread().getName());
        Thread thread = new Thread(new DownloadFileTask());
        thread.start();
    }
}
```

在创建线程结束后，我们调用 start 方法。当这个程序运行时，用于下载文件的代码将会运行在一个单独的线程中。

验证的方法是在创建线程之前，把当前线程的名字打印出来，用的方法是 `Thread.currentThread().getName()` 或 `Thread.currentThread().getId()`。

运行后得到的结果是：

> main
> Downloading a file: Thread-0

从结果看出我们有两个线程，一个是执行 main 方法的主线程，另一个是我们显式创建的线程。

每个线程都要经历开始、执行任务和结束三个过程，我们不能显式地结束一个线程。

接下来对上面的代码进行改造，把创造并开始一个线程的部分放入一个 for 循环中，来模拟下载 10 个文件的过程。

```java
public class ThreadDemo {
    public static void show() {
        System.out.println(Thread.currentThread().getName());

        for (var i = 0; i < 10; i++) {
            Thread thread = new Thread(new DownloadFileTask());
            thread.start();
        }
    }
}
```

得到的结果如下：

> main
> Downloading a file: Thread-8
> Downloading a file: Thread-6
> Downloading a file: Thread-2
> Downloading a file: Thread-1
> Downloading a file: Thread-3
> Downloading a file: Thread-7
> Downloading a file: Thread-0
> Downloading a file: Thread-5
> Downloading a file: Thread-9
> Downloading a file: Thread-4

除了主线程，我们还有 10 个额外的线程。

尽管上述的结果在终端是线性打印出来的，但这些线程是同时启动同时运行的。

## Pausing a Thread

Thread 类中有一个方法叫 sleep，我们可以用它来暂停一个线程的执行。

有了这个，我们可以模拟一个长时间运行的操作，比如说下载一场比赛。这个方法的参数是以毫秒为单位的。

如果我们输入 5000，这将会暂停当前的线程大约 5 秒，但一般不会正好 5 秒，这取决于底层操作系统。

在一个线程暂停后，其它线程就可以获得处理器时间。

在调用 sleep 方法后，Java 编辑器会报告说有一个未处理的异常，这个异常是 InterruptedException，这是一个抛出异常。将光标移到 sleep 上后用 option + enter 将其放入 try/catch 模块中即可。

```java
public class DownloadFileTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Downloading a file: " + Thread.currentThread().getName());

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Download completed: " + Thread.currentThread().getName());
    }
}
```

运行程序后，会发现 10 个下载任务基本都在开始 5 秒后完成下载。

如果只有一个单线程的程序，这个过程需要 50 秒，而不只是 5 秒。所以多线程可以是我们的程序变得更快。

现在假设我们想下载 100 个文件，这个机器有 6 个核心 12 个线程可用，会怎样运行？

JVM 中有一个东西叫做线程调度器（Thread Scheduler），这个调度器的任务就是去决定每个线程要执行多久的时间。所以如果我们的任务数量大于线程数量，调度器就会在任务之间不断切换，给每个任务一定的 CPU 时间，这个过程非常快，以至于我们误以为所有的任务都是在同时进行的。但这是在软件层面上的并行。

## Joining Threads

假设我们下载了一个文件，我们想要启动另一个线程来扫描下载文件的线程。

获取文件是一项耗时的工作，所以我们应该在一个单独的线程中运行它。

但我们只有在完成下载之后才能获取文件，那我们如何知道下载的线程已经完成？

由于下载文件需要的时间是不确定的，所以我们不能使用模拟时使用的 Thread.sleep 中传入的参数来得到具体的下载时间。

因此就需要用到 join 方法。

```java
public class ThreadDemo {
    public static void show() {
        System.out.println(Thread.currentThread().getName());

        Thread thread = new Thread(new DownloadFileTask());
        thread.start();

        try {
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("File is ready to be scanned.");
    }
}
```

在 start 后面使用 join 方法，将使当前的线程，也就是执行当前代码的主线程，等待下载线程的完成。

所以这个调用会阻塞当前的线程，直到下载线程结束。

一旦这个方法 return，我们就可以打印接下来的信息。

与 sleep 方法类似，join 方法也会抛出 InterruptedException，需要用 option + enter 将其放入 try/catch 模块中。

输出结果如下：

> main
> Downloading a file: Thread-0
> Download completed: Thread-0
> File is ready to be scanned.

如果去掉 join 方法，输出结果如下：

> main
> File is ready to be scanned.
> Downloading a file: Thread-0
> Download completed: Thread-0

当线程在等待的过程中时，它不能做任何动作。

例如在桌面或者移动应用程序中，主线程负责处理 UI，比如说鼠标的点击或键盘的输入，如果我们让主线程等待另一个线程，它在等待时就无法响应 UI 事件，此时用户就无法拖动窗口或改变窗口大小。

不过我们后面还会有更好的方法来实现让一个线程在另一个线程后面启动，而不会导致当前的线程等待。

## Interrupting a Thread

通常在处理长时间运行的任务时，我们希望给用户取消的权利。

这里我们开始一个下载任务，我们想一秒后把它取消掉。

首先回到 DownloadFileTask 中，做出如下修改：

```java
public class DownloadFileTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Downloading a file: " + Thread.currentThread().getName());

        for (var i = 0; i < Integer.MAX_VALUE; i++)
            System.out.println("Downloading byte " + i);

        System.out.println("Download completed: " + Thread.currentThread().getName());
    }
}
```

通过上述修改，可以清楚看到任务是否被取消。

然后回到 ThreadDemo 中，尝试在 1 秒后取消任务。

首先尝试用 sleep 方法让主线程暂停 1 秒，然后使用 interrupt 方法取消下载任务。

```java
public class ThreadDemo {
    public static void show() {
        Thread thread = new Thread(new DownloadFileTask());
        thread.start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        thread.interrupt();
    }
}
```

如果运行上述代码，会发现任务会一直下载，不会被取消。

这是因为调用 interrupt 方法并不能强制线程停止，它只是向线程发送一个中断请求，最终还是线程来决定是否要停止。

因此我们应该回到 DownloadFileTask 中，我们应该让它不断地检查是否有中断请求，如果有中断请求，就要停止下载。

```java
public class DownloadFileTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Downloading a file: " + Thread.currentThread().getName());

        for (var i = 0; i < Integer.MAX_VALUE; i++) {
            if (Thread.currentThread().isInterrupted()) return;
            System.out.println("Downloading byte " + i);
        }

        System.out.println("Download completed: " + Thread.currentThread().getName());
    }
}
```

修改后再次运行，就可以达到预期效果。

如果一个线程被暂停（sleep）的时候收到了中断请求（interrupt），就会抛出异常，这也是为何在使用 sleep 方法的时候必须要抛出 InterruptedException。

## Concurrency Issues

在展示的所有例子中，我们的线程是彼此隔离的，但在现实的情况下，有时我们的线程可能需要访问甚至修改共享资源。

例如在下载文件时，每个线程都可以报告已下载到共享资源中的字节大小，这样我们就可以跟踪整个下载任务的进度。

如果多个线程访问同一个对象，并且至少一个对其进行了修改，我们就会遇到一些问题：

1. 多个线程尝试在同一时间修改同一段数据

   这种情况下，我们有一个竞争条件（race condition），这意味着这些线程在竞争修改这段数据的权利，具体内容下一节会讲到。

2. 一个线程修改了共享的数据，但这一变化对其他线程来说是不可见的

   这一问题被称为可见性问题（visibility problem），不同的线程读取到的同一段数据的内容可能不同。

这些问题是由并发系统（concurrent system）的特性产生的，因此，如果我们想构建一个多线程程序，我们就需要对这些问题有合适的理解，并且知道该如何预防它们。

能够被多线程安全地并行运行的代码叫做线程安全代码（Thread-safe Code）。

## Race Conditions

假设我们想展示多个正在下载的文件的已下载比特总数，我们就需要把这个总数储存在某个地方，并让那些个线程一边下载一边让这个总数增加。

这就会产生竞争条件，它意味着这些线程在竞争修改这段数据的权利。

增加一个新类 DownloadStatus：

```java
public class DownloadStatus {
    private int totalBytes;

    public void incrementTotalBytes() {
        totalBytes++;
    }

    public int getTotalBytes() {
        return totalBytes;
    }
}
```

然后回到 ThreadDemo，进行如下修改：

```java
public class ThreadDemo {
    public static void show() {
        var status = new DownloadStatus();

        for (var i = 0; i < 10; i++) {
            var thread = new Thread(new DownloadFileTask(status));
            thread.start();
        }
    }
}
```

我们用 option + enter 让编译器智能创建一个构造函数用于接收 status 参数。

然后在 DownloadFileTask 中再用 option + enter 让编译器为 status 创建字段。

注意 DownloadFileTask 中的 run 方法中的 for 循环中的变化。

```java
public class DownloadFileTask implements Runnable {

    private DownloadStatus status;

    public DownloadFileTask(DownloadStatus status) {
        this.status = status;
    }

    @Override
    public void run() {
        System.out.println("Downloading a file: " + Thread.currentThread().getName());

        for (var i = 0; i < 10_000; i++) {
            if (Thread.currentThread().isInterrupted()) return;
            status.incrementTotalBytes();
        }

        System.out.println("Download completed: " + Thread.currentThread().getName());
    }
}
```

回到 ThreadDemo，在下载任务的线程结束后，我们应该打印下载的总比特数，所以我们得等待所有线程结束。

但我们不能在 for 循环中使用 join 方法，因为这会在下个线程开始前，让主线程暂停。

所以我们应该先让所有的线程同时开始，然后让主线程等待它们全部完成。

```java
public class ThreadDemo {
    public static void show() {
        var status = new DownloadStatus();

        List<Thread> threads = new ArrayList<>();

        for (var i = 0; i < 10; i++) {
            var thread = new Thread(new DownloadFileTask(status));
            thread.start();
            threads.add(thread);
        }

        for (var thread : threads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println(status.getTotalBytes());
    }
}
```

我们每个任务有 10_000 字节，总共 10 个任务，所以结果应该是 100_000。

但运行程序后会发现并不是这样，这就是多个线程竞争改写权利的结果。

在 DownloadStatus 中，我们有一行代码是 `totalBytes++;`，这个递增运算符包括 3 个步骤：

1. 从主存中读取 totalBytes 并将其存储在 CPU 中
2. CPU 将这个值增加
3. 这个值被存储回内存中

这种操作被称为非原子操作（non atomic operation），因为它涉及多个步骤。相反，原子操作就只有一个步骤。

假设有两个线程都想调用 incrementTotalBytes 这个方法，并假设这个值一开始是 0。那这两个线程并发地读取这个值，也就是 0，然后在 CPU 中增加，变成 1，然后被写回内存，内存中的值就是 1，而不是 2。

## Strategies for Thread Safety

我们有一些编写线程安全代码的策略。

### Confinement

最简单的方法是不在多个线程之间共享数据，这叫做约束法（Confinement），因为我们想把线程能用到的资源给限制住。

例如，我们不让所有下载线程都共享一个 DownloadStatus，而给每个线程分配一个，然后在下载完成后把它们合并起来。

### Immutability

另一个方法是使用不可更改的（Immutable）只读对象，只读意味着这个对象的数据在创建后不能被修改，例如 Java 中的字符串。所有对字符串的操作都会返回一个新的字符串，原始的字符串并没有改变。如果线程对一个共享资源只是读取而不是改写，就不会出现问题。

### Synchronization

另一个方法是防止多个线程同时访问同一个对象，这叫做同步（Synchronization），因为我们在协调它们对对象的访问。我们通过锁定资源来实现这一目的。我们在代码中设置一个锁，每次只有一个线程可以执行这一部分，其他线程必须等待。

因此同步强调代码按顺序（sequentially）执行，这与并发（concurrency）的思想相悖。

再加上实现同步会有新的挑战。我们可能遇到的问题之一是死锁，这一状况发生在两个线程互相等待对方资源的情况下。

综上，我们应当尽量避免使用同步。

### Atomic objects

另一个方法是使用原子类（atomic class），比如说原子整数。

这些类允许我们在不用锁的情况下实现线程安全。

如果我们想把一个原子整数增加，JVM 将会用一个原子操作来实现。

### Partitioning

最后一个方法是把数据划分为多个段，每个段每次只能被一个线程访问，但整个数据可以同时被多个线程访问，只要他们访问的不是同一个段。

Java 提供了许多通过分区来支持并发的集合（Collection）类，

所以多个线程可以同时访问一个集合对象，但每次只有一个线程可以访问一个集合中的某一个段。

## Confinement

根据约束法，我们要给每个线程分配一个 DownloadStatus。

DownloadFileTask 修改如下：

```java
public class DownloadFileTask implements Runnable {

    private DownloadStatus status;

    public DownloadFileTask() {
        this.status = new DownloadStatus();
    }

    @Override
    public void run() {
        System.out.println("Downloading a file: " + Thread.currentThread().getName());

        for (var i = 0; i < 10_000; i++) {
            if (Thread.currentThread().isInterrupted()) return;
            status.incrementTotalBytes();
        }

        System.out.println("Download completed: " + Thread.currentThread().getName());
    }

    public DownloadStatus getStatus() {
        return status;
    }
}
```

ThreadDemo 修改如下：

```java
public class ThreadDemo {
    public static void show() {
        List<Thread> threads = new ArrayList<>();
        List<DownloadFileTask> tasks = new ArrayList<>();

        for (var i = 0; i < 10; i++) {
            var task = new DownloadFileTask();
            tasks.add(task);

            var thread = new Thread(task);
            thread.start();
            threads.add(thread);
        }

        for (var thread : threads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        var totalBytes = tasks.stream()
                .map(task -> task.getStatus().getTotalBytes())
                .reduce(0, Integer::sum);

        System.out.println(totalBytes);
    }
}
```

运行后即可得到理想结果。

## Locks

另一种预防竞争条件和可见性问题的方法是在有线程访问资源时，就给共享资源上锁，防止其他线程的访问，这种方法叫做同步。

被上锁的部分被称为关键部分（critical section）。

通过这种方法，代码就会线性运行，这就失去了我们原本追求的并发性。

这一过程类似上厕所，在厕所内有人的时候，门就会被锁上，其他人就进不去，除非上一个人出来。

将 DownloadFileTask 和 ThreadDemo 还原到 Race Conditions 一节中的情况，然后将 DownloadStatus 修改如下：

```java
public class DownloadStatus {
    private int totalBytes;
    private Lock lock = new ReentrantLock();

    public void incrementTotalBytes() {
        lock.lock();
        try {
            totalBytes++;
        }
        finally {
            lock.unlock();
        }
    }

    public int getTotalBytes() {
        return totalBytes;
    }
}
```

Lock 是在 java.util.concurrent.locks 中声明的接口。

ReentrantLock 是 Lock 的实现之一。

lock 方法可以锁住这个 lock 对象，在实现增加操作后，再用 unlock 方法将其解锁。

unlock 方法最后应该放入到 finally 模块中，这样即使抛出异常，也能成功解锁，否则将会出现死锁。

这里因为递增符号不会抛出异常，所以不这么做也行。

运行后即可得到理想情况。

## The synchronised Keyword

除了 Lock，Java 中还有 synchronised 关键字可以实现同步。

synchronised 关键字的效果与 Lock 相同，只是没有了显式的上锁和解锁的过程。

```java
public class DownloadStatus {
    private int totalBytes;
    private int totalFiles;
    private Object totalBytesLock = new Object();
    private Object totalFilesLock = new Object();

    public void incrementTotalBytes() {
        synchronized (totalBytesLock) {
            totalBytes++;
        }
    }

    public synchronized void incrementTotalFiles() {
        totalFiles++;
    }

    public int getTotalFiles() {
        return totalFiles;
    }

    public int getTotalBytes() {
        return totalBytes;
    }
}
```

在 synchronised 关键字后的括号中，我们应当传入一个对象，这个对象被我们称为监控器对象。Java 中的每个对象都有一个内建的锁，所以 Java 将从监控器对象中获得内置锁，在底层使用。

例如我们可以使用 this 关键字，把当前对象传入，这也可以解决我们的问题，但这种做法并不合适，因为如果我们这个类中还有另外一个变量，比如说 totalFiles 用来记录下载的文件总数，而且我们也用 synchronised 关键字将其上锁，并且传入的对象仍是 this 关键字引用的当前对象，那就相当于两个厕所用了同一个门，totalFiles 和 totalBytes 两个变量，本来应该可以供两个线程单独访问，现在也只能让一个线程来使用，以为一旦某个线程尝试修改其中一个变量，另一个就也一起被锁死了。

综上，我们应该对每个区域使用专有的监视器对象。为此我们创建了两个 Object 对象，他们只是普通的对象实例，我们也可以用任意的其他类，但根据习惯我们使用 Object，因为我们不期望它们有其他操作。

还有一种使用 synchronised 关键字的方法，就是把整个方法声明为  synchronised 的，但它与使用 this 作为监视器对象的方法一样，所以并不推荐，也不应该在写代码时使用它，代码中只是作为演示，以防在维护旧代码时遇到。

## The volatile Keyword

这是另一种编写线程安全代码的方法，但它避免了同步所需要的开销，它解决了可见性问题，而不是竞争条件。所以它不会阻止两个线程同时修改同一个数据，它确保了如果一个线程修改了数据，其他线程也能够看到这些变化。

ThreadDemo 修改如下：

```java
public class ThreadDemo {
    public static void show() {
        var status = new DownloadStatus();
        var thread1 = new Thread(new DownloadFileTask(status));
        var thread2 = new Thread(() -> {
            while (!status.isDone()) {}
            System.out.println(status.getTotalBytes());
        });

        thread1.start();
        thread2.start();
    }
}
```

第二个线程中我们传入了一个 lambda 表达式用来表示一个 Runnable 对象，我们也可以创建一个单独的类来实现这个接口，但这里想做一些改变，从而可以看到更多形式的代码。

while 后面的花括号是空的，表示只是等待，什么也不做。

DownloadStatus 修改如下：

```java
public class DownloadStatus {
    private boolean isDone;
    private int totalBytes;
    private int totalFiles;
    private Object totalBytesLock = new Object();

    public void incrementTotalBytes() {
        synchronized (totalBytesLock) {
            totalBytes++;
        }
    }

    public synchronized void incrementTotalFiles() {
        totalFiles++;
    }

    public int getTotalFiles() {
        return totalFiles;
    }

    public int getTotalBytes() {
        return totalBytes;
    }

    public boolean isDone() {
        return isDone;
    }

    public void done() {
        isDone = true;
    }
}
```

DownloadFileTask 修改如下：

```java
public class DownloadFileTask implements Runnable {

    private DownloadStatus status;

    public DownloadFileTask(DownloadStatus status) {
        this.status = status;

    }

    @Override
    public void run() {
        System.out.println("Downloading a file: " + Thread.currentThread().getName());

        for (var i = 0; i < 1_000_000; i++) {
            if (Thread.currentThread().isInterrupted()) return;
            status.incrementTotalBytes();
        }

        status.done();

        System.out.println("Download completed: " + Thread.currentThread().getName());
    }
}
```

运行后发现第一个线程开始并结束，但第二个线程一直不结束，事实上它会一直持续下去，因为关于 isDone 字段的操作没有被上锁，如果给它也上了锁就可以得到理想效果。

但不论怎样，同步的开销太大。第二个线程一直不结束，因为它看不到 isDone 字段的变化。

因为 JVM 在底层实现了一些优化，使得代码运行速度更快，其中一种优化就是 caching values。

假设我们有一个值为 1 的整型字段，这个值存在主存或 RAM 中。而我们有两个线程，分别由不同的 CPU 来运行，每个 CPU 都有一个 Cache，它是一个 CPU 内部的小体量的本地内存，从 Cache 中读取数据比从主存或 RAM 中要快。CPU 将数据从外部读入 Cache 中，第一个线程将它所在的 CPU 中的 Cache 中的值改为了 2，而第二个线程所在的 CPU 中的 Cache 中的值不受影响。即是第一个 CPU 后来把数据写回主存或 RAM，第二个线程也只会看它的 CPU 内的 Cache 中的值，所以对第二个线程来说，这个值不曾发生改变。

为了避开同步来解决上述问题，我们可以把这个值声明为 volatile，这个词的意思是“不稳定”，所以这是在告诉 JVM 这个值是不稳定的，它可能会发生改变，所以不能依赖那个存储在 Cache 中的值，每次用它的时候都要从主存中读取。

声明方式如下：

```java
public class DownloadStatus {
    private volatile boolean isDone;
    private int totalBytes;
    private int totalFiles;
    private Object totalBytesLock = new Object();

    public void incrementTotalBytes() {
        synchronized (totalBytesLock) {
            totalBytes++;
        }
    }

    public synchronized void incrementTotalFiles() {
        totalFiles++;
    }

    public int getTotalFiles() {
        return totalFiles;
    }

    public int getTotalBytes() {
        return totalBytes;
    }

    public boolean isDone() {
        return isDone;
    }

    public void done() {
        isDone = true;
    }
}
```

## Thread Signalling

有时我们需要一个线程去等待另一个线程结束，上一节我们通过 while 循环来实现这一功能，问题是这么做会浪费 CPU 周期，因为这个 while 循环会不停地运行，直到它的条件变成 false。

我们可以通过 wait 和 notify 方法来解决这个问题。这两个方法是 Object 的方法，意味着所有的类都有这两个方法。

```java
public class ThreadDemo {
    public static void show() {
        var status = new DownloadStatus();
        var thread1 = new Thread(new DownloadFileTask(status));
        var thread2 = new Thread(() -> {
            while (!status.isDone()) {
                synchronized (status) {
                    try {
                        status.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
            System.out.println(status.getTotalBytes());
        });

        thread1.start();
        thread2.start();
    }
}
```

调用 wait 方法会使那个线程进入睡眠状态，直到另一个线程通知这个线程它在等待的那个值发生了改变。

调用 wait 方法后要调用 try/catch 模块。

JVM 希望我们在一个同步模块中使用 wait 方法。所以我们还需要 synchronized 模块来包裹上面的 try/catch 模块。这里我们使用 status 对象来作为监视器对象。

之后还要回到 DownloadFileTask 来添加 notify 方法。

notifyAll 方法在我们有多个线程等待被通知时使用。

同理 notify 方法也应该被包裹进 synchronized 模块中。

```java
public class DownloadFileTask implements Runnable {

    private DownloadStatus status;

    public DownloadFileTask(DownloadStatus status) {
        this.status = status;

    }

    @Override
    public void run() {
        System.out.println("Downloading a file: " + Thread.currentThread().getName());

        for (var i = 0; i < 1_000_000; i++) {
            if (Thread.currentThread().isInterrupted()) return;
            status.incrementTotalBytes();
        }

        status.done();
        
        synchronized (status) {
            status.notifyAll();
        }

        System.out.println("Download completed: " + Thread.currentThread().getName());
    }
}
```

运行上述代码就可以得到理想效果。

wait 和 notify 方法在复杂的问题中会变的很难理解，如果不合理使用，就会遇到很多奇奇怪怪的问题。所以尽力不要在代码中使用，这个问题还有更好的解决办法，下一节将会介绍。

## Atomic Objects

另一种实现线程安全的方法是使用原子类。

在 java.util.concurrent.atomic 包中有一堆原子类，比如 AtomicBoolean，AtomicInteger 等。通过这些原子类，我们可以进行原子操作。

之前说过，自增符号在底层包含三个操作，但如果通过原子类，就可以用原子操作来实现值的增减。

将 ThreadDemo 改回到 Race Conditions 一节中的版本。

```java
public class ThreadDemo {
    public static void show() {
        var status = new DownloadStatus();

        List<Thread> threads = new ArrayList<>();

        for (var i = 0; i < 10; i++) {
            var thread = new Thread(new DownloadFileTask(status));
            thread.start();
            threads.add(thread);
        }

        for (var thread : threads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println(status.getTotalBytes());
    }
}
```

将 DownloadFileTask 中的循环次数改回 10_000 次。

删除 DownloadStatus 中所有关于同步的代码（Lock 和 synchronized）。

在 DownloadStatus 中，将 totalBytes 改为 AtomicInteger 类型，对应的 getTotalBytes 和 incrementTotalBytes 方法也要改变。

```java
public class DownloadStatus {
    private boolean isDone;
    private AtomicInteger totalBytes = new AtomicInteger();
    private int totalFiles;

    public void incrementTotalBytes() {
        totalBytes.incrementAndGet();
    }

    public void incrementTotalFiles() {
        totalFiles++;
    }

    public int getTotalFiles() {
        return totalFiles;
    }

    public int getTotalBytes() {
        return totalBytes.get();
    }

    public boolean isDone() {
        return isDone;
    }

    public void done() {
        isDone = true;
    }
}
```

其中 incrementAndGet 方法是先增加后返回值（类似 ++a），getAndIncrement 方法是先返回值后增加（类似 a++）。本例中由于只是想增加这个值，并不介意返回的值是多少，所以两种方法都行。

运行上述代码，即可得到理想效果。

原子类的工作原理本质上是使用了比较（compare）与交换（swap）的技术，这种技术被大多数 CPU 支持。

当我们调用 incrementAndGet 方法的时候，这个原子类就会比较当前值与期望值，如果他们不一样，就进行交换。

例如当前值为 0，因为想增加它，所以期望值是 1，因为这两个值不一样，就会被交换。

综上，如果我们处理的是一个专门用来计数的变量，与同步相比，应该优先使用原子类型，因为它们更快，且更容易使用。

## Adders

上一节讲了用原子类来实现计数器，但如果我们有多个线程来频繁更新一个值，最好使用 Java 中的 Adder 类，它们比原子类更快。

Adder 类有 LongAdder 和 DoubleAdder 两种。

我们把上一节中的原子类去掉，常使用 Adder 来解决问题。

将 totalBytes 改为 LongAdder 类型，对应的 getTotalBytes 和 incrementTotalBytes 方法也要改变。

```java
public class DownloadStatus {
    private boolean isDone;
    private LongAdder totalBytes = new LongAdder();
    private int totalFiles;

    public void incrementTotalBytes() {
        totalBytes.increment();
    }

    public void incrementTotalFiles() {
        totalFiles++;
    }

    public int getTotalFiles() {
        return totalFiles;
    }

    public int getTotalBytes() {
        return totalBytes.intValue();
    }

    public boolean isDone() {
        return isDone;
    }

    public void done() {
        isDone = true;
    }
}
```

在内部，这个 LongAdder 对象有一个计数器区域，这个区域会根据需要进行扩充，这个区域中有一堆区域单元，每一个单元都有一个计数器，每当有线程想修改这个对象的值，它只需要修改这些计数器中的一个就行，所以不同的线程可以同时修改这个对象的值。这也是为何这个类比原子类型要快，因为它允许更多的吞吐量。

在我们调用 intValue 方法的时候，在内部，这个方法会调用 sum 方法，把所有计数器中的值求和，然后返回一个 int 类型的值。

这个对象的运算方法有 add，decrement，doubleValue，reset 等，我们调用的是 increment。

执行上述代码就可以得到理想效果。

如果我们有多个线程来频繁更新一个值，最好使用 Java 中的 Adder 类，它们比原子类更快。

## Synchronized Collections

有时我们需要在多个线程中共享一个集合。

```java
public class ThreadDemo {
    public static void show() {
        Collection<Integer> collection = new ArrayList<>();

        var thread1 = new Thread(() -> {
            collection.addAll(Arrays.asList(1, 2, 3));
        });

        var thread2 = new Thread(() -> {
            collection.addAll(Arrays.asList(4, 5, 6));
        });

        thread1.start();
        thread2.start();

        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(collection);
    }
}
```

运行上述代码，得到结果如下：

> [4, 5, 6]

这是因为竞争条件的存在。

为了解决这个问题，可以使用同步。

```java
public class ThreadDemo {
    public static void show() {
        Collection<Integer> collection = Collections.synchronizedCollection(new ArrayList<>());

        var thread1 = new Thread(() -> {
            collection.addAll(Arrays.asList(1, 2, 3));
        });

        var thread2 = new Thread(() -> {
            collection.addAll(Arrays.asList(4, 5, 6));
        });

        thread1.start();
        thread2.start();

        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(collection);
    }
}
```

在内部，我们的数据被存储在 ArrayList 中，但在我们调用 Collections.synchronizedCollection 方法后，这个数组会把这个 ArrayList 对象包裹在一个同步集合中。在这个同步集合中，所有的方法，比如 add，remove 等，都是同步的。所以这个方法作用于一个普通的集合后，就能让这个集合变成同步的集合。

运行上述代码就可以得到理想效果。

## Concurrent Collections

同步集合是用锁来实现线程安全的，它在性能和可扩展性方面，随着线程数量和并发操作次数的增加，会出现较大的缺陷。

在这种情况下，我们可以使用 Java 中的并发集合。

这种集合通过分区来实现并发，它们把数据划分为多个段，每个段每次只能被一个线程访问，但整个数据可以同时被多个线程访问，只要它们访问的不是同一个段。

并发集合比同步集合的效率更高，因为它的吞吐量更大。

这些类在 java.util.concurrent 包中声明。

例子有 ConcurrentHashMap，ConcurrentLinkedDeque，ConcurrentMap 等等。

```java
public class ThreadDemo {
    public static void show() {
        Map<Integer, String> map = new HashMap<>();
        map.put(1, "a");
        map.get(1);
        map.remove(1);
    }
}
```

上面代码创建的是一般的 HashMap，要想变成线程安全的并发 HashMap，只需要如下修改：

```java
public class ThreadDemo {
    public static void show() {
        Map<Integer, String> map = new ConcurrentHashMap<>();
        map.put(1, "a");
        map.get(1);
        map.remove(1);
    }
}
```

Map 是一个接口，HashMap 和 ConcurrentHashMap 是它的两种实现。

通过面向接口编程，我们减少了代码变动带来的影响。

## Summary

多线程允许我们用更少的时间做更多的事情。

但构建多线程要面临两个问题：竞争条件和可见性问题（由 Cache 导致）。

解决上述问题的方法有：

1. 约束法：不在线程间共享数据
2. 同步：包含多种实现：
   1. Lock
   2. synchronised 关键字
3. 原子类：利用比较和交换技术
4. 分割：将共享资源分段





