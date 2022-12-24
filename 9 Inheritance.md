# Inheritance

## Object Class

## Constructors

创建方式：从上方菜单栏找到 Code，然后选中 Generate，快捷键 command + N

在一个“类”中代码的排序为：

1. Fields，即各种参数
2. Constructors
3. All the public methods

如果父级类的 constructor 有参数，则子级类在构建时要用到 super 函数来传递所需要的参数。 

## Access Modifiers

<span style='background:green'>Public members 在类之外也是可以使用的。</span>

<span style='background:green'>Private members 只能在类内调用，不能被继承。</span>

Protected members 是在同一个包内可用，但可以在不同包的子类中使用。过于复杂，不常用，也应避免使用。

如果没有声明，则是 package private，则只能在同一个包内使用，即使是不同包的子类也不能继承。过于复杂，不常用，也应避免使用。

## Overriding Methods

在子类中重写父类的方法。

注意与重载（overloading）区分，重载指的是通过不同的 signature 来实现同一个 method。

在重写的方法前要进行备注，备注方法如下

```java
@Override
```

## Upcasting and Downcasting

Upcasting : 向上转型，转成父类

Downcasting : 向下转型，转成子类

举例：当一个函数需要一个父类作为参数，但传入的是一个子类时，子类会被自动转型为父类，但是在函数内的操作只能有父类包含的操作，没有子类特有的操作。如果要在函数中进行子类的操作，需要再将函数内的父类向下转型为子类。但这样做的风险是，如果传入的参数是父类，函数就会报错。

如下，若把  ``show(control)`` 的注释去掉，则会报错。因为每一个 ``TextBox`` 都是 ``UIControl`` 但不是每一个 ``UIControl`` 都是 ``TextBox``。

```java
public class Main {

    public static void main(String[] args) {
        var control = new UIControl(true);
        var textBox = new TextBox();
//        show(control);
        show(textBox);
    }

    public static void show(UIControl control) {
        System.out.println(control);
        var textBox = (TextBox)control;
        textBox.setText("Hello");
        System.out.println(control);
    }
}
```

更改后的如下：

```java
public class Main {

    public static void main(String[] args) {
        var control = new UIControl(true);
        var textBox = new TextBox();
        show(control);
        show(textBox);
    }

    public static void show(UIControl control) {
        if(control instanceof TextBox) {
            var textBox = (TextBox)control;
            textBox.setText("Hello");
        }
        System.out.println(control);
    }
}
```

其中 ``UIControl`` 为：

```java
public class UIControl {
    private boolean isEnabled = true;

    public UIControl(boolean isEnabled) {
        this.isEnabled = isEnabled;
    }

    public void enable() {
        isEnabled = true;
    }

    public void disable() {
        isEnabled = false;
    }

    public boolean isEnabled() {
        return isEnabled;
    }
}
```

其中 ``TextBox`` 为：

```java
public class TextBox extends UIControl {
    private String text = "";

    public TextBox() {
        super(true);
    }

    @Override
    public String toString() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }

    public void clear() {
        text = "";
    }
}
```

## Comparing Object

```java
public class Main {

    public static void main(String[] args) {
        var point1 = new Point(1, 2);
        var point2 = new Point(1, 2);
        System.out.println(point1 == point2);
        System.out.println(point1.equals(point2));
        System.out.println(point1.hashCode());
        System.out.println(point2.hashCode());
    }
}
```

上面的代码会依次输出“false true 994 994”。但若不重写 ``equals`` 函数，第二个也会是 ``false``。

要重写 ``equals`` 就要同时重写 ``hashCode``。

```java
public class Point {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;

        if (!(obj instanceof Point))
            return false;

        var otherPoint = (Point)obj;
        return otherPoint.x == x && otherPoint.y == y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }
}
```

简单的重写方式为：

1. command + N 打开生成面板
2. 选择 ``equals() and hashCode()`` 
3. 直接点击 ``Next``
4. 选择用于比较的字段
5. 选择用于计算哈希值的字段
6. 结束

## Polimorphism 多态性

解释：允许一个对象有不同的形式

举例：

创建一个新的类：Check Box 并继承 UIControl。

```java
public class CheckBox extends UIControl{
    public CheckBox() {
        super(true);
    }
}
```

此时，在主类中，我们有：

```java
public class Main {

    public static void main(String[] args) {
        UIControl[] controls = { new TextBox(), new CheckBox() };
    }
}
```

如果我们想 render（渲染）controls 中的对象，就要用 if 语句来判断对象的类型，这是很麻烦的。因此我们做出如下调整：

1. 在 UIControl 中构造虚拟方法 render：

   ```java
   public class UIControl {
       private boolean isEnabled = true;
   
       public void render() {
       }
   
       public UIControl(boolean isEnabled) {
           this.isEnabled = isEnabled;
       }
   
       public void enable() {
           isEnabled = true;
       }
   
       public void disable() {
           isEnabled = false;
       }
   
       public boolean isEnabled() {
           return isEnabled;
       }
   }
   ```

   

2. 在 TextBox 和 CheckBox 中分别实现 render：

   TextBox

   ```java
   public class TextBox extends UIControl {
       private String text = "";
   
       public TextBox() {
           super(true);
       }
   
       @Override
       public void render() {
           System.out.println("Render TextBox");
       }
   
       @Override
       public String toString() {
           return text;
       }
   
       public void setText(String text) {
           this.text = text;
       }
   
       public void clear() {
           text = "";
       }
   }
   ```

   CheeckBox

   ```java
   public class CheckBox extends UIControl{
       public CheckBox() {
           super(true);
       }
   
       @Override
       public void render() {
           System.out.println("Render CheckBox");
       }
   }
   ```

   

3. 此时，我们的主函数中只需要如下代码就可以对不同类别的 UIControl 进行 render：

   ```java
   public class Main {
   
       public static void main(String[] args) {
           UIControl[] controls = { new TextBox(), new CheckBox() };
           for(var control : controls) {
               control.render();
           }
       }
   }
   ```

## Abstract Classes and Methods 抽象类和方法

使用情况：我们声明了一个类或方法，但我们不想把它实例化。

抽象类存在的目的：为它的子类提供公共的代码。

当一个类的所有方法都是抽象的时，我们可以用关键词 abstract 把这个类声明为抽象的，abstract 的位置在 class 前。

当一个类被声明为 abstract 时，就不能被实例化了，只能被继承。

当一个方法被声明为 abstract 时，就会要求这个方法所在的类的所有的非抽象子类都要实现该方法。

```java
public abstract class UIControl {
    private boolean isEnabled = true;

    public abstract void render();

    public UIControl(boolean isEnabled) {
        this.isEnabled = isEnabled;
    }

    public void enable() {
        isEnabled = true;
    }

    public void disable() {
        isEnabled = false;
    }

    public boolean isEnabled() {
        return isEnabled;
    }
}
```

## Final Classes and Methods

当一个类被声明为 final 时，就不能再被继承。非特殊情况下不用。

当一个方法被声明为 final 时，就不能再被重写。

## Deep Inheritance Hierarchies 深度继承等级

不要多重继承，不要超过 3 层。

## Multiple Inheritance

Python 中，一个类可以有多个父类，这叫做 Multiple Inheritance，但 Java 中并没有这个设定，因为这会导致很多模棱两可的问题。

