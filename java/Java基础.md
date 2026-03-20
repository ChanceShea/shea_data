# Java基础
## 概念
### Java的特点
- **平台无关性**：Java有“一次编写，到处运行”的特点。Java编译器将Java代码编译成字节码，字节码可以在任何安装了JVM的系统上运行
- **面向对象**：Java是一个严格面向对象的编程语言，Java中几乎所有东西都是对象。面向对象编程(OOP)特性使得代码更易于维护和复用，包括类、对象、继承、封装、多态、抽象
- **内存管理**：Java有自己的垃圾回收机制，自动管理内存和回收不再使用的对象。开发者无需手动管理内存，从而减少内存泄漏和其他相关内存问题
### Java为什么是跨平台的
Java能够跨平台开发，主要依赖于JVM。我们编写的Java源码，编译后会生成.class文件，即字节码文件，而JVM就是负责将字节码文件翻译成对应系统的机器码，然后在这个系统上运行。而Java程序只需要编写一次，就能在不同操作系统的JVM上运行。这就是Java的**一次编写，到处运行**。
JVM是一个”桥梁“，是Java实现跨平台的关键。Java代码先被编译成字节码，再由JVM将字节码翻译成机器语言，从而达到运行Java程序的目的
与其他一些语言不同，Java编译后的结果不是机器码，而是字节码，字节码不能直接运行，必须通过JVM翻译成机器码之后才能在操作系统上运行。而像C/C++，编译后生成的是机器码，而不同操作系统的机器码不一样，所以C/C++就不能跨平台运行，JVM是用C/C++开发的，因此也不能跨平台运行，所以每种操作系统都有不一样的JVM
### JVM、JRE、JDK
- JVM是Java虚拟机，是Java程序运行的环境。负责将Java字节码编译成机器码，并执行程序。JVM提供了内存管理、垃圾回收、安全性等功能，使得Java具有跨平台性
- JRE是Java运行时环境，是Java程序运行所需的最小环境。它包含了JVM和一组Java类库，用于支持Java程序的运行。JRE不包含开发工具，只提供Java程序运行时所需的运行环境
- JDK是Java开发工具包，是开发Java程序所需的工具集合。它包含了JVM、编译器(javac)、调试器(jdb)等开发工具，以及一系列类库。JDK提供了开发、编译、调试和运行Java程序所需的全部工具和环境
### 为什么Java解释和编译都有
Java在经过编译后生成字节码文件，接下来进入JVM中，有两个步骤，解释和编译
- Java源代码首先被编译成字节码，JIT会把编译过的机器码保存起来，以备下次使用
- JVM中的一个方法调用计数器，当累计计数大于一定值时，就是用JIT进行编译生成机器码文件。否则就是用解析器进行解释执行，然后字节码也是经过解释器解释运行的
所以Java及时编译型语言也是解释型语言，默认采用的是解释器和编译器混合的模式
区别
- 编译型语言：在程序执行之前，整个源代码都会被编译成机器码或字节码，生成可执行文件。执行时直接运行编译后的代码，速度快，但跨平台性较差（典型的如C、C++）
- 解释型语言：在程序执行中，逐行解释执行源代码，不生成独立的可执行文件。通常由解释动态解释并执行代码，跨平台性好，但执行速度相对较慢（典型的如Python、JavaScript）
### 值传递和引用传递
Java中，参数传递只有值传递一种方式，不存在真正的引用传递。区别就是传递的是**值的副本**还是**引用的副本**
值传递实际传递的是值的副本，适用于基本数据类型，修改方法内的参数副本，不会影响原变量的值
```java
public static void main(String[] args) {
	int a = 10;
	change(a);
	System.out.println(a);
}
public static void change(int a) {
	a = 20;
}
```
例如上述代码，尽管change方法内将a的值修改成20，但是main方法中输出的值还是10
引用传递，对于对象传递的是对象引用的副本。两个引用指向同一个对象，因此通过副本修改时，会影响到原对象。但如果**修改副本的指向，不会印象原引用的指向**
```java
public class Test1 {  
  
    static class Student {  
        public String name;  
  
        public Student(String name) {  
            this.name = name;  
        }  
    }  
  
    public static void main(String[] args) {  
        Student stu = new Student("zhangsan");  
        changeName(stu);  
        System.out.println(stu.name);  
        changeName(stu);  
        System.out.println(stu.name);  
    }  
  
    private static void changeName(Student stu) {  
        stu.name = "lisi";  
    }  
    private static void changeRef(Student stu) {  
        stu = new Student("wangwu");  
    }  
}
```
```text
lisi
lisi
```
changeName方法修改的是引用的副本，由于引用的副本和原引用指向同一个对象，所以修改后，原引用的值也是lisi，但对于changeRef方法，由于引用的副本指向了一个新对象wangwu，因此对原引用不会有影响，因此原引用的值还是lisi
## 数据类型
Java中的数据类型分为基本数据类型和引用数据类型
下面是8种基本数据类型的默认值、位数、取值范围
- **byte**：占用1字节，8位，取值范围`-128(-2^7)~127(2^7-1)`，默认值为0
- **short**：占用2字节，16位，取值范围`-32768(-2^15)~32767(2^15-1)`，默认值为0
- **int**：占用4字节，32位，取值范围`-2147483648(-2^31)~2147483647(2^31-1)`，默认值为0
- **long**：占用8字节，64位，取值范围`-2^63~2^63-1`，默认值0L
- **float**：占用4字节，32位，取值范围`1.4E - 45 到 3.4028235E38`，默认值0.0f
- **double**：占用8字节，64位，取值范围`4.9E - 324 到 1.7976931348623157E308`，默认值0.0d
- **char**：占用2字节，16位，取值范围`'\u0000'(0) 到 '\uffff'(65535)`，默认值'\u0000'
- **boolean**：无明确字节大小（理论上为1字节），无明确位数，取值范围`true或false`，默认值false
### 数据类型转换方式
- 自动类型转换（隐式转换）：当目标类型的范围大于原类型时，Java会自动将原类型转换为目标类型，不需要显示的类型转换。例如`int`转换为`long`、`fload`转换为`double`
- 强制类型转换（显示转换）：当目标类型范围小于原类型时，需要使用强制类型转换将原类型转换为目标类型。但是可能导致数据丢失或溢出
- 字符串转换：Java提供了字符串表示的数据转换为其他类型数据的方法。例如`Integer.parseInt()`或`Double.parseDouble()`
- 数值之间的转换：Java提供了一些数值类型之间的转换方法，例如将整型转换为字符型、将字符型转换为整型。
### BigDecimal
浮点数在计算机中都是用二进制的形式存储，而对于浮点数来说，二进制无法精确表示小数，double和float中的小数都是以一种近似的形式存在，不是完全精确的小数
所以在进行浮点数运算的时候，会出现精度丢失的问题。那么如果我们在进行商品价格计算的时候，就会出现问题，当并发量非常大的时候，就会出现非常大的问题。导致对账出现错误
所以在进行精确计算，尤其是涉及到金钱运算的时候，一般都会使用Decimal
```java
public static void main(String[] args) {  
    BigDecimal a = new BigDecimal("0.1");  
    BigDecimal b = new BigDecimal("0.2");  
    BigDecimal sum = a.add(b);  
    System.out.println(sum);  
}
```
```
0.3
```
BigDecimal可以确保精确的十进制数值运算，避免使用double可能出现的舍入误差。在创建BigDecimal对象时，应使用字符串作为参数，而不是直接使用浮点数，避免出现误差
### 装箱和拆箱
装箱和拆箱是将基本数据类型和对应的包装类之间进行转换的过程
自动装箱主要发生在两种情况：赋值和方法调用时
**自动装箱的弊端**
在一个循环中进行自动装箱操作的情况，就会创建很多多余的对象，影响程序性能
```java
public static void main(String[] args) {  
    Integer a = 0;  
    for (int i = 0; i < 100; i++) {  
        a += i;  
    }  
}
```
对于上述代码，因为`+`这个操作符不适用于Integer对象，就需要先对sum进行自动拆箱操作，进行数值相加，最后发生自动装箱操作转换成Integer对象
```java
int res = sum.intValue() + i;
Integer sum = new Integer(res);
```
上述循环会创建100个无用的Integer对象，在一些庞大的循环中，会降低程序的性能并加重了垃圾回收的工作量
### Java中为什么要有Integer
Integer就是对int类型的包装，就是把int类型包装成Object对象，对象封装有很多好处，可以把属性也就是数据跟处理这些数据的方法结合在一起，比如Integer就有parseInt等方法来专门处理int型相关的数据
**泛型**
在Java中，泛型只能使用引用类型，而不能使用基本类型。因此，如果要在泛型中使用int类型，就必须使用Integer包装类。假设我们要使用列表，对其中的元素进行排序，并将结果放入一个新的列表。如果我们使用int类型，就无法直接使用`Collections.sort()`方法，但是如果使用Integer包装类，就可以使用该方法
**转换**
Java中，基本类型和引用类型不能直接转换，必须通过包装类来实现。例如将int类型的值转换成String类型，就必须先将其转换成Integer类型，然后再转换成String类型
```java
int i = 10;
Integer a = new Integer(i);
String str = a.toString();
```
**集合**
Java集合中只能存储对象，而不能存储基本类型。因此如果要将int类型的数据存储在集合中，就必须使用Integer包装类。
**Integer和int的区别**
- 基本类型和引用类型：int是一种基本数据类型，Integer是一种引用类型。基本数据类型是Java中最基本的数据类型，是预定义的，不需要实例化就可以使用。而引用类型需要通过实例化对象来使用。这意味着，使用int来存储一个整数时，不需要额外的内存分配，而使用Integer时，必须为对象分配内存。在性能方面，基本数据类型的操作通常比相应的引用类型更快
- 自动装箱和拆箱：Integer作为int的包装类，它可以实现自动装箱和拆箱。自动装箱是指将基本类型转化为相应的包装类类型，而自动拆箱则是将包装类类型转化为相应的基本类型。这使得Java程序员更加方便地进行数据类型的转换
- 空指针异常：int类型可以直接赋值为0，而Integer变量必须通过实例化对象来复制。如果对一个未经初始化的Integer变量进行操作，就会出现空指针异常。因为它被赋予了nul值，而对于null值是无法自动拆箱的
**Integer缓存**
Java的Integer类中实现了一个静态缓存池，用于存储特定范围内的整数值对应的Integer对象
默认情况下，这个范围是-128~127。当通过Integer.valuOf(int)方法创建一个在这个范围内的整数对象时，不需要每次都生成一个新的对象实例，而是复用缓存中的对象，会之间从内存中取出，不需要新建对象
## 面向对象
面向对象是一种编程范式，它将现实世界中的事物抽象为对象，对象具有属性和行为。面向对象的设计思想是以对象为中心，通过对象之间的交互来完成程序的功能，具有灵活性和可扩展性，通过封装和继承可以更好地应对需求变化
Java面向对象有三大特性
- **封装**：封装是指将对象的属性和行为结合在一起，对外隐藏对象的内部细节，仅通过对象提供的接口与外界交互。封装的目的是增强安全性和简化编程，使得对象更加独立
- **继承**：继承是一种可以使得子类自动共享父类数据结构和方法的机制。它是代码复用的重要手段，通过继承可以建立类与类之间的层次关系，使得结构更加清晰
- **多态**：多态是指允许不同类的对象对同一消息作出响应。即同一个接口，使用不同的实例而执行不同操作。多态性可以分为编译时多态（重载）和运行时多态（重写）。它使得程序具有良好的灵活性和扩展性
### 多态的实现
- **方法重载**：方法重载是指同一个类中可以有多个同名方法，它们具有不同的参数列表（参数类型、数量或顺序不同）。虽然方法名相同，但根据传入的参数不同，编译器会在编译时确定调用哪个方法。对于一个add方法，可以定义为add(int a,int b)和add(double a,double b)
- **方法重写**：方法重写是指子类能够提供对父类中同名方法的具体实现。在运行时，JVM会根据对象的实际类型确定调用哪个版本的方法。这是实现多态的主要方式。对于一个sount方法，子类Dog可以重写该方法，子类Cat也可以重写该方法。调用sound方法时，会判断是哪个类调用了，因此可以实现不同的输出
- **接口与实现**：多态也体现在接口的使用上，多个类可以实现同一个接口，并且用接口类型的调用来调用这些了类的方法。这使得程序在面对不同具体实现时保持一贯的调用方式。多个类都实现了一个Animal接口。，当用Animal类型的引用来调用sound方法时，会触发对应的实现
- **向上转型和向下转型**：Java中，可以使用父类类型的引用指向子类对象，这是向上转型。通过这种方式，可以在运行时采用不同的子类实现。将父类引用转回其子类实现，这是向下转型，但在执行前需要确认引用实际指向的对象类型以避免`ClassCastException`
多态可以提高代码的扩展性和复用性，是很多设计模式、设计原则、编程技巧的代码实现基础
### 抽象类和普通类的区别
- 实例化：普通类可以直接实例化对象；抽象类不能被实例化，只能被继承
- 方法实现：普通类中的方法可以有具体的实现；抽象类中的方法可以有实现也可以没有实现
- 继承：一个类既可以继承普通类，也可以继承抽象类（但只能继承一个）；同时可以实现多个接口。
- 实现限制：普通类可以被其他类继承和使用，而抽象类一般用作基类，被其他类继承和扩展使用
### 抽象类和接口的区别
**特点**
- 抽象类用于描述类的共同特性和行为，可以有成员变量、构造方法和具体方法。适用于有明显继承关系的场景
- 接口用于定义行为规范，可以多实现，只能有常量和抽象方法（Java8后可以有默认方法和静态方法）。适用于定义类的能力或功能
**区别**
- 实现方式：实现接口的关键字为`iplements`，继承抽象类的关键字为`extends`。一个类可以实现多个接口，但只能继承一个抽象类
- 方法方式：接口只有定义，不能有方法的实现（Java8后可以有默认方法），而抽象类可以有定义和实现，方法可以在抽象类中实现
- 访问修饰符：接口成员变量默认为`public static final`，必须赋初值，不能被修改；其所有成员方法都是public、abstrac的。抽象类中成员变量默认default，可在子类中被重新定义，也可以被重新赋值；抽象方法必须被abstrac修饰，不能被private、static、sunchronized和native等修饰
- 变量：抽象类可以包含实例变量和静态变量，接口只能包含常量
抽象类不能被final修饰，因为抽象类是用来被继承的，而final修饰的类被禁止继承或方法重写，因此抽象类不能被final修饰
### 接口中的方法
- 抽象方法：抽象方法是接口的核心部分，所有实现接口的类都必须实现这些方法。抽象方法默认是public和abstract
- 默认方法：默认方法是在Java8中引入的，允许接口提供具体实现，实现类可以选择重写默认方法
- 静态方法：静态方法也是Java8中引入的，它们属于接口本身可以直接通过接口名调用，不需要实现类的对象
- 私有方法：私有方法是在Java9中引入的，用于接口中为默认方法或其他私有方法提供辅助功能。这些方法不能被实现类访问，只能在接口内部使用
### 静态变量和静态方法
在Java中，静态变量和敬他方法是与类本身关联的，不是与类的实例关联。它们在内存中只存在一份，可以被类的所有实例共享
**静态变量**
静态变量是类中使用static关键字声明的变量。它们属于类而不是任何具体的对象，有以下特点
- 共享性：所有该类的实例共享同一个静态变量。如果一个实例修改了静态变量的值，其他实例也会看到这个更改
- 初始化：静态变量在类被加载时初始化，只会对其进行一次分配内存
- 访问方式：静态变量可以直接通过类名访问，也可以通过实例访问，但推荐使用类名
**静态方法**
静态方法是在类中使用static关键字声明的方法。类似于静态变量，静态方法也属于类，而不是任何具体的对象。主要有以下特点
- 无实例依赖：静态方法可以在没有创建实例的情况下调用。对于静态方法来说，不能直接访问非静态的成员变量或方法，因为静态方法没有上下文实例
- 访问静态成员：静态方法可以直接调用其他静态变量和静态方法，但不能直接访问非静态成员
- 多态性：静态方法不支持重写，但可以被隐藏
### 静态内部类和非静态内部类的区别
- 非静态内部类依赖外部类的实例，而静态内部类不依赖外部类的实例
- 非静态内部类可以访问外部类的实例变量和方法，而静态内部类只能访问外部类的静态成员
- 非静态内部类不能定义静态成员，而静态内部类可以定义静态成员
- 非静态内部类在外部类实例化后才能被实例化，而静态内部类可以独立实例化
- 非静态内部类可以访问外部类的私有成员，而静态内部类不能直接访问外部类的私有成员，需要通过实例化外部类来访问
非静态内部类可以直接访问外部方法是因为编译器在生成字节码时会为非静态内部类维护一个指向外部类实例的引用
这个引用使得非静态内部类可以访问外部类的实例变量和方法。编译器会在生成非静态内部类的构造方法时，将外部类的实例作为参数传入，并在内部类的实例化过程中建立外部类实例和内部类实例之间的联系，从而实现直接访问外部方法的功能
### final
final关键字主要有以下三方面的作用
- 修饰类：当final修饰一个类时，表示这个类不能被继承，是类继承体系中的最终形态。例如String类就是被final修饰的，保证了String类的不可变性和安全性，防止其他类通过继承来改变String类的行为和特性
- 修饰方法：用final修饰的方法不能在子类中被重写。比如`java.lang.Object`类中的getClass方法就是final的，因为这个方法的行为是有Java虚拟机底层实现来保证的，不应该被子类改写
- 修饰变量：当final修饰基本数据类型时，该变量一旦被赋值就不能再改变。例如`final int num = 10`，这个num就是一个常量，不能再对其进行重新赋值操作，否则会导致编译错误。对于引用数据类型，final修饰意味着这个引用变量不能再指向其他对象，但这个对象本身的内容是可以改变的。例如`final StringBuilder sb = new StringBuilder("Hello ")`，不能让sb变量指向其他的StringBuilder对象，但可以通过`sb.append("World")`来改变字符串内容
### static
static关键字主要用于西施类的成员和内部类，其核心作用是将成员与类本身关联，而非与类的实例关联，具体作用如下
- 修饰变量：被static修饰的变量输入类的本身，而非某个类的实例。所有对象共享一份静态变量，内存中只存在一份副本。可以通过`类名.变量名`直接访问，无需创建对象。通常用于存储所有对象共享的数据
- 修饰方法：静态方法属于类，不属于任何实例，因此不能直接访问类中的非静态成员，但可以访问静态成员，通过`类名.方法名`直接调用，无需创建对象
- 修饰代码块：静态代码块在类加载时执行，且只执行一次，用于初始化静态变量或执行类级别的预处理操作。多个静态代码块按定义顺序执行，且优先于非静态代码块和构造方法
- 修饰内部类：静态内部类不依赖于外部类的实例，可以独立存在，不能直接访问外部类的非静态成员，当内部类与外部类的实例无关时使用，避免内部类持有外部类的引用导致内存泄露
## 深拷贝和浅拷贝
- 浅拷贝是只复制对象本身和其内部的值类型字段，不会赋值对象的引用。浅拷贝知识创建一个新的对象，然后将远对象的字段值复制到新对象中，原对象内部如果有引用类型的字段，就只将引用复制到新对象中，两个对象指向的是同一个引用对象
- 深拷贝是指在复制对象的同事，将对象内部的所有引用类型的字段内容也复制一份，而不是共享引用。深拷贝会递归复制对象内部的所有引用类型的字段，生成一个全新的对象以及其内部的所有对象
### 实现深拷贝的三种方法
- 实现Cloneable接口并重写clone方法
	这种方法要求对象及其引用类型字段都实现了Cloneable接口，并且重写clone()方法。在clone()方法中，通过递归克隆引用类型字段来实现深拷贝
