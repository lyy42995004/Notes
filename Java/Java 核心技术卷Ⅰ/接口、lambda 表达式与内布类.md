# 接口

**接口的概念**

- 在 Java 中，接口不是类，而是对希望符合该接口的类的一组需求。例如`Arrays`类的`sort`方法要求对象所属的类必须实现`Comparable`接口才能进行排序。
- 接口中的方法默认是`public`的，声明时可不写`public`关键字。如`Comparable`接口中的`compareTo`方法，接受一个`Object`参数并返回整数。Java 5 起，`Comparable`接口升级为泛型类型。
- 接口中不能有实例字段，也不能实现方法（Java 8 前），实现接口的类需完成方法实现和实例字段定义。
- 类要实现接口，需声明实现给定接口，并为接口中的所有方法提供定义，使用`implements`关键字。如`Employee`类实现`Comparable`接口，需实现`compareTo`方法来定义比较规则。

- `compareTo`方法的细节：
  - **返回值**：小于时返回负数，等于时返回 0，大于时返回正数。进行整数比较排序时很实用，但要注意整数范围避免溢出。
  - `Comparable`接口文档建议`compareTo`与`equals`方法兼容，但存在如`BigDecimal`类这样的例外情况 。

- Java 是强类型语言，方法调用时编译器会检查方法是否存在。`Arrays`类的`sort`方法要求对象数组元素所属类实现`Comparable`接口，以确保有`compareTo`方法，否则会报错或运行时抛出异常 。

```java
public class Employee implements Comparable<Employee> {
    private String name;
    private double salary;

    public int compareTo(Employee other) {
        return Double.compare(salary, other.salary);
    }
}

Arrays.sort(staff);
```

**接口的属性**

- 接口不能使用`new`运算符实例化，但可声明接口变量，并指向实现该接口的类对象。还能用`instanceof`检查对象是否实现了特定接口。
- 接口可以扩展，形成接口链，从通用接口扩展到专用接口。
- 接口中可包含常量，且字段默认是`public static final`的。
- 一个类虽只能有一个超类，但能实现多个接口，为定义类的行为提供了灵活性。

**接口与抽象类**

- 抽象类可包含抽象方法，若将`Comparable`设计为抽象类，会限制类的扩展（Java 不支持多重继承），而接口能让类实现多个接口，避免多重继承的复杂性和低效性，提供类似多重继承的好处 。

> **C++注释**：C++ 支持多重继承，即一个类可以从多个基类中继承属性和方法，但这会带来诸如虚基类、复杂的访问控制规则以及横向指针类型转换等复杂情况。在实际编程中，多数 C++ 程序员很少使用多重继承。程序员建议在一个主要基类描述核心对象特征时使用多重继承，其他基类补充辅助功能，这种方式和 Java 中类继承一个基类同时实现多个接口的模式有相似之处 。

**静态和私有方法**

- **静态方法**：Java 8 开始允许接口中添加静态方法，一些原本放在伴随类中的静态方法可直接放在接口中，如`Path`接口在 Java 11 中的新特性。
- **私有方法**：Java 9 中接口方法可以是`private`，可以是静态或实例方法，且只能在接口自身方法中使用 。

**默认方法**

- 可以为接口方法提供默认实现，需用`default`修饰符标记。如`Comparable`接口的默认实现，虽通常会被覆盖，但在某些场景有用，像`Iterator`接口的`remove`方法和`Collection`接口的`isEmpty`方法。

```java
public interface Iterator<E> {
    boolean hashNext();
    E next();
    default void remove() { throw new UnsupportedOperationExpection{"remove"}}
    // ...
}

public interface Collection {
    int size();
    default boolean isEmpty() { return size() == 0; }
    // ...
}
```

- **接口演化**：默认方法可用于 “接口演化”，为接口新增方法时，若设为默认方法可保证原代码兼容，避免类因未实现新方法而编译出错。

**解决默认方法冲突**

- **超类优先**：超类提供具体方法时，同名且参数类型相同的接口默认方法会被忽略。
- **接口冲突**：当两个接口有同名且参数类型相同的方法，实现类必须覆盖该方法解决冲突。若一个接口无默认实现，实现类也需处理避免二义性。此外，类扩展超类并实现接口，且两者有相同方法时，遵循 “类优先” 规则，接口默认方法被忽略 。

