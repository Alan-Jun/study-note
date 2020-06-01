Java 虚拟机的字节码指令集的数量自从 Sun 公司的第一款 Java 虚拟机问世至 JDK 7 来临之前的十余年时间里，一直没有发生任何变化[ [1\] ](https://www.infoq.cn/article/jdk-dynamically-typed-language#_ftn1_7498)。随着 JDK 7 的发布，字节码指令集终于迎来了第一位新成员——invokedynamic 指令。这条新增加的指令是 JDK 7 实现“动态类型语言（Dynamically Typed Language）”支持而进行的改进之一，也是为 JDK 8 可以顺利实现 Lambda 表达式做技术准备。在这篇文章中，我们将去了解 JDK 7 这项新特性的出现前因后果和它的意义。

## 动态类型语言

在介绍 JDK 7 提供的动态类型语言支持之前，我们要先弄明白动态类型语言是什么？它与 Java 语言、Java 虚拟机有什么关系？了解 JDK 7 提供动态类型语言支持的技术背景，对理解这个语言特性是很必要的。

何谓动态类型语言[ [2\] ](https://www.infoq.cn/article/jdk-dynamically-typed-language#_ftn2_7498)？动态类型语言的关键特征是它的类型检查的主体过程是在运行期而不是编译期进行的，满足这个特征的语言有很多，常用的包括：APL、Clojure、Erlang、Groovy、JavaScript、Jython、Lisp、Lua、PHP、Prolog、Python、Ruby、Smalltalk 和 Tcl 等等。那相对地，在编译期就进行类型检查过程的语言，如 C++ 和 Java 等就是最常用的静态类型语言。

觉得上面定义过于概念化？那我们不妨通过两个例子以最浅显的方式来说明什么是“在编译期 / 运行期进行”和什么是“类型检查”。首先看这段简单的 Java 代码，它是否能正常编译和运行？

```
public static void main(String[] args) { 
    int[][][] array = new int[1][0][-1]; 
} 
```

这段代码能够正常编译，但运行的时候会报 NegativeArraySizeException 异常。在《Java 虚拟机规范》中明确规定了 NegativeArraySizeException 是一个运行时异常，通俗一点说，运行时异常就是只要代码不运行到这一行就不会有问题。与运行时异常相对应的是连接时异常，例如很常见的 NoClassDefFoundError 便属于连接时异常，即使会导致连接时异常的代码放在一条无法执行到的分支路径上，类加载时（Java 的连接过程不在编译阶段，而在类加载阶段）也照样会抛出异常。

不过，在 C 语言里，含义相同的代码的代码就会在编译期报错：

```
int main(void) {
    int i[1][0][-1]; // GCC 拒绝编译，报“size of array is negative” 
    return 0;
}
```

由此看来，一门语言的哪一种检查行为要在运行期进行，哪一种检查要在编译期进行并没有必然的因果逻辑关系，关键是在语言规范中人为规定的，再举一个例子来解释“类型检查”，例如下面这一句再普通不过的代码：

```
obj.println(“hello world”); 
```

显然，这行代码需要一个具体的上下文才有讨论的意义，假设它在 Java 语言中，并且变量 obj 的类型为 java.io.PrintStream，那 obj 的值就必须是 PrintStream 的子类（实现了 PrintStream 接口的类）才是合法的。否则，哪怕 obj 属于一个确实有用 println(String) 方法，但与 PrintStream 接口没有继承关系，代码依然不能运行——因为类型检查不合法。

但是相同的代码在 ECMAScript（JavaScript）中情况则不一样，无论 obj 具体是何种类型，只要这种类型的定义中确实包含有 println(String) 方法，那方法调用便可成功。

这种差别产生的原因是 Java 语言在编译期间却已将 println(String) 方法完整的符号引用（本例中为一项 CONSTANT_InterfaceMethodref_info 常量）生成出来，作为方法调用指令的参数存储到 Class 文件中，例如下面这个样子：

```
invokevirtual #4; //Method java/io/PrintStream.println:(Ljava/lang/String;)V 
```

这个符号引用包含了此方法定义在哪个具体类型之中、方法的名字以及参数顺序、参数类型和方法返回值等信息，通过这个符号引用，虚拟机就可以翻译出这个方法的直接引用（譬如方法内存地址或者其他实现形式）。而在 ECMAScript 等动态类型语言中，变量 obj 本身是没有类型的，变量 obj 的值才具有的类型，编译时候最多只能确定方法名称、参数、返回值这些信息，而不会去确定方法所在的具体类型（方法接收者不固定）。“变量无类型而变量值才有类型”这个特点也是动态类型语言的一个重要特征。

了解了动态和静态类型语言的区别后，也许读者的下一个问题就是动态、静态类型语言两者谁更好，或者谁更加先进？这种比较不会有确切答案，它们都有自己的优点，选择哪种语言是需要权衡的事情。静态类型语言在编译期确定类型，最显著的好处是编译器可以提供严谨的类型检查，这样与类型相关的问题能在编码的时候就及时发现，利于稳定性及代码达到更大的规模。而动态类型语言在运行期确定类型，这可以为开发人员提供更大的灵活性，某些在静态类型语言中要花大量臃肿代码来实现的功能，由动态类型语言来实现可能会很清晰简洁，清晰简洁通常也就意味着开发效率的提升。

另外，动态、静态类型语言的界限并非泾渭分明，如 C#是一门静态类型语言，但是在.NET 4.0 中新增加的 dynamic 关键字（它作用就是用来描述一个“无类型”的变量）以及相应的 DLR 支持后，C# 4.0 便拥有了动态语言的特性。

## JDK 7 与动态类型

现在，我们回到本专栏的主题，来看看 Java 语言、虚拟机与动态类型语言之间有什么关系。Java 虚拟机毫无疑问是 Java 语言的运行平台，但它的使命并不仅限于此，早在 1997 年出版的《Java 虚拟机规范》第一版中就规划了这样一个愿景：“在未来，我们会对 Java 虚拟机进行适当的扩展，以便更好的支持其他语言运行于 Java 虚拟机之上”。而目前确实已经有许多动态类型语言运行于 Java 虚拟机之上了，如 Clojure、Groovy、Jython 和 JRuby 等等，能够在同一个虚拟机之上可以实现静态类型语言的严谨与动态类型语言的灵活，这是一件很美妙的事情。

但遗憾的是 Java 虚拟机层面对动态类型语言的支持一直都有所欠缺，主要表现在方法调用方面：JDK 7 以前字节码指令集中，四条方法调用指令（invokevirtual、invokespecial、invokestatic、invokeinterface）的第一个参数都是被调用的方法的符号引用（CONSTANT_Methodref_info 或者 CONSTANT_InterfaceMethodref_info 常量），前面已经提到过，方法的符号引用在编译时产生，而动态类型语言只有在运行期才能确定接收者类型。这样，在 Java 虚拟机上实现的动态类型语言就不得不使用“曲线救国”的方式（如编译时留个占位符类型，运行时动态生成字节码实现具体类型到占位符类型的适配）来实现，这样势必让动态类型语言实现的复杂度增加，也可能带来额外的性能或者内存开销。尽管可以想一些办法（如 Call Site Caching）让这些开销尽量变小，但这种底层问题终归是应当在虚拟机层次上去解决才最合适，因此在 Java 虚拟机层面上提供动态类型的直接支持就成为了 Java 平台的发展趋势之一，这就是 JDK 7（JSR-292）中 invokedynamic 指令以及 java.lang.invoke 包出现的技术背景。

## java.lang.invoke 包

JDK 7 实现了 JSR 292 《Supporting Dynamically Typed Languages on the Java Platform》，新加入的 java.lang.invoke 包[ [3\] ](https://www.infoq.cn/article/jdk-dynamically-typed-language#_ftn3_7498)是就是 JSR 292 的一个重要组成部分，这个包的主要目的是在之前单纯依靠符号引用来确定调用的目标方法这条路之外，提供一种新的动态确定目标方法的机制，称为 Method Handle。这个表达也不好懂？那不妨把 Method Handle 与 C/C++ 中的 Function Pointer，或者 C#里面的 Delegate 类比一下。举个例子，如果我们要实现一个带谓词的排序函数，在 C/C++ 中常用做法是把谓词定义为函数，用函数指针来把谓词传递到排序方法，像这样：

```
void sort(int list[], const int size, int (*compare)(int, int)) 
```

但 Java 语言中做不到这一点，没有办法单独把一个函数作为参数进行传递。普遍的做法是设计一个带有 compare() 方法的 Comparator 接口，以实现了这个接口的对象作为参数，例如 Collections.sort() 就是这样定义的：

```
void sort(List list, Comparator c)
```

不过，在拥有 Method Handle 之后，Java 语言也可以拥有类似于函数指针或者委托的方法别名的工具了。下面代码演示了 MethodHandle 的基本用途，无论 obj 是何种类型（临时定义的 ClassA 抑或是实现 PrintStream 接口的实现类 System.out），都可以正确调用到 println() 方法。

```java
import static java.lang.invoke.MethodHandles.lookup;
import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodType;
/** 
 * JSR 292 MethodHandle 基础用法演示 
 * @author IcyFenix
 */
public class MethodHandleTest {
    static class ClassA {
        public void println(String s) {
            System.out.println(s);
        }
    }
    public static void main(String[] args) throws Throwable {
        Object obj = System.currentTimeMillis() % 2 == 0 ? System.out : new ClassA();
        // 无论 obj 最终是哪个实现类，下面这句都能正确调用到 println 方法。 
        getPrintlnMH(obj).invokeExact("icyfenix");
    }
    private static MethodHandle getPrintlnMH(Object reveiver) throws Throwable {
        // MethodType：代表“方法类型”，包含了方法的返回值（methodType() 的第一个参数）和具体参数（methodType() 第二个及以后的参数）。 
        MethodType mt = MethodType.methodType(void.class, String.class);
        // lookup() 方法来自于 MethodHandles.lookup，这句的作用是在指定类中查找符合给定的方法名称、方法类型，并且符合调用权限的方法句柄。 
        // 因为这里调用的是一个虚方法，按照 Java 语言的规则，方法第一个参数是隐式的，代表该方法的接收者，也即是 this 指向的对象，这个参数以前是放在参数列表中进行传递，现在提供了 bindTo() 方法来完成这件事情。 
        return lookup().findVirtual(reveiver.getClass(), "println", mt).bindTo(reveiver);
    }
}
```

方法 getPrintlnMH() 中实际上是模拟了 invokevirtual 指令的执行过程，只不过它的分派逻辑并非固化在 Class 文件的字节码上的，而是通过一个具体方法来实现。而这个方法本身的返回值（MethodHandle 对象），可以视为对最终调用方法的一个“引用”。以此为基础，有了 MethodHandle 就可以写出类似于这样的函数声明了：

```
void sort(List list, MethodHandle compare) 
```

从上面的例子看来，使用 MethodHandle 并没有多少困难，不过看完它的用法之后，读者大概就会疑问到，相同的事情，用反射不是早就可以实现了吗？

确实，仅站在 Java 语言的角度看，MethodHandle 的使用方法和效果上与 Reflection 都有众多相似之处。不过，它们也有以下这些区别：

- Reflection 和 MethodHandle 机制本质上都是在模拟方法调用，但是 Reflection 是在模拟 Java 代码层次的方法调用，而 MethodHandle 是在模拟字节码层次的方法调用。在 MethodHandles.Lookup 上的三个方法 findStatic()、findVirtual()、findSpecial() 正是为了对应于 invokestatic、invokevirtual & invokeinterface 和 invokespecial 这几条字节码指令的执行权限校验行为，而这些底层细节在使用 Reflection API 时是不需要关心的。
- Reflection 中的 java.lang.reflect.Method 对象远比 MethodHandle 机制中的 java.lang.invoke.MethodHandle 对象所包含的信息来得多。前者是方法在 Java 一端的全面映像，包含了方法的签名、描述符以及方法属性表中各种属性的 Java 端表示方式，还包含有执行权限等的运行期信息。而后者仅仅包含着与执行该方法相关的信息。用开发人员通俗的话来讲，Reflection 是重量级，而 MethodHandle 是轻量级。
- 由于 MethodHandle 是对字节码的方法指令调用的模拟，那理论上虚拟机在这方面做的各种优化（如方法内联），在 MethodHandle 上也应当可以采用类似思路去支持（但目前1.7实现还不完善）。而通过反射去调用方法则不行。

MethodHandle 与 Reflection 除了上面列举的区别外，最关键的一点还在于去掉前面讨论施加的前提“仅站在 Java 语言的角度看”之后：Reflection API 的设计目标是只为 Java 语言服务的，而 MethodHandle 则设计为可服务于所有 Java 虚拟机之上的语言，其中也包括了 Java 语言而已。

## invokedynamic 指令

本文一开始就提到了 JDK 7 为了更好地支持动态类型语言，引入了第五条方法调用的字节码指令 invokedynamic，但前面一直没有再提到它，甚至把之前使用 MethodHandle 的示例代码反编译后也不会看见 invokedynamic 的身影，它到底有什么应用呢？

某种程度上可以说 invokedynamic 指令与 MethodHandle 机制的作用是一样的，都是为了解决原有四条 invoke* 指令方法分派规则固化在虚拟机之中的问题，把如何查找目标方法的决定权从虚拟机转嫁到具体用户代码之中，让用户（包含其他语言的设计者）有更高的自由度。而且，它们两者的思路也是可类比的，可以想象作为了达成同一个目的，一个用上层代码和 API 来实现，另一个是用字节码和 Class 中其他属性和常量来完成。因此，如果前面 MethodHandle 的例子看懂了，理解 invokedynamic 指令并不困难。

每一处含有 invokedynamic 指令的位置都被称作“动态调用点（Dynamic Call Site）”，这条指令的第一个参数不再是代表方法符号引用的 CONSTANT_Methodref_info 常量，而是变为 JDK 7 新加入的 CONSTANT_InvokeDynamic_info 常量，从这个新常量中可以得到 3 项信息：引导方法（Bootstrap Method，此方法存放在新增的 BootstrapMethods 属性中）、方法类型（MethodType）和名称。引导方法是有固定的参数，并且返回值是 java.lang.invoke.CallSite 对象，这个代表真正要执行的目标方法调用。根据 CONSTANT_InvokeDynamic_info 常量中提供的信息，虚拟机可以找到并且执行引导方法，从而获得一个 CallSite 对象，最终调用要执行的目标方法上。我们还是照例拿一个实际例子来解释这个过程吧。如下面代码清单所示：

```java
import static java.lang.invoke.MethodHandles.lookup;
import java.lang.invoke.CallSite;
import java.lang.invoke.ConstantCallSite;
import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodHandles;
import java.lang.invoke.MethodType;
public class InvokeDynamicTest {
    public static void main(String[] args) throws Throwable {
        INDY_BootstrapMethod().invokeExact("icyfenix");
    }
    public static void testMethod(String s) {
        System.out.println("hello String:" + s);
    }
    public static CallSite BootstrapMethod(MethodHandles.Lookup lookup, String name, MethodType mt) throws Throwable {
        return new ConstantCallSite(lookup.findStatic(InvokeDynamicTest.class, name, mt));
    }
    private static MethodType MT_BootstrapMethod() {
        return MethodType.fromMethodDescriptorString("(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;", null);
    }
    private static MethodHandle MH_BootstrapMethod() throws Throwable {
        return lookup().findStatic(InvokeDynamicTest.class, "BootstrapMethod", MT_BootstrapMethod());
    }
    private static MethodHandle INDY_BootstrapMethod() throws Throwable {
        CallSite cs = (CallSite) MH_BootstrapMethod().invokeWithArguments(lookup(), "testMethod", MethodType.fromMethodDescriptorString("(Ljava/lang/String;)V", null));
        return cs.dynamicInvoker();
    }
}
```

这段代码与前面 MethodHandleTest 的作用基本上是一样的，虽然笔者没有加以注释，但是阅读起来应当不困难。真没读懂也不要紧，我没写注释的原因是这段代码并非写给人看的（@_@，我不是在骂人）。由于目前光靠 Java 语言的编译器 javac 没有办法生成带有 invokedynamic 指令的字节码（曾经有一个 java.dyn.InvokeDynamic 的语法糖可以实现，但后来被取消了），所以只能用一些变通的办法，John Rose（Da Vinci Machine Project 的 Leader）编写了一个把程序的字节码转换为使用 invokedynamic 的简单工具 INDY[ [4\] ](https://www.infoq.cn/article/jdk-dynamically-typed-language#_ftn4_7498)来完成这件事情，我们要使用这个工具来产生最终要的字节码，因此这个示例代码中的方法名称不能乱改，更不能把几个方法合并到一起写。

把上面代码编译、转换后重新生成的字节码如下（结果使用 javap 输出，因版面原因，精简了许多无关的内容）：

```
Constant pool: 
    #121 = NameAndType #33:#30 // testMethod:(Ljava/lang/String;)V 
    #123 = InvokeDynamic #0:#121 // #0:testMethod:(Ljava/lang/String;)V 
    public static void main(java.lang.String[]) throws java.lang.Throwable; 
Code: 
    stack=2, locals=1, args_size=1 
0: ldc #23 // String abc 
2: invokedynamic #123, 0 // InvokeDynamic #0:testMethod:(Ljava/lang/String;)V 
7: nop 
8: return 
    public static java.lang.invoke.CallSite BootstrapMethod(java.lang.invoke.MethodHandles$Lookup, java.lang.String, java.lang.invoke.MethodType) throws java.lang.Throwable; 
Code: 
    stack=6, locals=3, args_size=3 
0: new #63 // class java/lang/invoke/ConstantCallSite 
3: dup 
4: aload_0 
5: ldc #1 // class org/fenixsoft/InvokeDynamicTest 
7: aload_1 
8: aload_2 
9: invokevirtual #65 // Method         java/lang/invoke/MethodHandles$Lookup.findStatic:(Ljava/lang/Class;Ljava/lang/String;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/MethodHandle; 
12: invokespecial #71 // Method java/lang/invoke/ConstantCallSite."<init>":(Ljava/lang/invoke/MethodHandle;)V 
15: areturn 
```

从 main() 方法的字节码中可见，原本的方法调用指令已经被替换为 invokedynamic 了，它的参数为第 123 项常量（第二个值为 0 的参数在 HotSpot 中用不到，与 invokeinterface 那个的值为 0 的参数一样是占位的）：

```
2: invokedynamic #123, 0 // InvokeDynamic #0:testMethod:(Ljava/lang/String;)V 
```

从常量池中可见，第 123 项常量显示“#123 = InvokeDynamic #0:#121”说明它是一项 CONSTANT_InvokeDynamic_info 类型常量，常量值中前面“#0”代表引导方法取 BootstrapMethods 属性表的第 0 项（javap 没有列出属性表的具体内容，不过示例中仅有一个引导方法，即 BootstrapMethod()），而后面的“#121”代表引用第 121 项类型为 CONSTANT_NameAndType_info 的常量，从个常量中可以获取方法名称和描述符，既后面输出的“testMethod:(Ljava/lang/String;)V”。

再看 BootstrapMethod()，这个方法 Java 源码中没有，是 INDY 产生的，但是它的字节码很容易读懂，所有逻辑就是调用 MethodHandles$Lookup 的 findStatic() 方法，产生 testMethod() 方法的 MethodHandle，然后用它创建一个 ConstantCallSite 对象。最后，这个对象返回给 invokedynamic 指令实现对 testMethod() 方法的调用，invokedynamic 指令的调用过程到此就宣告完成了。