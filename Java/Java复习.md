# Chapter 1 Introduction to Computers, Programs, and Java

## Check Point:

### 1.33 What is the Java source filename extension, and what is the Java bytecode filename extension?

Java源文件扩展名：`.java`
Java字节码文件扩展名：`.class`

### 1.35 What is the command to compile a Java program?

编译Java程序的命令：

```bash
javac 文件名.java
```

（例如：`javac HelloWorld.java`）

### 1.37 What is the JVM?

**VM（Java Virtual Machine，Java虚拟机）** 是 Java 程序运行的核心组件，主要作用包括：

1. **执行字节码**：将编译后的 `.class` 文件（字节码）解释或编译成机器码运行。
2. **跨平台支持**：通过“一次编译，到处运行”实现平台无关性（依赖不同系统的 JVM 实现）。
3. **内存管理**：自动处理内存分配、垃圾回收（GC）。
4. **安全机制**：提供沙箱环境，限制恶意代码访问系统资源。

**特点**：

- JVM 本身是**平台相关**的（需针对不同操作系统安装对应版本）。
- 字节码（`.class`）才是**平台无关**的。

```bash
javac Main.java    # 编译生成Main.class
java Main          # JVM执行字节码
```

### 1.42 What are syntax errors (compile errors), runtime errors, and logic errors?

1. **语法错误（编译错误）**
   - 代码不符合语法规则，编译时直接报错（如缺少分号、拼写错误）。
2. **运行时错误**
   - 编译通过，但运行时报错（如数组越界、空指针异常）。
3. **逻辑错误**
   - 程序能运行，但结果错误（如计算错误、条件判断写反）。

### 1.43 Give examples of syntax errors, runtime errors, and logic errors

**语法错误（编译错误）**

- 示例：`System.out.println("Hello"` ❌ 缺少右括号
- 示例：`int x = "123"` ❌ 字符串不能直接赋给整型

**运行时错误**

- 示例：`int[] arr = new int[3]; System.out.println(arr[5]);` ❌ 数组越界
- 示例：`String s = null; System.out.println(s.length());` ❌ 空指针异常

**逻辑错误**

- 示例：计算平均数时：`avg = (a+b)/2` ❌ 整数运算应改为`(a+b)/2.0`
- 示例：判断成年时：`if(age > 18)` ❌ 应写`age >= 18`（漏判18岁）

#  Chapter 2 Elementary Programming

## Check Point:

### 2.8 What are the naming conventions for class names, method names, constants, and variables? Which of the following items can be a constant, a method, a variable, or a class according to the Java naming conventions? MAX_VALUE, Test, read, readDouble

**Java 命名规范**

1. **类名（Class）**：**大驼峰**（首字母大写）
   - 示例：`Test`
2. **方法名（Method）**：**小驼峰**（首字母小写）
   - 示例：`read`, `readDouble`
3. **变量名（Variable）**：**小驼峰**（首字母小写）
   - 示例：`count`, `userName`
4. **常量（Constant）**：**全大写 + 下划线**（`static final` 修饰）
   - 示例：`MAX_VALUE`

------

**判断以下名称的类型**

|     名称     |           类型           |      依据       |
| :----------: | :----------------------: | :-------------: |
| `MAX_VALUE`  |         **常量**         | 全大写 + 下划线 |
|    `Test`    |         **类名**         |     大驼峰      |
|    `read`    | **方法名** 或 **变量名** |     小驼峰      |
| `readDouble` | **方法名** 或 **变量名** |     小驼峰      |

### 2.10 Find the largest and smallest byte, short, int, long, float, and double. Which of these data types requires the least amount of memory?

**取值范围**

| 数据类型 |              最小值              |              最大值              |
| :------: | :------------------------------: | :------------------------------: |
|  `byte`  |              `-128`              |              `127`               |
| `short`  |            `-32,768`             |             `32,767`             |
|  `int`   |     `-2³¹` (-2,147,483,648)      |     `2³¹-1` (2,147,483,647)      |
|  `long`  |              `-2⁶³`              |             `2⁶³-1`              |
| `float`  |  `Float.MIN_VALUE` (约 1.4E-45)  |  `Float.MAX_VALUE` (约 3.4E+38)  |
| `double` | `Double.MIN_VALUE` (约 4.9E-324) | `Double.MAX_VALUE` (约 1.8E+308) |

**内存占用（从小到大）**

| 数据类型 | 内存占用（字节） |
| :------: | :--------------: |
|  `byte`  |  **1**（最小）   |
| `short`  |        2         |
|  `int`   |        4         |
|  `long`  |        8         |
| `float`  |        4         |
| `double` |        8         |

**`byte` 占用内存最少（1字节）**

### 2.19 Which of the following are correct literals for floating-point numbers? 12.3, 12.3e+2, 23.4e-2, –334.4, 20.5, 39F, 40D

**正确的浮点数字面量（✅）**

1. `12.3` → **double**（默认）
2. `12.3e+2` → **科学计数法（double）**（等价于 `12.3 × 10² = 1230.0`）
3. `23.4e-2` → **科学计数法（double）**（等价于 `23.4 × 10⁻² = 0.234`）
4. `–334.4` → **double**（带负号）
5. `20.5` → **double**
6. `39F` → **float**（显式声明为 `float` 类型）
7. `40D` → **double**（显式声明为 `double` 类型）

**总结**

以上所有选项都是合法的浮点数字面量

### 2.25 Which of these statements are true? 

```
a. Any expression can be used as a statement. 

b. The expression x++ can be used as a statement. 

c. The statement x = x + 5 is also an expression. 

d. The statement x = y = x = 0 is illegal
```

正确的陈述是：

**b. 表达式 `x++` 可以作为语句使用。**
​**​c. 语句 `x = x + 5` 也是一个表达式。​**​

解释：

- **a. 错误**：并非所有表达式都能作为语句使用（例如，字面量 `5` 单独作为语句无意义）。
- **b. 正确**：`x++` 是表达式，但可以单独作为语句（如 `x++;`）。
- **c. 正确**：赋值语句（如 `x = x + 5`）本身是表达式，返回被赋的值。
- **d. 错误**：`x = y = x = 0` 是合法的（从右向左赋值），但可能因变量未定义而报错。

### 2.27 Can different types of numeric values be used together in a computation?

是的，**不同类型的数值可以在计算中一起使用**，但会遵循**自动类型转换（隐式转换）**规则：

1. **小范围类型 → 大范围类型**：计算时，较小范围的类型（如 `int`）会自动转换为较大范围的类型（如 `double`）。
   - 例如：`int + double` → `double`。
2. **整数与浮点数混合**：整数会被提升为浮点数后再计算。
   - 例如：`5 / 2.0` → `2.5`（`int` 转 `double`）。
3. **赋值时的潜在问题**：若结果赋值给较小范围的变量，可能需**强制类型转换**，否则可能丢失精度或编译错误。
   - 例如：`int x = 3.14;` 会丢失小数部分（需显式转换）。

**结论**：可以混合计算，但需注意类型转换规则和精度问题。

### 2.28 What does an explicit casting from a double to an int do with the fractional part of the double value? Does casting change the variable being cast?

**显式转换（强制类型转换）** 将 `double` 转为 `int` 时：

1. **对小数部分的处理**：**直接截断**（丢弃小数部分，不四舍五入）。
   - 例如：`(int)3.7` → `3`，`(int)-2.9` → `-2`。
2. **是否改变原变量**：**不会**改变被转换的变量本身，仅生成一个临时的新值。

# Chapter 3 Selections

## Check Point:

### 3.16 a. How do you generate a random integer i such that 0 <= i < 20? b. How do you generate a random integer i such that 10 <= i < 20? c. How do you generate a random integer i such that 10 <= i <= 50? d. Write an expression that returns 0 or 1 randomly.

**a. 生成 0 ≤ i < 20 的随机整数**

```java
int i = (int)(Math.random() * 20);
```

**解释**：`Math.random()` 返回 `[0, 1)` 的浮点数，乘以 20 后取整得到 `[0, 19]`。

**b. 生成 10 ≤ i < 20 的随机整数**

```java
int i = 10 + (int)(Math.random() * 10);
```

**解释**：`Math.random() * 10` 得到 `[0, 10)`，加 10 后得到 `[10, 19]`。

**c. 生成 10 ≤ i ≤ 50 的随机整数**

```java
int i = 10 + (int)(Math.random() * 41);
```