```java
interface Person {
    default String getName() { return ""; }
}

interface Named {
    defalut String getName() { return getClass().getName() + "_" + hashCode(); }
}

class Student implements Person, Named { /* ... */ }
```

**接口与回调**

- 回调是一种程序设计模式，指定特定事件发生时应采取的动作。以`java.swing`中的`Timer`类为例，创建定时器时需设置时间间隔和操作，通过实现`ActionListener`接口的`actionPerformed`方法定义定时执行的内容，传递的`ActionEvent`参数包含事件相关信息 。

```java
public class TimeTest {
    public static void main(String[] args) {
        var listener = new TimePrinter();

        // construct a timer that calls the listener
        // once every second
        var timer = new Timer(1000, listener);
        timer.start();

        // keep program running until the user selects "OK"
        JOptionPane.showMessageDialog(null, "Quit program?");
        System.exit(0);
    }
}

class TimePrinter implements ActionListener {
    public void actionPerformed(ActionEvent event) {
        System.out.println("At the tone, the time is " + 
                            Instant.ofEpochMilli(event.getWhen()));
        Toolkit.getDefaultToolkit().beep();
    }
}
```

**Comparator 接口**

- `Arrays.sort`方法有另一个版本，可接受一个实现了`Comparator`接口的比较器对象来进行排序。`Comparator`接口是一个泛型接口，包含一个`compare`方法，用于比较两个对象 。

```java
public interface Comparator<T> {
    int compare(T first, T second);
}

class LengthComparator implements Comparator<String> {
    public int compare(String first, String second) {
        return first.length() - second.length();
    }
}
```

**对象克隆**

- 直接建立包含对象引用的变量副本，原变量和副本会引用同一个对象，任何一个变量的改变都会影响另一个。而使用`clone`方法可以创建具有独立状态的副本。`Object`类的`clone`方法是`protected`的，不能直接调用，只有实现了克隆功能的类才能调用 。

- `Cloneable`接口是一个标记接口，本身没有定义任何方法，仅用于指示类提供了安全的`clone`方法。如果一个对象请求克隆但未实现该接口，会抛出`CloneNotSupportedException`异常 。

- **浅拷贝**：默认的克隆操作是浅拷贝，即只复制对象的基本类型字段，对于对象引用字段，原对象和克隆对象会共享该引用。以`Employee`类为例，如果其中包含可变对象（如`Date`），修改克隆对象中可变对象的状态，会影响原对象 。
- **深拷贝**：为避免共享引用带来的问题，对于包含可变对象的类，需要实现深拷贝，即递归地复制所有引用的对象 。

<img src="https://s2.loli.net/2025/03/09/A9VF3o1XxguJa7R.png" alt="image.png" style="zoom: 67%;" />

- 实现克隆的步骤与要点：
  - 类要实现`Cloneable`接口，并将`clone`方法重新定义为`public`。
  - 考虑对象中字段的可变性，对于可变对象决定是采用浅拷贝还是深拷贝 。
  - 处理可能抛出的`CloneNotSupportedException`异常 。

```java
class Employee implements Cloneable {
    public Employee clone() throws CloneNotSupportedExpection {
        // call Object.clone()
        Employee cloned = (Employee) super.clone();
        
        // clone mutable fields
        cloned.hireDay = (Date) hireDay.clone();
        return cloned;
    }
}
```

- 所有数组类型都有公共的`clone`方法，可用于创建包含原数组副本的新数组 。

# lambda 表达式

**为什么引入 lambda 表达式**

- lambda 表达式是一段可传递的代码块，可在以后执行一次或多次。在 Java 中，以往传递代码块并不容易，因为 Java 是面向对象语言，必须构造一个包含所需代码方法的对象。
- 例如在`ActionListener`的`actionPerformed`方法中执行定时任务，以及在定制比较器完成排序时，都需构造相应类的实例来传递代码块。引入 lambda 表达式旨在更简便地传递代码块，提升编程效率，让代码更简洁

**lambda 表达式的语法**

