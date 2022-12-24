# Interfaces

## What are interfaces ?

We use interfaces to build loosely coupled, extensible and testable applications.

我们使用接口构造松散耦合的，可扩展，可测试的应用程序。

假设 A 是子类，B 是父类，通过 private 关键字来修饰某些变量可以减少 A 与 B 之间的关联，但这不足够，接口可以将 A 与 B 完全解偶。

接口和类很类似，但他只包含方法的声明，没有实现。

接口定义了需要做什么。

类定义了如何去做。

## Tightly coupled code

假设我们创建以下两个类：

1. TaxCalculator

   ```java
   public class TaxCalculator {
       private double taxableIncome;
   
       public TaxCalculator(double taxableIncome) {
           this.taxableIncome = taxableIncome;
       }
   
       public double calculateTax() {
           return taxableIncome * 0.3;
       }
   }
   ```

   

2. TaxReport

   ```java
   public class TaxReport {
       private TaxCalculator calculator;
   
       public TaxReport() {
           calculator = new TaxCalculator(100_000);
       }
   
       public void show() {
           var tax = calculator.calculateTax();
           System.out.println(tax);
       }
   }
   ```

其中，TaxReport 调用了 TaxCalculator，如果我们修改 TaxCalculator，TaxReport 就会报错。

## Creating an interface

1. 在左侧的项目面板中右击对应的包
2. 选择 New
3. 选择 Java Class
4. 在 kind 栏选择 Interface
5. 输入名称
6. OK

创建好的接口如下：

```java
public interface TaxCalculator {
}
```

没有状态、数据等。

在这里只进行方法的声明，如下：

```java
public interface TaxCalculator {
    public double calculateTax();
}
```

其中，public 在 IDEA 中是灰色的，表示它是多余的，因为这里声明的每一个方法都需要被其它的类来实现，因此它们需要被其他类来访问，所以这些方法必须是 public 的。因此这里的 public 应该被移除。

通过接口来实现的类如下：

```java
public class TaxCalculator2018 implements TaxCalculator {
    private double taxableIncome;

    public TaxCalculator2018(double taxableIncome) {
        this.taxableIncome = taxableIncome;
    }

    @Override
    public double calculateTax() {
        return taxableIncome * 0.3;
    }
}
```

## Dependency Injection

根据依赖注册原则，上述的 TaxReport 类不应该考虑“计算器”这一对象的实现，只需要考虑去使用它。

分离实现和使用的过程叫做 separation of concerns。

因此我们将把“创建一个计算器实例”这一任务从 TaxReport 类中移到另一个类中。这一过程叫做 Dependency Injection。

Dependency Injection 包括 3 类：

1. Constructor Injection
2. Setter Injection
3. Method Injection

### Constructor Injection

将原来的 TaxReport 类改为如下：

```java
public class TaxReport {
    private TaxCalculator calculator;

    public TaxReport(TaxCalculator calculator) {
        this.calculator = calculator;
    }

    public void show() {
        var tax = calculator.calculateTax();
        System.out.println(tax);
    }
}
```

通过 constructor 来实现独立性。

之后在主函数中通过以下方法来实现两个类的连接。

```java
public class Main {

    public static void main(String[] args) {
        var calculator = new TaxCalculator2018(100_000);
        var report = new TaxReport(calculator);
    }
}
```

### Setter Injection

在原来的 TaxReport 类为 calculator 构建 setter。

```java
public class TaxReport {
    private TaxCalculator calculator;

    public TaxReport(TaxCalculator calculator) {
        this.calculator = calculator;
    }

    public void show() {
        var tax = calculator.calculateTax();
        System.out.println(tax);
    }

    public void setCalculator(TaxCalculator calculator) {
        this.calculator = calculator;
    }
}
```

它的优点是可以随时更改计算器。

```java
public class Main {

    public static void main(String[] args) {
        var calculator = new TaxCalculator2018(100_000);
        var report = new TaxReport(calculator);
        report. show();

        report.setCalculator(new TaxCalculator2019());
        report.show();
    }
}
```

#### 技巧

在为一个接口创建对应的类时，在声明结束后（下面代码的第一行），将鼠标移到类名上，按 Option + Enter，选择 Implement methods，可以快速实现接口中的方法。

```java
public class TaxCalculator2019 implements TaxCalculator{
    @Override
    public double calculateTax() {
        return 0;
    }
}
```

### Method Injection

直接在 TaxReport 类的 show 方法中传入计算器。

```java
public class TaxReport {
//    private TaxCalculator calculator;

//    public TaxReport(TaxCalculator calculator) {
//        this.calculator = calculator;
//    }

    public void show(TaxCalculator calculator) {
        var tax = calculator.calculateTax();
        System.out.println(tax);
    }

//    public void setCalculator(TaxCalculator calculator) {
//        this.calculator = calculator;
//    }
}
```

对应的主函数如下：

```java
public class Main {

    public static void main(String[] args) {
        var calculator = new TaxCalculator2018(100_000);
        var report = new TaxReport();
        report.show(calculator);
        report.show(new TaxCalculator2019());
    }
}
```

### 总结

我们常用 Constructor Injection。

## Interface Segregation Principle 接口分离原理

原理：应该把比较大的接口划分为多个小的接口，这样来最小化改变带来的影响

假设有如下接口：

```java
public interface UIWidget {
    void drag();
    void resize();
    void render();
}
```

对应的类有：

```java
public class Dragger {
    public void drag(UIWidget widget) {
        widget.drag();
        System.out.println("Dragging done!");
    }
}
```

