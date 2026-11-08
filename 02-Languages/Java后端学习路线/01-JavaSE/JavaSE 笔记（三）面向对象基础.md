# JavaSE 笔记（三）：面向对象基础

> 面向对象可以先理解为：把程序中的事物表示成一个个“对象”，对象保存自己的数据，也能执行自己的操作。类像设计图，对象像按设计图创建出来的具体物品。
>
> 本篇覆盖类与对象、方法、构造器、`this`、静态成员、包与访问权限、封装、继承、多态、抽象类、接口，以及枚举、记录类和密封类。

## 学习路线

```text
类和对象 → 属性和方法 → 构造器与 this → static
     → 包与访问权限 → 封装 → 继承和重写 → 多态
     → 抽象类与接口 → 枚举 / record / sealed
```

## 1. 类和对象

### 1.1 类是设计图，对象是具体事物

假设我们要在程序中表示学生：每个学生有姓名、年龄，也能自我介绍。

- **类（class）**：描述一类事物有什么数据、能做什么。
- **对象（object）**：根据类创建出来的具体实例。
- **属性/字段（field）**：对象保存的数据，例如姓名、年龄。
- **方法（method）**：对象可以执行的操作，例如自我介绍。

```java
class Student {
    String name; // 属性
    int age;

    void introduce() { // 方法
        System.out.println("我叫" + name + "，今年" + age + "岁");
    }
}
```

创建对象要使用 `new`：

```java
Student s1 = new Student();
s1.name = "小林";
s1.age = 18;
s1.introduce();
```

`Student` 是类，`s1` 是引用变量，它引用了一个 Student 对象。通过点号 `.` 可以访问对象允许访问的属性或方法。

### 1.2 每个对象有自己的实例数据

同一个类可以创建许多对象，每个对象的实例字段互相独立：

```java
Student a = new Student();
Student b = new Student();

a.name = "小林";
b.name = "小周";
```

修改 `a.name` 不会自动修改 `b.name`。对象创建后，字段若没有显式赋值，会得到默认值：数值类型是 0 或 0.0，`boolean` 是 `false`，引用类型是 `null`。不过**局部变量没有这种自动默认值**，必须先赋值再使用。

`null` 表示引用变量当前没有引用对象。对 `null` 使用点号访问成员会在运行时出错，因此要确认对象存在后再调用。

### 1.3 引用变量赋值和比较

把一个对象变量赋给另一个变量，复制的是**引用**，不是复制出一个新对象：

```java
Student a = new Student();
a.name = "小林";
Student b = a;       // a 和 b 引用同一个对象
b.name = "小周";
System.out.println(a.name); // 小周
```

基本类型变量保存数值本身；引用类型变量保存对象的引用。比较两个引用用 `==` 时，检查的是它们是否指向同一个对象；比较字符串内容一般使用 `equals`，不要用 `==` 比字符串内容。

## 2. 方法：给对象安排行为

方法把一组相关语句组合起来，可以接收输入，也可以返回结果。

```java
返回类型 方法名(参数列表) {
    方法体
    return 返回值; // 返回类型不是 void 时
}
```

示例：

```java
class Calculator {
    int add(int a, int b) {
        return a + b;
    }

    void showMessage(String message) {
        System.out.println(message);
    }
}
```

调用方法：

```java
Calculator calculator = new Calculator();
int result = calculator.add(3, 4); // 7
calculator.showMessage("计算完成");
```

- 方法定义里的变量叫**形式参数**（形参）；调用时传入的值叫**实际参数**（实参）。
- `void` 表示方法不向调用者返回结果。
- `return` 返回结果并立即结束当前方法；`void` 方法也可以写 `return;` 提前结束。
- 只要方法声明了非 `void` 返回类型，所有可能执行到方法末尾的路径都必须返回相应类型的值。

### 2.1 Java 参数传递：始终是值传递

Java 调用方法时，总是把实参的值复制一份给形参：

```java
static void changeNumber(int number) {
    number = 99;
}

int n = 10;
changeNumber(n);
System.out.println(n); // 10，方法里的 number 是副本
```

如果传入对象引用，复制的是引用值。方法中的形参和外面的变量因此可以引用同一个对象，方法修改对象字段时，外面能看到变化：

```java
static void rename(Student student) {
    student.name = "小周";
}
```

