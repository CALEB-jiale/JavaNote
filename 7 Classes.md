# Classes

[toc]

## Creating Classes

1. 打开左侧的 Project 面板
2. 右击 main class 的商机文件夹，选择 New -> Java Class
3. 命名时每个单词的首字母大写

```java
public class TextBox {
    public String text = "";

    public void setText(String text) {
        this.text = text;
    }

    public void clear() {
        text = "";
    }
}
```

## Creating Objects

```java
public class Main {

    public static void main(String[] args) {
//        TextBox textBox1 = new TextBox();
        var textBox1 = new TextBox();
        textBox1.setText("Box 1");
        System.out.println(textBox1.text);
    }
}
```

## Memory Allocation

Java 管理着两个不同的内存区域：堆和栈。

堆用于存放 Objects。

栈用于存放短期变量，包括对 Objects 的引用。

<img src="/Users/lucas/Documents/School/计算机基础/Java ☕️/JavaNote/7.1 堆和栈的关系.png" alt="7.1 堆和栈的关系" style="zoom:50%;" />

## Encapsulation 封装

定义：将数据和与数据相关的操作绑定在一个单位中。

### 创建 setter 和 getter

1. control + enter
2. 创建 setter 和 getter

## Abstraction 抽象

定义：通过隐藏不必要的细节来减少复杂性

## Coupling 耦合

定义：Classes 之间的依赖程度。

编程时应尽力减少不同 class 之间的耦合。

### 减少耦合

```java
public class Browser {
    public void navigate(String address) {
        String ip = findIpAddress(address);
        String html = sendHttpRequest(ip);
        System.out.println(html);
    }

    public String sendHttpRequest(String ip) {
        return "<html></html>";
    }

    public String findIpAddress(String address) {
        return "127.0.0.1";
    }
}
```

在本例中，`sendHttpRequest` 和 `findIpAddress` 两个方法属于不必要的细节，应当改为 `private` 型方法。

## Constructors

```java
public class Employee {
    private int baseSalary;
    private int hourlyRate;

    public static int numberOfEmployees;

    public Employee(int baseSalary) {
        this(baseSalary, 0);
    }

    public Employee(int baseSalary, int hourlyRate) {
        setBaseSalary(baseSalary);
        setHourlyRate(hourlyRate);
        numberOfEmployees++;
    }

    private int getHourlyRate() {
        return hourlyRate;
    }

    private void setHourlyRate(int hourlyRate) {
        if (hourlyRate < 0)
            throw new IllegalArgumentException("Hourly rate can't be less than 0.");
        this.hourlyRate = hourlyRate;
    }

    private int getBaseSalary() {
        return baseSalary;
    }

    private void setBaseSalary(int baseSalary) {
        if (baseSalary <= 0)
            throw new IllegalArgumentException("Salary can't be 0 or less.");
        this.baseSalary = baseSalary;
    }

    public int calculateWage(int extraHours) {
        return baseSalary + (hourlyRate * extraHours);
    }

    public int calculateWage() {
        return calculateWage(0);
    }
  
  	public void printNumberOfEmployees() {
        System.out.println(numberOfEmployees);
    }
}
```



## Method Overloading

定义：通过不同的方式创建 class。

注意：不要过度使用重载！

 ```java
 public int calculateWage(int extraHours) {
 		return baseSalary + (hourlyRate * extraHours);
 }
 
 public int calculateWage() {
 		return calculateWage(0);
 }
 ```

## Constructor Overloading

```java
public Employee(int baseSalary) {
		this(baseSalary, 0);
}

public Employee(int baseSalary, int hourlyRate) {
		setBaseSalary(baseSalary);
		setHourlyRate(hourlyRate);
  	numberOfEmployees++;
}
```

## Static Members

在 object rented 编程中，一个 class 可以拥有两种类型的 members，包括 instance members 和 static members。

Instance members 属于实例 intances 或者对象 objects。

Static members 或者叫 class members，属于 class，不需要创建具体的对象也可以调用这些数据或方法，例如 Employee 类中的 `numberOfEmployees` 和 `printNumberOfEmployees()`。



## 技巧 1：快速创建 method

在已被调用但未创建的 method 上点击“红灯泡”，选择创建 method。

快捷键：option + enter

## 技巧 2：快速创建 object

直接输入`new class 名称` ，然后点击“黄灯泡”，选择创建局部变量。

快捷键：option + enter

再次重复上述步骤，可选择将变量名前的类名改为 var。



