**解释**：`Math.random() * 41` 得到 `[0, 41)`，加 10 后得到 `[10, 50]`。

**d. 随机返回 0 或 1**

```java
int randomBit = (int)(Math.random() * 2);
```

**解释**：`Math.random() * 2` 得到 `[0, 2)`，取整后得到 `0` 或 `1`。

### 3.19 (a) Write a Boolean expression that evaluates to true if a number stored in variable num is between 1 and 100. (b) Write a Boolean expression that evaluates to true if a number stored in variable num is between 1 and 100 or the number is negative.

**(a) 判断 `num` 是否在 1 到 100 之间**

```java
boolean isBetween = (num > 1 && num < 100);
```

**解释**：`num` 必须大于 1 **且** 小于 100 才返回 `true`。

**(b) 判断 `num` 是否在 1 到 100 之间，或者是负数**

```java
boolean isBetweenOrNegative = (num > 1 && num < 100) || (num < 0);
```

**解释**：

- `(num > 1 && num < 100)` → 1 到 100 之间
- `|| (num < 0)` → **或者** 是负数

### 3.29 What data types are required for a switch variable? If the keyword break is not used after a case is processed, what is the next statement to be executed? Can you convert a switch statement to an equivalent if statement, or vice versa? What are the advantages of using a switch statement?

**1. switch 变量的数据类型要求**

**允许的类型**：

- **整数类型**：`byte`、`short`、`int`、`char`
- **枚举（Enum）**
- **字符串（String，Java 7+）**
- **包装类**（如 `Integer`、`Character`）

**不允许的类型**：`long`、`float`、`double`、`boolean` 等。

**2. 如果 `break` 未使用，下一条执行的语句是什么？**

**会继续执行下一个 `case` 的代码（无论是否匹配）**，直到遇到 `break` 或 `switch` 结束。

- 这种现象称为 **"case 穿透"（fall-through）**。

**3. 能否在 `switch` 和 `if` 之间转换？**

- **可以**：`switch` 可以转换为等价的 `if-else if-else` 结构，反之亦然。
- **但 `switch` 更简洁**：适用于多条件匹配固定值的场景。

**4. `switch` 语句的优势**

- **代码更清晰**：比多层 `if-else` 更易读。
- **执行效率高**：编译器可能优化为 **跳转表**（比 `if` 的逐级判断更快）。
- **支持枚举和字符串**（`if` 也能实现，但 `switch` 更直观）。

#  Chapter 4 Mathematical Functions, Characters, and Strings

## Check Point:

### 4.4.6 Write one statement to return the number of digits in an integer i.

可以使用以下语句返回整数 `i` 的位数：

```java
int digits = String.valueOf(Math.abs(i)).length();
```

**说明**：

1. `Math.abs(i)` 确保负数也能正确计算位数。
2. `String.valueOf()` 将整数转为字符串。
3. `.length()` 返回字符串长度，即数字的位数。

---

如果不使用库函数（如 `String.valueOf()` 或 `Math.abs()`），可以用循环计算整数的位数：

```java
int count = 0;
int temp = i;
do {
    temp /= 10;
    count++;
} while (temp != 0);
```

**说明**：

1. 用 `do-while` 确保 `i = 0` 时返回 `1`。
2. 每次循环去掉最低位（`temp /= 10`），并计数。

### 4.4.7 Write one statement to return the number of digits in a double value d.

```java
int numDigits = String.valueOf(d).replaceAll("[^0-9]", "").length();
```

实现原理：

1. **`String.valueOf(d)`**
   - 将 double 转为字符串（如 `-123.45` → `"-123.45"`）
2. **`.replaceAll("[^0-9]", "")`**
   - 正则表达式 `[^0-9]` 删除所有**非数字字符**（包括负号 `.` 和正号 `+`）
   - 例如：`"-123.45"` → `"12345"`，`"+0.007"` → `"0007"`
3. **`.length()`**
   - 返回纯数字字符串的长度

---

**纯数学计算（不依赖任何库）**

```java
double temp = d;
long num = (long)temp; // 取整数部分
if (num < 0) num = -num; // 处理负数
int count = (num == 0) ? 1 : 0;
while (num != 0) {
    num /= 10;
    count++;
}
```

**说明**：

1. 用 `(long)d` 取整数部分。
2. 循环除以 10 计算位数。
3. 特殊处理 `d=0`，直接返回 `1`。

### 4.6 Why does the Math class not need to be imported?

`Math`类属于`java.lang`包，该包默认自动导入，因此无需手动导入。

# Chapter 5 Loops

## Programming Exercise:

### 5.11 (Find numbers divisible by 5 or 6, but not both) Write a program that displays all the numbers from 100 to 200, ten per line, that are divisible by 5 or 6, but not both. Numbers are separated by exactly one space.

```java
public class DivisibleBy5Or6 {
    public static void main(String[] args) {
        int count = 0; // 每行计数器

        for (int i = 100; i <= 200; i++) {
            // 判断能被 5 或 6 整除，但不能同时被 5 和 6 整除
            if ((i % 5 == 0 || i % 6 == 0) && !(i % 5 == 0 && i % 6 == 0)) {
                System.out.print(i + " ");
                count++;

                // 每行输出 10 个数，换行
                if (count % 10 == 0) {
                    System.out.println();
                }
            }
        }
    }
}
```

📊 程序说明：

- `count` 记录当前行已经输出了多少个数。
- `if ((i % 5 == 0 || i % 6 == 0) && !(i % 5 == 0 && i % 6 == 0))`
   判断：能被 5 或 6 整除，但不能同时整除。
- 每输出一个合格的数，`count++`。
- 当 `count` 是 10 的倍数时，换行。

### 5.26 (Compute e) You can approximate e using the following series:

$$
e = 1 + \frac{1}{1!} + \frac{1}{2!} + \frac{1}{3!} + \cdots + \frac{1}{i!}
$$

Write a program that displays the e value for i = 10000, 20000, …, and 100000. (Hint: Because i! = i * (i - 1) * c * 2 * 1, then
$$
\frac{1}{i!} is \frac{1}{i(i-1)!}
$$
Initialize e and item to be 1 and keep adding a new item to e. The new item is the previous item divided by i for i = 2, 3, 4, . . . .)

```java
public class ComputeE {
    public static void main(String[] args) {
        double e = 1.0;
        double item = 1.0;

        for (int i = 1; i <= 100000; i++) {
            item /= i;
            e += item;

            // 如果 i 达到 10000 的倍数，就输出一次 e 的值
            if (i % 10000 == 0) {
                System.out.printf("i = %d, e = %.15f\n", i, e);
            }
        }
    }
}
```

📊 程序说明：

- `e` 初始值设为 1.0（即第 0 项 `1`）。
- `item` 初始值 1.0，对应 `1/0!`。
- 循环从 1 到 100000，每次 `item /= i`，然后 `e += item`。
- 当 `i` 能被 10000 整除时，输出当前 `e` 值，保留 15 位小数。

### 5.33 (Perfect number) A positive integer is called a perfect number if it is equal to the sum of all of its positive divisors, excluding itself. For example, 6 is the first perfect number because 6 = 3 + 2 + 1. The next is 28 = 14 + 7 + 4 + 2 + 1. There are four perfect numbers less than 10,000. Write a program to find all these four numbers.

```java
public class PerfectNumberFinder {
    public static void main(String[] args) {
        for (int number = 2; number < 10000; number++) {
            int sum = 0;

            // 求 number 的所有正因子之和（不包括 number 本身）
            for (int divisor = 1; divisor <= number / 2; divisor++) {
                if (number % divisor == 0) {
                    sum += divisor;
                }
            }

            // 如果因子和等于 number 本身，说明是完美数
            if (sum == number) {
                System.out.println(number);
            }
        }
    }
}
```

📊 程序说明：

- 外层循环：遍历 2 到 9999 的所有正整数。
- 内层循环：找出当前数 `number` 的所有因子（小于 `number` 且能整除 `number` 的数）。
- 求因子和 `sum`，如果 `sum == number`，说明是完美数，输出。
- `divisor <= number / 2` 就行，因为一个数的因子不可能超过它的一半。

### 5.48 (Process string) Write a program that prompts the user to enter a string and displays the characters at odd positions. Here is a sample run:

```
Enter a string: Beijing Chicago
BiigCiao
```

```java
import java.util.Scanner;

public class ProcessString {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);

        // 提示用户输入字符串
        System.out.print("Enter a string: ");
        String str = input.nextLine();

        // 输出奇数下标位置的字符
        for (int i = 0; i < str.length(); i += 2) {
            System.out.print(str.charAt(i));
        }

        System.out.println(); // 换行
        input.close();
    }
}
```