但如果方法里把形参重新指向另一个对象，外面的引用变量并不会跟着改。这样理解就不会误以为 Java 有“按引用传递”。

### 2.2 方法重载（overload）

同一个类可以有多个同名方法，只要**参数列表不同**，这叫重载：

```java
class Printer {
    void print(String text) { }
    void print(int number) { }
    void print(String text, int copies) { }
}
```

参数列表指参数个数、类型和顺序。只改返回类型不构成重载。调用时，编译器根据传入的参数选择匹配的方法。

### 2.3 方法互调：一个方法调用另一个方法

方法不必各自独立工作。一个方法可以调用同一个类中的另一个方法，也可以调用其他对象的方法。把较大的任务拆成几个职责清楚的小方法，代码会更容易读和修改。

```java
class Greeting {
    void sayHello(String name) {
        System.out.println(makeMessage(name));
    }

    String makeMessage(String name) {
        return "你好，" + name + "！";
    }
}

Greeting greeting = new Greeting();
greeting.sayHello("小林");
```

这里 `sayHello` 调用了 `makeMessage`：先得到字符串，再打印出来。方法互调时要留意返回值类型、参数是否匹配，以及方法是否可访问。实例方法互相调用时，通常是在同一个对象的上下文中执行；静态方法则不能直接调用实例方法，除非先通过某个对象来调用。

调用过程可以想成“暂时转去做一件小事，做完后回到原来的位置继续”：

```text
sayHello 开始
  → 调用 makeMessage
      → 返回消息
  → 打印消息
sayHello 结束
```

### 2.4 递归：方法调用自己

递归是一种特殊的方法互调：方法在完成工作时再次调用自己。它适合把问题拆成“一个较小的同类问题”，例如计算阶乘：

```java
static int factorial(int n) {
    if (n == 1) {        // 终止条件：问题已经足够简单
        return 1;
    }
    return n * factorial(n - 1); // 把问题缩小为求 (n - 1)!
}
```

调用 `factorial(4)` 的过程是：

```text
factorial(4) = 4 × factorial(3)
             = 4 × 3 × factorial(2)
             = 4 × 3 × 2 × factorial(1)
             = 4 × 3 × 2 × 1
             = 24
```

每个递归方法都要想清楚两件事：

1. **终止条件**：什么时候不再继续调用自己？没有终止条件，递归可能一直进行，最后导致 `StackOverflowError`。
2. **问题如何变小**：每次调用是否更接近终止条件？例如 `n - 1` 会逐步接近 `1`；如果参数一直不变或离终止条件越来越远，就无法正常结束。

上面的阶乘示例假设 `n >= 1`，实际使用时还应处理 `n < 1` 的输入。递归调用会暂存每一层尚未完成的工作，因此层数过多会占用较多调用栈空间。很多递归问题也能用循环解决；初学时先用递归理解拆解思路，再比较循环写法。

## 3. `this`、构造器和初始化

### 3.1 `this` 表示当前对象

当参数名和字段名相同时，可用 `this` 区分当前对象的字段：

```java
class Student {
    String name;

    void setName(String name) {
        this.name = name; // 左边是对象字段，右边是参数
    }
}
```

`this` 只能在实例上下文中使用，代表当前对象；静态方法没有“当前对象”，因此不能使用 `this`。

### 3.2 构造器负责创建时的初始化

构造器（constructor）在创建对象时运行，通常用于设置对象的初始状态。构造器名称必须和类名相同，且**没有返回类型**：

```java
class Student {
    String name;
    int age;

    Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

Student s = new Student("小林", 18);
```

如果一个类完全没有声明构造器，Java 会提供一个无参构造器。**只要自己声明了任何构造器，编译器就不再自动提供无参构造器。**若仍需要 `new Student()`，就要自己声明无参构造器。

构造器也可以重载，用不同参数创建不同初始状态。一个构造器可以用 `this(...)` 调用同类另一个构造器；这条调用必须是构造器的第一条语句。

### 3.3 字段初始化顺序（入门理解）

实例字段可以直接赋初值，也可以通过构造器初始化。创建对象时，字段先获得默认值，再按声明顺序执行字段初始化和实例初始化代码块，最后执行构造器主体。初学阶段通常直接在字段声明处或构造器中初始化即可，避免复杂初始化代码。

## 4. `static`：属于类的共享成员

普通字段属于每个对象；静态字段属于类本身，所有对象共享一份。静态方法也属于类，不需要先创建对象就能调用。

