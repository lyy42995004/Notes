# 语法

lambda 表达式是一种匿名函数，它允许在代码中直接定义和使用函数对象，而无需像传统函数那样先声明再定义。

```C++
[capture list] (parameter list) -> return type { function body }
```

- **捕获列表**：用于指定 lambda 表达式可以访问的外部变量。捕获列表可以为空，表示不捕获任何外部变量，也可以包含变量名，以指定要捕获的变量。捕获方式有值捕获（`[var]`）和引用捕获（`[&var]`）等。
- **参数列表**：与普通函数的参数列表类似，用于指定 lambda 表达式接受的参数。可以为空，表示该 `lambda` 表达式不接受任何参数。
- **返回类型**：指定 lambda 表达式的返回值类型。如果省略，则编译器会根据函数体中的返回语句自动推断返回类型。
- **函数体**：包含了 lambda 表达式要执行的具体操作，与普通函数的函数体类似。

# 捕获方式

- **值捕获**：值捕获是指 lambda 表达式在创建时会复制外部变量的值，在 lambda 表达式内部对该变量的修改不会影响到外部变量。

```C++
int x = 10;
auto f = [x] (int y) -> int { return x + y; }; // 值捕获 x
x = 20; // 修改外部的 x
cout << f(5) << endl; // 输出 15，不受外部 x 的影响
```

- **引用捕获**：引用捕获是指 lambda 表达式在创建时会捕获外部变量的引用，在 lambda 表达式内部对该变量的修改会直接影响到外部变量。

```C++
int x = 10;
auto f = [&x] (int y) -> int { return x + y; }; // 引用捕获 x
x = 20; // 修改外部的 x
cout << f(5) << endl; // 输出 25，受外部 x 的影响
```

- **隐式捕获**：隐式捕获是指 lambda 表达式可以根据使用情况自动推断要捕获的变量。隐式捕获有两种方式：隐式值捕获（`[=]`）和隐式引用捕获（`[&]`）；在捕获列表中使用 `=` 或 `&`，表示按值或按引用捕获 Lambda 表达式中使用的所有外部变量。


```C++
int x = 10;
int y = 20;
auto f = [=] (int z) -> int { return x + y + z; }; // 隐式按值捕获 x，y
cout << f(5) << endl; // 输出 35
```

```C++
int x = 10;
int y = 20;
auto f = [&] (int z) -> int { return x + y + z; }; // 显式按引用捕获 x，y
x = 30; // 修改外部的 x
y = 40; // 修改外部的 y
cout << f(5) << endl; // 输出 75
```

- **混合捕获**：lambda 表达式还可以同时使用值捕获和引用捕获，以及隐式捕获和显式捕获的混合。

```C++
int x = 10;
int y = 20;
auto f = [=, &y] (int z) -> int { return x + y + z; }; // 隐式按值捕获 x，显式按引用捕获 y
x = 30; // 修改外部的 x
y = 40; // 修改外部的 y
cout << f(5) << endl; // 输出 55，不受外部 x 的影响，受外部 y 的影响
```

# 使用场景

- **作为函数参数**：lambda 表达式最常见的用途之一是作为函数的参数，特别是在使用算法库时。例如，在`std::sort`函数中，可以使用 lambda 表达式来定义自定义的比较函数。

```C++
// 使用 lambda 表达式作为 std::sort 的比较函数
std::sort(numbers.begin(), numbers.end(), [](int a, int b) {
    return a < b;
});
```

- **用于函数对象**：lambda 表达式可以用于创建函数对象，这些函数对象可以像普通函数一样被调用，也可以存储在变量中，以便在需要时进行调用。

```C++
// 将 lambda 表达式赋值给一个变量
auto add = [](int a, int b) {
    return a + b;
};
// 调用 lambda 表达式
int result = add(3, 5);
```

- **在 STL 算法中的使用**：lambda 表达式在 STL 算法中非常有用，可以方便地定义各种操作。例如，`std::for_each`算法可以用于对容器中的每个元素执行一个操作，这个操作可以用 lambda 表达式来定义。

```C++
// 使用 std::for_each 和 lambda 表达式打印每个元素
std::for_each(numbers.begin(), numbers.end(), [](int num) {
    std::cout << num << " ";
});
```

# C++14扩展

- **初始化捕获**：允许在 Lambda表达式的捕获列表中使用初始化表达式，从而实现初始化捕获，即可以在捕获列表中创建和初始化一个新的变量，而不是捕获一个已存在的变量。

```C++
// 初始化捕获
auto f = [z = 10] (int x, int y) -> int { return x + y + z; };
// 调用 Lambda表达式
cout << f(1, 2) << endl; // 输出 13
```

- **泛型 Lambda**：C++14 允许在 Lambda表达式的参数列表和返回值类型中使用 `auto` 关键字，从而实现泛型 Lambda，即可以接受任意类型的参数和返回任意类型的值的 Lambda表达式。

```C++
// 泛型 lambda 表达式
auto lambda = [](auto a, auto b) {
    return a + b;
};

std::cout << lambda(1, 2) << std::endl;
std::cout << lambda(1.5, 2.5) << std::endl;
```

# C++17扩展

- **捕获 `*this`**：使用 `[*this]` 捕获，它捕获的是当前对象的一个副本。即使原对象在 lambda 执行期间被修改或者销毁，lambda 表达式使用的是自己持有的对象副本，不会受到影响。

> 表达式使用 `[this]` 捕获，它捕获的是当前对象的指针。lambda 表达式内部通过该指针访问对象的成员。这意味着如果原对象在 lambda 执行期间被修改或者销毁，lambda 表达式的行为可能会受到影响。

```C++
class MyClass {
public:
    int value = 10;
    void printValue() {
        // 捕获 *this
        auto lambda = [*this]() {
            std::cout << value << std::endl;
        };
        lambda();
    }
};
```

- **constexpr lambda 表达式**：允许在编译时调用 lambda 表达式，前提是满足 `constexpr` 的条件。有利于编译期计算和优化。

```C++
// constexpr lambda 表达式
constexpr auto lambda = [](int a, int b) constexpr {
    return a + b;
};
constexpr int result = lambda(1, 2); // 3
```

# 优点

- **代码简洁性**：lambda 表达式可以在需要的地方直接定义函数，避免了单独定义函数的繁琐过程，使代码更加紧凑和易读。
- **可移植性**：lambda 表达式可以作为函数对象传递给其他函数，使得代码更加灵活和可移植。
- **闭包特性**：lambda 表达式可以捕获外部变量，形成闭包。闭包允许在函数内部访问和操作外部变量，即使外部变量的作用域已经结束，这在某些场景下非常有用。

# 注意事项

- **捕获列表的性能**：在使用值捕获时，如果捕获的变量较大，可能会导致性能问题。在这种情况下，可以考虑使用引用捕获或其他优化方法。
- **生命周期问题**：当使用引用捕获时，需要注意被捕获变量的生命周期。如果被捕获的变量在 lambda 表达式执行之前就已经销毁，可能会导致未定义行为。
- **可变性**：默认情况下，lambda 表达式中的函数体是不可变的，即不能修改捕获的变量。如果需要在 lambda 表达式中修改捕获的变量，可以在参数列表后添加`mutable`关键字。