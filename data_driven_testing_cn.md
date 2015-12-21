###Data Driven Testing
通常，执行相同的测试代码多次是有用的，在不同的输入与预期结果。spock数据驱动测试支持使他成为第一类级别特性




###Introduction
Suppose we want to specify the behavior of the Math.max method:

    class MathSpec extends Specification {
        def "maximum of two numbers"() {
            expect:
            // exercise math method for a few different inputs
            Math.max(1, 3) == 3
            Math.max(7, 4) == 7
            Math.max(0, 0) == 0
        }
    }

尽管这种方式是好的在简单的用例像这个，他有一些潜在的缺点：
代码和数据时混合的，但是不容易独立改变
数据不容易自动生成或者获取从外部源
顺序实施相同的代码多次，它或者已经被复制或者被提取到到一个分离的方法
在失败的用例中，不能立刻清理失败引起的输入
实施相同的代码多次不利于从相同的隔离，作为执行分离方法方式。
spock的数据驱动支持试图解决这些问题。在开始之前，让我们重构上面的代码使用数据驱动特性方法。首先，我们介绍三个方法参数替换掉硬编码interge 值。



    class MathSpec extends Specification {
        def "maximum of two numbers"(int a, int b, int c) {
            expect:
            Math.max(a, b) == c

            ...
        }
    }

我们完成这个逻辑测试，但是仍需要数据值被使用。这样做where block 放置方法的最后。在这个简单的用例，where：block持有数据表。

###Data Tables
Data tables are a convenient way to exercise a feature method with a fixed set of data values:

    class Math extends Specification {
        def "maximum of two numbers"(int a, int b, int c) {
            expect:
            Math.max(a, b) == c

            where:
            a | b | c
            1 | 3 | 3
            7 | 4 | 4
            0 | 0 | 0
        }
    }
数据表是一个方便的方式实施一个特性方法使用一确定组的数据。

where:
a | _
1 | _
7 | _
0 | _
表第一行，称为数据头，定义了变量。子行，称作数据行，持有相应的数据。对每一行，特性方法都会被执行一次。我们称作迭代的方法。如果一个迭代方法失败了，意味着其他迭代仍然被执行。所有的
失败都会被报告。
###Isolated Execution of Iterations

迭代式独立的从互相间相同的分割特性方法。每个迭代获取自己实例从spec类。并setup cleanup会被分别调用在每个迭代执行前后。

共享对象在迭代间
顺序的共享一个对象在迭代间，它使用@Shared 或者静态字段

提示
只有@Shared 和静态字段 能被访问在where：block里面。
注意，这些对象也能被共享给其他方法。不是一个好的方式共享对象在相同的方法迭代。如果你思考这个问题，思考放入每个方法在隔离的spec，所有能被处理在相同的文件。这个实现更好的隔离在一些样板代码的成本。

###Syntactic Variations

    class DataDriven extends Specification {
        def "maximum of two numbers"() {
            expect:
            Math.max(a, b) == c

            where:
            a | b || c
            3 | 5 || 5
            7 | 0 || 7
            0 | 0 || 0
        }
    }
上面的代码能被调整在几个方面。首先，从where：block 已经定义所有的数据变量，这个方法参数能被提交。其次，输入与期待输出能被分离使用双线去虚拟设置分离。使用它，代码变成这样。

###Reporting of Failures

让我们设想我们实现max方法有一个错误，其中一个迭代失败了。

两个数字中最大的数字 失败

明显的问题是：迭代失败，数据值时什么。在我们的例子里，它非常难指出是第二个迭代失败。在其他时候可能是更加困难甚至是不可能。在任何用例里，如果spock能大声并清晰哪次失败是非常好的，
超过只是报告失败。这是@Unroll注解的目标

###Method Unrolling

一个方法注释使用@Unroll 将会有迭代过程独立的报告。
为何@Unroll 不默值？
一个原因是一些执行环境期待提前告诉测试方法的数量，并且确定实际数量问题。宁一个原因是该注解能极大改变测试报告数量，可能不是明智的。
注意unrolling没有影响在方法怎么样执行上。他只是在报告里交替。依赖执行环境，输出像这样



    maximum of two numbers[0]   PASSED

    maximum of two numbers[1]   FAILED

    Math.max(a, b) == c
        |    |  |  |  |
        |    7  0  |  7
        42         false

maximum of two numbers[2]   PASSED
This tells us that the second iteration (with index 1) failed. With a bit of effort, we can do even better:

    @Unroll
    def "maximum of #a and #b is #c"() { ... }