```java
class A implements Cloneable {  
    private B b;  
      
    @Override  
    protected Object clone() throws CloneNotSupportedException {  
        A cloned = (A) super.clone();  
        cloned.b = (B) b.clone();  
        return cloned;  
    }  
}  
  
class B implements Cloneable {  
    @Override  
    protected Object clone() throws CloneNotSupportedException {  
        return super.clone();  
    }  
}
```
- 使用序列化和反序列化
	通过讲对象序列化为字节流，再从字节流反序列化为对象来实现深拷贝。要求对象及其所有引用类型字段都实现Serializable接口
```java
class A implements Serializable {  
  
    private B b;  
  
    public A deepCopy() {  
        try{  
            ByteArrayOutputStream bos = new ByteArrayOutputStream();  
            ObjectOutputStream oos = new ObjectOutputStream(bos);  
            oos.writeObject(b);  
            oos.flush();  
            oos.close();  
            ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());  
            ObjectInputStream ois = new ObjectInputStream(bis);  
            return (A) ois.readObject();  
        } catch (IOException | ClassNotFoundException e) {  
            throw new RuntimeException(e);  
        }  
    }  
}  
  
class B implements Serializable {  
    }
```
- 手动递归复制
	针对特定对象结构，手动递归复制对象及其引用类型字段，适用于对象结构复杂度不高的情况