📊 程序说明：

- `Scanner` 获取用户输入。
- `for` 循环从 0 开始，每次 `i += 2`，跳过偶数下标。
- `str.charAt(i)` 取出当前字符并输出。
- 最后换行，美化输出。

# Chapter 6 Methods

## Check Point:

### 6.4 True or false? A call to a method with a void return type is always a statement itself, but a call to a value-returning method cannot be a statement by itself.

False。对返回void的方法的调用可以单独作为语句，但对返回值的方法的调用也可以单独作为语句。

### 6.6 What would be wrong with not writing a return statement in a value-returning method? Can you have a return statement in a void method? Does the return statement in the following method cause syntax errors? 

```java
public static void xMethod(double x, double y) {
    System.out.println(x + y);
    return x + y;
}
```

1. **不写 `return` 语句在返回值的方法中会怎样？**
   - 会导致**编译错误**，因为 Java 要求值返回方法必须返回一个与声明类型匹配的值。
2. **`void` 方法中可以有 `return` 语句吗？**
   - 可以，但**不能返回值**，只能写成 `return;` 用于提前退出方法。
3. **示例代码是否有语法错误？**
   - **有错误**，因为 `void` 方法不能返回 `x + y`（一个 `double` 值）。
   - 修正方式：要么去掉 `return x + y;`，要么将方法改为返回值类型（如 `double`）。

### 6.7 Define the terms parameter, argument, and method signature

1. **参数（Parameter）**：
   - 方法定义时声明的变量，用于接收传入的值。
   - 例如：`void print(int num)` 中的 `num` 是参数。
2. **实参（Argument）**：
   - 调用方法时实际传入的值或变量。
   - 例如：`print(5)` 中的 `5` 是实参。
3. **方法签名（Method Signature）**：
   - 方法的名称 + 参数列表（类型、顺序、数量），用于唯一标识方法。
   - 例如：`int sum(int a, int b)` 的签名是 `sum(int, int)`。
   - **不包括返回类型和参数名**。

### 6.8 Write method headers (not the bodies) for the following methods: 

```
a. Return a sales commission, given the sales amount and the commission rate.
b. Display the calendar for a month, given the month and year.
c. Return a square root of a number.
d. Test whether a number is even, and returning true if it is.
e. Display a message a specified number of times.
f. Return the monthly payment, given the loan amount, number of years, and annual
interest rate.
g. Return the corresponding uppercase letter, given a lowercase letter.
```

a. 返回销售佣金（给定销售额和佣金率）

```java
public static double getSalesCommission(double salesAmount, double commissionRate)
```

b. 显示某年某月的日历

```java
public static void displayCalendar(int month, int year)
```

c. 返回一个数的平方根

```java
public static double getSquareRoot(double number)
```

d. 判断一个数是否为偶数，如果是则返回 `true`

```java
public static boolean isEven(int number)
```

e. 显示消息指定次数

```java
public static void displayMessage(String message, int times)
```

f. 返回月供（给定贷款金额、年数和年利率）

```java
public static double getMonthlyPayment(double loanAmount, int years, double annualRate)
```

g. 返回给定小写字母对应的大写字	母

```java
public static char toUpperCase(char lowercaseLetter)
```

## Programming Exercise:

### 6.2 (Sum the digits in an integer) Write a method that computes the sum of the digits in an integer. Use the following method header: 

```java
public static int sumDigits(long n) 
```

