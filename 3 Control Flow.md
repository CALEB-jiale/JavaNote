# Control Flow

[toc]

## Comparison Operators

* \>
* \<
* \=
* \>=
* \<=

## Logical Operators

* &&
* ||
* !

## Conditional Statements

### If

```java
public class Main {

    public static void main(String[] args) {
	    int temp = 32;
	    if (temp > 30) {
            System.out.println("Il fait chaud!");
        } else if (temp >20) {
            System.out.println("Il fait bon!");
        } else {
            System.out.println("Il fait froid!");
        }
    }
}
```

等价方式：

```java
public class Main {

    public static void main(String[] args) {
	    int income = 120_000;
	    boolean hasHighIncome = income > 100_000;
    }
}
```

### The Ternary Operator

```java
public class Main {

    public static void main(String[] args) {
	    int income = 120_000;
	    String className = income >= 100_000 ? "First" : "Economy";
    }
}
```

### Switch

```java
public class Main {

    public static void main(String[] args) {
        String role = "admin";
        switch (role) {
            case "admin":
                System.out.println("You are an admin.");
                break;

            case "moderator":
                System.out.println("You are a moderator.");
                break;

            default:
                System.out.println("You are a guest.");
        }
    }
}
```



## Loops

### For

```java
public class Main {

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            System.out.println("Hello World");
        }
    }
}
```

### While

```java
public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String input = "";
        while (!input.equals("quit")) {
            System.out.print("Input: ");
            input = scanner.next().toLowerCase();
            System.out.println(input);
        }
    }
}
```

### Do While

```java
public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String input = "";
        do {
            System.out.print("Input: ");
            input = scanner.next().toLowerCase();
            System.out.println(input);
        } while (!input.equals("quit"));
    }
}
```

### Break and Continue

```java
public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String input = "";
        while (true) {
            System.out.print("Input: ");
            input = scanner.next().toLowerCase();
            if (input.equals("pass")) {
                continue;
            }
            if (input.equals("quit")) {
                break;
            }
            System.out.println(input);
        }
    }
}
```

### For Each

```java
public class Main {

    public static void main(String[] args) {
        String[] fruits = {"Apple", "Mango", "Orange"};
        for (String fruit : fruits) {
            System.out.println(fruit);
        }
    }
}
```

局限：

1. 只能从前往后遍历
2. 不能访问索引