```java
class A {  
    private B b;  
  
    public void setB(B b) {  
        this.b = b;  
    }  
  
    public B getB() {  
        return b;  
    }  
  
    public A deepCopy(){  
        A a = new A();  
        a.setB(this.b.deepCopy());  
        return a;  
    }  
}  
  
class B {  
    public B deepCopy(){  
        return new B();  
    }  
}
```
## 泛型
泛型是Java编程语言中的一个重要特性，它允许类、接口和方法在定义时使用一个或多个类型参数，这些类型参数在使用时可以被指定为具体类型
泛型的主要目的是在编译时提供更强的类型检查，并且在编译后能够保留类型信息，避免在运行时出现类型转换异常
泛型有以下特点
- 适用于多种数据类型执行相同的代码
```java
private static <T extends Number> double add(T a, T b) {  
    return a.doubleValue() + b.doubleValue();  
}
```
对于上述代码，如果没有泛型，我们就需要重复定义多种类型的add函数，才能保证可以适用于多种基本数据类型
- 泛型中的类型在使用时定义，不需要强制的类型转换、
```java
List list = new ArrayList();  
list.add("a");  
list.add(1);  
list.add(100L);
```
对于上面这样一段代码，list中的元素都是Object类型，所以在取出集合元素时要人为的强制类型转换，且这样子还容易出现`ClassCastException`异常，而通过泛型约束，就可以让list中只存储一种类型的数据
## 对象
### Java中创建对象的方式
- 使用new关键字，这是最常见、最基础的创建对象的方式。通过调用类的构造器来实例化对象。优点是简单、直接、明确。缺点是紧密耦合，必须知道具体类名
- 使用Class类的newInstance()方法，通过Java的反射API，在运行时动态创建对象，这种方式不需要再编译时知道具体的类。JDK9之后，通常会使用`Constructor.newInstance()`方法，因为`Class.newInstance()`方法只能调用无参公有构造器，且会抛出异常，而`Constructor.newInstance()`方法更强大，灵活
```java
Constructor<A> constructor = A.class.getConstructor();  
A a = constructor.newInstance();
```
- 使用clone方法，通过实现Cloneable接口并重写clone()方法，可以基于一个现有对象创建一个新的副本对象
- 使用反序列化，通过`ObjectInputStream`从一个字节流中重建一个对象
- 工厂模式，这是一种设计模式，不直接使用new关键字，而是通过一个方法来返回对象实例。将对象的使用和创建分离，降低耦合，还可以隐藏创建对象的复杂逻辑
### new出的对象什么时候回收
通过new创建的对象，有GC负责回收。垃圾回收器的工作是在程序运行过程中自动进行的，它会周期性地检测不再被引用的对象，并将其回收释放内存
主要有以下几种回收方法
- 引用技术法：某个对象的引用计数为0时，表示该对象不再被引用，可以被回收
- 可达性分析：从GC Root对象出发，通过对象的引用链遍历，所有通过GC Root遍历后可以达到的对象不需要回收，反之不可达的对象，全都会被GC回收
- 终结器(Finalizer)：如果对象重写了`finalize()`方法，垃圾回收器会在回收该对象之前调用`finalize()`方法，对象可以在该方法中进行一些清理操作。但是终结器机制不推荐使用，因为它的执行时间是不确定的，可能会导致不可预测的性能问题
### 获取私有对象
通常情况下，类中声明private的成员变量或方法，这些成员只能在类内部被访问，无法通过外部访问
可以通过以下两种情况访问私有对象
- 使用getter方法，类一般都会为私有成员变量提供公共访问方法，通过调用该方法就可以安全地获取私有对象
- 反射机制：反射机制允许在运行时检查和修改类、方法、字段等信息，通过反射可以绕过private修饰符的限制获取私有对象
```java
public class Test2 {  
    public static void main(String[] args) throws IllegalAccessException {  
        A a = new A();  
        Class<? extends A> clazz = a.getClass();  
        Field[] fields = clazz.getDeclaredFields();  
        for (Field field : fields) {  
            field.setAccessible(true);  
            String val = (String) field.get(a);  
            System.out.println(val);  
            String change = "ababa";  
            field.set(a, change);  
            System.out.println(field.get(a));  
        }  
    }  
}  
  
class A {  
    private String name = "abcsd";  
}
```
```text
abcsd
ababa
```
## 反射
反射机制是在运行状态中，对于任意一个类，都能够知道这个类中的所有属性和方法，对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取信息以及动态调用对象的方法的功能称为反射机制
反射具有以下特性
- 运行时类信息访问：反射机制允许程序在运行时获取类的完整结构信息，包括类名、包名、父类、实现的接口、构造函数、方法、字段等
- 动态对象创建：通过反射API动态获取对象实例，即使在编译时不知道具体的类名。这是通过Class类的newInstance方法或者Constructor类的newInstance方法实现的
- 动态方法调用：可以在运行时动态地调用对象的方法，包括私有方法。这是通过Method的invoke方法实现的，允许传入对象实例和参数值来执行方法
- 访问和修改字段值：反射允许程序在运行时访问和修改对象的值，即使是私有的。通过Field的get和set方法实现
### 应用场景
- 加载数据库驱动：项目底层使用的数据库，需要动态地根据实际情况去加载，有时使用mysql，有时使用oracle。JDBC在不知道是什么类型数据库的情况下，就需要动态地加载获取，可以通过`Class.forName()`方法，通过反射加载数据库驱动程序，如果是mysql则加载mysql驱动，如果是oracle则加载oracle驱动
- 配置文件加载：Spring的IOC，Spring通过配置文件配置各种各样的bean，需要哪些bean就配置哪些，spring会根据需求去动态加载
	Spring通过XML配置模式装载bean的过程：将程序中所有xml或properties配置文件加载入内存；Java类里面解析xml或properties里面的内容，得到对应的实体类字节码字符串以及相关属性信息；使用反射机制，根据字符串获得某个类的Class实例；动态配置实力的属性