```java
class Student {
    String name;                 // 每个学生各自有一份
    static int studentCount = 0; // 全班共享一份

    Student(String name) {
        this.name = name;
        studentCount++;
    }

    static void showCount() {
        System.out.println(studentCount);
    }
}

Student a = new Student("小林");
Student b = new Student("小周");
Student.showCount(); // 2
```

访问静态成员推荐用类名：`Student.studentCount`、`Student.showCount()`。静态方法不能直接访问实例字段或调用实例方法，因为它没有一个特定对象作为当前对象；它可以直接访问本类的静态成员。

**类初始化**大致发生在类首次主动使用时，例如读取或写入非编译期常量的静态字段、调用静态方法、创建实例等。静态字段初始化和静态代码块会在类初始化过程中按源码顺序执行。这里先理解“类初始化一次、实例构造每次创建对象都会执行”即可。

## 5. 包和访问权限

### 5.1 包：给类分组并避免重名

包（package）用于组织类。包名通常用小写字母，并常采用反向域名形式，例如 `com.example.app`。

```java
package com.example.app; // 通常是源文件第一条非注释语句

import com.example.model.Student;

public class Main {
    public static void main(String[] args) {
        Student student = new Student("小林", 18);
    }
}
```

- `package` 声明当前类属于哪个包。
- `import` 导入其他包中的类型，省去每次写完整包名。
- `java.lang` 中常用类型（如 `String`、`System`）会自动导入。
- `import some.package.*` 只导入该包中的类型，不包含其子包。

### 5.2 四种访问级别

| 修饰符 | 当前类 | 同包其他类 | 不同包子类 | 不同包非子类 |
|---|---:|---:|---:|---:|
| `private` | ✓ | — | — | — |
| 默认（不写） | ✓ | ✓ | — | — |
| `protected` | ✓ | ✓ | ✓* | — |
| `public` | ✓ | ✓ | ✓ | ✓ |

`protected` 在不同包的子类中有具体访问规则，初学阶段可先记为“同包可访问，包外子类在继承上下文中可访问”。类本身（顶层类）通常只能声明为 `public` 或包级默认访问，不能声明为 `private` 或 `protected`。

## 6. 封装：保护对象的有效状态

如果字段随便开放，外部代码可能把年龄改成负数、把状态改成不存在的值。封装通常把字段设为 `private`，并由方法提供受控的读写方式：

```java
class Student {
    private String name;
    private int age;

    public Student(String name, int age) {
        this.name = name;
        setAge(age);
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if (age < 0) {
            throw new IllegalArgumentException("年龄不能为负数");
        }
        this.age = age;
    }
}
```

封装不是“所有字段都机械地生成 getter/setter”，而是让类控制自己的数据和规则。对象的创建方式也可以被限制，例如把构造器设为 `private`，再通过类提供的方法创建对象；这属于特定设计场景，初学先知道它可行即可。

## 7. 继承：复用并扩展已有类型

继承用 `extends` 表示“子类是一种父类”。子类可以获得父类可继承的成员，并添加自己的字段和方法：

```java
class Person {
    protected String name;

    Person(String name) {
        this.name = name;
    }

    void introduce() {
        System.out.println("我叫" + name);
    }
}

class Student extends Person {
    Student(String name) {
        super(name); // 调用父类构造器
    }

    void study() {
        System.out.println(name + "正在学习");
    }
}
```

Java 类只能直接继承一个父类，但可以实现多个接口。构造子类时，父类部分也必须先初始化：`super(...)` 用于调用父类构造器，且若显式使用必须写在构造器第一句。若父类没有无参构造器，子类构造器必须显式调用一个合适的父类构造器。

- `this(...)` 调用本类的另一个构造器；`super(...)` 调用父类构造器。
- `this.成员` 指当前对象成员；`super.成员` 用于明确访问父类实现。
- `private` 成员不能被子类直接访问，但父类可以通过自身公开或受保护的方法管理它们。
- `final` 类不能被继承；`final` 方法不能被重写。

继承适合表达真正的“是一种”关系。若只是想复用某个功能，而不是类型上的父子关系，组合（一个对象持有并使用另一个对象）往往更合适。

## 8. 多态：同一种调用，不同对象有不同表现

父类或接口类型的变量可以引用它的子类对象：

