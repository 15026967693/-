# 与Java的不同点
Groovy尝试对Java开发者尽可能的自然。我们在设计Groovy的时候遵循最小惊奇的原则,尤其是原先是Java后台但现在学习Groovy的开发者。

在这里我门列出了Java和Groovy的最主要的区别。

### 1.默认导入

以下的这些这些包和类默认被导入了,所以您不需要特地使用一个明确的***import***语句去使用他们。

* java.io.*

* java.lang.*

* java.math.BigDecimal

* java.math.BigInteger

* java.net.*

* java.util.*

* groovy.lang.*

* groovy.util.*

### 2.多重方法

在groovy中,被调用的方法在运行的时候确定。这被称作运行时分发或者多重方法。这就意味着那个方法将在运行的时候基于他的参数类型被决定
在Java中,方法的选择是基于Groovy的对立面的:方法在编译的时候基于声明的类型被决定。

这下面的代码,能够同时被编译成Java和Groovy的,但是却有着不同的行为:

``` java
int method(String arg) {
    return 1;
}
int method(Object arg) {
    return 2;
}
Object o = "Object";
int result = method(o);
```
在Java中您可能得到:

```java
assertEquals(2, result);
```

但在groovy:

```java
assertEquals(1, result);
```

这是因为Java会使用静态的类型信息,**o**被声明为一个**Object**类型,但是Groovy将会在运行时方法被调用的时候进行选择。由于他被声明为了**String**类型,所以参数类型为**String**的方法被调用了。

### 3.数组的初始化

在Groovy形如{ ... }的语句被视作闭包。这意味着您不能使用这个语法创建数组(如下所示是错误的):

``` groovy
int[] array = { 1, 2, 3}
```

实际上您不得不使用以下的语法:

``` groovy
int[] array = [1,2,3]
```

### 4.包可见的作用域

在Groovy里面,省略成员变量修饰符并不会像Java一样变成一个包私有的成员变量:

``` java
class Person {
    String name
}
```

相对的他被用来创建一个属性,这意味着他被声明为一个私有的成员变量,一个对应的Getter方法和一个对应的Setter方法。

您也可以通过用@PackageScope修饰一个包私有的成员变量使他变成包公有的成员变量。

``` java
class Person {
    @PackageScope String name
}
```

###5.ARM(自动资源回收管理)语句块

从Java 7开始的自动资源回收管理语句块(Automatic Resource Management)在Groovy中并不被支持。相反Groovy提供了很多依赖于闭包的拥有相同效果并且符合语言习惯的方法。例如:

``` java
Path file = Paths.get("/path/to/file");
Charset charset = Charset.forName("UTF-8");
try (BufferedReader reader = Files.newBufferedReader(file, charset)) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }

} catch (IOException e) {
    e.printStackTrace();
}
```

能够被写成下面这样:

``` java
new File('/path/to/file').eachLine('UTF-8') {
   println it
}
```

或者如果您想要想要一个和Java相像的版本:

``` java
new File('/path/to/file').withReader('UTF-8') { reader ->
   reader.eachLine {
       println it
   }
}
```

6.内部类

匿名内部类和嵌套类的实现遵循Java的指导,但是您不应该在使用Java语言规范的同时坚决否认不同的编程规范。这个实现的完成看起来很想我们为**groovy.lang.Closure**做的:既有一些好处但也有一些不同。例如允许访问私有成员变量和方法可能导致一些问题,但是另一方面来说局部变量并不需要被声明成为final的。

#### 6.1.静态内部类

这里有一个静态内部类的例子:

``` java
class A {
    static class B {}
}
new A.B()
```

静态内部类的使用是被支持的最好的。如果你实在需要一个内部类,你应该使他成为静态的。

#### 6.2.匿名内部类

``` java
import java.util.concurrent.CountDownLatch
import java.util.concurrent.TimeUnit

CountDownLatch called = new CountDownLatch(1)

Timer timer = new Timer()
timer.schedule(new TimerTask() {
    void run() {
        called.countDown()
    }
}, 0)

assert called.await(10, TimeUnit.SECONDS)
```

#### 6.3.创建非静态内部类的实例

在Java中您能够这样做

``` java
public class Y {
    public class X {}
    public X foo() {
        return new X();
    }
    public static X createX(Y y) {
        return y.new X();
    }
}
```