## 注解
注解本质上是一个继承了Annotation的特殊接口，其具体实现类是Java运行时生成的动态代理类。通过反射获取注解是，返回的是Java运行时生成的动态代理对象。通过代理对象调用自定义注解的方法，会最终调用`AnnotationInvocationHandler`的invoke方法，该方法会从`memberValues`这个map中索引出对应的值。而memberValues的来源是Java常量池
### 底层实现
注解本质上是一种特殊的接口，继承自`java.lang.annotation.Annotation`接口，所以注解也叫声明式接口
编译后，Java编译器会将其转换为一个继承自Annotation的接口，并生成相应的字节码文件
根据注解的作用范围，Java注解可以分为以下几类
- 源码级注解：仅存在于源码中，编译后不会保留`@Retention(RetentionPolicy.SOURCE)`
- 类文件级别注解：保留在.class文件中，但运行时不可见`@Retention(RetentionPolicy.CLASS)`
- 运行时注解：保留在.class文件中，并且可以通过反射在运行时访问`@Retention(RetentionPolicy.RUNTIME)`
只有运行时注解可以通过反射机制进行解析
当注解被标记为RUNTIME时，Java编译器会在生成的.class文件中保存注解信息。这些信息存储在字节码的属性表中，包括以下内容
- **RuntimeVisibleAnnotations**：存储运行时可见的注解信息
- **RuntimeInvisibleAnnotations**：存储运行时不可见的注解信息
- **RuntimeVisibleParameterAnnotations**和**RuntimeInvisibleParameterAnnotations**：存储方法参数上的注解信息
注解的解析主要依赖于Java的反射机制，下面是解析注解的基本流程
- 获取注册信息：通过反射API可以获取类、方法、字段等元素上的注解
```java
@MyAnnotation("abcabc")  
public class Test2 {  
    public static void main(String[] args) {  
        Class<Test2> clazz = Test2.class;  
        MyAnnotation annotation = clazz.getAnnotation(MyAnnotation.class);  
        if(annotation != null) {  
            System.out.println(annotation.value());  
        }  
    }  
}
```
```text
abcabc
```
- 底层原理：反射机制的核心类是`java.lang.reflect.AnnotatedElement`，它是所有可以被注解修饰的元素的（Class、Method、Field）父接口，该接口提供了以下方法
	`getAnnotation(Class<?>annotationClass)`：获取指定类型的注解
	`getAnnotations()`：获取所有注解
	`isAnnotationPresent(Class<? extends Annotation> annotationClass)`：判断是否包含指定注解
	JVM在加载类时会解析.class文件中的注解信息，并将其存储在内存中，供反射机制使用，因此注解解析的底层实现主要依赖于Java的反射机制和字节码文件的存储。通过`@Retention`元注解可以控制注解的保留策略，当使用`RetentionPolicy.RUNTIME`时，可以在运行时通过反射API来解析注解信息。在JVM层面，会从字节码文件中读取注解信息，并创建注解的代理对象来获取注解的属性值
### 作用域
注解的作用域指的是注解可以应用在哪些程序元素上，例如类、方法、字段等
Java注解的作用域可以分为三种
- 类级别作用域：用于描述类的注解，通常放置在类定义上面，可以用来指定类的一些属性，如类的访问级别、继承关系、注释等
- 方法级别作用域：用于描述方法的注解，通常放置在方法定义上面，可以用来指定方法的一些属性，如方法的访问级别、返回值类型、异常类型、注释等
- 字段级别作用域：用于描述字段的注解，通常放置在字段定义上面，可以用来指定字段的一些属性，如字段的访问级别、默认值、注释等
除了以上三种作用域，Java还提供了其他的注解作用域，例如构造函数作用域和局部变量作用域。这些注解作用域可以用来对构造函数和局部变量进行描述和注释
## 异常
Java的异常体系主要基于`Throwable`及其子类，Throwable下有两个重要子类，分别是`Error`和`Exception`
- Error：表示运行环境的错误，错误是程序无法处理的严重问题，如虚拟机错误、动态链接库失效等。程序不应该捕获这类错误。例如`OutOfMemoryError`、`StackOverflowError`
- Exception：表示程序本身可以处理的异常情况。异常分为两类
	- 非运行时异常（受检异常，Checked Exception）：这类异常在编译时就必须被捕获或声明抛出。它们通常是外部错误，如文件不存在、类未找到等。非运行时异常强制程序员处理这些可能出现的问题，增强了程序的健壮性
	- 运行时异常（非受检异常，Unchecked Exception或RuntimeException）：这类异常特指RuntimeException及其子类。它与Error一起构成了Java中的非受检异常家族。运行时异常有程序逻辑错误导致，如空指针访问、数组越界等。运行时异常是不需要在编译时强制捕获或声明的