```java
Person person = new Student("小林");
person.introduce();
```

这就是多态的一种表现：代码通过较通用的类型操作对象，运行时会根据对象实际类型选择被重写的方法。引用变量的声明类型限制编译时能直接调用哪些成员；对象的实际类型决定实例方法的重写版本。

### 8.1 向上转型和向下转型

- **向上转型**：子类对象赋给父类变量，通常自动完成，例如 `Person p = new Student(...)`。
- **向下转型**：父类变量转回子类类型，需要显式转换，并确保对象确实是该子类。

```java
if (person instanceof Student student) { // Java 16 正式支持的模式匹配写法
    student.study();
}
```

较旧 JDK 可写成先 `instanceof` 判断，再强制转换。不要在不确认实际类型时随意向下转型，否则会发生 `ClassCastException`。

### 8.2 方法重写与重载不是一回事

- **重载（overload）**：同一类里方法同名、参数列表不同，编译时选择。
- **重写（override）**：子类为继承来的实例方法提供新实现，运行时按实际对象类型选择。

```java
class Person {
    void speak() {
        System.out.println("人说话");
    }
}

class Student extends Person {
    @Override
    void speak() {
        System.out.println("学生回答问题");
    }
}
```

推荐写 `@Override`，如果签名没写对，编译器会提示。重写方法不能缩小访问权限；`private` 方法不会被子类重写；`static` 方法是隐藏而不是实例方法重写。

### 8.3 `Object`：所有类的共同父类

没有显式写 `extends` 的类，默认直接继承 `Object`；其他类也最终沿继承链继承它。常见方法包括：

- `toString()`：把对象转成便于查看的文字。IDE 常能生成适合调试的版本。
- `equals(Object)`：比较对象是否相等。默认实现和 `==` 一样比较是否为同一对象；类可按需要重写为比较内容。
- `hashCode()`：生成哈希值。若重写 `equals`，通常也要遵守规则重写 `hashCode`，相等对象必须有相同哈希值。

`==` 对基本类型比较数值；对引用类型比较是否为同一个对象。比较对象内容要使用该类定义的 `equals` 语义。`Object` 还有线程协作等方法，后续学习多线程时再了解即可。

## 9. 抽象类和接口

### 9.1 抽象类：共享基础，同时留下未完成部分

用 `abstract` 声明的类不能直接 `new`。抽象类可以有字段、构造器、普通方法，也可以声明没有方法体的抽象方法：

```java
abstract class Shape {
    abstract double area();

    void describe() {
        System.out.println("这是一个图形");
    }
}

class Circle extends Shape {
    private final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * radius * radius;
    }
}
```

具体子类必须实现继承到的抽象方法；若不实现，就必须自己也声明为抽象类。抽象方法不能是 `private`，因为子类需要实现它。

### 9.2 接口：约定一组能力

接口用 `interface` 声明，类用 `implements` 实现接口。一个类可以实现多个接口：

```java
interface Study {
    void study();
}

class Student implements Study {
    @Override
    public void study() {
        System.out.println("正在学习");
    }
}
```

接口方法是公开的，实现方法必须是 `public`。接口适合表达不同类都能具备的能力，例如 `Runnable`、`Comparable`。接口不是类的多继承：类仍只能继承一个类，但可实现多个接口。接口也可以 `extends` 一个或多个接口。

接口字段隐含为 `public static final` 常量。现代接口还支持默认方法和静态方法：

```java
interface Study {
    default void pause() {
        System.out.println("休息一下");
    }

    static void tips() {
        System.out.println("保持专注");
    }

    private void helper() { // Java 9 起接口可有 private 辅助方法
        System.out.println("内部复用逻辑");
    }
}
```

默认方法可被实现类继承或重写；接口静态方法通过接口名调用，例如 `Study.tips()`。私有接口方法只供接口内部复用。

`Cloneable` 是一个标记接口：它本身不声明方法，但表示某些克隆机制允许该对象被克隆。`Object.clone()` 涉及浅拷贝、异常和可变成员共享等问题，初学时不要把它当成通用复制方案；多数业务类更适合明确编写复制逻辑。

### 9.3 怎么选抽象类或接口？

| 需求 | 常见选择 |
|---|---|
| 多个相关类共享状态和部分实现 | 抽象类 |
| 描述一种能力或行为约定，供不同类实现 | 接口 |
| 需要一个类结合多种能力 | 一个父类 + 多个接口 |