For example, sumDigits(234) returns 9 (2 + 3 + 4). (Hint: Use the % operator to extract digits, and the / operator to remove the extracted digit. For instance, to extract 4 from 234, use 234 % 10 (= 4). To remove 4 from 234, use 234 / 10 (= 23). Use a loop to repeatedly extract and remove the digit until all the digits are extracted. Write a test program that prompts the user to enter an integer and displays the sum of all its digits.

```java
import java.util.Scanner;

public class SumDigits {
    
    // 方法：求整数 n 的各位数字之和
    public static int sumDigits(long n) {
        int sum = 0;
        while (n != 0) {
            sum += n % 10;  // 取最后一位
            n /= 10;        // 去掉最后一位
        }
        return sum;
    }

    // 主程序
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);

        // 提示用户输入一个整数
        System.out.print("Enter an integer: ");
        long number = input.nextLong();

        // 调用方法并输出结果
        int result = sumDigits(number);
        System.out.println("The sum of the digits is: " + result);

        input.close();
    }
}
```

📊 程序说明：

- `sumDigits(long n)`：
  - 用 `% 10` 取最后一位数字，加到 `sum` 中。
  - 用 `/ 10` 去掉最后一位。
  - 循环直到 `n` 变成 0。
- `main` 方法中：
  - 用 `Scanner` 获取用户输入。
  - 调用 `sumDigits` 方法，输出结果。

### 6.12 (Display characters) Write a method that prints characters using the following header: 

```java
public static void printChars(char ch1, char ch2, int numberPerLine) 
```

This method prints the characters between ch1 and ch2 with the specified numbers per line. Write a test program that prints ten characters per line from 1 to Z. Characters are separated by exactly one space.

```java
public class DisplayCharacters {

    // 方法：打印从 ch1 到 ch2 的字符，每行 numberPerLine 个
    public static void printChars(char ch1, char ch2, int numberPerLine) {
        int count = 0;

        for (char c = ch1; c <= ch2; c++) {
            System.out.print(c + " ");
            count++;

            // 每 numberPerLine 个字符换行
            if (count % numberPerLine == 0) {
                System.out.println();
            }
        }

        // 如果最后一行没满，补个换行
        if (count % numberPerLine != 0) {
            System.out.println();
        }
    }

    // 主程序
    public static void main(String[] args) {
        // 调用方法，打印 1 到 Z，每行 10 个字符
        printChars('1', 'Z', 10);
    }
}
```

📊 程序说明：

- `for` 循环遍历从 `ch1` 到 `ch2` 的字符。
- `count` 用来记录当前行已经输出了多少个字符。
- 每输出 `numberPerLine` 个就换行。
- 最后如果不满一行，再补一个换行，保证输出美观。

### 6.20 (Count the letters in a string) Write a method that counts the number of letters in a string using the following header:

```java
public static int countLetters(String s)
```

 Write a test program that prompts the user to enter a string and displays the number of letters in the string.

```java
import java.util.Scanner;

public class CountLetters {

    // 方法：统计字符串中字母的个数
    public static int countLetters(String s) {
        int count = 0;
        for (int i = 0; i < s.length(); i++) {
            // 判断是否为字母
            if (Character.isLetter(s.charAt(i))) {
                count++;
            }
        }
        return count;
    }

    // 主程序
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);

        // 提示用户输入字符串
        System.out.print("Enter a string: ");
        String str = input.nextLine();

        // 调用方法，显示结果
        int letterCount = countLetters(str);
        System.out.println("The number of letters in the string is: " + letterCount);

        input.close();
    }
}
```

📊 程序说明：

- `Character.isLetter(char c)` 是 Java 自带的方法，判断一个字符是不是字母。
- 遍历字符串每个字符，遇到字母 `count++`。
- 最后返回字母个数。

# Chapter 7 Single-Dimensional Arrays

## Check Point:

### 7.1 How do you declare an array reference variable and how do you create an array?

在Java中声明数组引用变量和创建数组的方法如下：

1. **声明数组引用变量**（指定类型和名称）：

   ```java
   数据类型[] 变量名;  // 推荐方式
   // 或
   数据类型 变量名[];  // 合法但不推荐
   ```

   示例：

   ```java
   int[] nums;  // 声明整型数组引用
   ```

2. **创建数组**（分配内存空间）：

   ```java
   变量名 = new 数据类型[数组长度];  // 动态初始化
   // 或直接初始化（静态初始化）
   数据类型[] 变量名 = {值1, 值2, ...};
   ```

   示例：

   ```java
   nums = new int[5];          // 创建长度为5的整型数组
   String[] names = {"A", "B"}; // 直接初始化字符串数组
   ```

👉 关键点：声明仅创建引用，`new`或`{}`才会实际分配数组内存。

### 7.4 Indicate true or false for the following statements: 

```
■ Every element in an array has the same type.
■ The array size is fixed after an array reference variable is declared.
■ The array size is fixed after it is created.
■ The elements in an array must be a primitive data type.
```

1. **✅ True** - 数组中的每个元素类型相同（声明时确定）。
2. **❌ False** - 数组大小在声明引用变量时未固定（仅创建时通过`new`或`{}`确定）。
3. **✅ True** - 数组一旦创建，大小固定（不可修改）。
4. **❌ False** - 数组元素可以是**基本类型**（如`int`）或**对象类型**（如`String`）。

### 7.9 What happens when your program attempts to access an array element with an invalid index?

程序会抛出 **`ArrayIndexOutOfBoundsException`** 并终止（除非用 `try-catch` 捕获）。

### 7.16 True or false? When an array is passed to a method, a new array is created and passed to the method.

**False**

当数组传递给方法时，传递的是**数组引用**（内存地址），**不会创建新数组**。方法内修改数组会影响原始数组。

### 7.26 What types of array can be sorted using the java.util.Arrays.sort method? Does this sort method create a new array?

1. **支持排序的数组类型**：
   - 基本类型数组（如 `int[]`, `double[]`, `char[]` 等）
   - 对象类型数组（如 `String[]`, `Integer[]` 等，需实现 `Comparable` 接口）
2. **是否创建新数组**：
   → ​**​不会​**​。`Arrays.sort()` 直接修改原数组（​**​原地排序​**​），不返回新数组。

## Programming Exercise:

### 7.8 (Average an array) Write two overloaded methods that return the average of an array with the following headers: 

```java
public static int average(int[] array)
public static double average(double[] array)
```

Write a test program that prompts the user to enter ten double values, invokes this method, and displays the average value.

```java
import java.util.Scanner;

public class AverageArray {

    // 方法1：计算 int 数组平均值，返回 int
    public static int average(int[] array) {
        int sum = 0;
        for (int value : array) {
            sum += value;
        }
        return sum / array.length;
    }

    // 方法2：计算 double 数组平均值，返回 double
    public static double average(double[] array) {
        double sum = 0;
        for (double value : array) {
            sum += value;
        }
        return sum / array.length;
    }

    // 主程序
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        double[] numbers = new double[10];

        // 提示用户输入 10 个 double 值
        System.out.print("Enter ten double values: ");
        for (int i = 0; i < numbers.length; i++) {
            numbers[i] = input.nextDouble();
        }

        // 调用 double 数组版 average 方法并输出结果
        double avg = average(numbers);
        System.out.println("The average value is: " + avg);

        input.close();
    }
}
```

📊 程序说明：

- **重载方法**
  - `average(int[] array)`：遍历 `int` 数组，求和，除以数组长度，返回 `int`
  - `average(double[] array)`：遍历 `double` 数组，求和，除以数组长度，返回 `double`
- **主程序**
  - 用 `Scanner` 获取 10 个 `double` 值
  - 调用 `average(double[])` 方法，输出平均值

### 7.14 (Computing gcd) Write a method that returns the gcd of an unspecified number of integers. The method header is specified as follows: 

```java
public static int gcd(int... numbers) 
```

Write a test program that prompts the user to enter five numbers, invokes the method to find the gcd of these numbers, and displays the gcd.

```java
import java.util.Scanner;

public class ComputeGCD {

    // 方法：求两个数的最大公约数（辗转相除法）
    public static int gcd(int a, int b) {
        while (b != 0) {
            int temp = b;
            b = a % b;
            a = temp;
        }
        return a;
    }

    // 方法：求任意数量数的最大公约数
    public static int gcd(int... numbers) {
        int result = numbers[0];
        for (int i = 1; i < numbers.length; i++) {
            result = gcd(result, numbers[i]);
        }
        return result;
    }

    // 主程序
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        int[] numbers = new int[5];

        // 提示用户输入 5 个整数
        System.out.print("Enter five numbers: ");
        for (int i = 0; i < numbers.length; i++) {
            numbers[i] = input.nextInt();
        }

        // 调用 gcd 方法并输出结果
        int result = gcd(numbers);
        System.out.println("The greatest common divisor is: " + result);

        input.close();
    }
}
```

📊 程序说明：

- **gcd(int a, int b)**：用辗转相除法（欧几里得算法）计算两个数的最大公约数
- **gcd(int... numbers)**：
  - 先把第一个数赋给 `result`
  - 然后依次跟后面的数求 `gcd(result, numbers[i])`
  - 最后 `result` 就是所有数的最大公约数
- 主程序：
  - 提示用户输入 5 个整数，放进 `int[]`
  - 调用 `gcd(numbers)` 方法输出结果

# Chapter 8 Multidimensional Arrays

## Check Point:

### 8.2 Can the rows in a two-dimensional array have different lengths?

**可以**。Java 的二维数组允许每行长度不同（称为**不规则数组/Jagged Array**）。

### 8.4 Which of the following statements are valid? 

```java
int[][] r = new int[2];
int[] x = new int[];
int[][] y = new int[3][];
int[][] z = {{1, 2}};
int[][] m = {{1, 2}, {2, 3}};
int[][] n = {{1, 2}, {2, 3}, };
```

✅ **有效的声明**：

```java
int[][] y = new int[3][];  // 合法：二维数组，行数固定，列数未定
int[][] z = {{1, 2}};      // 合法：静态初始化，1行2列
int[][] m = {{1, 2}, {2, 3}}; // 合法：2行2列
int[][] n = {{1, 2}, {2, 3}, }; // 合法：末尾逗号允许（2行2列）
```

❌ **无效的声明**：

```java
int[][] r = new int[2];    // 错误：缺少第二维长度（应为 new int[2][...]）
int[] x = new int[];       // 错误：未指定长度或初始化值
```

## Programming Exercise:

### 8.2 (Sum the major diagonal in a matrix) Write a method that sums all the numbers in the major diagonal in an n * n matrix of double values using the following header:

```java 
public static double sumMajorDiagonal(double[][] m)
```

Write a test program that reads a 4-by-4 matrix and displays the sum of all its elements on the major diagonal. Here is a sample run:

```
Enter a 4-by-4 matrix row by row:
1 2 3 4.0
5 6.5 7 8
9 10 11 12
13 14 15 16
Sum of the elements in the major diagonal is 34.5 
```

```java
import java.util.Scanner;

public class MajorDiagonalSum {

    // 方法：求矩阵主对角线元素之和
    public static double sumMajorDiagonal(double[][] m) {
        double sum = 0;
        for (int i = 0; i < m.length; i++) {
            sum += m[i][i];
        }
        return sum;
    }

    // 主程序
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        double[][] matrix = new double[4][4];

        // 提示用户输入 4×4 矩阵
        System.out.println("Enter a 4-by-4 matrix row by row:");
        for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 4; j++) {
                matrix[i][j] = input.nextDouble();
            }
        }

        // 调用方法，计算主对角线和
        double sum = sumMajorDiagonal(matrix);

        // 输出结果
        System.out.println("Sum of the elements in the major diagonal is " + sum);

        input.close();
    }
}
```

📊 程序说明：

- `sumMajorDiagonal(double[][] m)`：
  - 用 `for` 循环，主对角线的元素是 `m[0][0], m[1][1], m[2][2], m[3][3]`
  - 用 `m[i][i]` 累加求和
- 主程序：
  - 用嵌套循环输入 4×4 的 double 数组
  - 调用方法输出结果

### 8.6 (Algebra: multiply two matrices) Write a method to multiply two matrices. The header of the method is:

```java
public static double[][] multiplyMatrix(double[][] a, double[][] b)
```

To multiply matrix a by matrix b, the number of columns in a must be the same as the number of rows in b, and the two matrices must have elements of the same or compatible types. Let c be the result of the multiplication. Assume the column size of matrix a is n. Each element cij is ai1 * b1j + ai2 * b2j + c + ain * bnj. For example, for two 3 * 3 matrices a and b, c is
$$
\begin{pmatrix}
a_{11} & a_{12} & a_{13} \\
a_{21} & a_{22} & a_{23} \\
a_{31} & a_{32} & a_{33}
\end{pmatrix}
\times
\begin{pmatrix}
b_{11} & b_{12} & b_{13} \\
b_{21} & b_{22} & b_{23} \\
b_{31} & b_{32} & b_{33}
\end{pmatrix}
=
\begin{pmatrix}
c_{11} & c_{12} & c_{13} \\
c_{21} & c_{22} & c_{23} \\
c_{31} & c_{32} & c_{33}
\end{pmatrix}
$$
where cij = ai1 * b1j + ai2 * b2j + ai3 * b3j . Write a test program that prompts the user to enter two 3 * 3 matrices and displays their product. Here is a sample run:

```
Enter matrix1: 1 2 3 4 5 6 7 8 9
Enter matrix2: 0 2 4 1 4.5 2.2 1.1 4.3 5.2
The multiplication of the matrices is
 1 2 3 0 2.0 4.0 5.3 23.9 24
 4 5 6 * 1 4.5 2.2 = 11.6 56.3 58.2
 7 8 9 1.1 4.3 5.2 17.9 88.7 92.4
```

```java
import java.util.Scanner;

public class MatrixMultiplication {

    // 方法：矩阵相乘
    public static double[][] multiplyMatrix(double[][] a, double[][] b) {
        double[][] c = new double[3][3];

        for (int i = 0; i < 3; i++) {         // 行
            for (int j = 0; j < 3; j++) {     // 列
                c[i][j] = 0;
                for (int k = 0; k < 3; k++) { // 累加计算
                    c[i][j] += a[i][k] * b[k][j];
                }
            }
        }

        return c;
    }

    // 方法：输出矩阵
    public static void printMatrix(double[][] matrix) {
        for (double[] row : matrix) {
            for (double value : row) {
                System.out.printf("%6.1f", value);
            }
            System.out.println();
        }
    }

    // 方法：输出带中缀 * 和 =
    public static void printFormatted(double[][] a, double[][] b, double[][] c) {
        for (int i = 0; i < 3; i++) {
            // 矩阵 a
            for (int j = 0; j < 3; j++) {
                System.out.printf("%6.1f", a[i][j]);
            }

            // * 和 = 号
            if (i == 1) {
                System.out.print("  * ");
            } else {
                System.out.print("    ");
            }

            // 矩阵 b
            for (int j = 0; j < 3; j++) {
                System.out.printf("%6.1f", b[i][j]);
            }

            if (i == 1) {
                System.out.print("  = ");
            } else {
                System.out.print("    ");
            }

            // 结果矩阵 c
            for (int j = 0; j < 3; j++) {
                System.out.printf("%6.1f", c[i][j]);
            }

            System.out.println();
        }
    }

    // 主程序
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);

        double[][] matrix1 = new double[3][3];
        double[][] matrix2 = new double[3][3];

        // 输入矩阵1
        System.out.println("Enter matrix1: ");
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                matrix1[i][j] = input.nextDouble();

        // 输入矩阵2
        System.out.println("Enter matrix2: ");
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                matrix2[i][j] = input.nextDouble();

        // 矩阵相乘
        double[][] result = multiplyMatrix(matrix1, matrix2);

        // 输出结果
        System.out.println("The multiplication of the matrices is");
        printFormatted(matrix1, matrix2, result);

        input.close();
    }
}
```

📌 程序说明：

- `multiplyMatrix` 方法：三重循环实现矩阵相乘
- `printFormatted` 方法：
  - 每行同时输出 matrix1、matrix2 和 result
  - 第 2 行显示 `*` 和 `=`
- 主程序：读取用户输入，调用方法计算、输出

### 8.13 (Locate the largest element) Write the following method that returns the location of the largest element in a two-dimensional array

```java
public static int[] locateLargest(double[][] a) 
```

The return value is a one-dimensional array that contains two elements. These two elements indicate the row and column indices of the largest element in the two-dimensional array. Write a test program that prompts the user to enter a twodimensional array and displays the location of the largest element in the array. Here is a sample run:

```
Enter the number of rows and columns of the array: 3 4
Enter the array:
23.5 35 2 10
4.5 3 45 3.5
35 44 5.5 9.6
The location of the largest element is at (1, 2)
```

```java
import java.util.Scanner;

public class LocateLargest {

    // 方法：查找最大元素的位置
    public static int[] locateLargest(double[][] a) {
        int[] location = new int[2]; // [行, 列]
        double max = a[0][0];
        location[0] = 0;
        location[1] = 0;

        for (int i = 0; i < a.length; i++) {
            for (int j = 0; j < a[i].length; j++) {
                if (a[i][j] > max) {
                    max = a[i][j];
                    location[0] = i;
                    location[1] = j;
                }
            }
        }
        return location;
    }

    // 主程序
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);

        // 输入行列数
        System.out.print("Enter the number of rows and columns of the array: ");
        int rows = input.nextInt();
        int cols = input.nextInt();

        double[][] array = new double[rows][cols];

        // 输入数组元素
        System.out.println("Enter the array:");
        for (int i = 0; i < rows; i++)
            for (int j = 0; j < cols; j++)
                array[i][j] = input.nextDouble();

        // 查找最大值位置
        int[] location = locateLargest(array);

        // 输出位置
        System.out.println("The location of the largest element is at (" +
                           location[0] + ", " + location[1] + ")");

        input.close();
    }
}
```

📌 程序说明：

- `locateLargest` 方法：
  - 用一个 `int[]` 数组存储最大值的行列索引
  - 两层循环逐个比较更新最大值和位置
- 主程序：
  - 先输入行列数，再输入数组值
  - 调用方法查找最大元素位置并输出

# Chapter 9 Objects and Classes

## Check Point:

### 9.1 Describe the relationship between an object and its defining class.

对象是类的实例，类定义了对象的属性和方法。对象通过类创建，继承类的结构。

### 9.5 What are the differences between constructors and methods?

**构造函数（Constructor）和方法（Method）的区别：**

1. **用途**
   - 构造函数：**初始化对象**（在 `new` 时自动调用）。
   - 方法：**执行具体功能**（需手动调用）。
2. **命名**
   - 构造函数：**必须与类名相同**。
   - 方法：**可自定义名称**。
3. **返回值**
   - 构造函数：**无返回值**（连 `void` 也不写）。
   - 方法：**可有返回值**（或 `void`）。
4. **调用时机**
   - 构造函数：**对象创建时自动执行一次**。
   - 方法：**按需调用**，可多次执行。

### 9.6 When will a class have a default constructor?

一个类在**没有显式定义任何构造函数**时，编译器会自动生成一个**无参数的默认构造函数**。

**两种情况会失去默认构造函数：**

1. **手动定义了任意构造函数**（即使是有参的）→ 编译器不再自动生成默认构造。
2. **类被声明为抽象类（abstract）** → 不能直接实例化，无需默认构造。

### 9.10 Is an array an object or a primitive type value? Can an array contain elements of an object type? Describe the default value for the elements of an array.

**数组（Array）的性质：**

1. **数组是对象**，不是基本类型（如 `int`、`char` 等）。
2. **可以包含对象类型的元素**（例如 `String[]`、自定义类的对象数组）。

**数组元素的默认值：**

- **基本类型数组**：数值型（`int`/`double` 等）默认为 `0`，`boolean` 默认为 `false`，`char` 默认为 `\u0000`。
- **对象类型数组**（如 `String[]`、自定义类）：默认为 `null`。

### 9.19 Can you invoke an instance method or reference an instance variable from a static method? Can you invoke a static method or reference a static variable from an instance method? What is wrong in the following code?

**1. 静态方法（static method）能否调用实例方法或访问实例变量？**

❌ **不能**。静态方法属于类，不依赖对象实例，因此无法直接访问实例方法或变量（需先创建对象）。

**2. 实例方法能否调用静态方法或访问静态变量？**

✅ **可以**。实例方法可以访问静态成员（因为静态成员属于类，与对象无关）。

**3. 代码错误分析（问题代码）：**

```java
public class C {
    public static void main(String[] args) {
        method1();  // ❌ 错误：静态方法直接调用了实例方法 method1()
    }

    public void method1() {  // 实例方法
        method2();  // ✅ 合法：实例方法可以调用静态方法
    }

    public static void method2() {
        System.out.println("What is radius " + c.getRadius());  
        // ❌ 错误：静态方法 method2() 直接访问了实例变量 c
    }

    Circle c = new Circle();  // 实例变量
}
```

### 9.20 What is an accessor method? What is a mutator method? What are the naming conventions for accessor methods and mutator methods?

**访问器方法（Getter）**

- 作用：读取私有属性值
- 命名：`get`+属性名（如`getName()`）
- 布尔属性用`is`（如`isValid()`）

**修改器方法（Setter）**

- 作用：修改私有属性值
- 命名：`set`+属性名（如`setAge()`）

### 9.21 What are the benefits of data field encapsulation?

1. **保护数据安全**
   - 防止外部直接修改数据，避免非法值（如`age = -10`）。
2. **可控的访问权限**
   - 通过`getter/setter`方法控制读写（如只读属性）。
3. **隐藏实现细节**
   - 内部逻辑修改不影响外部代码（如计算属性`getTotal()`）。
4. **便于维护和扩展**
   - 可在方法中添加校验、日志等逻辑，统一管理数据。

### 9.32 Describe the role of the this keyword.

1. **指代当前对象**（实例方法/构造器中）
2. **区分同名变量**（如 `this.name = name`）
3. **调用其他构造方法**（`this(...)`，必须在首行）
4. **作为参数传递**（如 `obj.method(this)`）

## Programming Exercise:

### 9.6 (Stopwatch) Design a class named StopWatch. The class contains:

```
■ Private data fields startTime and endTime with getter methods.
■ A no-arg constructor that initializes startTime with the current time.
■ A method named start() that resets the startTime to the current time.
■ A method named stop() that sets the endTime to the current time.
■ A method named getElapsedTime() that returns the elapsed time for the stopwatch in milliseconds.
```

Draw the UML diagram for the class and then implement the class. Write a test program that measures the execution time of sorting 100,000 numbers using selection sort.

UML 类图

```markdown
-----------------------------
|         StopWatch         |
-----------------------------
- startTime: long
- endTime: long
-----------------------------
+ StopWatch()
+ start(): void
+ stop(): void
+ getStartTime(): long
+ getEndTime(): long
+ getElapsedTime(): long
-----------------------------
```

```java
public class StopWatch {
    private long startTime;
    private long endTime;

    // 无参构造器，初始化 startTime 为当前时间
    public StopWatch() {
        startTime = System.currentTimeMillis();
    }

    // 重置 startTime 为当前时间
    public void start() {
        startTime = System.currentTimeMillis();
    }

    // 设置 endTime 为当前时间
    public void stop() {
        endTime = System.currentTimeMillis();
    }

    // 获取 startTime
    public long getStartTime() {
        return startTime;
    }

    // 获取 endTime
    public long getEndTime() {
        return endTime;
    }

    // 获取耗时（毫秒）
    public long getElapsedTime() {
        return endTime - startTime;
    }

    // 测试程序：用 selection sort 排序 100,000 个数，并计时
    public static void main(String[] args) {
        int[] numbers = new int[100000];

        // 随机生成 100,000 个 0~999,999 之间的数
        for (int i = 0; i < numbers.length; i++) {
            numbers[i] = (int)(Math.random() * 1000000);
        }

        // 创建秒表对象
        StopWatch stopwatch = new StopWatch();
        stopwatch.start();

        // 选择排序
        for (int i = 0; i < numbers.length - 1; i++) {
            int minIndex = i;
            for (int j = i + 1; j < numbers.length; j++) {
                if (numbers[j] < numbers[minIndex]) {
                    minIndex = j;
                }
            }

            // 交换
            int temp = numbers[i];
            numbers[i] = numbers[minIndex];
            numbers[minIndex] = temp;
        }

        stopwatch.stop();

        // 输出耗时
        System.out.println("Selection sort of 100,000 numbers took " +
                           stopwatch.getElapsedTime() + " milliseconds.");
    }
}
```

### 9.10 (Algebra: quadratic equations) Design a class named QuadraticEquation for a quadratic equation ax2 + bx + x = 0. The class contains:

```
■ Private data fields a, b, and c that represent three coefficients.
■ A constructor for the arguments for a, b, and c.
■ Three getter methods for a, b, and c.
■ A method named getDiscriminant() that returns the discriminant, which is b2 - 4ac.
■ The methods named getRoot1() and getRoot2() for returning two roots of the equation 
```

$$
\begin{aligned}
r_1 &= \frac{-b + \sqrt{b^2 - 4ac}}{2a} \\
r_2 &= \frac{-b - \sqrt{b^2 - 4ac}}{2a}
\end{aligned}
$$

These methods are useful only if the discriminant is nonnegative. Let these methods return 0 if the discriminant is negative. 

Draw the UML diagram for the class and then implement the class. Write a test program that prompts the user to enter values for a, b, and c and displays the result based on the discriminant. If the discriminant is positive, display the two roots. If the discriminant is 0, display the one root. Otherwise, display “The equation has no roots.” See Programming Exercise 3.1 for sample runs.

```markdown
----------------------------------------------------
|              QuadraticEquation                   |
----------------------------------------------------
- a: double  
- b: double  
- c: double  
----------------------------------------------------
+ QuadraticEquation(a: double, b: double, c: double)  
+ getA(): double  
+ getB(): double  
+ getC(): double  
+ getDiscriminant(): double  
+ getRoot1(): double  
+ getRoot2(): double  
----------------------------------------------------
```

QuadraticEquation.java

```java
public class QuadraticEquation {
    private double a;
    private double b;
    private double c;

    // 构造器
    public QuadraticEquation(double a, double b, double c) {
        this.a = a;
        this.b = b;
        this.c = c;
    }

    // Getter 方法
    public double getA() {
        return a;
    }

    public double getB() {
        return b;
    }

    public double getC() {
        return c;
    }

    // 计算判别式
    public double getDiscriminant() {
        return b * b - 4 * a * c;
    }

    // 求根1
    public double getRoot1() {
        double discriminant = getDiscriminant();
        if (discriminant < 0) {
            return 0;
        } else {
            return (-b + Math.sqrt(discriminant)) / (2 * a);
        }
    }

    // 求根2
    public double getRoot2() {
        double discriminant = getDiscriminant();
        if (discriminant < 0) {
            return 0;
        } else {
            return (-b - Math.sqrt(discriminant)) / (2 * a);
        }
    }
}
```

测试程序

```java
import java.util.Scanner;

public class TestQuadraticEquation {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);

        // 输入 a, b, c
        System.out.print("Enter a, b, c: ");
        double a = input.nextDouble();
        double b = input.nextDouble();
        double c = input.nextDouble();

        // 创建方程对象
        QuadraticEquation equation = new QuadraticEquation(a, b, c);

        double discriminant = equation.getDiscriminant();

        if (discriminant > 0) {
            System.out.println("The equation has two roots " +
                    equation.getRoot1() + " and " + equation.getRoot2());
        } else if (discriminant == 0) {
            System.out.println("The equation has one root " +
                    equation.getRoot1());
        } else {
            System.out.println("The equation has no roots.");
        }

        input.close();
    }
}
```

# Chapter 10 Object-Oriented Thinking

## Programming Exercise:

### 10.3 (The MyInteger class) Design a class named MyInteger. The class contains:

```
■ An int data field named value that stores the int value represented by this object.
■ A constructor that creates a MyInteger object for the specified int value.
■ A getter method that returns the int value.
■ The methods isEven(), isOdd(), and isPrime() that return true if the value in this object is even, odd, or prime, respectively.
■ The static methods isEven(int), isOdd(int), and isPrime(int) that return true if the specified value is even, odd, or prime, respectively.
■ The static methods isEven(MyInteger), isOdd(MyInteger), and isPrime(MyInteger) that return true if the specified value is even, odd, or prime, respectively.
■ The methods equals(int) and equals(MyInteger) that return true if the value in this object is equal to the specified value.
■ A static method parseInt(char[]) that converts an array of numeric characters to an int value.
■ A static method parseInt(String) that converts a string into an int value.
```

Draw the UML diagram for the class and then implement the class. Write a client program that tests all methods in the class. 

```markdown
---------------------------------------------------
|                   MyInteger                     |
---------------------------------------------------
- value: int  
---------------------------------------------------
+ MyInteger(value: int)  
+ getValue(): int  
+ isEven(): boolean  
+ isOdd(): boolean  
+ isPrime(): boolean  
+ static isEven(value: int): boolean  
+ static isOdd(value: int): boolean  
+ static isPrime(value: int): boolean  
+ static isEven(myInt: MyInteger): boolean  
+ static isOdd(myInt: MyInteger): boolean  
+ static isPrime(myInt: MyInteger): boolean  
+ equals(value: int): boolean  
+ equals(myInt: MyInteger): boolean  
+ static parseInt(chars: char[]): int  
+ static parseInt(str: String): int  
---------------------------------------------------
```

MyInteger.java

```java
public class MyInteger {
    private int value;

    // 构造方法
    public MyInteger(int value) {
        this.value = value;
    }

    // Getter
    public int getValue() {
        return value;
    }

    // 实例方法：判断偶数
    public boolean isEven() {
        return value % 2 == 0;
    }

    // 实例方法：判断奇数
    public boolean isOdd() {
        return value % 2 != 0;
    }

    // 实例方法：判断质数
    public boolean isPrime() {
        if (value < 2) return false;
        for (int i = 2; i <= Math.sqrt(value); i++) {
            if (value % i == 0) return false;
        }
        return true;
    }

    // 静态方法：判断偶数
    public static boolean isEven(int value) {
        return value % 2 == 0;
    }

    // 静态方法：判断奇数
    public static boolean isOdd(int value) {
        return value % 2 != 0;
    }

    // 静态方法：判断质数
    public static boolean isPrime(int value) {
        if (value < 2) return false;
        for (int i = 2; i <= Math.sqrt(value); i++) {
            if (value % i == 0) return false;
        }
        return true;
    }

    // 静态方法：判断偶数（MyInteger对象）
    public static boolean isEven(MyInteger myInt) {
        return myInt.isEven();
    }

    // 静态方法：判断奇数（MyInteger对象）
    public static boolean isOdd(MyInteger myInt) {
        return myInt.isOdd();
    }

    // 静态方法：判断质数（MyInteger对象）
    public static boolean isPrime(MyInteger myInt) {
        return myInt.isPrime();
    }

    // 判断相等（int值）
    public boolean equals(int anotherValue) {
        return this.value == anotherValue;
    }

    // 判断相等（MyInteger对象）
    public boolean equals(MyInteger anotherInt) {
        return this.value == anotherInt.getValue();
    }

    // 静态方法：char数组转int
    public static int parseInt(char[] chars) {
        int result = 0;
        for (char c : chars) {
            result = result * 10 + (c - '0');
        }
        return result;
    }

    // 静态方法：字符串转int
    public static int parseInt(String str) {
        return Integer.parseInt(str);
    }
}
```

TestMyInteger.java

```java
public class TestMyInteger {
    public static void main(String[] args) {
        MyInteger myInt = new MyInteger(17);

        System.out.println("Value: " + myInt.getValue());
        System.out.println("Is Even? " + myInt.isEven());
        System.out.println("Is Odd? " + myInt.isOdd());
        System.out.println("Is Prime? " + myInt.isPrime());

        System.out.println("Static isEven(20): " + MyInteger.isEven(20));
        System.out.println("Static isOdd(20): " + MyInteger.isOdd(20));
        System.out.println("Static isPrime(20): " + MyInteger.isPrime(20));

        MyInteger anotherInt = new MyInteger(20);
        System.out.println("Static isEven(MyInteger 20): " + MyInteger.isEven(anotherInt));

        System.out.println("Equals 17? " + myInt.equals(17));
        System.out.println("Equals anotherInt (20)? " + myInt.equals(anotherInt));

        char[] charArray = {'1', '2', '3'};
        System.out.println("parseInt(char[] {'1','2','3'}): " + MyInteger.parseInt(charArray));
        System.out.println("parseInt(String \"456\"): " + MyInteger.parseInt("456"));
    }
}
```

### 10.11 (Geometry: the Circle2D class) Define the Circle2D class that contains:

```
■ Two double data fields named x and y that specify the center of the circle with getter methods.
■ A data field radius with a getter method.
■ A no-arg constructor that creates a default circle with (0, 0) for (x, y) and 1 for radius.
■ A constructor that creates a circle with the specified x, y, and radius.
■ A method getArea() that returns the area of the circle.
■ A method getPerimeter() that returns the perimeter of the circle.
■ A method contains(double x, double y) that returns true if the specified point (x, y) is inside this circle (see Figure 10.21a).
■ A method contains(Circle2D circle) that returns true if the specified circle is inside this circle (see Figure 10.21b).
■ A method overlaps(Circle2D circle) that returns true if the specified circle overlaps with this circle (see Figure 10.21c).
```

![image-20250609211400683](C:/Users/42995004/AppData/Roaming/Typora/typora-user-images/image-20250609211400683.png)

Draw the UML diagram for the class and then implement the class. Write a test program that creates a Circle2D object c1 (new Circle2D(2, 2, 5.5)), displays its area and perimeter, and displays the result of c1.contains(3, 3), c1.contains(new Circle2D(4, 5, 10.5)), and c1.overlaps(new Circle2D(3, 5, 2.3)).

```markdown
----------------------------------------------------
|                    Circle2D                      |
----------------------------------------------------
- x: double  
- y: double  
- radius: double  
----------------------------------------------------
+ Circle2D()  
+ Circle2D(x: double, y: double, radius: double)  
+ getX(): double  
+ getY(): double  
+ getRadius(): double  
+ getArea(): double  
+ getPerimeter(): double  
+ contains(x: double, y: double): boolean  
+ contains(circle: Circle2D): boolean  
+ overlaps(circle: Circle2D): boolean  
----------------------------------------------------
```

Circle2D.java

```java
public class Circle2D {
    private double x;
    private double y;
    private double radius;

    // 无参构造方法
    public Circle2D() {
        this.x = 0;
        this.y = 0;
        this.radius = 1;
    }

    // 带参构造方法
    public Circle2D(double x, double y, double radius) {
        this.x = x;
        this.y = y;
        this.radius = radius;
    }

    // Getter方法
    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }

    public double getRadius() {
        return radius;
    }

    // 计算面积
    public double getArea() {
        return Math.PI * radius * radius;
    }

    // 计算周长
    public double getPerimeter() {
        return 2 * Math.PI * radius;
    }

    // 判断点是否在圆内
    public boolean contains(double x, double y) {
        double distance = Math.hypot(this.x - x, this.y - y);
        return distance <= this.radius;
    }

    // 判断圆是否完全包含另一个圆
    public boolean contains(Circle2D circle) {
        double distance = Math.hypot(this.x - circle.x, this.y - circle.y);
        return distance + circle.radius <= this.radius;
    }

    // 判断两个圆是否相交
    public boolean overlaps(Circle2D circle) {
        double distance = Math.hypot(this.x - circle.x, this.y - circle.y);
        return distance < this.radius + circle.radius &&
               distance + Math.min(this.radius, circle.radius) > Math.max(this.radius, circle.radius);
    }
}
```

TestCircle2D.java

```java
public class TestCircle2D {
    public static void main(String[] args) {
        Circle2D c1 = new Circle2D(2, 2, 5.5);

        System.out.printf("Area: %.2f\n", c1.getArea());
        System.out.printf("Perimeter: %.2f\n", c1.getPerimeter());

        System.out.println("c1.contains(3, 3): " + c1.contains(3, 3));
        System.out.println("c1.contains(new Circle2D(4, 5, 10.5)): " +
                c1.contains(new Circle2D(4, 5, 10.5)));
        System.out.println("c1.overlaps(new Circle2D(3, 5, 2.3)): " +
                c1.overlaps(new Circle2D(3, 5, 2.3)));
    }
}
```

# Chapter 11 Inheritance and Polymorphism

## Check Point:

### 11.7.1 What are the three pillars of object - oriented programming? What is polymorphism?

**面向对象编程的三大支柱：**

1. **封装（Encapsulation）**：隐藏对象内部细节，通过方法控制数据访问。
2. **继承（Inheritance）**：子类复用父类的属性和方法，实现代码复用。
3. **多态（Polymorphism）**：同一操作作用于不同对象时表现出不同行为。

**多态的含义**：

- 表现方式：
  - **编译时多态**：方法重载（Overload）
  - **运行时多态**：方法重写（Override）+ 父类引用指向子类对象（如 `Animal a = new Dog()`）
- **核心作用**：提高代码扩展性，实现“同一接口，不同实现”。

### 11.1 True or false? A subclass is a subset of a superclass.

**错误（False）**

**原因**：

- 子类（subclass）**扩展**（extends）了父类（superclass），而非“子集”。
- 子类包含父类的属性和方法，并可以**新增或修改**功能（如添加新方法、重写父类方法）。

### 11.3 What is single inheritance? What is multiple inheritance? Does Java support multiple inheritance?

**单继承（Single Inheritance）**：

- 一个子类只能继承**一个父类**（如 `class B extends A`）。

**多继承（Multiple Inheritance）**：

- 一个子类可以继承**多个父类**（如 `class C extends A, B`）。

**Java 是否支持多继承？**

- **不支持类之间的多继承**（避免“菱形问题”带来的复杂性）。
- **通过接口（Interface）实现多继承**（如 `class C implements A, B`）。

### 11.5 How does a subclass invoke its superclass’s constructor?

**子类调用父类构造方法的方式：**

1. **隐式调用**：
   - 如果子类构造方法**没有显式调用** `super(...)`，编译器会**自动调用父类的无参构造方法**（若父类无无参构造，则编译报错）。
2. **显式调用**：
   - 使用 `super(参数)` **手动调用父类的指定构造方法**，必须放在子类构造方法的**第一行**。

### 11.6 True or false? When invoking a constructor from a subclass, its superclass’s no-arg constructor is always invoked.

**错误（False）**

**原因**：

1. **显式调用时**：如果子类构造方法**手动调用了父类的有参构造**（如 `super(参数)`），则不会自动调用父类的无参构造。
2. **父类无无参构造时**：若父类**没有无参构造**，子类必须显式调用父类的其他构造方法，否则编译报错。

### 11.24 Indicate true or false for the following statements: 

```
■ You can always successfully cast an instance of a subclass to a superclass. 
■ You can always successfully cast an instance of a superclass to a subclass.
```

**判断以下陈述的正误：**

1. **✅ 将子类实例转换为父类（向上转型）总是成功**

   - 原因：子类“是”父类（`Dog`是`Animal`），无需强制类型转换即可自动完成。

   - 示例：

     ```java
     Animal a = new Dog(); // 直接赋值，合法  
     ```

2. **❌ 将父类实例转换为子类（向下转型）不一定成功**

   - 原因：父类对象可能不是目标子类的实例，运行时可能抛出 `ClassCastException`。

   - **安全做法**：先用 `instanceof` 检查类型。

   - 示例：

     ```java
     Animal a = new Animal();  
     Dog d = (Dog) a; // 运行时错误！  
     ```

     ```java
     Animal a = new Dog();  
     if (a instanceof Dog) {  
         Dog d = (Dog) a; // 安全转换  
     }  
     ```

### 11.28 Does every object have a toString method and an equals method? Where do they come from? How are they used? Is it appropriate to override these methods?

**1. 是否所有对象都有 `toString()` 和 `equals()` 方法？**
✅ ​**​是的​**​。所有 Java 对象都继承自 `Object` 类，因此默认拥有这两个方法。

**2. 这些方法的来源**

- 来自 **`Object` 类**（所有类的超类）。
- 默认实现：
  - `toString()`：返回类名@哈希码（如 `Dog@1a2b3c`）。
  - `equals()`：比较对象地址（等同于 `==`）。

**3. 用途**

- `toString()`：
  - 打印对象信息（如 `System.out.println(obj)` 会自动调用）。
  - **建议重写**以返回有意义的字符串（如 `"Dog[name=Tom]"`）。
- `equals()`：
  - 比较对象内容是否逻辑相等（如两个学生对象的学号相同）。
  - **必须重写**以自定义相等规则（同时需重写 `hashCode()`）。

**4. 是否需要重写？**

- ✅ 通常需要重写：
  - `toString()`：提供可读性更好的输出。
  - `equals()`：实现内容比较而非地址比较。

### 11.42 Indicate true or false for the following statements: 

```
a. A protected datum or method can be accessed by any class in the same package.
b. A protected datum or method can be accessed by any class in different packages.
c. A protected datum or method can be accessed by its subclasses in any package.
d. A final class can have instances.
e. A final class can be extended.
f. A final method can be overridden.
```

**判断以下陈述的正误：**

**a. ✅ 正确**

- `protected` 成员可以被**同一包中的任何类**访问。

**b. ❌ 错误**

- `protected` 成员**不能**被不同包中的无关类访问（仅允许子类访问）。

**c. ✅ 正确**

- `protected` 成员可以**被任何包中的子类**访问（继承权限）。

**d. ✅ 正确**

- `final` 类**可以实例化**（如 `String` 是 `final` 类，但能创建对象）。

**e. ❌ 错误**

- `final` 类**不能被继承**（如 `public final class Math { }` 不可扩展）。

**f. ❌ 错误**

- `final` 方法**不能被子类重写**（锁定方法实现，禁止修改）。

## Programming Exercise:

### 11.1 (The Triangle class) Design a class named Triangle that extends GeometricObject. The class contains:

```
■ Three double data fields named side1, side2, and side3 with default values 1.0 to denote three sides of the triangle.
■ A no-arg constructor that creates a default triangle.
■ A constructor that creates a triangle with the specified side1, side2, and side3.
■ The accessor methods for all three data fields.
■ A method named getArea() that returns the area of this triangle.
■ A method named getPerimeter() that returns the perimeter of this triangle.
■ A method named toString() that returns a string description for the triangl
```

For the formula to compute the area of a triangle, see Programming Exercise 2.19. The toString() method is implemented as follows:  

```java
return "Triangle: side1 = " + side1 + " side2 = " + side2 + " side3 = " + side3;
```

Draw the UML diagrams for the classes Triangle and GeometricObject and implement the classes. Write a test program that prompts the user to enter three sides of the triangle, a color, and a Boolean value to indicate whether the triangle is filled. The program should create a Triangle object with these sides and set the color and filled properties using the input. The program should display the area, perimeter, color, and true or false to indicate whether it is filled or not.

**GeometricObject**

```markdown
-------------------------------------------------
|               GeometricObject                |
-------------------------------------------------
- color: String  
- filled: boolean  
- dateCreated: java.util.Date  
-------------------------------------------------
+ GeometricObject()  
+ GeometricObject(color: String, filled: boolean)  
+ getColor(): String  
+ setColor(color: String): void  
+ isFilled(): boolean  
+ setFilled(filled: boolean): void  
+ getDateCreated(): java.util.Date  
+ toString(): String  
-------------------------------------------------
```

**Triangle**

```markdown
----------------------------------------
|              Triangle                |
----------------------------------------
- side1: double = 1.0  
- side2: double = 1.0  
- side3: double = 1.0  
----------------------------------------
+ Triangle()  
+ Triangle(side1: double, side2: double, side3: double)  
+ getSide1(): double  
+ getSide2(): double  
+ getSide3(): double  
+ getArea(): double  
+ getPerimeter(): double  
+ toString(): String  
----------------------------------------
```

**GeometricObject.java**

```java
import java.util.Date;

public class GeometricObject {
    private String color = "white";
    private boolean filled;
    private Date dateCreated;

    public GeometricObject() {
        dateCreated = new Date();
    }

    public GeometricObject(String color, boolean filled) {
        dateCreated = new Date();
        this.color = color;
        this.filled = filled;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public boolean isFilled() {
        return filled;
    }

    public void setFilled(boolean filled) {
        this.filled = filled;
    }

    public Date getDateCreated() {
        return dateCreated;
    }

    public String toString() {
        return "created on " + dateCreated + "\ncolor: " + color + " and filled: " + filled;
    }
}
```

**Triangle.java**

```java
public class Triangle extends GeometricObject {
    private double side1 = 1.0;
    private double side2 = 1.0;
    private double side3 = 1.0;

    public Triangle() {
    }

    public Triangle(double side1, double side2, double side3) {
        this.side1 = side1;
        this.side2 = side2;
        this.side3 = side3;
    }

    public double getSide1() {
        return side1;
    }

    public double getSide2() {
        return side2;
    }

    public double getSide3() {
        return side3;
    }

    public double getArea() {
        double s = (side1 + side2 + side3) / 2;
        return Math.sqrt(s * (s - side1) * (s - side2) * (s - side3));
    }

    public double getPerimeter() {
        return side1 + side2 + side3;
    }

    public String toString() {
        return "Triangle: side1 = " + side1 + " side2 = " + side2 + " side3 = " + side3;
    }
}
```

**测试程序**

**TestTriangle.java**

```java
import java.util.Scanner;

public class TestTriangle {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);

        System.out.print("Enter side1, side2, side3 of the triangle: ");
        double side1 = input.nextDouble();
        double side2 = input.nextDouble();
        double side3 = input.nextDouble();

        System.out.print("Enter the color of the triangle: ");
        String color = input.next();

        System.out.print("Is the triangle filled (true/false)? ");
        boolean filled = input.nextBoolean();

        Triangle triangle = new Triangle(side1, side2, side3);
        triangle.setColor(color);
        triangle.setFilled(filled);

        System.out.println("\n" + triangle);
        System.out.printf("Area: %.2f\n", triangle.getArea());
        System.out.printf("Perimeter: %.2f\n", triangle.getPerimeter());
        System.out.println("Color: " + triangle.getColor());
        System.out.println("Filled: " + triangle.isFilled());
    }
}
```