### Java异常处理的方法
- try-catch块：用于捕获并处理可能抛出的异常。try块中包含可能抛出异常的代码，catch块用于捕获并处理特定类型的异常。可以有多个catch块来处理不同类型的异常
- throw语句：用于手动抛出异常，可以根据需要在代码中使用throw语句主动抛出特定类型的异常
- throws关键字：用于在方法声明中声明可能抛出的异常类型。如果一个方法可能抛出异常，但不想在方法内部进行处理，可以使用throws关键字将异常传递给调用者来处理
- finally块：用于定义无论异常是否发生都会执行的代码块。通常用于释放资源，确保资源正常关闭
```java
try {
	return a;
}finally {
	return b;
}
```
上述代码中，最终的返回结果是b不是a
## Object
Object类是所有类的超类，默认提供11个核心方法，用于对象比较、哈希、字符串表示、线程同步等
- equals：默认实现是比较两个对象的内存地址，和\==的效果一样，但实际开发中通常需要按对象的内容比较
- hashCode：如果重写了equals方法就必须重写hashCode方法。因为Java的约定是如果两个对象的equals返回true，它们的hashCode必须相等；如果hashCode不相等，equals必须返回false。如果只重写equals不重写hashCode，会导致对象在HashMap和HashSet等集合中无法正确存储
- toString：默认返回`类名@对象的哈希码十六进制表示`，可读性差。实际开发中重写它是为了返回对象的具体信息，方便日志打印和调试
- getClass：返回对象运行时的实际类对象，和编译时类型可能不同，这个方法不能重写。比如父类Animal和子类Dog，`Animal animal = new Dog()`，animal.getClass()返回的是Dog的类对象，常用于反射场景
- clone：用于创建对象的浅拷贝，默认浅拷贝意味着如果对象有引用类型属性，拷贝后的对象和原对象会共享该引用属性。使用clone需要让类实现Cloneable接口，否则会抛出`CloneNotSupportedException`
- notify和notifyAll：它们都是用于多线程同步的，和synchronized配合使用，作用是唤醒等待当前对象锁的线程。notify是随机唤醒一个等待线程，notifyAll是唤醒所有等待线程，比如生产者消费者模式中，生产者生产完数据后调用notifyAll，唤醒等待的消费者线程
- wait：作用是让当前持有对象锁的线程释放锁并进入等待状态，直到被notify或notifyAll唤醒或等待时间到期。同样需要在synchronized同步块或方法中使用，否则会跑`IlleaglMonitorStateException`
- finalize：它是对象被垃圾回收器回收前会调用的方法，默认是空实现。现在基本不推荐使用，因为它的执行时机不确定，可能很久才执行甚至不执行，而且可能导致对象复活，影响垃圾回收效率。Java 9 之后使用`try with resources`或`PhantomReference`来处理资源释放
### \==和equals的区别
`==`比较的是地址，`equals`比较的是内容
Java中，`==`对于基本数据类型，比较的是数值相不相等，对于引用类型，比较的是两个变量是不是指向内存中的同一个对象
`equals`方法是Object类里定义的一个方法，默认实现和`==`一样，比较的是地址，但是有许多常用的类(String、Integer)都把这个方法给重写了，重写之后比较的就不是地址，而是对象里实际存储的内容是否相等
对于字符串常量池中的对象
```java
String a = "hello";
String b = "hello";
```
a和b虽然是两个不同的对象，通过`==`来比较，返回的是true，这是因为通过双引号创建字符串的时候，JVM会将其放入到字符串常量池中。如果池中有了对应的字符串，就会直接复用这个字符串，而不是新建一个字符串，所以a和b的地址是相同的，\==就返回true了
### hashCode和equals的关系
Java中，对于重写equals方法的类，通常也需要重写hashCode方法，并且要满足一下规定
- 如果两个对象使用equals方法比较结果为true，那么他们的hashCode值必须要相等，即`obj1.equasl(obj2)`返回true，`obj1.hashCode()`就必须等于`obj2.hashCode()`
- 如果两个对象的hashCode值相同，它们的equals返回的不一定是true，即如果`obj1.hashCode() == obj2.hashCode()`返回true，`obj1.equals(obj2)`不一定返回true，因为有哈希冲突的情况
hashCode和equals方法是紧密相关的，重写equals方法时必须重写hashCode方法，以保证在使用哈希表等数据结构的时候，对象的相等性判断和存储查找操作能够正常工作。重写hashCode方法时，需要确保相等的对象具有相同的哈希码，相同哈西码的对象不一定相等
### String、StringBuilder、StringBuffer的区别
String是不可变对象，创建了之后就不能再修改。每次修改时都会创建出一个新的String对象，因此适用于一些修改较少的情况
StringBuilder是可变的，但是StringBuilder线程不安全，多线程环境下可能会出现错误
StringBuffer是线程安全的StringBuilder，其方法通过synchronized实现同步
String性能最低，因为在修改时会创建大量的临时对象，StringBuffer的性能其次，因为其内部需要通过synchronized来保证线程安全，StringBuilder的性能最好，因为它不需要保证线程安全，少了同步的开销，适合单线程下的字符串操作
## Java8新特性
Java8主要有以下新特性：
- Lambda表达式：Lambda表达式是一种简洁的语法，用于创建匿名函数，主要用于简化函数式接口的使用。有以下两种形式
	`(paramters) -> expression`：当Lambda中只有一个表达式时，表达式的结果会作为返回值
	`(paramters) -> {statements;}`：当Lambda中包含多条语句时，需要使用大括号将所有语句括起来，若有返回值则使用return语句
- 函数式接口：仅包含一个抽象方法的接口，可用`@FunctionalInterface`注解标记
- Stream API：提供链式操作处理集合数据，支持并行处理
- Optional类：封装可能为null的对象，减少空指针异常
- 方法引用：简化Lambda表达式，直接引用现有方法
- 接口的默认方法和静态方法：接口可定义默认实现和静态方法，增强扩展性
- 并行数组排序：使用多线程加速数组排序，`Arrays.parallelSort(array)`
- 重复注解：允许同一位置多次使用相同注解，`@Repeatable`注解配合容器注解使用
- 类型注解：注解可以应用于更多位置（泛型、异常等），`List<@Nonnull String> list`
- CompletableFuture
	Future用于表示异步计算的结果，只能通过阻塞或者轮询的方式获取结果，而且不支持设置回调方法，Java8之前若要设置回调一般会使用guava的ListenableFuture，回调的应用会导致回调地狱
	CompletableFuture对Future进行了扩展，可以通过设置回调的方式处理计算结果，同时也支持组合操作，支持进一步编排，同时一定程度解决了回调地狱的问题
	假设三个操作step1、step2、step3存在依赖关系，其中step3的执行依赖step1和step2的结果
