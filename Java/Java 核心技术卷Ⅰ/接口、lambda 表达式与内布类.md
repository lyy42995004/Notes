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

直接建立包含对象引用的变量副本，原变量和副本会引用同一个对象，任何一个变量的改变都会影响另一个。而使用`clone`方法可以创建具有独立状态的副本。`Object`类的`clone`方法是`protected`的，不能直接调用，只有实现了克隆功能的类才能调用 。

- **`Cloneable`接口**：该接口是一个标记接口，本身没有定义任何方法，仅用于指示类提供了安全的`clone`方法。如果一个对象请求克隆但未实现该接口，会抛出`CloneNotSupportedException`异常 。
- 浅拷贝与深拷贝
  - **浅拷贝**：默认的克隆操作是浅拷贝，即只复制对象的基本类型字段，对于对象引用字段，原对象和克隆对象会共享该引用。以`Employee`类为例，如果其中包含可变对象（如`Date`），修改克隆对象中可变对象的状态，会影响原对象 。
  - **深拷贝**：为避免共享引用带来的问题，对于包含可变对象的类，需要实现深拷贝，即递归地复制所有引用的对象 。
- 实现克隆的步骤与要点
  - 类要实现`Cloneable`接口，并将`clone`方法重新定义为`public`。
  - 考虑对象中字段的可变性，对于可变对象决定是采用浅拷贝还是深拷贝 。
  - 处理可能抛出的`CloneNotSupportedException`异常 。
- **特殊情况说明**：所有数组类型都有公共的`clone`方法，可用于创建包含原数组副本的新数组 。
