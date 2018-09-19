# Groovy开发工具
## (对JDK的扩展GDK)
### 1.操作IO

Groovy为操作IO提供了很多有用的方法。您可以使用默认的java代码在Groovy中处理IO操作(Groovy完全兼容),不仅如此,Groovy还提供了更多更 方便的方法处理文件、二进制流、字符流、……

特别的,您应该看一下增加的方法。

* java.io.File类: [ http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/File.html]( http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/File.html)

* java.io.InputStream 类: [http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/InputStream.html](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/InputStream.html)

* java.io.OutputStream 类: [http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/OutputStream.html](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/OutputStream.html)

*  java.io.Reader 类: [http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/Reader.html](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/Reader.html)

* the java.io.Writer 类: [http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/Writer.html](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/Writer.html)

* java.nio.file.Path 类: [http://docs.groovy-lang.org/latest/html/groovy-jdk/java/nio/file/Path.html](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/nio/file/Path.html)

接下来的部分着重关注使用上面提到更加有帮助的方法来编写更加符合语言习惯的结构,但是并不意味着是所有可用方法的完整描述。如果您想要关于那些方法的完整描述的话,请阅读[GDK API](http://groovy-lang.org/gdk.html)。

#### 1.1读取文件

第一个例子然我们看看您 怎样在Groovy中输出一个文本文件的所有内容:

``` groovy
new File(baseDir, 'haiku.txt').eachLine { line ->
    println line
}
```

**eachLine**方法是Groovy自动添加到File类中的(关于如何扩展类之后相关章节会有详细描述)并且拥有许多的方法变体,例如如果您需要知道行数,您能够使用这个变量:

``` groovy
new File(baseDir, 'haiku.txt').eachLine { line, nb ->
    println "Line $nb: $line"
}
```

如果由于某些原因在**ecahLine**的方法体内抛出了异常,这个方法会将资源正确的关闭。Groovy添加的所有使用IO资源的方法都会那么做。

例如某些情况下您选择使用**Reader**但您仍能够从Groovy的自动资源管理中受益。在下一个例子中,字符流将会被关闭即使发生了异常。

``` groovy
def count = 0, MAXSIZE = 3
new File(baseDir,"haiku.txt").withReader { reader ->
    while (reader.readLine()) {
        if (++count > MAXSIZE) {
            throw new RuntimeException('Haiku should only have 3 verses')
        }
    }
}
```

加入您需要手机一个文本文件的所有行到一个列表里(文本文件的一行为一个元素),您能够这么做:
``` groovy
def list = new File(baseDir, 'haiku.txt').collect {it}
```

您甚至可以使用**as**操作符使文件的内容变为一个数组,数组的每个元素为文本文件的一行的内容。

``` groovy
def array = new File(baseDir, 'haiku.txt') as String[]
```

有多少次您不得不使用一个byte数组来获得一个文件的内容但又有多少代码是必须的额?
Groovy使他实际上变的非常简单:

``` groovy
byte[] contents = file.bytes
```

 操作I/O并不仅仅局限于文件。事实上许多操作依赖于输入输出流,这也是为什么Groovy添加了了许多推荐的方法去操作输入输出流,您能够在这个[文档](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/InputStream.html)中看到。
 
 在下面的这个例子中,您能够非常简单的从一个**File**类中获取**InputStream**:
 
 ``` groovy
def is = new File(baseDir,'haiku.txt').newInputStream()
// do something ...
is.close()
 ```
 
 然而在上面的例子中您需要自己去关闭InputStream。在Groovy中其实有个更符合习惯的**withInputStream**方法,他能为您自动关闭InputStream:
 
 ``` groovy
new File(baseDir,'haiku.txt').withInputStream { stream ->
    // do something ...
}
 ``` 
 
 #### 1.2.写文件
 
 当然某些情况下您并不像读文件而想写文件。一个写文件的方法是使用**Writer**:
 
 ``` groovy
new File(baseDir,'haiku.txt').withWriter('utf-8') { writer ->
    writer.writeLine 'Into the ancient pond'
    writer.writeLine 'A frog jumps'
    writer.writeLine 'Water’s sound!'
}
```

但是对于这样一个简单的例子,使用<<操作符就足够了:

``` groovy
new File(baseDir,'haiku.txt') << '''Into the ancient pond
A frog jumps
Water’s sound!'''
```

当然我们并不总是处理文本内容,所以您可以直接在这个例子中写入字节:
``` groovy
file.bytes = [66,22,11]
```

当然您也可以直接处理输出流。下面的例子展示了您怎样创建一个输出流写入文件:

``` groovy
def os = new File(baseDir,'data.bin').newOutputStream()
// do something ...
os.close()
```

然而你会发现这需要你自己去关闭输出流,与上文的输入流一样您可以使用更符合语言规范的并且会帮助您自动处理错误和关闭输出流的**withOutputStream**方法:

``` groovy
new File(baseDir,'data.bin').withOutputStream { stream ->
    // do something ...
}
```

#### 1.3.遍历文件树

在脚本上下文为了寻找一些特定的文件并且对他们进行一些操作进行文件树遍历操作是一个常见的任务。Groovy提供了多种方法实现文件树遍历。例如您能够在一个文件夹下的所有文件中执行:

``` groovy
dir.eachFile { file ->               //(1)
    println file.name
}
dir.eachFileMatch(~/.*\.txt/) { file ->     //(2)
    println file.name
}
```

(1)在文件夹找到的每个文件中执行闭包。
(2)在文件夹符合正则表达式的文件中执行闭包。

你经常不得不处理有很深层次的文件,在这种情况下您能够使用**eachFileRecurse**:

``` groovy
dir.eachFileRecurse { file ->                      //(1)
    println file.name
}

dir.eachFileRecurse(FileType.FILES) { file ->      //(2)
    println file.name
}
```
(1)在文件夹中的所有文件和文件夹递归的执行闭包。
(2)只在文件中(不包括文件夹子)递归的执行闭包。

你能够使用**traverse**方法来运用更多的复合遍历技术,这需要您设置一个特殊的标志指明您想要怎样遍历:

``` groovy
dir.traverse { file ->
    if (file.directory && file.name=='bin') {
        FileVisitResult.TERMINATE                   //(1)
    } else {
        println file.name
        FileVisitResult.CONTINUE                    //(2)
    }

}
```
(1) 如果当前文件是一个文件夹并且他的名字是bin就停止遍历。
(2)否则输出文件名并继续。

#### 1.4.数据与对象

Java中分别使用 **java.io.DataOutputStream**和**java.io.DataInputStream**类来进行序列化和反序列化数据的场景很常见。Groovy是他们处理起来更加易于使用。例如,您能够使用一下代码将数据序列化到文件中或者从文件中反序列化:

``` groovy
boolean b = true
String message = 'Hello from Groovy'
// Serialize data into a file
file.withDataOutputStream { out ->
    out.writeBoolean(b)
    out.writeUTF(message)
}
// ...
// Then read it back
file.withDataInputStream { input ->
    assert input.readBoolean() == b
    assert input.readUTF() == message
}
```

与此相像,如果您想要序列化的数据实现了**Serializable**接口,您能够使用一个对象输出流处理它,就像这儿说明的那样:

``` groovy
Person p = new Person(name:'Bob', age:76)
// Serialize data into a file
file.withObjectOutputStream { out ->
    out.writeObject(p)
}
// ...
// Then read it back
file.withObjectInputStream { input ->
    def p2 = input.readObject()
    assert p2.name == p.name
    assert p2.age == p.age
}
```

#### 1.5.执行外部进程(调用命令行)

之前的部分描述了用Groovy处理文件,字符流,字节流的简便之处。但在系统管理和开发领域经常需要与外部进程进行通信。
Groovy提供了一个简便的方法执行命令行进程。简单的写下命令行命令并将其作为一个字符串然后调用他的**execute()**方法。例如:在一个类unix系统上(或者一个安装了类unix命令行工具的windows系统上),您能够执行这些:

``` groovy
def process = "ls -l".execute()             //(1)
println "Found text ${process.text}"//(2)
```
(1)在外部进程中执行ls命令。
(2)消耗命令行的输出并取到文本中。

**execute()**方法将返回一个包含要被处理的in/out/err流和要被检查的退出值的**java.lang.Process**实例。

例如这是与上面的例子同样的命令但现在我们将一次只处理结果流的一行:
``` groovy
def process = "ls -l".execute()             //(1)
process.in.eachLine { line ->               //(2)
    println line                            //(3)
}
```
(1)在外部进程中执行ls命令。
(2)进程输入流的每一行。
(3)输出该行。

值得注意的是**in**指向一个指向默认命令输出流的输入流。**out**将指向一个您能够发送数据前往的进程的流(该进程的默认输入)。

请记住许多的命令是shell内置的并且需要特殊的处理。所以,如果您想要在windows机器上列出一个文件夹下所有文件的话并写下如下语句:
``` groovy
def process = "dir".execute()
println "${process.text}"
```

你将会得到一个**IOException**异常说:Cannot run program "dir": CreateProcess error=2, The system cannot find the file specified.
这是因为**dir**是Windows shell(cmd.exe)的内建命令并且不能简单的运行为一个可执行文件。相反,你需要写:
``` groovy
def process = "cmd /c dir".execute()
println "${process.text}"
```

另外由于这个功能现阶段底层使用**java.lang.Process**,所以必须要考虑那个类本身的不足之处。特别的,javadoc对于这个类说明如下:

因为一些本地平台只为默认的输入输出流提供有限的缓冲区大小,如果没有及时的写入子进程的输入流或者及时的读出子进程的输出流可能引起子进程的阻塞,甚至是死锁。

因为这个原因Groovy提供了一些附加的更有用的方法来使处理进程流更简单。

下面是如何从进程中获取所有输出（包括错误流输出）：
``` groovy
def p = "rm -f foo.tmp".execute([], tmpDir)
p.consumeProcessOutput()
p.waitFor()
```

也有许多使用**StringBuffer**,**InputStream**, **OutputStream**等等的
**consumeProcessOutput**的变体。为了获取完整的列表,请阅读[GDK API for java.lang.Process](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/lang/Process.html)

此外还有一个将一个进程的输出流送入另一个进程的输入流的pipeTo命令(映射成**|**来允许重载)。
下面是一些使用的例子:
Pipes的使用:
``` groovy
proc1 = 'ls'.execute()
proc2 = 'tr -d o'.execute()
proc3 = 'tr -d e'.execute()
proc4 = 'tr -d i'.execute()
proc1 | proc2 | proc3 | proc4
proc4.waitFor()
if (proc4.exitValue()) {
    println proc4.err.text
} else {
    println proc4.text
}
```
消费错误:
``` groovy
def sout = new StringBuilder()
def serr = new StringBuilder()
proc2 = 'tr -d o'.execute()
proc3 = 'tr -d e'.execute()
proc4 = 'tr -d i'.execute()
proc4.consumeProcessOutput(sout, serr)
proc2 | proc3 | proc4
[proc2, proc3].each { it.consumeProcessErrorStream(serr) }
proc2.withWriter { writer ->
    writer << 'testfile.groovy'
}
proc4.waitForOrKill(1000)
println "Standard output: $sout"
println "Standard error: $serr"
```

### 2.操作集合

Groovy提供了丰富的集合类型的原生支持,包括 lists, maps 或者ranges。
他们中许多基于Java集合类型同时被[Groovy development kit](http://www.groovy-lang.org/gdk.html)中的附加方法所修饰。

#### 2.1.列表
##### 2.1.1.列表字面量
您能够像下面这样创建一个列表。注意**[]**是空列表的表达式。
``` groovy
def list = [5, 6, 7, 8]
assert list.get(2) == 7
assert list[2] == 7
assert list instanceof java.util.List

def emptyList = []
assert emptyList.size() == 0
emptyList.add(5)
assert emptyList.size() == 1
```

每个列表表达式创建了一个[java.util.Lis](https://docs.oracle.com/javase/8/docs/api/java/util/List.html)的实现。
当然列表也能作为构建另一个列表的源列表。
``` groovy
def list1 = ['a', 'b', 'c']
//construct a new list, seeded with the same items as in list1
def list2 = new ArrayList<String>(list1)

assert list2 == list1 // == 检查每个相应的元素是否相等

// clone() 能够被调用
def list3 = list1.clone()
assert list3 == list1
```

一个列表是一个有序的对象的集合:

``` groovy
def list = [5, 6, 7, 8]
assert list.size() == 4
assert list.getClass() == ArrayList     // 被使用的列表的指定的种类

assert list[2] == 7                     //索引从0开始
assert list.getAt(2) == 7               // 下表运算符 []的等价方法
assert list.get(2) == 7                 // 替代方法

list[2] = 9
assert list == [5, 6, 9, 8,]           // 尾逗号也是可以的

list.putAt(2, 10)                       // 值变换[]的等效方法
assert list == [5, 6, 10, 8]
assert list.set(2, 11) == 10            // 返回旧值的替代方法
assert list == [5, 6, 11, 8]

assert ['a', 1, 'a', 'a', 2.5, 2.5f, 2.5d, 'hello', 7g, null, 9 as byte]
//对象可以是不同类型的,允许多种类型

assert [1, 2, 3, 4, 5][-1] == 5             // 使用负数从末尾进行计数
assert [1, 2, 3, 4, 5][-2] == 4
assert [1, 2, 3, 4, 5].getAt(-2) == 4       // getAt()允许传入负值
try {
    [1, 2, 3, 4, 5].get(-2)                 // 但是get()方法不允许负值
    assert false
} catch (e) {
    assert e instanceof IndexOutOfBoundsException
}
```

#####2.1.2.列表作为布尔表达式
列表能够被视作是一个布尔值:
``` groovy
assert ![]             // 一个空列表被视作false

//所有其他的列表,不论内容如何,都视为true
assert [1] && ['a'] && [0] && [0.0] && [false] && [null]
```

##### 2.1.3.列表的迭代
 元素的迭代经常依靠调用**each**和**eachWithIndex**方法来完成,他们在列表的每个元素上执行代码:
 ``` groovy
[1, 2, 3].each {
    println "Item: $it" // `it` 是一个隐含的参数,它指向当前的元素
}
['a', 'b', 'c'].eachWithIndex { it, i -> // `it` 是当前的元素, `i`是当前的索引
    println "$i: $it"
}
 ```
 
 此外为了迭代,经常要通过改变转变一个列表自身的所有元素到另一个元素来创建一个新列表。这个操作,经常被叫做mapping,在Groovy**collect**方法已经实现了这个目的:
 ``` groovy
assert [1, 2, 3].collect { it * 2 } == [2, 4, 6]

//相比collect更加简短的语法
assert [1, 2, 3]*.multiply(2) == [1, 2, 3].collect { it.multiply(2) }

def list = [0]
//给予 `collect` 搜集元素的列表也是可行的。
assert [1, 2, 3].collect(list) { it * 2 } == [0, 2, 4, 6]
assert list == [0, 2, 4, 6]
```

##### 2.1.4.操纵集合
###### 筛选和查询
Groovy开发工具包含了许多用来增加强默认集合的 实用方法,其中一些方法说明如下:

``` groovy
assert [1, 2, 3].find { it > 1 } == 2           // 寻找符合条件的第一个方法
assert [1, 2, 3].findAll { it > 1 } == [2, 3]   // 寻找所有符合条件的元素
assert ['a', 'b', 'c', 'd', 'e'].findIndexOf {      //  寻找第一个符合条件的元素索引
    it in ['c', 'e', 'g']
} == 2

assert ['a', 'b', 'c', 'd', 'c'].indexOf('c') == 2  // 返回索引
assert ['a', 'b', 'c', 'd', 'c'].indexOf('z') == -1 // 索引-1意味着列表中不存在此元素
assert ['a', 'b', 'c', 'd', 'c'].lastIndexOf('c') == 4

assert [1, 2, 3].every { it < 5 }               // 如果所有元素满足条件返回true
assert ![1, 2, 3].every { it < 3 }
assert [1, 2, 3].any { it > 2 }                 // 如果任何元素满足条件返回true
assert ![1, 2, 3].any { it > 3 }

assert [1, 2, 3, 4, 5, 6].sum() == 21                //使用plus()方法对所有元素求和
assert ['a', 'b', 'c', 'd', 'e'].sum {
    it == 'a' ? 1 : it == 'b' ? 2 : it == 'c' ? 3 : it == 'd' ? 4 : it == 'e' ? 5 : 0
    // 在sum中使用自定义值
} == 15
assert ['a', 'b', 'c', 'd', 'e'].sum { ((char) it) - ((char) 'a') } == 10
assert ['a', 'b', 'c', 'd', 'e'].sum() == 'abcde'
assert [['a', 'b'], ['c', 'd']].sum() == ['a', 'b', 'c', 'd']

//能够提供一个初始值
assert [].sum(1000) == 1000
assert [1, 2, 3].sum(1000) == 1006

assert [1, 2, 3].join('-') == '1-2-3'           // 字符串连接
assert [1, 2, 3].inject('counting: ') {
    str, item -> str + item                     //reduce操作
} == 'counting: 123'
assert [1, 2, 3].inject(0) { count, item ->
    count + item
} == 6
```

下面是一个符合语言习惯的寻找最大值和最小值的Groovy代码:
``` groovy
def list = [9, 4, 2, 10, 5]
assert list.max() == 10
assert list.min() == 2

//我们能够比较任何能够比较的单个字符
assert ['x', 'y', 'a', 'z'].min() == 'a'

//我们能够使用闭包指定排序的行为
def list2 = ['abc', 'z', 'xyzuvw', 'Hello', '321']
assert list2.max { it.size() } == 'xyzuvw'
assert list2.min { it.size() } == 'z'
```

除了闭包,你也能使用**Comparator**来定义比较的条件:
``` groovy
Comparator mc = { a, b -> a == b ? 0 : (a < b ? -1 : 1) }

def list = [7, 4, 9, -6, -1, 11, 2, 3, -9, 5, -13]
assert list.max(mc) == 11
assert list.min(mc) == -13

Comparator mc2 = { a, b -> a == b ? 0 : (Math.abs(a) < Math.abs(b)) ? -1 : 1 }


assert list.max(mc2) == -13
assert list.min(mc2) == -1

assert list.max { a, b -> a.equals(b) ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 } == -13
assert list.min { a, b -> a.equals(b) ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 } == -1
```

###### 添加或者移除元素
我们能够使用**[]**来分配一个空列表并使用**<<**往里面添加元素

``` groovy
def list = []
assert list.empty

list << 5
assert list.size() == 1

list << 7 << 'i' << 11
assert list == [5, 7, 'i', 11]

list << ['m', 'o']
assert list == [5, 7, 'i', 11, ['m', 'o']]

//在<<链中的第一个元素就是目标列表
assert ([1, 2] << 3 << [4, 5] << 6) == [1, 2, 3, [4, 5], 6]

//使用leftShift等同于使用<<
assert ([1, 2, 3] << 4) == ([1, 2, 3].leftShift(4))


```

我们能够使用多种方式添加元素到列表中
``` groovy
assert [1, 2] + 3 + [4, 5] + 6 == [1, 2, 3, 4, 5, 6]
//等同于调用 `plus`方法
assert [1, 2].plus(3).plus([4, 5]).plus(6) == [1, 2, 3, 4, 5, 6]

def a = [1, 2, 3]
a += 4      // 创建一个新的列表并将它分配到 `a`
a += [5, 6]
assert a == [1, 2, 3, 4, 5, 6]

assert [1, *[222, 333], 456] == [1, 222, 333, 456]
assert [*[1, 2, 3]] == [1, 2, 3]
assert [1, [2, 3, [4, 5], 6], 7, [8, 9]].flatten() == [1, 2, 3, 4, 5, 6, 7, 8, 9]

def list = [1, 2]
list.add(3)
list.addAll([5, 4])
assert list == [1, 2, 3, 5, 4]

list = [1, 2]
list.add(1, 3) // 在索引1对应的元素前添加3这个元素
assert list == [1, 3, 2]

list.addAll(2, [5, 4]) //在索引2对应的元素前面添加[5,4]
assert list == [1, 3, 5, 4, 2]

list = ['a', 'b', 'z', 'e', 'u', 'v', 'g']
list[8] = 'x' //  [] 操作符 会在需要的时候扩展列表的容量
//如果需要的话null会被插入。
assert list == ['a', 'b', 'z', 'e', 'u', 'v', 'g', null, 'x']
```
然而一个列表上的+运算是不可变的这很重要。比较**<<**，他会创建一个新的列表，但这经常并不是你想要的并且可能会导致性能问题。

[Groovy开发工具](http://www.groovy-lang.org/gdk.html)也包括允许您简单的依靠值移除列表元素的方法：
``` groovy
assert ['a','b','c','b','b'] - 'c' == ['a','b','b','b']
assert ['a','b','c','b','b'] - 'b' == ['a','c']
assert ['a','b','c','b','b'] - ['b','c'] == ['a']

def list = [1,2,3,4,3,2,1]
list -= 3           // 从原来的列表移除`3`这个元素来创建一个新的列表。 
assert list == [1,2,4,2,1]
assert ( list -= [2,4] ) == [1,1]
```

也能够通过在**remove**方法中传入索引来移除一个元素，在这种情况下列表时可变的:

``` groovy
def list = ['a','b','c','d','e','f','b','b','a']
assert list.remove(2) == 'c'        // 移除元素并返回他
assert list == ['a','b','d','e','f','b','b','a']
```

如果您仅仅想要移除一个列表中具有相同值的第一个元素而不是具有相同值所有的元素，您能够调用**remove**方法并传入那个值。

``` groovy
def list= ['a','b','c','b','b']
assert list.remove('c')             // 移除'c'，会返回true因为元素移除了
assert list.remove('b')             // 移除第一个'b'并返回true因为有元素被移除了

assert ! list.remove('z')           // 返回false因为没有元素被移除了
assert list == ['a','b','b']

就像您所看到的那样，有两个可用的**remove**方法。一个传入一个整数同时根据他的索引移除元素，另一个则会移除第一个与传入值想匹配的元素。那么当我们有一个整数类型的列表会怎么样呢？在这种情况下，您可能需要使用**removeAt**来使用索引移除元素，**removeElement**来移除第一个与值相匹配的元素。

``` groovy
def list = [1,2,3,4,5,6,2,2,1]

assert list.remove(2) == 3          //移除索引2的元素并返回他
assert list == [1,2,4,5,6,2,2,1]

assert list.removeElement(2)        // 移除第一个2并返回true
assert list == [1,4,5,6,2,2,1]

assert ! list.removeElement(8)      //返回false因为8并不在这个列表中
assert list == [1,4,5,6,2,2,1]

assert list.removeAt(1) == 4        // 移除索引1的元素并返回他
assert list == [1,5,6,2,2,1]
```

当然**removeAt**和**removeElement**可以在任何类型的列表上使用。

此外可以通过调用**clear**方法来移除所有列表中的元素：

``` groovy
def list= ['a',2,'c',4]
list.clear()
assert list == []
```
##### 设置操作

[Groovy开发工具](http://www.groovy-lang.org/gdk.html)也有使集合运算更简单的方法:
``` groovy
assert 'a' in ['a','b','c']             // 如果元素属于集合返回true
assert ['a','b','c'].contains('a')      // 等同于Java的 `contains` 方法
assert [1,3,4].containsAll([1,4])       // `containsAll` 将检查是否所有的元素都被找到了

assert [1,2,3,3,3,3,4,5].count(3) == 4  //计算有多少个元素具有指定的相同值
assert [1,2,3,3,3,3,4,5].count {
    it%2==0                             // 计算有多少个元素符合断言
} == 2

assert [1,2,4,6,8,10,12].intersect([1,3,6,9,12]) == [1,6,12]

assert [1,2,3].disjoint( [4,6,9] )
assert ![1,2,3].disjoint( [2,4,6] )


```

##### 排序

操作集合经常需要实现排序。Groovy提供了多种多样的方法来排序列表，从使用闭包到比较器（comparators)，就像下面的例子那样：

``` groovy
assert [6, 3, 9, 2, 7, 1, 5].sort() == [1, 2, 3, 5, 6, 7, 9]

def list = ['abc', 'z', 'xyzuvw', 'Hello', '321']
assert list.sort {
    it.size()
} == ['z', 'abc', '321', 'Hello', 'xyzuvw']

def list2 = [7, 4, -6, -1, 11, 2, 3, -9, 5, -13]
assert list2.sort { a, b -> a == b ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 } ==
        [-1, 2, 3, 4, 5, -6, 7, -9, 11, -13]

Comparator mc = { a, b -> a == b ? 0 : Math.abs(a) < Math.abs(b) ? -1 : 1 }

// 仅在JDK 8以上
// list2.sort(mc)
// assert list2 == [-1, 2, 3, 4, 5, -6, 7, -9, 11, -13]

def list3 = [6, -3, 9, 2, -7, 1, 5]

Collections.sort(list3)
assert list3 == [-7, -3, 1, 2, 5, 6, 9]

Collections.sort(list3, mc)
assert list3 == [1, 2, -3, 5, 6, -7, 9]


```

##### 重复元素

[Groovy开发工具](http://www.groovy-lang.org/gdk.html)也利用了运算符重载的优势提供了允许复制列表元素的方法：
``` groovy
assert [1, 2, 3] * 3 == [1, 2, 3, 1, 2, 3, 1, 2, 3]
assert [1, 2, 3].multiply(2) == [1, 2, 3, 1, 2, 3]
assert Collections.nCopies(3, 'b') == ['b', 'b', 'b']

//JDK的nCopies与列表乘法有着不同的语义
assert Collections.nCopies(2, [1, 2]) == [[1, 2], [1, 2]] //不是 [1,2,1,2]


```
