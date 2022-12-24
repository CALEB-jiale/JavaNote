# Types

[toc]

## Primitive types

* Primitive types: for storing simple values
* Refrences: for storing complex objects

|  Type   | Bytes |    Range     |
| :-----: | :---: | :----------: |
|  byte   |   1   | [-128, 127]  |
|  short  |   2   | [-32k, 32k]  |
|   int   |   4   |  [-2B, 2B]   |
|  long   |   8   |              |
|  float  |   4   |              |
| double  |   8   |              |
|  char   |   2   | A, B, C, ... |
| boolean |   1   | true / false |

## Reference types

* Primitive types: numbers, characters, booleans
* Refrences: data, mail message

```java
public class Main {

    public static void main(String[] args) {
        byte age = 21;
        Date now = new Date();
        System.out.println(now);
    }
}
```

* primitive types 存储直接存储值

* reference types 存储地址

## Casting and Parse

```java
public class Main {

    public static void main(String[] args) {
        // Implicit casting
        // byte > short > int > long > float > double
        short x = 1;
        int y = x + 2;

        // Explicit casting
        double a = 1.1;
        int b = (int)a + 2;

        // Parse
        String m = "1";
        int y = Integer.parseInt(m) + 2;
    }
}
```

## Math

```java
public class Main {

    public static void main(String[] args) {
        // 四舍五入
        int round = Math.round(1.1F);

        // 取上取下
        int ceil = (int)Math.ceil(1.1F);
        int floor = (int)Math.floor(1.1F);

        // 取大取小
        int max = Math.max(1, 2);
        int min = Math.min(1, 2);

        // 随机
        double random = Math.random();
    }
}
```



## Formating numbers

```java
public class Main {

    public static void main(String[] args) {
        NumberFormat currency = NumberFormat.getCurrencyInstance();
        String result = currency.format(12345);
        System.out.println(result);
    }
}
```

## Reading input

```java
public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("Nom: ");
        String name = scanner.next().trim();
        String name = scanner.nextLine().trim();
        System.out.print("Âge: ");
        byte age = scanner.nextByte();
        System.out.println("Tu es " + name);
        System.out.println("Tu as " + age);
    }
}
```