这只是设计时的常见思路，具体还要看类型之间的关系和项目约定。

## 10. 其他常用类型

### 10.1 枚举 `enum`：有限个固定选项

如果状态只能是“运行、学习、睡觉”之一，用任意字符串会允许写出无效值。枚举把可选值限定好：

```java
enum Status {
    RUNNING,
    STUDYING,
    SLEEPING
}

Status status = Status.STUDYING;
```

枚举值是枚举类型自身的固定实例。枚举也可以有字段、构造器和方法；枚举构造器不能由外部调用。

### 10.2 记录类 `record`：简洁地承载数据

Java 16 起正式支持记录类，适合表达以数据为主的类型：

```java
record Point(int x, int y) { }

Point point = new Point(3, 4);
System.out.println(point.x());
System.out.println(point); // 自动生成有用的 toString
```

记录类会自动提供组件访问器（名称是 `x()`，不是传统 JavaBean 风格的 `getX()`）、`equals`、`hashCode`、`toString` 和规范构造器。记录组件是不可重新赋值的，但若组件引用可变对象，对象内部仍可能变化；所以 record 并不自动保证深层不可变。记录类不能显式继承其他类，但可以实现接口，也可添加方法。

### 10.3 密封类 `sealed`：明确允许哪些子类型

普通类默认可被符合条件的代码继承；`final` 类完全禁止子类。Java 17 起正式支持密封类，可列出允许的直接子类型：

```java
sealed class Vehicle permits Car, Bus { }

final class Car extends Vehicle { }
non-sealed class Bus extends Vehicle { }
```

被允许的直接子类必须选择 `final`、`sealed` 或 `non-sealed`：分别表示不再允许子类、继续限制子类、重新开放继承。密封类型可用于普通类、抽象类或接口。它适合子类型范围明确的模型；需要满足源码组织和模块/包等语言规则。

## 11. 核心概念对照

| 名称 | 记忆方式 |
|---|---|
| 类 | 描述一类对象的设计图 |
| 对象 | 类创建出的具体实例 |
| 字段 | 对象或类保存的数据 |
| 方法 | 对象或类能执行的操作 |
| 构造器 | 创建对象时初始化对象的特殊成员 |
| `this` | 当前对象 |
| `static` | 属于类、由该类对象共享的成员 |
| 封装 | 控制数据访问并保护对象规则 |
| 继承 | 建立父类与子类关系，复用并扩展类型 |
| 多态 | 通用类型引用不同对象，调用表现可随实际类型变化 |
| 抽象类 | 可以保留未实现方法的共同基类 |
| 接口 | 一组能力或行为约定 |

## 12. 复习与练习

### 先回答这些问题

1. 类和对象有什么区别？同一类创建的两个对象，实例字段是否共享？
2. `Student b = a;` 是复制对象还是复制引用？
3. Java 方法参数是如何传递的？为什么方法能改对象字段，却不能让外部基本变量变值？
4. 构造器与普通方法有什么不同？自己写了带参构造器后还能否直接 `new X()`？
5. `static` 字段和实例字段分别属于谁？
6. 重载和重写有什么区别？
7. 抽象类和接口分别适合表达什么？
8. 为什么比较字符串内容一般使用 `equals` 而不是 `==`？

### 动手练习

1. 创建 `Book` 类，包含书名、作者、页数，并写一个打印书籍信息的方法。
2. 给 `Book` 添加构造器，并确保页数不能小于 1。
3. 创建 `Shape` 抽象类，要求子类实现面积计算；实现圆形和矩形，并通过 `Shape` 类型的变量调用面积方法。
4. 用枚举表示订单状态，只允许 `NEW`、`PAID`、`SHIPPED`、`CANCELLED`。
5. 用 `record` 表示二维坐标，并创建两个内容相同的坐标对象，观察 `equals` 的结果。

> 建议边学边在 IDEA 中运行小例子。看到错误时先读编译器信息，再检查类型、方法签名、访问权限和括号，不要急着整段重写。

---

**整理说明：** 本文依据原始《面向对象基础》笔记重排，保留类与对象、方法、参数传递、构造器、静态成员、包、访问控制、封装、继承、多态、Object、抽象类、接口及枚举、record、sealed 等主题，并对引用语义、`equals`/`hashCode`、访问权限和 Java 版本特性作了补充说明。