此时如果修改 UIWidget，比如我们希望在 resize 方法中添加参数 size，就可能会导致其他的相关类报错。在本例中并不会报错，因为 Dragger 类只用到了 drag 方法。但对于一个大的接口，这并不能保证。

因此应该让每个接口都专注于一个功能。

将 drag 方法独立出来：

```java
public interface Draggable {
    void drag();
}
```

让 UIWidget 继承 Draggable：

```java
public interface UIWidget extends Draggable {
    void resize(int size);
    void render();
}
```

### 快捷方式

#### 对 method

1. 将鼠标停留在要提取的方法上，如 resize
2. 按 control + T 调出重构面板
3. 选择 Extract Interface
4. 设定新接口的名字
5. 选择要移动的方法

#### 对 class

1. 右击类名
2. Refactor
3. Extract Interface
4. 选择 Rename original class and use interface where possible，以此来使提取的接口使用本来的类的名字，只需要重命名类
5. 在 Member to form interface 中选择要提取的 method
6. 点击 Refactor

### 注意

本次可的目的不是说每个接口里都只能有一个方法，而是应该专注于一个功能，比如：

```java
public interface Resizable {
    void resize(int size);
    void resize(int x, int y);
    void resizeTo(UIWidget widget);
}
```

一个类不能有多个父类，一个接口可以有多个父接口

## 练习题 MyTube

## Interface 中的不好的特性

接下来要介绍的特性被 Mosh 批评为不好的特性。

### Fields in Interfaces

```java
public interface TaxCalculator {
    float minimumTax = 100;
    double calculateTax();
}
```

在接口中声明的字段是 public，static，final 的字段，这意味着之后不能改变。

这个特性的目的是为了防止 magic numbers，指的是那些不知道从哪里突然冒出来的数字。这能使得我们的代码更清晰并易于维护。

但要注意以下几点：

1. 要注意这个值是否在所有的 implementation 中都相等
2. 即使在所有的实现中都相等，它也应该只是实现的细节

所以，不要在接口中声明字段。

### Static Methods in Interfaces

```java
public interface TaxCalculator {
    double calculateTax();
    static double getTaxableIncome(double income, double expenses) {
        return income - expenses;
    }
}
```

在此注意，接口定义的是 what，不是 how，因此这个特性不是好特性。

如果想在某个接口下的所有实现中 重复使用某个逻辑，应该定义一个抽象类，并在这个抽象类中实现这个逻辑，这样它的子类都能共享这个逻辑。

在转移到抽象类后，这个逻辑的 static 修饰符应该被改为 protected 修饰符，因为我们希望只有这个抽象类的具体实现可以使用这个方法。

```java
public abstract class AbstractTaxCalculator implements TaxCalculator{
    protected double getTaxableIncome(double income, double expenses) {
        return income - expenses;
    }
}
```

此时因该把原来作为接口的实现的类转为作为这个虚拟类的实现的类，使用 extends 而不是 implements。

```java
public class TaxCalculator2018 extends AbstractTaxCalculator {
    private double taxableIncome;

    public TaxCalculator2018(double taxableIncome) {
        this.taxableIncome = taxableIncome;
    }

    @Override
    public double calculateTax() {
        return taxableIncome * 0.3;
    }
}
```

### Private Method

这一特性是为了服务上一个特性 Static Method 的，是为了提取出 Static Method 中的重复部分，减少冗余，但由于 Static Method 就不应该存在，所以这一特性也不应该存在。

## Interfaces 与 Abstract Method

### Interfaces

是一个“合同”，用来规定需要做什么，不在乎怎么做。

存在的意义是，使得代码低耦合，可扩展，可测试。

### Abstract Method

是部分实现的类，用于在一些类中分享代码。

### 总结

由于上一节中提到的不好的特性，接口正在被滥用，模糊了接口和类的边界，这给了黑客可乘之机。

因为在 Java 中一个类只能继承一个父类，但可以继承多个接口。

所以如果在接口中实现方法，就会导致很多麻烦。

所以应该明确区分接口和类，这样就可以把改变带来的影响最小化，而不是在改变某一点后就需要对原有的代码进行大量的更改或重写。

从而可以构建 losely coupled, extensible, testable, applications.

## 何时去使用接口

应该在你希望将类与它的依赖项解偶的时候使用。

You should use interfaces in situations where you want to decouple a class from its dependences.

这带来的好处有：

1. Easily swap one implementation with another.

   比如我们有一个可以用来把视频编码的代码，将来我们可能会想用一个更快的代码来替代这个代码。此时我们就可以通过一个接口来将他们解偶，从而使得算法的改变对程序的影响最小或为 0。

   接口相当于一个转换器，也是一种协议。

2. Easily extend your applications again with minimal impact.

   比如我们希望设计一个可以供他人使用的框架，这时接口将很有帮助。

   比如我们要构建一个用于构建 WEB 应用程序的框架，这个框架应该有一个模版引擎（Template Engine），它能用于解析一些 HTML 模版。我们已经有了自己的模版引擎，但我们希望别人能继续建造他们自己的模版引擎。因此，与其构建一个完整的 implementations，不如面对接口编程，让别人可以实现我们的接口，这样他们就可以在我们的要求的基础上实现他们自己的功能，增加框架的可扩展点。

3. Test your classes in isolation.

   如果一个类使用电子邮件服务或存储引擎，我们可以通过接口将其解偶然后单独测试这个类，即单元测试。

   但如果我们并没有进行单元测试的需求，则使用接口并不能给我们带来收益，所以不如不用。

   
