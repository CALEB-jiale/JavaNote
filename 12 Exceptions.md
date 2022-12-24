# Exceptions

## 什么是异常

首先，添加一个包

1. 右击 src 下的子文件，即代码文件所在的文件夹
2. 选择 New，Package
3. 输入包名

在这个包下创建新的类 ExceptionsDemo

```java
public class ExceptionsDemo {
    public static void show() {
        sayHello(null);
    }

    public static void sayHello(String name) {
        System.out.println(name.toUpperCase());
    }
}
```

在主程序中调用

```java
public class Main {

    public static void main(String[] args) {
        ExceptionsDemo.show();
    }
}
```

运行后将会报错

> Exception in thread "main" java.lang.NullPointerException: Cannot invoke "String.toUpperCase()" because "name" is null
> 	at com.company.exceptions.ExceptionsDemo.sayHello(ExceptionsDemo.java:11)
> 	at com.company.exceptions.ExceptionsDemo.show(ExceptionsDemo.java:7)
> 	at com.company.Main.main(Main.java:8)

从以上信息可以读出异常的类型以及发生的地点

## 不同类型的异常

Java 中的异常分为 3 类：

1. checked exceptions：开发者提前预测到会发生的错误，在编译时会背编译器提醒
2. unchecked exceptions (runtime exceptions)：不会在编译时被编译器检查到的异常
   1. NullPointerException 指针异常
   2. ArithmeticException 算数异常
   3. IllegalArgumentException 参数异常
   4. IndexOutOfBoundException 索引异常
   5. IllegalStateException 状态异常
3. errors： 程序以外的错误，如栈溢出或内存溢出

### Exceptions Hierarchy 异常等级

最高级：Throwable

次级：Exception (checked 与 unchecked) 和 Error

最低级（Exception 的子集）：RuntimeException (unchecked)

##  处理异常的不同技术

### Catching Exceptions

构建以下程序

```java
public class ExceptionsDemo {
    public static void show() {
        try {
            var reader = new FileReader("file.txt");
            System.out.println("File opened");
        } catch (FileNotFoundException ex) {
            System.out.println("File does not exist.");
            System.out.println(ex.getMessage());
        }
    }
}
```

在主函数中调用并运行，将会得到以下结果：

> File does not exist.
> file.txt (No such file or directory)

当 try 模块中的某一行抛出异常后，将直接跳转到 catch 模块，执行 catch 中的语句。

#### 快捷方式

```java
public class ExceptionsDemo {
    public static void show() {
        var reader = new FileReader("file.txt");
    }
}
```

1. 在上面代码中鼠标点上 FileReader
2. 按 option + enter
3. 选择 surround with try/catch

之后获得如下代码