Groovy不支持**y.new X()**的语法。相反,你不得不写**new X(y)**,像下面的代码这样:

``` groovy
public class Y {
    public class X {}
    public X foo() {
        return new X()
    }
    public static X createX(Y y) {
        return new X(y)
    }
}
```

请特别小心,Groovy支持不提供参数变量调用只有一个参数的方法。在这种情况下,那个参数的值会变成null。同样的规则在调用构造方法的时候也适用。举例来说当你写new X()而不是new X(this)的时候是很危险的,由于我们还没有找到一个好的方法去防止这个问题 。

### 7.Lambda表达式

Java 8 支持Lambda表达式和方法引用:

``` java
Runnable run = () -> System.out.println("Run");
list.forEach(System.out::println);
```

Java 8的Lambda表达式可以或多或少的被看作是匿名内部类,Groovy并不支持这种语法但是相对的Groovy有闭包:

``` groovy
Runnable run = { println 'run' }
list.each { println it } // or list.each(this.&println)
```

### 8.GStrings

由于双引号字符串字面量被解释为**GString**值,当Groovy和Java编译器编译一个包含"$"字符的字符串字面量的类Groovy可能由于编译错误而失败,或者产生一些微妙的有些不同的代码。

通常情况下,如果一个API声明参数类型为GString或String时Groovy将会自动在这两中类型间进行转换,当心接受一个Object最为参数的Java APIS并且检查实际的类型。

### 9.字符串和字符字面量

单引号字面量在Groovy被用作**String**类型,而双引号字面量将会依据在字面量中是否有字符串插值被视作**String**类型或者**GString**类型。

``` groovy
assert 'c'.getClass()==String
assert "c".getClass()==String
assert "c${1}".getClass() in GString
```

Groovy仅仅在分配一个**char**变量的时候才会自动的将一个单字符字符串(如'c'在groovy中他的类型并不是char而是String)装换成字符类型变量(char类型)。在调用有**char**类型的参数的方法的时候,我们需要显式的将变量转换成char类型或者确信这个变量在之前已经被转换成了char类型。

``` groovy
char a='a'
assert Character.digit(a, 16)==10 : 'But Groovy does boxing'
assert Character.digit((char) 'a', 16)==10

try {
  assert Character.digit('a', 16)==10
  assert false: 'Need explicit cast'
} catch(MissingMethodException e) {
}
```

Groovy支持两种风格的类型转换,但是当转换一个多字符字符串的时候他们有一些微妙的不同。Groovy风格的转换更加的宽松,而C风格的转换(Java强制装换的那种风格)将会由于异常而失败。

``` groovy
// for single char strings, both are the same
assert ((char) "c").class==Character
assert ("c" as char).class==Character

// for multi char strings they are not
try {
  ((char) 'cx') == 'c'
  assert false: 'will fail - not castable'
} catch(GroovyCastException e) {
}
assert ('cx' as char) == 'c'
assert 'cx'.asType(char) == 'c'
```

### 10.原始类型和包装类(int和Integer等)

