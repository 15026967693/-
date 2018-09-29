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

但是对于这样一个简单的例子,使用**<<**操作符就足够了:

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

### 2.2.Map
#### 2.2.1.Map字面量
在groovy,maps(也被看作是关系数组能够使用map字面量语法:[:]来创建:

``` groovy

def map = [name: 'Gromit', likes: 'cheese', id: 1234]
assert map.get('name') == 'Gromit'
assert map.get('id') == 1234
assert map['name'] == 'Gromit'
assert map['id'] == 1234
assert map instanceof java.util.Map

def emptyMap = [:]
assert emptyMap.size() == 0
emptyMap.put("foo", 5)
assert emptyMap.size() == 1
assert emptyMap.get("foo") == 5


```

Map的键值默认是字符串类型的:**[a:1]**等同于**['a':1]**。如果你定义了一个变量**a**并且你想要使用**a**的值作为你的map的键这会造成混乱。在这种情况下，你必须添加括号转义键，就像接下来的例子那样:

``` groovy


def a = 'Bob'
def ages = [a: 43]
assert ages['Bob'] == null // `Bob` 没有被找到
assert ages['a'] == 43     // 因为`a`是字面量

ages = [(a): 43]            // 现在我们使用括号转义`a`
assert ages['Bob'] == 43   // 值被找到了


```

此外对于map字面量，能够获得一个map的拷贝，像下面这样克隆他：
``` groovy


def map = [
        simple : 123,
        complex: [a: 1, b: 2]
]
def map2 = map.clone()
assert map2.get('simple') == map.get('simple')
assert map2.get('complex') == map.get('complex')
map2.get('complex').put('c', 3)
assert map.get('complex').get('c') == 3


```

生成的map是原map的一个浅拷贝，就像之前例子说明的那样。

#### 2.2.2.Map属性标记
Maps也想beans一样所以只要键是String类型的并且是有效的Groovy标识符你就能够使用属性标记去获取/设置Map中的属性：

``` groovy


def map = [name: 'Gromit', likes: 'cheese', id: 1234]
assert map.name == 'Gromit'     //能够被用来代替 map.get('name')
assert map.id == 1234

def emptyMap = [:]
assert emptyMap.size() == 0
emptyMap.foo = 5
assert emptyMap.size() == 1
assert emptyMap.foo == 5


```

注意：在设计上**map.foo**将总会去寻找map上的名字是**foo**的键。这意味着在一个不包含**class**键名的map中**foo.class**将会返回**null**。如果您确实需要知道类名，那么你必须使用**getClass()**方法:
``` groovy


def map = [name: 'Gromit', likes: 'cheese', id: 1234]
assert map.class == null
assert map.get('class') == null
assert map.getClass() == LinkedHashMap // 这个可能才是实际您所需要的

map = [1      : 'a',
       (true) : 'p',
       (false): 'q',
       (null) : 'x',
       'null' : 'z']
assert map.containsKey(1) //1并不是一个标识符所有像这样用
assert map.true == null
assert map.false == null
assert map.get(true) == 'p'
assert map.get(false) == 'q'
assert map.null == 'z'
assert map.get(null) == 'x'


```

#### 2.2.3.迭代map
就像通常在在[Groovy开发工具](http://www.groovy-lang.org/gdk.html)的那样，在map上符合语言习惯的迭代是使用**each**和**eachWithIndex**方法。
值得注意的是使用map语法（[:]）创建的map是有序的，那意味着你在map 的entries（map内部实际存储的类）上迭代的时候，能够他们返回的entries与他们添加的entries具有相同的顺序。

``` groovy


def map = [
        Bob  : 42,
        Alice: 54,
        Max  : 33
]

// `entry` 是一个map entry
map.each { entry ->
    println "Name: $entry.key Age: $entry.value"
}

// `entry`是一个 map entry, `i` 是map的索引
map.eachWithIndex { entry, i ->
    println "$i - Name: $entry.key Age: $entry.value"
}

//作为替代你可以直接使用key和value
map.each { key, value ->
    println "Name: $key Age: $value"
}

// Key, value和作为map索引的i
map.eachWithIndex { key, value, i ->
    println "$i - Name: $key Age: $value"
}


```

#### 2.2.4.操作map
##### 添加或者删除元素

在一个map中添加元素能够通过使用put方法，下标操作符或者使用putAll方法来完成:
``` groovy


def defaults = [1: 'a', 2: 'b', 3: 'c', 4: 'd']
def overrides = [2: 'z', 5: 'x', 13: 'x']

def result = new LinkedHashMap(defaults)
result.put(15, 't')
result[17] = 'u'
result.putAll(overrides)
assert result == [1: 'a', 2: 'z', 3: 'c', 4: 'd', 5: 'x', 13: 'x', 15: 't', 17: 'u']


```

移除一个map上的所有元素能够使用clear方法来完成：
``` groovy


def m = [1:'a', 2:'b']
assert m.get(1) == 'a'
m.clear()
assert m == [:]


```

使用map的[:]语法生成的map的键的唯一性是靠object的**equals**和**hashcode**方法来确定的。这意味着你不应该使用一个hash code会随着时间变化的对象，否则你将不能取键关联的值。
另外要注意的是您不应该使用一个GString作为一个map的键值，因为GString的hash code与相同值的String类型的hash code并不是一样的：
``` groovy


def key = 'some key'
def map = [:]
def gstringKey = "${key.toUpperCase()}"
map.put(gstringKey,'value')
assert map.get('SOME KEY') == null


```

##### 键，值和键值对
我们可以在下面这个例子中检查一下键，值和键值对：
``` groovy


def map = [1:'a', 2:'b', 3:'c']

def entries = map.entrySet()
entries.each { entry ->
  assert entry.key in [1,2,3]
  assert entry.value in ['a','b','c']
}

def keys = map.keySet()
assert keys == [1,2,3] as Set


```

在上面这个例子中返回的可变的值（可能是键、值、或者键值对）是非常不被提倡的因为直接操作的成功与否直接依赖于被操作的map的类型。特别的，Groovy大体上依赖JDK的集合类型所以并不能保证一个集合能够安全的通过**keySet**, **entrySet**或者**values**来操作。

##### 筛选和查询

[Groovy开发工具](http://www.groovy-lang.org/gdk.html)包含了与列表相似的筛选、查询和聚合的方法：
``` groovy


def people = [
    1: [name:'Bob', age: 32, gender: 'M'],
    2: [name:'Johnny', age: 36, gender: 'M'],
    3: [name:'Claire', age: 21, gender: 'F'],
    4: [name:'Amy', age: 54, gender:'F']
]

def bob = people.find { it.value.name == 'Bob' } // find a single entry
def females = people.findAll { it.value.gender == 'F' }

//全部都返回entries, 但是你能够使用collect来重新得到年龄例如：
def ageOfBob = bob.value.age
def agesOfFemales = females.collect {
    it.value.age
}

assert ageOfBob == 32
assert agesOfFemales == [21,54]

// 但是你也能使用一个键/键值对 的值 作为闭包的参数
def agesOfMales = people.findAll { id, person ->
    person.gender == 'M'
}.collect { id, person ->
    person.age
}
assert agesOfMales == [32, 36]

// `every`返回 true 如果 所有键值对与条件匹配
assert people.every { id, person ->
    person.age > 18
}

// `any` r返回 true 若果任何键值对与条件匹配

assert people.any { id, person ->
    person.age == 54
}


```

##### 分组
我们能够使用一些准则将一个列表分组成一个map：

``` groovy


assert ['a', 7, 'b', [2, 3]].groupBy {
    it.class
} == [(String)   : ['a', 'b'],
      (Integer)  : [7],
      (ArrayList): [[2, 3]]
]

assert [
        [name: 'Clark', city: 'London'], [name: 'Sharma', city: 'London'],
        [name: 'Maradona', city: 'LA'], [name: 'Zhang', city: 'HK'],
        [name: 'Ali', city: 'HK'], [name: 'Liu', city: 'HK'],
].groupBy { it.city } == [
        London: [[name: 'Clark', city: 'London'],
                 [name: 'Sharma', city: 'London']],
        LA    : [[name: 'Maradona', city: 'LA']],
        HK    : [[name: 'Zhang', city: 'HK'],
                 [name: 'Ali', city: 'HK'],
                 [name: 'Liu', city: 'HK']],
]


```

2.3.Ranges
Ranges允许你创建顺序值列表。这也能够被当做是List因为Range继承了java.util.List。

使用**..**标记定义的的Ranges是一个封闭区间（同时包含左边界和右边界）
使用**..<**标记定义的Ranges是一个半开半闭区间，他们包括最小值但不包括最大值。
``` groovy

// 一个封闭区间
def range = 5..8
assert range.size() == 4
assert range.get(2) == 7
assert range[2] == 7
assert range instanceof java.util.List
assert range.contains(5)
assert range.contains(8)

//然我们使用一个半开半闭区间
range = 5..<8
assert range.size() == 3
assert range.get(2) == 7
assert range[2] == 7
assert range instanceof java.util.List
assert range.contains(5)
assert !range.contains(8)

//不适用索引获取range的最大值
range = 1..10
assert range.from == 1
assert range.to == 10

注意到int数据的Ranges被实现的很高效，创建了一个包括from值和to值的轻量的Java对象。
Ranges能够被用来给一个实现了java.lang.Comparable 接口的对象作比较同时也有next() 和 previous()方法来返回range的下一个/上一个成员。例如，你能创建一个字符串类型的元素的range:
``` groovy


//一个封闭的range
def range = 'a'..'d'
assert range.size() == 4
assert range.get(2) == 'c'
assert range[2] == 'c'
assert range instanceof java.util.List
assert range.contains('a')
assert range.contains('d')
assert !range.contains('e')


```
你能使用一个传统的**for**循环来迭代一个range:
``` groovy
for (i in 1..10) {
    println "Hello ${i}"
}
```
但是作为替代你能使用一个更加符合Groovy语言习惯的拥有相同效果的风格，使用each方法来迭代each方法:
``` groovy


(1..10).each { i ->
    println "Hello ${i}"
}


```
Range也能够被用在switch语句块中：
``` groovy


switch (years) {
    case 1..10: interestRate = 0.076; break;
    case 11..25: interestRate = 0.052; break;
    default: interestRate = 0.037;
}


```

### 2.4.集合的增强语法
#### 2.4.1.GPath支持
得益于列表和map的对于属性表示法的支持，Groovy提供了语法糖使嵌套结构处理起来更加简单，就像接下来的例子说明的那样：
``` groovy


def listOfMaps = [['a': 11, 'b': 12], ['a': 21, 'b': 22]]
assert listOfMaps.a == [11, 21] //GPath表示法
assert listOfMaps*.a == [11, 21] //展开点表示法

listOfMaps = [['a': 11, 'b': 12], ['a': 21, 'b': 22], null]
assert listOfMaps*.a == [11, 21, null] //适用于空值
assert listOfMaps*.a == listOfMaps.collect { it?.a } //等同于上面的表示法
// But this will only collect non-null values
assert listOfMaps.a == [11,21]


```

#### 2.4.2.展开表示法
展开表示法能够“内联”一个集合到另外一个集合。他是避免调用putAll方法和
便捷的将嵌套结构扁平化的语法糖：
``` groovy


assert [ 'z': 900,
         *: ['a': 100, 'b': 200], 'a': 300] == ['a': 300, 'b': 200, 'z': 900]
//在map定义中使用展开表示法
assert [*: [3: 3, *: [5: 5]], 7: 7] == [3: 3, 5: 5, 7: 7]

def f = { [1: 'u', 2: 'v', 3: 'w'] }
assert [*: f(), 10: 'zz'] == [1: 'u', 10: 'zz', 2: 'v', 3: 'w']
//在函数参数中使用map展开表示法
f = { map -> map.c }
assert f(*: ['a': 10, 'b': 20, 'c': 30], 'e': 50) == 30

f = { m, i, j, k -> [m, i, j, k] }
//使用展开map表示法混合命名参数和未命名参数
assert f('e': 100, *[4, 5], *: ['a': 10, 'b': 20, 'c': 30], 6) ==
        [["e": 100, "b": 20, "c": 30, "a": 10], 4, 5, 6]


```

#### 2.4.3.星点`*.`运算符

星点运算符是一个快捷操作允许你在一个集合的所有元素上的调用方法或属性：

``` groovy


assert [1, 3, 5] == ['a', 'few', 'words']*.size()

class Person {
    String name
    int age
}
def persons = [new Person(name:'Hugo', age:17), new Person(name:'Sandra',age:19)]
assert [17, 19] == persons*.age


```

#### 2.4.4.使用下标运算符进行切片
你可以使用下标表达式来通过索引访问数组。有趣的是字符串在上下文中被看成是特殊类型的集合：
``` groovy


def text = 'nice cheese gromit!'
def x = text[2]

assert x == 'c'
assert x.class == String

def sub = text[5..10]
assert sub == 'cheese'

def list = [10, 11, 12, 13]
def answer = list[2,3]
assert answer == [12,13]


```
注意你能够使用ranges来提取集合的一部分：
``` groovy


list = 100..200
sub = list[1, 3, 20..25, 33]
assert sub == [101, 103, 120, 121, 122, 123, 124, 125, 133]


```

下表运算符能够用来更新已经存在的集合（这个集合的种类必须是可变的）：
``` groovy


list = ['a','x','x','d']
list[1..2] = ['b','c']
assert list == ['a','b','c','d']


```

值得注意的是负的索引也是被允许的，被用来更简单的选取集合的尾部：
``` groovy


text = "nice cheese gromit!"
x = text[-1]
assert x == "!"


```

你能够使用负索引来从列表，数组，字符串......的尾部来进行计数。
``` groovy


def name = text[-7..-2]
assert name == "gromit"


```最后，如果你使用一个开始索引大于结束索引的range那么答案就会颠倒过来。
``` groovy


text = "nice cheese gromit!"
name = text[3..1]
assert name == "eci"


```

#### 2.5.集合的增强方法
除了列表，maps和ranges,Groovy还提供了许多额外的方法来进行筛选，聚合，分组，计数......这些方法可用于集合或者是更简单的迭代。
特别的，我们邀请您阅读[Groovy开发工具API文档] (http://www.groovy-lang.org/gdk.html)并且我们对其进行了一个大致的分类：
* Iterable增加的方法在[这儿](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/lang/Iterable.html)。
* Iterator增加的方法在[这儿](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/Iterator.html)。
* Collection增加的方法在[这儿](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/Collection.html)。
* List增加的方法在[这儿](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/List.html)。
* Map增加的方法在[这儿](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/Map.html)。

### 3.使用遗留的Date/Calendar类型
groovy-dateutil模块支持操作java的传统的Date和Canlendar类的众多扩展。
你能够使用普通的索引下标（此时索引为Calendar类的常量属性）来访问Date或者Calenda的属性，就像下面这样：
``` groovy
import static java.util.Calendar.*    //(1)

def cal = Calendar.instance
cal[YEAR] = 2000                      //(2)
cal[MONTH] = JANUARY                  //(2)
cal[DAY_OF_MONTH] = 1                 //(2)
assert cal[DAY_OF_WEEK] == SATURDAY   //(3)
```
(1)导入常量
(2)设置calendar的年月日
（3访问calendar的星期数）

Groovy支持在Date和Calendar中进行计算和迭代就像下面这个例子那样：
``` groovy


def utc = TimeZone.getTimeZone('UTC')
Date date = Date.parse("yyyy-MM-dd HH:mm", "2010-05-23 09:01", utc)

def prev = date - 1
def next = date + 1

def diffInDays = next - prev
assert diffInDays == 2

int count = 0
prev.upto(next) { count++ }
assert count == 3


```

你能够将字符串解析成date也能将日期输出成格式化的字符串：
``` groovy


def orig = '2000-01-01'
def newYear = Date.parse('yyyy-MM-dd', orig)
assert newYear[DAY_OF_WEEK] == SATURDAY
assert newYear.format('yyyy-MM-dd') == orig
assert newYear.format('dd/MM/yyyy') == '01/01/2000'


```
你也能够基于一个已有的Date或者Calendar来创建一个新的Date或Calendar对象：
``` groovy


def newYear = Date.parse('yyyy-MM-dd', '2000-01-01')
def newYearsEve = newYear.copyWith(
    year: 1999,
    month: DECEMBER,
    dayOfMonth: 31
)
assert newYearsEve[DAY_OF_WEEK] == FRIDAY


```
### 4.操作日期/时间类型
 groovy-datetime模块提供了Java 8中介绍的操作 [Date/Time API](https://www.oracle.com/technetwork/articles/java/jf14-date-time-2125367.html)的众多扩展。这个文档指向"JSR310类型"定义的数据类型。（JSR:JAVA规范提案）

#### 4.1.格式化和解析
使用date/time类型的一个通常的情况就是吧他们转化成字符串（格式化）和从字符串解析（解析）。Groovy提供了这些额外的格式化方法。

| 方法 | 说明 | 例子 |
|:-------:|:----------:|:----------:|
|getDateString()|对于LocalDate和 LocalDateTime ，使用 DateTimeFormatter.ISO_LOCAL_DATE来格式化。|2018-03-10|
|     |对于OffsetDateTime，使用DateTimeFormatter.ISO_OFFSET_DATE来格式化。|2018-03-10+04:00|
|        |对于ZonedDateTime，使用DateTimeFormatter.ISO_LOCAL_DATE 来格式化并且添加ZoneId 的短名称。|2018-03-10EST|
|getDateTimeString()|对于LocalDateTime ，使用 DateTimeFormatter.ISO_LOCAL_DATE_TIME来格式化。|2018-03-10T20:30:45|
|      |对于OffsetDateTime，使用DateTimeFormatter.ISO_OFFSET_DATE_TIME来格式化。|2018-03-10T20:30:45+04:00|
|      |对于ZonedDateTime，使用DateTimeFormatter.ISO_LOCAL_DATE _TIME来格式化并且添加ZoneId 的短名称。|2018-03-10T20:30:45EST|
|getTimeString()|对于LocalDate和 LocalDateTime ，使用 DateTimeFormatter.ISO_LOCAL_TIME来格式化。|20:30:45|
|        |对于OffsetTime和OffsetDateTime，使用 DateTimeFormatter.ISO_OFFSET_TIME来格式化。|20:30:45+04:00|
|                  |对于 ZonedDateTime，使用 DateTimeFormatter.ISO_LOCAL_TIME来格式化并且添加ZoneId 的短名称。|20:30:45EST|
|format(FormatStyle style)|对于LocalTime和OffsetTime, 使用 DateTimeFormatter.ofLocalizedTime(style)来格式化|4:30 AM (with style FormatStyle.SHORT, e.g.)|
|                                 |对于LocalDate, 使用 DateTimeFormatter.ofLocalizedDate(style)来格式化|Saturday, March 10, 2018 (with style FormatStyle.FULL, e.g.)|
|                     |对于LocalDateTime, OffsetDateTime,和ZonedDateTime formats 使用 DateTimeFormatter.ofLocalizedDateTime(style)来格式化|Mar 10, 2019 4:30:45 AM (with style FormatStyle.MEDIUM, e.g.)|
|format(String pattern)|使用 DateTimeFormatter.ofPattern(pattern)来格式化|03/10/2018 (with pattern ’MM/dd/yyyy', e.g.)|

对于解析，Groovy给很多JSR310类型添加了一个静态的parse方法。这个方法接受两个参数：需要格式化的值和所使用的模式。这个模式是用[java.time.format.DateTimeFormatter API](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html)定义的。下面是一个例子：
``` groovy


def date = LocalDate.parse('Jun 3, 04', 'MMM d, yy')
assert date == LocalDate.of(2004, Month.JUNE, 3)

def time = LocalTime.parse('4:45', 'H:mm')
assert time == LocalTime.of(4, 45, 0)

def offsetTime = OffsetTime.parse('09:47:51-1234', 'HH:mm:ssZ')
assert offsetTime == OffsetTime.of(9, 47, 51, 0, ZoneOffset.ofHoursMinutes(-12, -34))

def dateTime = ZonedDateTime.parse('2017/07/11 9:47PM Pacific Standard Time', 'yyyy/MM/dd h:mma zzzz')
assert dateTime == ZonedDateTime.of(
        LocalDate.of(2017, 7, 11),
        LocalTime.of(21, 47, 0),
        ZoneId.of('America/Los_Angeles')
)


```

你应该注意到相比Groovy添加给java.util.Date的静态parse方法这些parse方法有着不同的参数顺序。这与Date/Time API的已经存在的parse方法相一致。

#### 4.2.改变日期/时间
##### 4.2.1.加和减
Temporal类型有plus和minus方法来增加或者减少一个提供的java.time.temporal.TemporalAmount变量。因为Groovy把+和-运算符映射成了
plus和minus的单参数方法，所以现在可以使用更自然的表达式语法来做加减。
``` groovy


def aprilFools = LocalDate.of(2018, Month.APRIL, 1)

def nextAprilFools = aprilFools + Period.ofDays(365) //增加365天
assert nextAprilFools.year == 2019

def idesOfMarch = aprilFools - Period.ofDays(17) //减去17天
assert idesOfMarch.dayOfMonth == 15
assert idesOfMarch.month == Month.MARCH


```

Groovy提供了额外的接受一个整型变量的plus和minus方法来使上面的例子写的更加简洁：
``` groovy


def nextAprilFools = aprilFools + 365 // 增加365天
def idesOfMarch = aprilFools - 17 // 减去17天


```

这些整型的单位依赖于JSR310（JAVA规范提案）类型操作数。上面的例子很明显，ChronoLocalDate类型就像LocalDate一样单位是天。Year 和YearMonth的整型分别是年和月的单位。所有其他的类型的单位是秒，就像LocalTime，例如：
``` groovy


def mars = LocalTime.of(12, 34, 56) // 12:34:56 pm

def thirtySecondsToMars = mars - 30 //退回 30 秒
assert thirtySecondsToMars.second == 26


```

##### 4.2.2.乘法和除法
*运算符能够用整型乘以Period和 Duration实例；/运算符能够将一个Duration除以一个整形：
``` groovy


def period = Period.ofMonths(1) * 2 // 一个一个月的 period乘以 2
assert period.months == 2

def duration = Duration.ofSeconds(10) / 5// 一个十秒的 duration除以5
assert duration.seconds == 2


```

##### 4.2.3.自增和自减
++和--运算符能够用一个单位来增加减少date/time值。由于JSR310类型是不可变的。这个操作将会使用增加后的/减少后的值创建一个新的实例并且重新指派它作为指针的指向。

``` groovy


def year = Year.of(2000)
--year // 减少一年
assert year.value == 1999

def offsetTime = OffsetTime.of(0, 0, 0, 0, ZoneOffset.UTC) // 00:00:00.000 UTC
offsetTime++ // 增加1秒
assert offsetTime.second == 1


```
##### 4.2.4.逆值
Duration和Period类型表示一个负向的或者正向的时间长度。他们能够通过一元运算符-来进行否定。
``` groovy


def duration = Duration.ofSeconds(-15)
def negated = -duration
assert negated.seconds == 15



```
#### 4.3.与date/time值进行交互
##### 4.3.1.属性标记
TemporalAccessor类型（例如：LocalDate, LocalTime, ZonedDateTime......）的getLong(TemporalField) 方法和TemporalAmount类型（即Period和 Duration）的get(TemporalUnit) 方法，能够使用Groovy的属性标记来进行调用。例如：

``` groovy


def date = LocalDate.of(2018, Month.MARCH, 12)
assert date[ChronoField.YEAR] == 2018
assert date[ChronoField.MONTH_OF_YEAR] == Month.MARCH.value
assert date[ChronoField.DAY_OF_MONTH] == 12
assert date[ChronoField.DAY_OF_WEEK] == DayOfWeek.MONDAY.value

def period = Period.ofYears(2).withMonths(4).withDays(6)
assert period[ChronoUnit.YEARS] == 2
assert period[ChronoUnit.MONTHS] == 4
assert period[ChronoUnit.DAYS] == 6


```
##### 4.3.2.Ranges,upto, 和 downto
JSR310类型能够使用range操作符。下面的例子在今天和六天前的LocalDate进行迭代，打印出每个迭代的星期数。由于所有的range是全封闭的范围，这会打印一周的所有7天时间。

``` groovy


def start = LocalDate.now()
def end = start + 6 // 6天之后
(start..end).each { date ->
    println date.dayOfWeek
}


```
upto方法也会完成上面的例子中range做的一样的事，upto方法从最早的开始值（包括它）迭代到最晚的那个值（也包括他自己），在每次迭代中使用增加的值来调用闭包。
``` groovy


def start = LocalDate.now()
def end = start + 6 // 6 天后
start.upto(end) { date ->
    println date.dayOfWeek
}
downto方法在反方向进行迭代。从一个较晚的初始值迭代到一个较早的结束值。

```
upto,downto和ranges的迭代单位与加法和减法相同。LocalDate一次迭代一天, YearMonth一次迭代一个月, Year一次迭代一年, 其他的一次迭代一秒。 所有方法支持一个可选的TemporalUnit 参数来改变迭代的单位。

思考一下接下来的这个例子, 从2018年3月1日 迭代到2018年3月2日 使用的事月的迭代单位。
``` groovy


def start = LocalDate.of(2018, Month.MARCH, 2)
def end = start + 1 // 1天后

int iterationCount = 0
start.upto(end, ChronoUnit.MONTHS) { date ->
    println date
    ++iterationCount
}

assert iterationCount == 1


```
由于开始时间包括自己在内，闭包使用3月1日来进行调用。然后upto方法给这个date增加了一个月，生成了4月1日的date。因为这个时间在指定的结束日期3月2日之后，迭代立即就停止了并且只调用了一次闭包。downto方法也有相同的行为，他也会在值比目标结束值更早的时候停止迭代。
概括地说当upto或者downto方法使用自定义迭代单位进行迭代的时候当前值永远不会超过最终值。
##### 4.3.3.结合date/time值

左移运算符（<<）能够被用来将两个JSR310类型结合成一个结合量。例如：一个LocalDate能够被左移到一个LocatTime来生成一个符合的LocatDateTime实例。

``` groovy


MonthDay monthDay = Month.JUNE << 3 //6月3日
LocalDate date = monthDay << Year.of(2015) //2015年6月3日
LocalDateTime dateTime = date << LocalTime.NOON // 2015年6月3日 下午12点
OffsetDateTime offsetDateTime = dateTime << ZoneOffset.ofHours(-5) //  2015年6月3日 下午12点UTC-5（时区）
```
左移运算符是可以互逆的；与操作对象的顺序无关。


``` groovy


def year = Year.of(2000)
def month = Month.DECEMBER

YearMonth a = year << month
YearMonth b = month << year
assert a == b



```
4.3.4.创建periods 和durations
右移运算符（>>）产生一个表示 period或者duration运算对象的中间值。
例如ChronoLocalDate, YearMonth, 和Year,这个操作生成一个Period实例:
``` groovy
def newYears = LocalDate.of(2018, Month.JANUARY, 1)
def aprilFools = LocalDate.of(2018, Month.APRIL, 1)

def period = newYears >> aprilFools
assert period instanceof Period
assert period.months == 3
``` 
这个运算给时间相关的JSR类型产生一个Duration：
``` groovy


def duration = LocalTime.NOON >> (LocalTime.NOON + 30)
assert duration instanceof Duration
assert duration.seconds == 30


```
如果运算符的左值比右值早那么结果是个正值。否则，结果就会是一个负值。
``` groovy


def decade = Year.of(2010) >> Year.of(2000)
assert decade.years == -10


```

#### 4.4.JSR310类型和遗留类型的互相转化
尽管java.util包中的日期、日历和时区类型存在缺陷，但它们在Java API中是常见的（至少在Java 8之前）。为了适应这种API的使用，Groovy提供了在JSR 310类型和遗留类型之间进行转换的方法。大多数JSR类型都配备了toDate()和toCalendar()方法，用于转换为相对等效的java.util.Date和java.util.Calendar值。ZoneId和ZoneOffset都有一个ToTimeZONE（）方法，用于转换为java. util.TimeZone。

``` groovy


// LocalDate 转 java.util.Date
def valentines = LocalDate.of(2018, Month.FEBRUARY, 14)
assert valentines.toDate().format('MMMM dd, yyyy') == 'February 14, 2018'

// LocalTime 转 java.util.Date
def noon = LocalTime.of(12, 0, 0)
assert noon.toDate().format('HH:mm:ss') == '12:00:00'

// ZoneId 转 java.util.TimeZone
def newYork = ZoneId.of('America/New_York')
assert newYork.toTimeZone() == TimeZone.getTimeZone('America/New_York')

// ZonedDateTime 转 java.util.Calendar
def valAtNoonInNY = ZonedDateTime.of(valentines, noon, newYork)
assert valAtNoonInNY.toCalendar().getTimeZone().toZoneId() == newYork


```

请注意当转换成一个遗留的类型时：
* 纳秒值被截断成一个毫秒值。一个LocalTime,例如：999,999,999的ChronoUnit.NANOS值 所代表的纳秒值被翻译成 999毫秒值。
* 当转换"local"类型 (LocalDate, LocalTime, 和 LocalDateTime),  Date 或者 Calendar 返回的失去会变成系统默认的。

* 当转换一个仅有时间的类型(LocalTime 或者 OffsetTime),  Date 或者 Calendar的year/month/day 被设置成当前日期。

* 当转换一个仅有日期的类型 (LocalDate),  Date 或者 Calendar的时间值会被清除, 例如： 00:00:00.000。

* 当将一个 OffsetDateTime 转换成 Calendar,只有ZoneOffset的 hours 和 minutes 被转换成相适应的 TimeZone.幸运的是, 有着非零的seconds的Zone Offsets就是原来的样子。

### 5.方便的工具

#### 5.1.ConfigSlurper
ConfigSlurper是一个用来读取被定义成Groovy脚本形式的配置文件的工具类。就像Java的*.properties文件一样，ConfigSlurper允许你使用点符号（.）来访问。此外，他还允许闭包作用域的配置值和任意的对象类型。
``` groovy
def config = new ConfigSlurper().parse('''
    app.date = new Date()  //（1）
    app.age  = 42
    app {                  //（2）
        name = "Test${42}"
    }
''')

assert config.app.date instanceof Date
assert config.app.age == 42
assert config.app.name == 'Test42'
```
(1)使用点标记
(2）使用闭包作用于作为点标记的替代。

就像上面例子中所看到的那样，parse方法能够检索groovy.util.ConfigObject实例。ConfigObject是一个指定的返回配置值或者一个新的ConfigObject实例的java.util.Map实现并且永不为null。
``` groovy
def config = new ConfigSlurper().parse('''
    app.date = new Date()
    app.age  = 42
    app.name = "Test${42}"
''')

assert config.test != null //(1)
```
(1)config.test没有被指定但是他在被调用的时候返回了一个ConfigObject 。
在点成为配置变量名一部分的情况下，你能够使用单引号或者双引号来进行转义。

``` groovy


def config = new ConfigSlurper().parse('''
    app."person.age"  = 42
''')

assert config.app."person.age" == 42


```

此外ConfigSlurper 也支持 environments。environments方法能够被用来决定一个拥有很多个配置块的配置文件哪个闭包生效。设想一下我们想要创建一个特别的配置值只在开发环境下生效。当创建ConfigSlurper实例的时候我们能够使用ConfigSlurper(String)构造方法来指定目标环境。
``` groovy


def config = new ConfigSlurper('development').parse('''
  environments {
       development {
           app.port = 8080
       }

       test {
           app.port = 8082
       }

       production {
           app.port = 80
       }
  }
''')

assert config.app.port == 8080


```

ConfigSlurper的环境并不会限制到任何特殊的环境名。他只取决于ConfigSlurper的客户端代码支持的值和相应的解释。
environments方法是默认的的环境配置的闭包的名字但是registerConditionalBlock方法能够用来注册除了environments名称的环境配置的闭包名。
``` groovy


def slurper = new ConfigSlurper()
slurper.registerConditionalBlock('myProject', 'developers')   //(1)

def config = slurper.parse('''
  sendMail = true

  myProject {
       developers {
           sendMail = false
       }
  }
''')

assert !config.sendMail


```
(1)一旦这个新的块被注册了ConfigSlurper就能解析他

出于Java集成的目的toProperties方法能够被用来将一个ConfigObject 转换成一个能够被存储成*.properties文本文件的java.util.Properties对象。请注意，在将配置值添加到新创建的Properties实例的过程中，配置值会被转换为String实例。 

``` groovy


def config = new ConfigSlurper().parse('''
    app.date = new Date()
    app.age  = 42
    app {
        name = "Test${42}"
    }
''')

def properties = config.toProperties()

assert properties."app.date" instanceof String
assert properties."app.age" == '42'
assert properties."app.name" == 'Test42'


```
##### 5.2.Expando
Expando类能够被用来创建一个动态的可扩展对象。尽管他的名字不再之后使用ExpandoMetaClass。每个Expando对象表现为一个可以再运行时扩展属性（或方法的）独立的，可动态制作的实例。
``` groovy


def expando = new Expando()
expando.name = 'John'

assert expando.name == 'John'


```
当动态属性注册了一个闭包代码块时会发生特殊情况。一旦被注册，就可以调用它，因为它将使用方法调用来完成。

``` groovy


def expando = new Expando()
expando.toString = { -> 'John' }
expando.say = { String s -> "John says: ${s}" }

assert expando as String == 'John'
assert expando.say('Hi') == 'John says: Hi'


```

##### 5.3.可观察list, map 和set
Groovy有可观察的list, map 和set。 当添加、移除或更改元素时，这些集合中都能触发java.beans.PropertyChangeEvent事件。 请注意，PropertyChangeEvent不仅发出了事件确实已经发生的信号，更多的，他还保存了已经确实发生改变的属性的属性名和新值/旧值的信息。根据已发生的改变的类型，可观察的集合可能会触发PropertyChangeEvent类型的子类类型的事件。例如，将一个元素添加到一个可观察的列表中会触发一个ObservableList.ElementAddedEvent 事件。
``` groovy


def event                                       (1)
def listener = {
    if (it instanceof ObservableList.ElementEvent)  { (2) 
        event = it
    }
} as PropertyChangeListener


def observable = [1, 2, 3] as ObservableList    (3)
observable.addPropertyChangeListener(listener)  (4)

observable.add 42                               (5)

assert event instanceof ObservableList.ElementAddedEvent

def elementAddedEvent = event as ObservableList.ElementAddedEvent
assert elementAddedEvent.changeType == ObservableList.ChangeType.ADDED
assert elementAddedEvent.index == 3
assert elementAddedEvent.oldValue == null
assert elementAddedEvent.newValue == 42


```

（1）声明一个捕获触发事件的 PropertyChangeEventListener 
（2）	ObservableList.ElementEvent 及其派生类型和这个监听器是有关的
（3）注册监听器
（4）由给定的list创建一个ObservableList
（5）触发一个ObservableList.ElementAddedEvent事件
请注意，事实上添加一个元素会触发两个事件。第一个类型是ObservableList.ElementAddedEvent，第二个类型是普通的PropertyChangeEvent，它通知监听器size属性的改变。
ObservableList.ElementClearedEvent 事件类型是另一个有趣的事件类型。每当删除多个元素时，例如调用clear（）时，它就保存从list中移除的元素。
``` groovy


def event
def listener = {
    if (it instanceof ObservableList.ElementEvent)  {
        event = it
    }
} as PropertyChangeListener


def observable = [1, 2, 3] as ObservableList
observable.addPropertyChangeListener(listener)

observable.clear()

assert event instanceof ObservableList.ElementClearedEvent

def elementClearedEvent = event as ObservableList.ElementClearedEvent
assert elementClearedEvent.values == [1, 2, 3]
assert observable.size() == 0


``` 

为了获得所有支持的事件类型的概览，我们希望您查看JavaDoc文档或可观察集合的源代码。
ObservableMap 和 ObservableSet具有与我们在本节中观察到的观察者相同的概念。