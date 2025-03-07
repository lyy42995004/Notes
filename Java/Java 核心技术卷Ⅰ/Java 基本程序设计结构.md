# 一个简单的java应用程序

```java
public class FirstSample {
    public static void main(String[] args) {
        System.out.println("Hello Java!");
    }
}
```

- Java 区分大小写，`main`方法拼写必须正确，否则程序无法运行。`public`是访问修饰符，`class`表明程序内容在类中，Java 应用程序所有内容都在类里。
- 类名命名规则宽松，但须以字母开头，不能用 Java 保留字。标准命名用骆驼命名法，源文件名要和公共类名相同且扩展名为`.java`。编译后生成同名`.class`文件，运行程序用`java`命令且不加`.class`扩展名。
- 按 Java 语言规范，`main`方法须声明为`public`，不过早期有些编译器在非`public`时也能运行，Java 1.4 及以后版本强制要求。每个 Java 应用程序都必须有`main`方法，其标准格式固定。

> **C++注释**：Java 中所有函数都是类的方法，`main`方法必须在一个外壳类中且是静态的，这点与 C++ 静态成员函数不同。和 C/C++ 一样，`void`表示方法无返回值，Java 中`main`方法正常退出无返回值，若要在终止程序时返回其他退出码，需用`System.exit`方法。

# 注释

- **单行注释**：最常用的是使用 “//”，注释内容从 “//” 开始到本行结尾。
- **多行注释**：当需要较长注释时，可在每行注释前标记 “//”，也能用 “/\*” 和 “\*/” 界定注释，将一段内容注释起来。
- **文档注释**：第三种注释以 “/\*\*” 开始，以 “\*/” 结束。

# 数据类型

Java 是强类型语言，每个变量都必须声明类型，共有 8 种基本类型，包括 4 种整型、2 种浮点类型、1 种字符类型`char`和 1 种表示真值的`boolean`类型，此外还有 “大数” 这种 **Java 对象**。

**整形**