- **基本形式**：由参数、箭头（`->`）和表达式组成。如`(String first, String second) -> first.length() - second.length()`，用于比较两个字符串长度。
- **无参数情况**：即使没有参数，也需提供空括号，像`() -> { for (int i = 100; i >= 0; i--) System.out.println(i); }` 。
- **类型推断**：若能推导出 lambda 表达式的参数类型，则可省略类型声明，编译器能根据上下文推导参数类型。
- **单参数且类型可推导**：参数类型可省略，还能省略小括号，如`event -> System.out.println("The time is " + Instant.ofEpochMilli(event.getWhen()))` 。
- **返回类型**：lambda 表达式的返回类型由上下文推导得出，但需注意代码分支中返回值的一致性，若部分分支返回值，部分不返回则不合法。

**函数式接口**

- 函数式接口是指只有一个抽象方法的接口，如`ActionListener`和`Comparator` 。lambda 表达式与这类接口兼容，当需要一个实现了函数式接口的对象时，可以提供一个 lambda 表达式。
- 以`Arrays.sort`方法为例，它需要一个`Comparator`接口的实现类对象作为参数，此时可以使用 lambda 表达式来代替传统的内部类实现，使代码更简洁高效。

```java
Arrays.sort(words, (first, second) -> first.length() - second.length());
```

- `java.util.function`包中定义了很多函数式接口，如`BiFunction`（接受两个参数并返回一个结果）、`Predicate`（用于传递条件判断的 lambda 表达式，返回布尔值）、`Supplier`（无参数，生成一个指定类型的值）等。

**方法引用**

- 方法引用是一种让代码更简洁的方式，它指示编译器生成一个函数式接口的实例，该实例的抽象方法会调用给定的方法。
- 主要有 3 种情况：
  - `Object::instanceMethod`：对象的实例方法引用，例如`out::println`等价于`x->System.out.println(x)`。
  - `Class::instanceMethod`：类的实例方法引用，此时 lambda 参数会作为方法的调用者，例如`String::compareToIgnoreCase`等价于`(x, y) -> x.compareToIgnoreCase(y)`。
  - `Class::staticMethod`：类的静态方法引用，例如`Math::pow()`等价于`(x, y) -> Math.pow(x, y)`。

- 以使用`this`参数，`this::equals`等同于`x -> this.equals(x)` 。这意味着可以通过`this`来引用当前对象的实例方法，以简洁的方式表达方法调用。
- `super`在方法引用中也是合法的。使用`super::instanceMethod`形式的方法表达式时，以`this`作为目标，会调用给定方法的超类版本。

**构造器引用**

- 构造器引用和方法引用类似，方法名为`new`。比如`Person::new`就是`Person`构造器的一个引用。具体使用哪个构造器由上下文决定。
- 在将字符串列表转换为`Person`对象数组时，通过`stream.map(Person::new)`，编译器会根据上下文选择合适的`Person`构造器（这里是带`String`参数的构造器）。

```java
Person[] people = stream.toArray(Person[]::new);
```

**变量作用域**

- lambda 表达式可以访问外围方法或类中的变量，这些被访问的非参数且不在代码中定义的变量称为自由变量。在 Java 中，lambda 表达式捕获的变量必须是事实最终变量（effectively final），即初始化后值不会改变的变量。
- 对捕获变量进行限制是为了保证线程安全，避免在 lambda 表达式执行多个动作时因变量值改变而引发问题。
- 在 lambda 表达式中使用`this`关键字，指的是创建该 lambda 表达式的方法的`this`参数，其含义和在所在方法中的其他位置一致，没有特殊之处 。

**处理 lambda 表达式**

- lambda 表达式的重点是延迟执行，若要立即执行代码则无需封装在 lambda 中。适合使用 lambda 延迟执行代码的场景：在单独线程运行、多次运行、算法特定位置运行等。

**再谈 Comparator**

- 接口包含方便的静态方法创建比较器，可用于 lambda 表达式或方法引用。
- `comparing`方法通过 “键提取器” 函数映射对象为可比较类型后进行比较；`thenComparing`方法可处理比较结果相同的情况；这些方法还有变体形式以避免基本类型值装箱问题。

# 内部类

内部类是定义在另一个类中的类，主要作用有两个，一是可以对同一个包中的其他类隐藏，二是其方法能够访问定义该类的作用域中的数据，包括原本私有的数据。虽然 lambda 表达式在实现回调方面表现出色，但内部类在构建代码时依然有用。