```java
public static void main(String[] args) {  
    ExecutorService executor = Executors.newFixedThreadPool(5);  
    ListeningExecutorService guavaExecutor = MoreExecutors.listeningDecorator(executor);  
    ListenableFuture<String> future1 = guavaExecutor.submit(() -> {  
        System.out.println("step1");  
        return "step1 result";  
    });  
    ListenableFuture<String> future2 = guavaExecutor.submit(() -> {  
        System.out.println("step2");  
        return "step2 result";  
    });  
    ListenableFuture<List<String>> future1And2 = Futures.allAsList(future1, future2);  
    Futures.addCallback(future1And2,new FutureCallback<List<String>>() {  
        @Override  
        public void onSuccess(List<String> result) {  
            System.out.println(result);  
            ListenableFuture<String> future3 = guavaExecutor.submit(() -> {  
                System.out.println("step3");  
                return "step3 result";  
            });  
            Futures.addCallback(future3,new FutureCallback<String>() {  
                @Override  
                public void onSuccess(String result) {  
                    System.out.println(result);  
                }  
  
                @Override  
                public void onFailure(Throwable t) {  
                }            },guavaExecutor);  
        }  
  
        @Override  
        public void onFailure(Throwable t) {  
  
        }},guavaExecutor);  
}
```
CompletableFuture的实现
```java
public static void main(String[] args) {  
    ExecutorService executor = Executors.newFixedThreadPool(5);  
    CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> {  
        System.out.println("step1");  
        return "step1 result";  
    });  
    CompletableFuture<String> cf2 = CompletableFuture.supplyAsync(() -> {  
        System.out.println("step2");  
        return "step2 result";  
    });  
    cf1.thenCombine(cf2, (result1,result2)-> {  
        System.out.println(result1+","+result2);  
        System.out.println("step3");  
        return "step3 result";  
    }).thenAccept(result3 -> System.out.println(result3));  
}
```
很明显，CompletableFuture的实现更简洁，可读性更好
CompletableFuture实现了两个接口，Future和CompletionStage
- Future表示异步计算的结果，CompletionStage用于表示异步执行过程中的一个步骤，这个步骤可能是由另外一个CompletionStage出发的，随着当前步骤的完成，也可能会出发其他一系列CompletionStage的执行
- CompletionStage接口定义了这样的能力，让我们可以根据实际业务对这些步骤进行多样化的编排组合，我们可以通过其提供的thenApply、thenCompose等函数式编程方法来组合编排这些步骤
### Java21新特性
- Switch语句的模式匹配：允许在switch的case标签中使用模式匹配，使操作更加灵活和类型安全，减少了样板代码和潜在错误。例如，可以在switch语句中直接根据账户类型的模式来获取相应的余额，如`case savingsAccount sa -> result = sa.getSavings()`
- 数组模式：将模式匹配扩展到数组中，使开发者能够在条件语句中更高效地解构和检查数组内容。例如，`if(arr instanceof int[]{1,2,3})`，可以直接判断数组arr是否匹配指定的模式
- 字符串模板：提供了一种更可读、更易维护的方式来构建复杂字符串，支持在字符串字面量中嵌入表达式，比如`hello {name},welcom to the world`
- 虚拟线程：这是一种轻量级并发的新选择。通过共享堆栈的方式，大大降低了内存消耗，同时提高了应用程序的吞吐量和响应速度。可以使用静态构建方法，
# Java集合
## List
List中主要有以下几个重要的实现类
- Vector是Java早期提供的线程安全的动态数组，Vector内部是使用对象数组来保存数据的，可以根据需要自动的增加容量，当数组已满时，会创建新的数组，并拷贝原有数组数据
- ArrayList是应用更加广泛的动态数组实现，它本身不是线程安全的，性能更好。与Vector类似，ArrayList也可以根据需要调整容量，不过两者的调整逻辑有区别，Vector在扩容时会提高1倍，而ArrayList是增加50%
- LinkedList，是Java提供的双向链表，所以不需要自动调整容量，也不是线程安全的
使用普通的for循环遍历，可以在遍历过程中修改元素，只要修改的索引不超过List范围即可
而对于forEach循环，一般不建议在forEach循环中直接修改正在遍历的元素，这可能会导致意外的结果或`ConcurrentModificationException`异常，在forEach循环中修改元素可能会破坏迭代器内部状态，在遍历过程中修改集合结构，会导致迭代器的预期结构和实际结构不一致
```java
public void forEach(Consumer<? super E> action) {  
    Objects.requireNonNull(action);  
    final int expectedModCount = modCount;  
    final Object[] es = elementData;  
    final int size = this.size;  
    for (int i = 0; modCount == expectedModCount && i < size; i++)  
        action.accept(elementAt(es, i));  
    if (modCount != expectedModCount)  
        throw new ConcurrentModificationException();  
}
```
在forEach时会设置一个exceptedModCount，当forEach循环中修改了元素，就会导致modCount被修改，从而导致exceptedModCount和modCount不一致，抛出异常
对于线程安全的List，如CopyOnWriteArrayList，由于采用了写时复制的机制，在遍历的同事可以进行修改操作，不会抛出`ConcurrentModificationException`异常，但可能会读取到旧数据，因为修改操作是在新副本上进行的
### ArrayList和LinkedList
ArrayList和LinkedList有以下区别
- 底层的数据结构不同：ArrayList使用数组实现，通过索引进行快速访问元素。LinkedList使用链表实现，通过节点之间的指针进行元素的访问和操作
- 插入和删除操作的效率不同：ArrayList在尾部的插入和删除操作效率高，但在中间或开头的插入和删除效率较低，需要移动元素。LinkedList在任意位置的插入和删除效率都较高，因为只需要调整节点之间的指针，但是LinkedList不支持随机访问，除了头结点外插入和珊瑚虫的时间复杂度都是O(n)，效率也不是很高
- 随机访问的效率不同：ArrayList支持通过索引进行快速随机访问，时间复杂度为O(1)。LinkedList需要从头到尾开始遍历链表，时间复杂度为O(n)
- 空间占用：ArrayList在创建时需要分配一段连续的内存空间，因此会占用较大的空间，LinkedList每个节点只需要存储元素和指针，因此相对较小
- 使用场景：ArrayList适用于频繁随机访问和尾部插入删除操作，而LinkedList适用于频繁的中间插入删除操作和不需要随机访问的场景
- 线程安全：这两个线程都不是线程安全的，Vector是线程安全的
### 为什么ArrayList线程不安全
在高并发添加数据下，ArrayList会暴露三个问题
- 部分值为null，即使我们没有add null
- 索引越界异常
- size与我们add的数量不符
Java中的add方法大致分为以下步骤
- 判断如果将当前的新元素添加到列表后面，列表的elementData数组的大小是否满足，如果size+1的这个需求长度大于了elementData这个数组的长度，那么就要对这个数组进行扩容
- 判断数组需不需要扩容，如果需要，调用grow方法进行扩容
- 将数组的size位置设置值
- 将当前集合的大小加一
对于上述三种线程不安全的问题，由以下产生
- 部分值为null：当线程1走到扩容那里时发现当前size是9，而数组容量是10，所以不用扩容，这时候cpu让出执行权，线程2也进来了，发现size是9，而数组容量是10，所以不用扩容，这时候线程1继续执行，将数组下标索引为9的位置占用了，还没有来得及执行size++，这时候线程2也来执行了，又把索引为9的位置set了一遍，这时候两个线程先后进行size++，导致下标索引10的地方为null
- 索引越界异常：线程1走到扩容那里发现当前size是9，数组容量是10，不用扩容，cpu让出执行权，线程2也发现不用扩容，这时候数组容量就是10，而线程1set完之后size++，这时候线程2再进来size就是10，而数组大小只有10，而要设置下标索引为10的时候，就会导致越界
- size与add的数量不符：size++本身就不是原子操作，可以分为三步，获取size的值，将size的值加1，将新的size值覆盖掉原来的，线程1和线程2拿到一样的size值加完了同时覆盖，就会导致有一次没有加上，所以会导致size和add的数量不符
### ArrayList的扩容机制
ArrayList在添加元素时，如果当前元素个数已经达到了内部数组的容量上限，就会触发扩容操作。扩容操作主要包含以下几个内容：
- 计算新的容量：一般情况下，新的容量会扩大为原容量的1.5倍，然后检查是否超过了最大容量限制
- 创建新的数组：根据计算得到的新容量，创建一个新的更大的数组
- 将元素复制：将原来数组中的元素逐个复制到新数组中
- 更新引用：将ArrayList内部指向原数组的引用指向新数组
- 完成扩容：扩容完成后，可以继续添加新元素
ArrayList的扩容操作涉及到数组的复制和内存的重新分配，所以在频繁添加大量元素时，可能会影响性能。为了减少扩容带来的性能损耗，可以在初始化ArrayList时预分配足够大的容量，避免频繁触发扩容操作
之所以扩容1.5倍，是因为1.5可以通过移位操作，减少浮点数或者运算时间和运算次数
```java
int newCapacity = oldCapacity + (oldCapacity >> 1);
```
### CopyonWriteArrayList
CopyonWriteArrayList底层是通过一个数组保存数据，使用volatile关键字修饰数组，保证当前线程对数组对象重新赋值后，其他线程可以及时感知到
```java
private transient volatile Object[] array;
```
对于add操作，CopyonWriteArrayList加了一把互斥锁以保证线程安全
```java
public boolean add(E e) {
	final ReentrantLock lock = this.lock;
	lock.lock();
	try {
		Object elements = getArray();
		int len = elements.length;
		Object newElements = Arrays.copyOf(elements,len+1);
		newElements[len] = e;
		setArray(newElements);
		return true;
	} finally {
		lock.unlock();
	}
}
```
从源码中可以知道，写入新元素时，会先讲原数组拷贝一份并且让原数组的长度+1后就得到了新数组，新数组里的元素和旧数组的元素一样并且长度比旧数组多1，然后将新加入的元素放置在新数组的最后一个位置后，再用新数组的地址替换旧数组的地址
在执行替换地址操作时，读取到的是老数组的数据，数据是有效数据；执行替换地址操作之后，读取的数据是新数组的数据，同样也是有效数据，而且使用该方式能比读写都加锁要有更好的性能
在之后的版本，由于jdk对synchronized锁进行了许多优化，因此就改用了synchronized锁而不是ReentrantLock
```java
public boolean add(E e) {  
    synchronized (lock) {  
        Object[] es = getArray();  
        int len = es.length;  
        es = Arrays.copyOf(es, len + 1);  
        es[len] = e;  
        setArray(es);  
        return true;  
    }  
}
```
### List泛型里面填基本数据类型为什么会报错
Java的泛型机制在设计的时候就只支持引用类型，不支持基本数据类型
原因如下
- 泛型的类型擦除机制：Java泛型在编译后会被擦除为Object类型，而Object只能几首引用类型，不能接受基本数据类型
- Java最初设计时，基本数据类型和引用类型是严格区分的，泛型时后续才引入的特性，为了兼容已有的类型系统，选择只支持引用类型