这个告诉我们第二个迭代失败，索引为1，随着一点点努力，我们能做更好。
这个方法名使用占位符，表示通过一前置一个＃符号，关联数据变量a b c,在输出，占位符将会被替换使用具体的值。




    maximum of 3 and 5 is 5   PASSED
    maximum of 7 and 0 is 7   FAILED

    Math.max(a, b) == c
        |    |  |  |  |
        |    7  0  |  7
        42         false

    maximum of 0 and 0 is 0   PASSED

现在我们一眼能看出是max 方法失败在输入7与0。看主题上的更多细节在on Unrolled Method Names 这小节

###Data Pipes
Data tables aren’t the only way to supply values to data variables. In fact, a data table is just syntactic sugar for one or more data pipes:

...
    where:
    a << [3, 7, 0]
    b << [5, 0, 0]
    c << [5, 7, 0]


数据表不知是提供数据变量的一种方式。实际上，一个数据表只是一个语法糖为一个或者更多个数据管道。
一个数据管道，通过left-shift (<<)  操作符号，连接一个数据变量到一个数据提供者。数据提供者持有所有的值对变量，每次迭代之一。任何groovy已知对象如何遍历被使用所有数据提供者。这个包含的对象有类型有 Collection, String, Iterable, and 实现了迭代器约定的对象。数据提供者不需要必须有数据。他们能获取数据从外部数据源如文本文件，数据库，电子表格，随即生成数据。数据提供者被查询
只在需要时获取下一个值。


###Multi-Variable Data Pipes

@Shared sql = Sql.newInstance("jdbc:h2:mem:", "org.h2.Driver")

如果一个数据提供者返回多个值在每个迭代。它将同步连接多个数据变量。这个语法时类似groovy 多任务但使用元括号取代左侧括号

    def "maximum of two numbers"() {
        ...
        where:
        [a, b, c] << sql.rows("select a, b, c from maxdata")
    }

不感兴趣的数据值能被忽略使用一个下划线
...
    where:
    [a, b, _, c] << sql.rows("select * from maxdata")


###Data Variable Assignment

一个数据变量呗直接分配一个值
...
    where:
    a = 3
    b = Math.random() * 100
    c = a > b ? a : b

分配被重新评估在每个迭代。如上面所示，右边部分分配可以关联其他数据变量。
    ...
    where:
    row << sql.rows("select * from maxdata")
    // pick apart columns
    a = row.a
    b = row.b
    c = row.c
###Combining Data Tables, Data Pipes, and Variable Assignments
Data tables, data pipes, and variable assignments can be combined as needed:
数据表 数据管道 多个变量分配被组合作为需要
...
    where:
    a | _
    3 | _
    7 | _
    0 | _

    b << [5, 0, 0]

    c = a > b ? a : b
###Number of Iterations

迭代间的数量依赖多少数据时可变的。连续执行相同的方法能产生不同数量的迭代。如果一个数据提供者运行出值比他的同行快，一个异常将产生。多个变量分配不能影响迭代数量。一个where：block只包含分配确切产生一个迭代。

###Closing of Data Providers
After all iterations have completed, the zero-argument close method is called on all data providers that have such a method.
所有迭代完成后，没有参数的关闭方法被调用在所有数据提供者有如此的方法

###More on Unrolled Method Names

一分unrolled方法名根grooy的字符串类似。除了接下来的不同：
表达被标注# 代替 $，没有相等于 ${…​}语法
表达式只支持属性访问和无参调用
给一个Person类只有name与age以及数据类型为person的变量。接下来的校验方法名是：


    def "#person is #person.age years old"() { ... } // property access
    def "#person.name.toUpperCase()"() { ... } // zero-arg method call
Non-string values (like #person above) are converted to Strings according to Groovy semantics.

The following are invalid method names:
    def "#person.name.split(' ')[1]" { ... } // cannot have method arguments
    def "#person.age / 2" { ... } // cannot use operators
If necessary, additional data variables can be introduced to hold more complex expression:

    def "#lastName"() {
        ...
        where:
        person << ...
        lastName = person.name.split(' ')[1]
    }

1. 想法背后运行方法参数更好的被IDE支持。然后 最新版本的IntelliJ IDEA 自动认出数据变量，甚至从数据表的值推断出他们的类型
2. 例如，一个特性方法能使用数据变量在setup: block 但不能在其他任何条件
3. groovy语法不运行$符号在方法名称中

