> **C++注释**： C++ 有嵌套类。被嵌套的类包含再在外围类的作用域内。
>
> ```cpp
> class LinkedList {
> public:
>     class Iterator {
>     public:
>         void insert(int x);
>         int erase();
>         // ...
>     private:
>         Link* current;
>         LinkedList* owner;
>     }
>     // ...
> private:
>     Link* head;
>     Link* tail;
> };
> ```
>
> C++ 的嵌套类和 Java 的内部类相似。但 Java 内部类的对象有一个隐式引用，指向实例化该对象的外部类对象，凭借这个隐式引用，Java 内部类能访问外部对象的全部状态，比如 Java 中的`Iterator`类无需显式指针指向它所属的链表类。而 Java 的静态内部类没有这个附加指针，在这一点上，它和 C++ 的嵌套类相当 。

**使用内部类访问对象状态**

- `TimePrinter`作为`TalkingClock`的内部类，实现了`ActionListener`接口。内部类`TimePrinter`的对象有一个隐式引用指向创建它的外部类`TalkingClock`对象，因此其`actionPerformed`方法不仅能访问自身数据字段，还能访问外部类`TalkingClock`的`beep`标志等数据字段。

```java
class TalkingClock {
    private int interval;
    private boolean beep;

    public TalkingClock(int interval, boolean beep) {
        this.interval = interval;
        this.beep = beep;
    }

    public void start() {
        ActionListener listener = new TimePrinter();
        Timer t = new Timer(interval, listener);
        t.start();
    }

    public class TimePrinter implements ActionListener {
        public void actionPerformed(ActionEvent event) {
            System.out.println("At the tone, the time is " + Instant.ofEpochMilli(event.getWhen()));
            if (beep) {
                Toolkit.getDefaultToolkit().beep();
            }
        }
    }
}
```

- 编译器会修改内部类构造器，添加对应外部类引用的参数。此外，将内部类声明为私有，可限制只有外部类方法能构造该内部类对象，增强数据访问控制 。

**内部类的特殊语法规则**

- **外围类引用语法**：使用`OuterClass.this`表示外围类引用，如在`TimePrinter`内部类的`actionPerformed`方法中，可通过`TalkingClock.this.beep`访问外围类的成员。

```java
public void actionPerformed(ActionEvent event) {
    if (TalkingClock.this.beep)
        Toolkit.getDefaultToolkit().beep();
}
```

- **内部类构造器语法**：创建内部类对象时，可以用`outerObject.new InnerClass(construction parameters)`的形式，此时新构造的内部类对象的外围类引用被设为创建它的方法的`this`引用。也可显式指定外围类引用。在外部引用内部类时，需使用`OuterClass.InnerClass`的形式 。

```java
ActionListener= this.new TimePrinter();
```

- 内部类中声明的静态字段必须是`final`且初始化为编译时常量，内部类不能有普通的`static`方法，但可以有访问外围类静态字段和方法的静态方法 。

**内部类是否有用、必要和安全**

- 内部类是编译器现象，与虚拟机无关。编译器会将内部类转换为常规类文件，如`TimePrinter`类会被转换为`TalkingClock$TimePrinter.class`，并生成额外字段存储对外围类的引用。

**局部内部类**

- 当一个类只在某个方法中被使用一次时，可以在该方法中局部定义这个类，即局部内部类。
- 如在`TalkingClock`类的`start`方法中定义的`TimePrinter`类。局部内部类不能有访问说明符（`public`或`private`），其作用域被限定在声明它的块中，对外部世界完全隐藏，除了所在方法外，其他地方无法知晓其存在 。

```java
public void start() {    
    public class TimePrinter implements ActionListener {
        public void actionPerformed(ActionEvent event) {
            System.out.println("At the tone, the time is " + Instant.ofEpochMilli(event.getWhen()));
            if (beep) {
                Toolkit.getDefaultToolkit().beep();
            }
        }
        
    var listener = new TimePrinter();
   	var timer = new Timer(interval, listener);
    timer.start();
}	
```

**由外部方法访问变量**

- 局部内部类不仅能访问外部类的字段，还能访问局部变量，但这些局部变量必须是事实最终变量（`effectively final`），即初始化后值不再改变。以`TalkingClock`的例子，将构造器参数`interval`和`beep`移至`start`方法中，局部内部类`TimePrinter`仍能访问这些变量。

```java
public void start(int interval, boolean beep) {    
    public class TimePrinter implements ActionListener {
        public void actionPerformed(ActionEvent event) {
            System.out.println("At the tone, the time is " + Instant.ofEpochMilli(event.getWhen()));
            if (beep) {
                Toolkit.getDefaultToolkit().beep();
            }
        }
        
    var listener = new TimePrinter();
   	var timer = new Timer(interval, listener);
    timer.start();
}	
```