![](https://s2.loli.net/2025/03/06/YfXxo2zGcpNI5Vn.png)

> **C++ 注释**： C 和 C++ 中整型大小与目标平台相关，而 Java 的数值类型则与平台无关，且 Java 没有无符号整型形式。

**浮点类型**

![image.png](https://s2.loli.net/2025/03/06/xHswW3Um65hqokj.png)

**char 类型**

- 原本用于表示单个字符，现有些 Unicode 字符需两个`char`值表示。

**Unicode 和 char 类型**

- Java 采用 Unicode 编码，解释了码点、代码平面等概念，以及`char`类型在处理 Unicode 字符时的情况 。

**boolean 类型**

- 只有`false`和`true`两个值，用于判定逻辑条件，不能和整数类型转换。

> **C++注释**：在 C++ 中，数值甚至指针可以替代`boolean`值，值`0`相当于布尔值`false`，非`0`值相当于布尔值`true` 。在 Java 中，不能像 C++ 那样将整数表达式转换为布尔值。例如`if (x = 0)`在 Java 中不能通过编译，因为 Java 严格区分数据类型，不允许这种隐式的类型转换 。

# 变量和常量

**声明变量**

- 声明变量时要先指定类型，后写变量名，每个声明以分号结束。

- 变量名由字母或数字构成，以字母开头，区分大小写。
- 不能用 Java 保留字，Java 9 后单下划线不能作变量名。

- 虽然可一行声明多个变量，但逐一声明可提高可读性。

```java
double salary;
```

**变量初始化**

- 声明后需显式初始化变量，赋值时变量名在等号左侧，值在右侧，也可将声明和初始化放同一行。
- 变量声明应靠近首次使用的地方。
- 从 Java 10 开始，局部变量声明可使用`var`关键字，由系统根据初始值推断类型。

> **C++注释**：C 和 C++ 区分变量声明与定义，`int i  =  10;`是一个定义，而`extern int i`是一个声明；Java 不区分。
>
> Java `var` 仅用于局部变量；C++ `auto` 用途更广，可用于局部变量、函数返回类型、模板编程等。

```java
// double salary;
// salary = 65000.0;
double salary = 65000.0;
```

**常量**

- 用`final`关键字指示常量，赋值后不能更改，常量名通常全大写。
- 若希望常量在类的多个方法中使用，可用`static final`设置为类常量，其定义在`main`方法外部，若声明为`public`，其他类的方法也能使用。

> **C++注释**：`const`是 Java 保留关键字，但目前未使用，需用`final`定义常量。

```java
public class Constants {
    final double CM_PER_INCH1 = 2.54;
    
    public static void main(String[] args) {
        final double CM_PER_INCH2 = 2.54;
    }
}
```

**枚举类型**

- 当变量取值在有限集合内时，可自定义枚举类型。
- 枚举类型包含有限个命名值，声明后可声明该类型的变量，变量只能存储给定的枚举值或特殊值`null` 。

```java
enum Size { SMALL, MEDIUM, LAGRE, EXTRA_LAGRE};
```

# 运算符

**算术运算符**

- 在 Java 中，“+、-、*、/” 分别表示加、减、乘、除运算。
- 整数被 0 除会产生异常，浮点数被 0 除结果为无穷大或 NaN。

> **Java 运算的可移植性与浮点计算细节**：可移植性是 Java 语言设计目标之一，但实现浮点数算术运算的可移植性较困难，因为不同处理器存储浮点数的精度有差异 。Java 虚拟机最初规范要求中间计算截断，后因会导致溢出且耗时，在默认情况下允许中间计算采用扩展精度。使用`strictfp`关键字标记方法或类，其中的浮点计算会采用严格的标准，使用严格计算可能产生溢出，而默认方式一般可避免溢出。

**数学函数与常量**

- `Math`类包含各种数学函数，如计算平方根用`sqrt`方法。

- `Math`类中的`sqrt`是静态方法。求幂用`pow`方法，`floorMod`可处理一些取模问题 。

**数值类型之间的转换**

<img src="https://s2.loli.net/2025/03/06/9AkT1J2YGciwtVv.png" alt="image.png" style="zoom:67%;" />

上面展示了数值类型间的合法转换路径，如小范围整数类型可自动转换为大范围整数类型或浮点数类型，转换可能有精度损失 。

**强制类型转换**

- 必要时`int`会自动转换为`double`，但`double`转`int`可能丢失信息，需强制类型转换，格式为在圆括号中给出目标类型，紧跟待转换变量名。
- 使用`Math.round`方法后仍需强制类型转换取整 。

```java
double x = 9.997;
int nx1 = (int)x; // 9

int nx2 = (int)Math.round(x); // 10
```

> **C++注释**：不要在`boolean`类型与任何数值类型之间进行强制类型转换，这样做能够防止出现一些常见的错误。只有在极少数情况下，才需要将布尔类型转换为数值类型 ，这时可以使用条件表达式 `b?1:0`。

**运算赋值和运算符**

- 在赋值中使用二元运算符简化代码，如`x += 4`等价于`x = x + 4`，运算符在`=`左边类型不同时会发生强制类型转换。

**自增与自减运算符**

- `n++`使`n`值加 1，`n--`使`n`值减 1，有前缀和后缀两种形式，前缀形式会先完成加，后缀形式会使用变量原来的值。

**关系和 boolean 运算符**

- 用`==`检测相等性，`!=`检测不相等，还有`<`、`>`、`<=`、`>=`等关系运算符，`&&`和`||`是逻辑运算符，遵循 “短路” 原则 。

**三元运算符**

- 格式为`condition ? expression1 : expression2`，条件为真返回`expression1`的值，否则返回`expression2`的值 。

**位运算符**

- 处理整型时可对二进制位操作，包括按位与（`&`）、按位或（`|`）、按位异或（`^`）、取反（`~`），还有左移（`<<`）和右移（`>>`、`>>>`）运算符，移位操作时需注意数据类型和符号位 。

- `>>>`会用 0 填充高位，这与`>>`不同，它会用符号位填充高位。不存在`<<<`运算符。


> **C++注释**：在 C/C++ 里，右移运算符`>>`存在不确定性，无法保证是进行算术移位（扩展符号位）还是逻辑移位（填充 0）。而 Java 消这种不确定性。

**括号与运算符优先级**

> **C++注释**：与 C 或 C++ 不同，Java不使用逗号运算符。不过，可以在for语句的第1部分和第3部分使用逗号分隔表达式列表。

![image.png](https://s2.loli.net/2025/03/06/mOrfxKvXMSbQhAI.png)

# 字符串

Java 字符串是 Unicode 字符序列，由标准类库中的`String`类表示，用双引号括起来的都是`String`类实例，如`String e = ""`（空串）和`String greeting = "Hello"` 。

**子串**

- `String`类的`substring`方法可从大字符串提取子串，该方法从索引 0 开始计数，第二个参数是不复制的第一个位置，子串长度为两参数差值。

> **C++注释**：类似与 C 和 C++，Java字符串中的代码单元从 0 开始计数。

```java
String greeting = "Hello";
String s = greeting.substring(0, 3); // Hel
```

**拼接**

- 可使用`+`连接两个字符串，非字符串值会转换为字符串后拼接 。
- Java 11 新增`join`方法，用界定符分隔多个字符串；`repeat`方法可重复字符串 。

```java
int age = 13;
String rating = "PG" + 	age; // PG13

String all = String.join(" / ", "S", "M", "L", "XL"); // S/M/L/XL

String repeated = "Java".repeat(3); // JavaJavaJava		
```

**不可变字符串**

- `String`类没有直接修改字符串中字符的方法，如需修改，通常提取保留子串再与替换字符拼接。

- Java 中`String`类是不可变的，字符串存于公共存储池，共享带来的效率优势大于拼接等操作的低效。

```java
greeting = greeting.substring(0, 3) + "p!";
```

> **C++注释**： Java 字符串不是字符数组，实际类似`char*`指针，赋值操作会重新指向新字符串，原字符串由系统自动回收。C++ 中`string`对象也能自动管理内存，但 C++ 字符串的单个字符可修改，与 Java 不同 。

**检测字符串是否相等**

- 可使用`equals`方法判断两个字符串内容是否相等，字符串变量或字面量都能参与比较，相等返回`true`，否则返回`false`。
- 若要不区分大小写比较，可使用`equalsIgnoreCase`方法。
- 强调不能用`==`运算符检测字符串相等，因为`==`判断的是字符串是否存于同一位置，而内容相同的字符串副本可能在不同位置。

```java
String greeting = "Hello";
if (greeting == "Hello") // probably true
if (greeting.substring(0, 3) == "Hel") // probably false
```

> **C++ 注释**：在 Java 中需注意，Java 没有像 C++ 那样重载`==`运算符来检测字符串内容相等，其`==`更像指针比较；C 语言用`strcmp`函数比较字符串，Java 的`compareTo`方法类似，但使用`equals`方法更清晰 。

**空串与 Null 串**

- 空串是长度为 0 的字符串，可通过`str.length() == 0`或`str.equals("")`来判断一个字符串是否为空串。
- `null`表示字符串变量未关联任何对象，检查字符串是否为`null`用`if (str == null)` 。同时检查字符串既非`null`又非空串，需使用`if (str != null && str.length() != 0)` ，在`null`值上调用方法会出错。

**码点与代码单元**

- Java 字符串由`char`值序列组成，`char`类型采用 UTF - 16 编码表示 Unicode 码点，多数常用 Unicode 字符用一个代码单元表示，辅助字符需两个。
- `length`方法返回的是代码单元数量，获取实际长度（码点数量）用`codePointCount`方法 。`charAt`方法返回指定位置的代码单元，获取指定位置码点用`codePointAt`方法。

**String 类 API**

- Java 的`String`类包含 50 多个方法，实用性强、使用频率高。

**阅读联机 API 文档**

- [Java SE 9 API](https://docs.oracle.com/javase/9/docs/api/overview-summary.html)

**构建字符串**

- 需要由较短字符串构建字符串，若用普通字符串拼接方式，每次拼接都会创建新的`String`对象，效率低且浪费空间。而`StringBuilder`类可避免该问题。

```java
StringBuilder builder = new StringBuilder();
builder.append(ch);
builder.append(str);
String completedString = builder.toString();
```

> `StringBuilder`类在 Java 5 中引入，其前身是`StringBuffer`，`StringBuffer`效率稍低，但允许多线程方式添加或删除字符。若所有字符串编辑操作都在单线程中执行，应使用`StringBuilder`，且这两个类的 API 相同。

# 输入和输出

**读取输入**

- 在 Java 中，输出到控制台很容易，调用`System.out.println`即可，但读取 “标准输入流”`System.in`则需构造与`System.in`关联的`Scanner`对象（`Scanner in = new Scanner(System.in);`） 。
- 可以使用`Scanner`类的多种方法读取输入，如`nextLine`读取一行，`nextInt`读取一个整数，`nextDouble`读取一个浮点数等。同时要注意导入`java.util.*`包。

> 因为输入是可见的，`Scanner`类不适用于读取密码，Java 6 引入`Console`类实现此目的，其`readPassword`方法返回的密码存于字符数组而非字符串，以增强安全性。

**格式化输出**

- 使用`System.out.print(x)`可将数值`x`输出到控制台，按`x`类型允许的最大非 0 位数打印。

**文件输入与输出**

- 文件读取：读取文件时，先构造`Scanner`对象，如`Scanner in = new Scanner(Path.of("myfile.txt"), "UTF-8")` ，若文件名含反斜杠需转义。

- 文件写入：写入文件需构造`PrintWriter`对象，如`PrintWriter out = new PrintWriter("myfile.txt", "UTF-8")` ，若文件不存在则创建。可像使用`System.out`一样使用`print`、`println`等方法进行输出。

# 控制流程

**块作用域**

- 块（复合语句）由多条 Java 语句组成，用大括号括起，它定义了变量的作用域，一个块可以嵌套在另一个块中，但不能在嵌套的两个块中声明同名变量。

> **C++注释**：在C++中，可以在嵌套块中重定义一个变量，在内存定义的变量会覆盖 外层定义的变量，Java 中不能允许这样做。

**条件语句**

- **if 语句**：形式为`if (condition) statement`，当条件为真时执行语句，若要执行多条语句，需用大括号括起来形成块语句。
- **if/else 语句**：形式是`if (condition) statement1 else statement2`，当条件为真执行`statement1`，否则执行`statement2` ，`else`部分是可选的。在多个`if/else`嵌套时，`else`与最邻近的`if`配对，使用大括号可使代码更清晰。

**循环**

- **while 循环**：形式为`while (condition) statement`，条件为真时循环执行语句（可以是块语句），若开始时条件就为假，则循环一次也不执行。
- **do/while 循环**：形式为`do statement while (condition);`，先执行语句，再检测循环条件，条件为真则重复执行，适用于至少要执行一次循环体的情况。

**确定循环**

-  for 循环的基本语法和使用方式，如`for (int i = 10; i > 0; i--)` 。
- 在 for 语句第 1 部分声明的变量，作用域扩展到循环体末尾，且该变量不能在循环体外使用。
- 不同 for 循环中可定义同名变量，for 循环是 while 循环的简化形式，两者可相互改写。

**多重选择：switch 语句**

- 当处理多重选择时，使用 switch 语句比 if/else 结构更合适。
- switch 语句从与选项值匹配的 case 标签开始执行，遇到 break 语句或 switch 结束处停止。若无匹配 case 标签且有 default 子句，则执行 default 子句。case 分支若缺少 break 语句会导致意外执行后续分支，存在风险。
- case 标签可以是 char、byte、short、int 的常量表达式，从 Java 7 开始还可以是字符串字面量。使用枚举类型时，switch 表达式可直接得出枚举值 。

**中断控制流程的语句**

- Java 中保留 goto 关键字但未使用其作为程序设计语句，因为随意使用 goto 可能使程序逻辑混乱，不过在某些复杂情况下，其他语言（如 C）可能会用到类似功能。
- break 语句用于跳出循环或 switch 语句，可带标签用于跳出多重嵌套循环；continue 语句用于中断当前循环体剩余部分，跳到循环首部继续执行 。

# 大数

当基本整数和浮点数精度无法满足需求时，可使用`java.math`包中的`BigInteger`和`BigDecimal`类。`BigInteger`实现任意精度的整数运算，`BigDecimal`实现任意精度的浮点数运算。

# 数组

**声明数组**

- 数组用于存储相同类型值的序列，是一种数据结构，通过整型下标访问元素。
- 声明数组变量时需指出数组类型（数据元素类型紧跟`[]`）和变量名，如`int[] a;` 。
- 使用`new`操作符创建数组，如`int[] a = new int[100];` ，数组长度不必是常量。
- 可采用简写形式在创建数组时初始化，如`int[] smallPrimes = { 2, 3, 5, 7, 11, 13 };` ，还能重新初始化数组而无需创建新变量。
- Java 允许创建长度为 0 的数组，其与`null`不同。

**访问数组元素**

- 数组元素下标从 0 开始，创建数组后可填充元素。
- 数字数组元素初始值为 0，`boolean`数组为`false`，对象数组为`null`。
- 可通过`array.length`获取数组元素个数。
- 访问数组时若下标越界，会引发 “`array index out of bounds`” 异常。

**for each 循环**

- 功能很强的循环结构，用于依次处理数组或实现了`Iterable`接口的集合中的每个元素，无需考虑下标值。语法为`for (variable : collection) statement` 。

```java
for (int element : a)
    System.out.println(element);
```

**数组拷贝**

- 直接将一个数组变量赋值给另一个数组变量，两个变量将引用同一个数组。
- 若要将一个数组的所有值拷贝到新数组，需使用`Arrays`类的`copyOf`方法 ，该方法也常用于增加数组大小。
- 若新数组长度大于原数组，数值型元素额外部分赋值为 0，布尔型赋值为`false`；若新数组长度小于原数组，则只拷贝前面的值 。

```java
int[] LuckNumbers = smallPrimes;
LuckNumbers[5] = 12;

int[] copiedLuckyNumbers = Arrays.copyOf(luckyNumbers, luckyNumbers.length);

LuckyNumbers = Arrays.copyOf(luckyNumbers, 2 * luckyNumbers.length);
```

![image.png](https://s2.loli.net/2025/03/06/lOLf8rgKdCjSaRx.png)

> **C++注释**：Java 数组与 C++ 堆栈上的数组不同，Java 数组类似于在堆上分配的数组指针。且 Java 的`[]`运算符预定义了越界检查，没有指针运算 。

**命令行参数**

- 每个 Java 应用程序都有一个带`String[] args`参数的`main`方法，该参数是一个字符串数组，用于接收命令行上指定的参数。

> **C++注释**：在 Java 应用程序的`main`方法中，`args`数组用于接收命令行参数，程序名并不存储在`args`数组中。

**数组排序**

- 使用`Arrays`类的`sort`方法可对数值型数组进行排序，该方法采用优化的快速排序算法。

```java
Array.sort(result);
```

**多维数组**

- 多维数组通过多个下标访问元素，适用于表示表格或复杂排列的数据。

```java
double[][] balances;
balances = new double[YEARS][RATES];

int[][] magicSquare = { 
    {16, 3, 2, 13}, 
    {5, 10, 11, 8}, 
    {9, 6, 7, 12}, 
    {4, 15, 14, 1} };
```

**不规则数组**

- Java 实际上没有真正的多维数组，而是由数组组成的数组。

> **C++注释**： Java 中二维数组的声明方式`double[][] balances = new double[10][6];` 。
>
> **不同**与 C++ 中，`double balances[10][6];`的声明方式与 Java 不同；也不同与`(*balances)[6] = new double[10];`**而类似于**：
>
> ```cpp
> double** balances = new double*[10];
> for (i = 0; i < 10; i++) 
> 	balances[i] = new double[6];
> ```