```java
public class ExceptionsDemo {
    public static void show() {
        try {
            var reader = new FileReader("file.txt");
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

### Catching Multiple Types of Exceptions

 有时需要捕捉多种不同类型的异常

```java
public class ExceptionsDemo {
    public static void show() {
        try {
            var reader = new FileReader("file.txt");
            var value = reader.read();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

如上所示，需要在 catch 模块后面添加 catch 模块，不同的模块对应不同的异常类型。

如果把上述的两个 catch 模块位置互换，会报错。因为 `FileNotFoundException` 继承了 `IOException`，根据多态性，`FileNotFoundException` 只是 `IOException` 的多种形式的其中一种，因此不需重复声明。

但是，通过上述代码，虽有部分冗余，但可以更准确地描述异常的类型。

另一种捕捉多种不同类型的异常的方式如下

```java
public class ExceptionsDemo {
    public static void show() {
        try {
            var reader = new FileReader("file.txt");
            var value = reader.read();
            new SimpleDateFormat().parse("");
        } catch (IOException | ParseException e) {
            e.printStackTrace();
        }
    }
}
```

利用 `|` 将不同的异常类型合并在一个 catch 模块中。

### Final Block

对于以下代码

```java
public class ExceptionsDemo {
    public static void show() {
        try {
            var reader = new FileReader("file.txt");
            var value = reader.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

让我们假设 `file.txt` 存在，也就是说我们可以成功打开文件，但在读取数据时发生了错误（比如硬件故障）。这样我们会出现一个问题：我们已经打开这个文件进行读取，但我们没有关闭它。由于文件是系统资源，因此当我们利用完以后总要及时释放，否则其它进程将无法访问这些资源。

因此上述代码应添加关闭文件的命令 `reader.close();` 。但是，由于上述代码在读取数值失败后就会直接跳转到 catch 模块， 所以把关闭文件的命令添加到 try 模块中无法起作用，所以需要 final 模块来释放资源。

修改后的代码如下

```java
public class ExceptionsDemo {
    public static void show() {
        FileReader reader = null;
        
        try {
            reader = new FileReader("file.txt");
            var value = reader.read();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

由于之前 `reader` 是 try 模块中的变量，不能在 try 模块外被识别，因此要提前在 try 模块之前声明变量。

并且，因为 `reader.close();` 语句也可能发生 `IOException`，因此也需要 try/catch 模块。

这是比较蠢笨的解决方法，后续会介绍更好的方法。

这里要学习的是，finally 模块总会被执行，无论是否有异常发生。

### The try-with-resources Statement

这是一种更好的解决资源释放问题的方法。

删除上一节代码中的 finally 模块，在 try 的空格后，花括号前添加一对括号，在括号内创建并初始化外部资源。

```java
public class ExceptionsDemo {
    public static void show() {

        try (var reader = new FileReader("file.txt")) {
            var value = reader.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

上述代码可以自动实现 finally 模块的功能。

但要上述的代码能成功运行，需要 FileReader 这个类实现 AutoCloseable 接口。

```java
public class ExceptionsDemo {
    public static void show() {

        try (
                var reader = new FileReader("file.txt");
                var writer = new FileWriter("file.txt")
        ) {
            var value = reader.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

在同时声明多个外部资源的时候，括号的排布应该遵循上例的格式。

### Throwing Exceptions

我们在新加一个类，命名为 Account，具体内容如下：

```java
public class Account {
    public void deposit(float value) {
        if (value <= 0)
            throw new IllegalArgumentException();
    }
}
```

此时如果调用 `deposit` 方法并传入一个负数，则程序会崩溃，并抛出 `IllegalArgumentException`，这是一个运行时异常（unchecked exception），不应该用 try/catch 模块来处理这个异常。

这种编程叫做防御性编程，因为当在现异常后，这种方式可以阻止后续代码的运行。

如果想抛出一个受控异常（checked exception），则会用到 throws 关键字，方法如下：

```java
public class Account {
    public void deposit(float value) throws IOException {
        if (value <= 0)
            throw new IOException();
    }
}
```

这是在告诉所有调用该方法的 “caller” 说，这个方法可能会抛出这个异常。

这是这个方法的 API 的一部分，API 是 Application Programming Interface 的缩写。

之后调用该方法的部分都应该处于一个 try/catch 模块之中。

```java
public class ExceptionsDemo {
    public static void show() {
        var account = new Account();
        try {
            account.deposit(-1);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

或者在调用该方法的方法上声明抛出异常。

```java
public class ExceptionsDemo {
    public static void show() throws IOException {
        var account = new Account();
        account.deposit(-1);
    }
}
```

上述两个操作都可以通过 option + enter 快速实现。

综上所述，对于一个异常，我们有两种处理方法，要么用 try/catch 模块来处理，要么在可能出现该异常的的方法中声明，然后让该方法的 caller 通过 try/catch 模块来处理。

### Re-throwing Exceptions

有时我们在捕捉到异常后希望进行其他操作，这时只需在上述代码的 catch 模块中进行即可，但我们仍需要告诉用户发生错误了，于是需要重新抛出捕捉到的异常。

```java
public class ExceptionsDemo {
    public static void show() throws IOException {
        var account = new Account();
        try {
            account.deposit(-1);
        } catch (IOException e) {
            System.out.println("Logging");
            throw e;
        }
    }
}
```

直接加上 `throw e;` 后，会报错，提醒我们 `Unhandled exception` ，但我们并不想在这里使用 try/catch 模块，因为当我们在这里出现异常时，我们希望这个函数的 caller 来处理异常，因此为了解决这个问题，我们在这个方法的声明中利用关键字 throws 来声明这个异常，如上一节所述。

在 main 函数中，我们利用 try/catch 模块来处理抛出的异常，利用快捷键得到以下结果：

```java
public class Main {
    public static void main(String[] args) {
        try {
            ExceptionsDemo.show();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

为了处理多种异常，我们把上面的结果一般化处理：

```java
public class Main {
    public static void main(String[] args) {
        try {
            ExceptionsDemo.show();
        } catch (Throwable e) {
            System.out.println("An unexpected error occurred.");
        }
    }
}
```

### Custom Exceptions 自定义异常

自定义异常可以帮助用户和其他开发人员更好地理解出现的问题。

举例说明，让我们在 Account 类中新加一个取款方法 withdraw。

如果用户余额不足，取款时应该有对应的异常提醒。

对于自定义异常，我们需要决定它是 checked 还是 unchecked。

如果是 Checked，则应该 extend Exception。

如果是 Unchecked，则应该 extend RuntimeException。

在本例中，我们希望它是 checked，因为这是我们应该预想到的并能修复的情况。

构建的异常如下：

```java
// Checked -> Exception
// Unchecked (runtime) -> RuntimeException

public class InsufficientFundsException extends Exception {
    public InsufficientFundsException() {
        super("Insufficient funds in your account.");
    }

    public InsufficientFundsException(String message) {
        super(message);
    }
}
```

此时的 Account 改为：

```java
public class Account {
    private float balance;

    public void deposit(float value) throws IOException {
        if (value <= 0)
            throw new IOException();
    }

    public void withdraw(float value) throws InsufficientFundsException {
        if (value > balance) {
            throw new InsufficientFundsException();
        }
    }
}
```

对应的 ExceptionsDemo 改为：

```java
public class ExceptionsDemo {
    public static void show(){
        var account = new Account();
        try {
            account.withdraw(10);
        } catch (InsufficientFundsException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

main 函数维持上一节的不变，运行后可看到报错结果。

## Chaining Exceptions

当抛出异常时，我们可以使用一种技术叫做 Chaining Exceptions。

这意味着把一个异常包裹进一个更一般化的异常中去。

例如在上一节中我们抛出了 InsufficientFundsException，但假设取款还可能因为其他原因导致失败，这是我们可以抛出不同类型的异常，比如说账户异常。

然后我们可以把  InsufficientFundsException 包裹进账户异常内，从而可以得到一个更加通用的异常。然后可以指出是什么具体的异常导致了这个一般化的异常。

### 实例操作

首先先创建一个新类：

```java
public class AccountException extends Exception {
}
```

然后去 Account 类中作如下更改：

```java
public class Account {
    private float balance;

    public void deposit(float value) throws IOException {
        if (value <= 0)
            throw new IOException();
    }

    public void withdraw(float value) throws AccountException {
        if (value > balance) {
            var fundsException = new InsufficientFundsException();
            var accountException = new AccountException();
            accountException.initCause(fundsException);
            throw accountException;
        }
    }
}
```

其中 AccountException 会有两个，一个是 java 自带的，一个是我们刚才创建的，这里选择我们自己创建的。

还有一种更简单的方法来实现上述结果。

去 AccountException 那里创建一个 constructor。

```java
public class AccountException extends Exception {
    public AccountException(Exception cause) {
        super(cause);
    }
}
```

再回到 Account 类中：

```java
public class Account {
    private float balance;

    public void deposit(float value) throws IOException {
        if (value <= 0)
            throw new IOException();
    }

    public void withdraw(float value) throws AccountException {
        if (value > balance) {
            throw  new AccountException(new InsufficientFundsException());
        }
    }
}
```

再对 ExceptionsDemo 作出如下更改：

```java
public class ExceptionsDemo {
    public static void show(){
        var account = new Account();
        try {
            account.withdraw(10);
        } catch (AccountException e) {
            e.printStackTrace();
        }
    }
}
```

运行即可得到报错以及报错的原因，但看不到 “Insufficient funds in your account.” 信息。

Exception 对象不只有 `initCause` 方法，还有 ` getCause` 方法，这个方法可以返回一个 Throwable 对象。

因此还可以有如下操作：

```java
public class ExceptionsDemo {
    public static void show(){
        var account = new Account();
        try {
            account.withdraw(10);
        } catch (AccountException e) {
            var cause = e.getCause();
            System.out.println(cause.getMessage());
        }
    }
}
```

这个程序运行后会出现 “Insufficient funds in your account.” 信息。