- 为了使局部内部类能访问所在方法的局部变量，编译器会为局部内部类创建相应的实例字段，并将局部变量复制到构造器中进行初始化。例如`TimePrinter`类在编译后会有对应`beep`值的实例字段，确保在`actionPerformed`方法执行时能获取到正确的变量值 。

**匿名内部类**

-  在使用局部内部类时，若只需创建一个类的一个对象，可使用匿名内部类，即无需为类指定名字。常用于创建实现特定接口（如`ActionListener`）或扩展某个类的对象 。

```java
public void start(int interval, boolean beep) {    
    var listener = new ActionListener() {
        public void actionPerformed(ActionEvent event) {
            System.out.println("At the tone, the time is " + Instant.ofEpochMilli(event.getWhen()));
            if (beep) {
                Toolkit.getDefaultToolkit().beep();
            }
        }
        
   	var timer = new Timer(interval, listener);
    timer.start();
}	
```

- 语法形式：`new SuperType(construction parameters) { inner class methods and data }`。`SuperType`可以是接口，此时内部类需实现该接口的方法；也可以是类，内部类则扩展这个类。由于匿名内部类没有类名，所以不能有构造器。若实现接口，虽无构造参数，但仍需保留一组小括号 。

**静态内部类**

- 当使用内部类只是为了将一个类隐藏在另一个类内部，且不需要内部类持有外围类对象的引用时，可将内部类声明为`static`，即静态内部类。它可以避免生成指向外部类对象的隐式引用 。
- 以计算数组中最小值和最大值为例，先给出了不使用内部类的实现方式，因需返回两个值存在不便。然后定义了`Pair`类来封装两个值，在`ArrayAlg`类中通过`minmax`方法返回`Pair`类型的对象。由于`Pair`类仅在`ArrayAlg`类中使用，为避免命名冲突并隐藏该类，将其声明为`ArrayAlg`的静态内部类 。

```java
class ArrayAlg {
    public static class Pair {
        private double first;
        private double second;

        public Pair(double f, double s) {
            first = f;
            second = s;
        }

        public double getFirst() {
            return first;
        }

        public double getSecond() {
            return second;
        }
    }

    public static Pair minmax(double[] values) {
        double min = Double.POSITIVE_INFINITY;
        double max = Double.NEGATIVE_INFINITY;
        for (double v : values) {
            if (min > v) min = v;
            if (max < v) max = v;
        }
        return new Pair(min, max);
    }
}
```

- 与常规内部类不同，静态内部类的对象不会持有外部类对象的引用，且可以有静态字段和方法。在接口中声明的内部类会自动是`static`和`public`的。如果内部类不需要访问外围类对象，使用静态内部类是更好的选择，部分程序员也用 “嵌套类（nested class）” 表示静态内部类 。

# 服务加载器

在一些复杂应用开发中会采用服务架构，部分平台（如 OSGi）支持相关开发，但 JDK 提供了一个简单的服务加载机制，由 Java 平台模块系统支持。通过`ServiceLoader`类，程序能加载符合公共接口的多个服务实现，给予服务设计者实现自由，也方便程序选择合适的实现 。

# 代理

**何时使用代理**

- 在运行时创建实现一组给定接口的新类可利用代理，适用于编译期无法确定要实现哪个接口的情况。代理类能实现指定接口，包含接口方法及`Object`类的部分方法，且需提供调用处理器（`InvocationHandler`接口的实现类对象），调用代理对象方法时，调用处理器的`invoke`方法会被调用 。

**创建代理对象**

- 通过`Proxy`类的`newProxyInstance`方法创建代理对象，该方法有三个参数，分别是类加载器（用于加载平台和应用类）、`Class`对象数组（指定要实现的接口）、调用处理器。使用代理可实现多种目的，如将方法调用路由到远程服务器、关联用户界面事件与动作、跟踪方法调用 。

**代理类的特性**

- 代理类是在程序运行过程中动态创建的，不过一旦创建完成，就和虚拟机中的常规类没有区别 。
- 所有代理类都扩展自`Proxy`类，且仅有一个实例字段，即调用处理器，该字段在`Proxy`超类中定义