由于在Groovy中一切都是对象,他会[自动包装](http://docs.groovy-lang.org/latest/html/documentation/core-object-orientation.html#_primitive_types)原生类型。由于这个原因,他并不会遵循Java扩宽类型优先与包装的原则。这里有一个使用**int**的例子:
``` groovy
int i
m(i)

void m(long l) {                //(1) 
  println "in m(long)"
}

void m(Integer i) {         //(2)
  println "in m(Integer)"
}
```
(1)这是Java会调用的方法,因为扩宽类型优先于类型自动拆箱。
(2)只是Groovy实际上会调用的方法 ,因为所有原始类型都被他们的包装类型替换了。

###  11.==的行为
在Java中==意味着原始类型的值相等(int,double等未包装的类型)或者对象的指针相等。在Groovy,如果他们是**Comparable**接口的实现则**==**被翻译成**a.compareTo(b)==0**,否则就是**a.equals(b)**。为了检查指针是否相等,请使用关键字is,例如:**a.is(b)**。

### 12.转换
Java使用自动扩宽和狭窄的[转换](https://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html)。

                                   Table 1.Java 转换
|               |目标类型|                 |          |         |      |                 |          |              |
|:--------:|:-----------:|:-----:|:------:|:----:|:----:|:-----:|:------:|:--------:|
|原类型|boolean|byte|short|char|int|long|float|double|
|boolean|-|N|N|N|N|N|N|N|
|byte|N|-|Y|C|Y|Y|Y|Y
|short|N|C|-|C|Y|Y|Y|Y
|char|N|C|C|-|Y|Y|Y|Y|
|int|N|C|C|C|-|Y|T|Y|
|long|N|C|C|C|C|-|T|T|
|float|N|C|C|C|C|C|-|Y|
|double|N|C|C|C|C|C|C|-|

行标签为原类型,列标签为目标类型。'Y'表明Java能够自动转换的类型。'C'表明当有显示转换的时候Java能够转换的类型。'T'表示Java能够转换的类型但是数据会被截断。'N'表示Java不能够转换的类型。

Groovy在这基础上扩展了很多。

                                  Table2.Groovy 转换
 |        |目标类型|    |     |     |    |    |     |    |     |    |    |    |     |    |     |    |    |    |
 |:---:|:---------:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
 |原类型|boolean|Boolean|byte|Byte|short|Short|char|Character|int|Integer|long|Long|BigInteger|float|Float|double|Double|BigDecimal|
 |boolean|-|B|N|N|N|N|N|N|N|N|N|N|N|N|N|N|N|N|
 |Boolean|B|-|N|N|N|N|N|N|N|N|N|N|N|N|N|N|N|N|
 |byte|T|T|-|B|Y|Y|Y|D|Y|Y|Y|Y|Y|Y|Y|Y|Y|Y|
 |Byte|T|T|B|-|Y|Y|Y|D|Y|Y|Y|Y|Y|Y|Y|Y|Y|Y|
|short|T|T|D|D|-|B|Y|D|Y|Y|Y|Y|Y|Y|Y|Y|Y|Y|
 |Short|T|T|D|T|B|-|Y|D|Y|Y|Y|Y|Y|Y|Y|Y|Y|Y|
 |char|T|T|Y|D|Y|D|-|D|Y|D|Y|D|D|Y|D|Y|D|D|
 |Character|T|T|D|D|D|D|D|-|D|D|D|D|D|D|D|D|D|D|
 |int|T|T|D|D|D|D|Y|D|-|B|Y|Y|Y|Y|Y|Y|Y|Y|
 |Integer|T|T|D|D|D|D|Y|D|B|-|Y|Y|Y|Y|Y|Y|Y|Y|
 |long|T|T|D|D|D|D|Y|D|D|D|-|B|Y|T|T|T|T|Y|
 |Long|T|T|D|D|D|T|Y|D|D|T|B|-|Y|T|T|T|T|Y|
 |BigInteger|T|T|D|D|D|D|D|D|D|D|D|D|-|D|D|D|D|T|
 |float|T|T|D|D|D|D|T|D|D|D|D|D|D|-|B|Y|Y|Y|
 |Float|T|T|D|T|D|T|T|D|D|T|D|T|D|B|-|Y|Y|Y|
 |double|T|T|D|D|D|D|T|D|D|D|D|D|D|D|D|-|B|Y|
 |Double|T|T|D|T|D|T|T|D|D|T|D|T|D|D|T|B|-|Y|
 |BigDecimal|T|T|D|D|D|D|D|D|D|D|D|D|D|T|D|T|D|-|
  
行标签为原类型,列标签为目标类型。'Y'表明Groovy能够自动转换的类型,'D'表示Groovy能够通过显示转换或者动态编译转换的类型,'T'表示Groovy能够转换的类型但是数据会被截断。'B'表示一个装箱/拆箱操作。'N'表示Groovy不能够转换的类型。

当转换成**boolean/Boolean**时截断使用[Groovy Truth](http://docs.groovy-lang.org/latest/html/documentation/core-semantics.html#Groovy-Truth)。从数字到字符的转换将**Number.intvalue()**转换成**char**。当从**Float**或者**Double**转换时Groovy使用**BigInteger**和**BigDecimal**进行构造,否则使用**toSting()**构造。
**java.lang.Number**定义了其他的转换的行为。

### 13.其他关键字
Groovy拥有比java更多的关键在。请不要把他们作为变量名。如下:

* **as**

* **def**

* **in**

* **trait**