## Map
**常见的Map集合**
- **HashMap**是基于哈希表实现的Map，它根据键的哈希值来存储和获取键值对，JDK1.8中使用数组+链表+红黑树实现。HashMap是非线程安全的，多线程环境下，多个线程同时对HashMap进行操作时，可能会导致数据不一致或出现死循环的问题
- **LinkedHashMap**继承自HashMap，在HashMap的基础上，使用双向链表维护了键值对的插入顺序或访问顺序，使得迭代顺序与插入顺序或访问顺序一致
- **TreeMap**是基于红黑树实现的Map，它可以对键进行排序，默认按照自然顺序排序，也可以通过指定比较起进行排序。TreeMap是非线程安全的，在多线程环境下，如果多个线程对TreeMap进行操作，可能会破坏红黑树的结构，导致数据不一致
- **Hashtable**是早期Java提供的线程安全的Map实现，它的实现方式和Map类似，但在方法上使用了synchronized关键字来保证线程安全。通过在每个可能修改Hashtable状态的方法上加上了synchronized关键字，使得同一时刻只能有一个线程访问Hashtable的这些方法，从而保证了线程安全
- **ConcurrentHashMap**在JDK1.8以前，采用分段锁的技术来提高并发性能。在ConcurrentHashMap中，将数据分为多个段，每个段都有自己的锁。在进行修改操作时，只需要获取相应段的锁，而不是整个Map的锁，就可以允许多个线程同时访问不同的段，提高了并发效率。JDK1.8之后，采用volatile+CAS或synchronized来保证线程安全
**对Map进行快速遍历**
- 可以采用forEach循环和entrySet()方法，可以同时获取Map中的键和值
```java
public class MapTest {  
    public static void main(String[] args) {  
        Map<String,Integer> map = new HashMap<>();  
        map.put("A", 1);  
        map.put("B", 2);  
        map.put("C", 3);  
        map.put("D", 4);  
        map.put("E", 5);  
        for (Map.Entry<String, Integer> entry : map.entrySet()) {  
            System.out.println(entry.getKey() + " " + entry.getValue());  
        }  
    }  
}
```
- 采用forEach循环和keySet()方法，可以只遍历Map中的键，性能较好
```java
public class MapTest {  
    public static void main(String[] args) {  
        Map<String,Integer> map = new Hashtable<>();  
        map.put("A", 1);  
        map.put("B", 2);  
        map.put("C", 3);  
        map.put("D", 4);  
        map.put("E", 5);  
        for (String key : map.keySet()) {  
            System.out.println(key + " : " + map.get(key));  
        }  
    }  
}
```
- 使用迭代器，获取entrySet()或keySet()的迭代器，也可以实现遍历Map，这种方法在需要删除元素等操作时比较有效
- lambda表达式和forEach()方法，这种方式简洁和函数式
- Stream API，Java 8 后使用Stream API也可以遍历Map，将Map转换为Stream流，可以进行各种操作
### HashMap
HashMap底层是一个长为16的动态数组，当有键值对添加时，就会通过哈希函数映射到对应下标位置，数组的每个元素都是一个链表
当链表长度大于8时，链表就会变成红黑树，当链表长度小于6时，就会从红黑树变回链表
当数组中的元素数量大于默认负载因子(0.75，即达到数组容量的0.75)时，数组就会扩容为原来的两倍大小
多线程环境下，同时对数据进行修改，会发生数据丢失的问题。HashMap是线程不安全的
#### HashMap实现原理
jdk1.7之前，HashMap底层是由数组+链表实现的，HashMap通过哈希算法将元素的键映射到数组对应的槽位中。如果多个键映射到同一个槽位，它们就会以链表的形式存储在同一个槽位上。但是由于链表的查询时间是O(n)，所以一旦链表长度变长了，查询效率就会很低
jdk1.8之后，**当一个链表的长度超过8之后，就会将链表转换成红黑树**，查找时使用红黑树，时间复杂度就变成了O(logn)，**当链表长度小于6的时候，会将红黑树转换回链表**
**为什么不使用平衡二叉树**
AVL树是严格平衡的二叉树，要求任意节点的左右子树高度差不超过1，这就意味着：
- 插入、删除操作会触发大量的旋转操作，来保证AVL树的平衡
- HashMap的场景是链表转树，仅发生在链表长度>=8时，本身是低频场景，为了这种低频场景付出高频的平衡开销，很不划算
- 红黑树金宝铮黑色高度平衡，旋转次数远少于AVL树，插入、删除操作的平均时间复杂度仍为O(logn)，但实际执行效率更高
HashMap的核心是`哈希+数组+链表/树`，树的作用只是解决哈希冲突严重导致链表过长的问题，而非纯做树形存储
- 红黑树的查找、删除、插入的时间复杂度都是O(logn)，虽然比AVL树的查找略慢，但增删的开销远低于AVL树
- 对于HashMap来说，增删和查找的频率几乎持平，红黑树的综合性能更优
#### 解决哈希冲突的方法
- 链接法：使用链表或其他数据结构来存储冲突位置键值对，将它们链接在同一个哈希桶中
- 开放寻址法：在哈希表中找到另一个可用的位置来存储冲突的键值对，而不是存储在链表中。可以使用线性探测、二次探测或双重散列等方法
- 再哈希法：当发生冲突时，使用另一个哈希函数再次计算键的哈希值，直到找到一个空槽来存储键值对
- 哈希桶扩容：当哈希冲突过多时，可以动态的扩大哈希桶的数量，重新分配键值对，减少冲突的概率
#### get元素的过程
```java
public V get(Object key) {  
    Node<K,V> e;  
    return (e = getNode(key)) == null ? null : e.value;  
}
```
上面是HashMap中的get方法
```java
final Node<K,V> getNode(Object key) {  
    Node<K,V>[] tab; Node<K,V> first, e; int n, hash; K k;  
    if ((tab = table) != null && (n = tab.length) > 0 &&  
        (first = tab[(n - 1) & (hash = hash(key))]) != null) {  
        if (first.hash == hash && // always check first node  
            ((k = first.key) == key || (key != null && key.equals(k))))  
            return first;  
        if ((e = first.next) != null) {  
            if (first instanceof TreeNode)  
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);  
            do {  
                if (e.hash == hash &&  
                    ((k = e.key) == key || (key != null && key.equals(k))))  
                    return e;  
            } while ((e = e.next) != null);  
        }  
    }  
    return null;  
}
```
从getNode方法中，可以知道
- 先判断HashMap中存储数据的数组table不为null且长度大于0，且通过hash值计算出的节点存放的位置有节点存在
- 位置上有节点存在时，就到指定的位置来查询
- 先判断第一个节点key是否与参数的key值相等，若相等，则说明这个节点就是我们要找的节点，直接返回；若不相等，则判断第一个节点是否有下一个节点，若有继续判断，若没有则表示要找的节点不存在
- 若第一个节点是树节点，则通过红黑树的查找算法进行查找，若第一个节点不是树节点，通过do...while循环遍历链表来查找
- 没有找到对应的节点，则返回null
#### put过程
- 先根据要添加的键的哈希码计算在数组中的位置
- 检查该位置是否为空，如果为空，则直接在该位置创建一个新的Entry对象来存储键值对。将要添加的键值对作为该Entry的键和值，并保存在数组的对应位置。将HashMap的修改次数(modCount)加1
- 如果该位置已经存在其他键值对，检查该位置的第一个键值对的哈希码和键是否与要添加的键值对相同。如果相同，则表示找到了相同的键，直接将新值替换旧值，完成更新操作
- 如果第一个键值对的哈希码和键不相同，则需要遍历链表或红黑树来查找是否有相同的键
	如果键值对集合是链表结构，从链表的头部开始逐个比较键的哈希码和equals()方法，直到找到相同的键或到达链表尾部。如果找到了相同的键，则用新值替代旧值。如果没找到相同的键，则将新的键值对添加到链表头部
	如果键值对集合是红黑树结构，在红黑树中使用哈希码和equals()方法进行查找。根据键的哈希码，定位到红黑树的某个节点，然后逐个比较，直到找到相同的键或到达红黑树末尾。如果找到了相同的键，则用新值替代旧值。如果没找到相同的键，则将新的键值对添加到红黑树中
- 检查负载因子是否超过阈值(0.75)，如果键值对的数量与数组长度的比值大于阈值，则需要进行扩容操作
- 扩容操作。先创建一个两倍大小的数组，将旧数组中的键值对重新计算哈希码并分配到新数组中的对应位置。更新HashMap的数组引用和阈值参数
#### 为什么String适合做key
用String做key，因为String对象是不可变的，一旦创建就不能被修改，这保证了key的稳定性。如果key是可变的，可能会导致hashCode和equals方法的不一致，进而影响HashMap的正确性
#### HashMap key可以为null吗
可以
HashMap中使用hash()方法来计算key的哈希值，当key为空时，会直接令key的哈希值为0，不使用hashCode()方法
HashMap虽然支持key和value为null，但是null作为key只能有一个，null作为value可以有多个。因为HashMap中，如果key值一样，那么会覆盖相同key值的value为最新，所以key为null只能有一个
```java
static final int hash(Object key) {  
    int h;  
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);  
}
```
#### 重写HashMap的equals和hashCode方法
HashMap使用key对象的hashCode()和equals()方法去决定key-value对的索引。当我们试着从HashMap中获取值的时候，这些方法也会被用到。如果这些方法没有被正确实现，就有可能出现两个不同的key产生相同的hashCode()和equals()输出，然后HashMap就认为它们是相同的，然后覆盖它们，而非把它们存储到不同的地方
同样的，所有不允许存储重复数据的集合类都使用hashCode()和equals()去查找重复，所以实现这两个方法必须要遵循以下规则
- 如果`o1.equals(o2)`那么`o1.hashCode() == o2.hashCode()`总是为true
- 如果`o1.hashCode == o2.hashCode()`，不意味着`o1.equals(o2)`总是为true
#### 重新HashMap的equals方法不当会出现的问题
HashMap在比较元素时，会先通过hashCode进行比较,如果相同，再使用equals方法进行比较
所以equals相等的两个对象，hashCode一定相等 hashCode相等的两个对象，equals不一定相等（散列冲突）
重写了equals方法，不重写hashCode方法时，可能会出现equals方法返回true，而hashCode方法却返回false，就可能导致HashMap等类存储多个一模一样的对象，导致出现覆盖存储的数据的问题，这与HashMap只能由唯一的key的规范不符合
#### 多线程下出现的问题
- jdk1.7的HashMap使用头插法插入元素，多线程环境下，扩容的时候可能导致环形链表的出现，形成死循环。因此，1.8之后使用尾插法插入元素，在扩容时会保持链表元素原本的顺序，不会出现环形链表的问题
- 多线程同时执行put操作，如果计算出来的索引位置时相同的，会造成前一个key被后一个key覆盖，从而导致元素的丢失。
#### HashMap的扩容机制
HashMap默认的负载因子是0.75，即存储的元素数量超过了总容量的75%时，就会触发扩容机制，有以下两个步骤
- 对哈希表长度的扩展
- 将旧哈希表中的数据放到新的哈希表中
因为使用的是2次幂的扩展，所以元素的位置要么是在原位，要么是在原位置再移动2次幂的位置
![](assets/Java基础/file-20260312163249378.png)
因此在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit，因此新的index就会发生一些变化
![](assets/Java基础/file-20260312163756989.png)
因此，我们在扩充HashMap的时候，不需要重新计算hash，只需要看看原来的hash值新增的那个bit是0还是1就好了，是0的话索引不变，是1的话索引变成"原索引+oldCap"
```java
if ((e.hash & oldCap) == 0) {  
    // ...
}  
else {  
    // ...
}
```
上述代码是resize()时发生的，因为HashMap的长度都是2次幂，因此oldCap的高位是1，而对于hash，如果hash的高位是1，在进行`hash & oldCap`时，就会加上oldCap的值，如果hash的高位是0，在进行`hash & oldCap`时，就不变
#### HashMap的大小为什么是2的n次方大小
jdk1.7中，HashMap整个扩容过程就是分别取出数组元素，一般该元素是最后一个放入链表中的元素，然后遍历以该元素为头的单向链表元素，依据每个被遍历元素的hash值计算其在新数组中的下标，然后进行交换。这样的扩容方式会将原来哈希冲突的单向链表尾部变成扩容后的单向链表头部
jdk1.8中，HashMap对扩容操作做了优化。由于扩容数字的长度是2倍关系，所以对于假设初始tableSize=4要扩容到8来说，就是从0100变成了1000（左移一位），在扩容中就只用判断原来的hash值和左移动的一位按位与操作是0或1就行，0的话索引不变，1的话变成原索引加上扩容前数组长度
之所以能通过这种与运算来重新分配索引，是因为hash值本来就是随机的，而hash按位与上newTable得到的0和1就是随机的，所以扩容的过程就能把之前哈希冲突的元素再随机分不到不同的索引中
#### HashMap和Hashtable的区别
- HashMap线程不安全，效率更高，可以存储null的key和value，null的key只能有一个，null的value可以有多个。默认初始容量为16，每次扩充变为原来的两倍。创建时如果给定了初始容量，则扩充为2的幂次方大小。底层使用数组+链表，插入元素后如果链表长度大于阈值，先判断数组长度是小于64，如果小于则扩充数字，反之将链表转化为红黑树，减少搜索时间
- Hashtable线程安全，效率更低，其内部方法基本都经过synchronized修饰，不可以有null的key和value。默认初始容量为11，每次扩容都变为原来的2n+1。创建时给定了初始容量，就会直接使用给定的大小。底层数据结构为数组+链表。现在已经基本被淘汰了
### ConcurrentHashMap
ConcurrentHashMap就是在HashMap的基础上，保证线程安全
jdk1.7中，使用数组+链表的形式实现，而数组又分为大数组Segment和小数组HashEntry
Segment是一种可重入锁，HashEntry则用于存储键值对数据
一个ConcurrentHashMap中包含一个Segment数组，一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素
jdk1.7 ConcurrentHashMap分段锁技术将数据分成一段一段存储，然后给每段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问，就能够实现并发访问
jdk1.8后，ConcurrentHashMap使用了数组+链表/红黑树的方式优化了ConcurrentHashMap的实现
jdk1.8 ConcurrentHashMap主要通过volatile+CAS或者synchronized来实现线程安全。添加元素时首先会判断容器是否为空
- 如果为空，则使用volatile+CAS来初始化
- 如果不为空，则根据存储的元素计算该位置是否为空
	- 如果根据存储的元素计算结果为空，则利用CAS设置该节点
	- 如果根据存储的元素计算结果不为空，则使用synchronized，然后遍历桶中的数据，并替换或新增节点到桶中，最后在判断是否需要转换为红黑树，这样就能保证并发访问时的线程安全了
ConcurrentHashMap通过对头结点枷锁来保证线程安全，锁的粒度相比Segment来说更小了，发生冲突和加锁的频率降低了，并发操作的性能提高了
#### 分段锁是怎么加锁的
ConcurentHashMap中，将整个数据结构分为多个Segment，每个Segment都类似于一个小的HashMap，每个Segment都有自己的锁，不同Segment之间的操作互不影响，从而提高并发性能
在ConcurrentHashMap中，对于插入、更新、删除等操作，需要先定位到具体的Segment，然后再在该Segment上加锁，而不是像传统HashMap一样对整个数据结构加锁。这样可以使得不同Segment之间的操作可以并行进行，提高了并发性能
且jdk1.7中ConcurrentHashMap中的分段锁使用的是ReentrantLock，是可重入的
#### 为什么有了synchronized还要使用CAS
ConcurrentHashMap会权衡考虑使用哪个来加锁
在putVal中，如果计算出来的hash槽没有存放元素，那么就可以直接使用CAS来进行设置值，这是因为在设置元素的时候，因为hash值经过了各种扰动后，造成hash碰撞的几率较低，那么就可以预测使用较少的自旋来完成具体的hash落槽操作
当发生了hash碰撞的时候，就说明容量不够用了，或者已经有大量线程访问了，因此这时候使用synchronized来处理hash碰撞比CAS效率更高，因为发生了hash碰撞大概率是线程竞争比较激烈的情况
#### Hashtable和ConcurrentHashMap的区别
- jdk1.8之前，ConcurrentHashMap采用分段锁，对整个数组进行分段，每一把锁只锁容器里的一部分数据，多线程访问不同数据段里的数据，就不会存在锁竞争，提高了并发访问。jdk1.8之后，直接采用数组+链表/红黑树，并发控制使用CAS和synchronized操作，更提高了速度
- Hashtable所有的方法都通过synchronized加锁来保证线程安全，效率很低。当两个线程同步访问时，就会陷入阻塞或者轮询状态
## Set
Set集合中的元素是唯一的，不会出现重复的元素
Set集合通过内部的数据结构来实现key的无重复。当向Set集合中插入元素时，会先根据元素的hashCode值来确定元素的存储位置，然后再通过equals来判断是否已经存在相同的元素，如果存在则不会再次插入，保证了元素的唯一性
其中
TreeSet和LinkedHashSet是有序的。TreeSet是基于红黑树实现的，保证元素的自然顺序。LinkedHashSet是基于双重链表和哈希表的结合来实现元素的有序存储，保证元素添加的自然顺序。LinkedHashSet不仅保证元素的唯一性，还可以保证元素的插入顺序。




