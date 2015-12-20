###Data Driven Testing
Oftentimes, it is useful to exercise the same test code multiple times, with varying inputs and expected results. Spock’s data driven testing support makes this a first class feature.

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

Although this approach is fine in simple cases like this one, it has some potential drawbacks:

Code and data are mixed and cannot easily be changed independently

Data cannot easily be auto-generated or fetched from external sources

In order to exercise the same code multiple times, it either has to be duplicated or extracted into a separate method

In case of a failure, it may not be immediately clear which inputs caused the failure

Exercising the same code multiple times does not benefit from the same isolation as executing separate methods does

Spock’s data-driven testing support tries to address these concerns. To get started, let’s refactor above code into a data-driven feature method. First, we introduce three method parameters (called data variables) that replace the hard-coded integer values:

尽管这种方式是好的在简单的用例像这个，他有一些潜在的缺点：
代码和数据时混合的，但是不容易独立改变
数据不容易自动生成或者获取从外部源
顺序实施相同的代码多次，它或者已经被复制或者被提取到到一个分离的方法

失败的用例，它不能被立刻清理，输入引起的失败

实施相同的代码多次不利于从相同的隔离作为执行分离方法。

spock的数据驱动支持试图解决这些问题。在开始之前，让我们重构上面的代码使用数据驱动特性方法。首先，我们介绍三个方法参数替换掉硬编码interge 值。



    class MathSpec extends Specification {
        def "maximum of two numbers"(int a, int b, int c) {
            expect:
            Math.max(a, b) == c

            ...
        }
    }
We have finished the test logic, but still need to supply the data values to be used. This is done in a where: block, which always comes at the end of the method. In the simplest (and most common) case, the where: block holds a data table.

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
The first line of the table, called the table header, declares the data variables. The subsequent lines, called table rows, hold the corresponding values. For each row, the feature method will get executed once; we call this an iteration of the method. If an iteration fails, the remaining iterations will nevertheless be executed. All failures will be reported.

Data tables must have at least two columns. A single-column table can be written as:

where:
a | _
1 | _
7 | _
0 | _
表第一行，称为数据头，定义了变量。子行，称作数据行，持有相应的数据。对每一行，特性方法都会被执行一次。我们称作迭代的方法。如果一个迭代方法失败了，意味着其他迭代仍然被执行。所有的
失败都会被报告。
###Isolated Execution of Iterations
Iterations are isolated from each other in the same way as separate feature methods. Each iteration gets its own instance of the specification class, and the setup and cleanup methods will be called before and after each iteration, respectively.

Sharing of Objects between Iterations
In order to share an object between iterations, it has to be kept in a @Shared or static field.

NOTE
Only @Shared and static variables can be accessed from within a where: block.
Note that such objects will also be shared with other methods. There is currently no good way to share an object just between iterations of the same method. If you consider this a problem, consider putting each method into a separate spec, all of which can be kept in the same file. This achieves better isolation at the cost of some boilerplate code.


迭代式独立的从互相间相同的分割特性方法。每个迭代获取自己实例从spec类。并setup cleanup会被分别调用在每个迭代执行前后。

共享对象在迭代间
顺序的共享一个对象在迭代间，它使用@Shared 或者静态字段

提示
只有@Shared 和静态字段 能被访问在where：block里面。
注意，这些对象也能被共享给其他方法。不是一个好的方式共享对象在相同的方法迭代。如果你思考这个问题，思考放入每个方法在隔离的spec，所有能被处理在相同的文件。这个实现更好的隔离在一些样板代码的成本。

###Syntactic Variations
The previous code can be tweaked in a few ways. First, since the where: block already declares all data variables, the method parameters can be omitted.[1] Second, inputs and expected outputs can be separated with a double pipe symbol (||) to visually set them apart. With this, the code becomes:

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
Let’s assume that our implementation of the max method has a flaw, and one of the iterations fails:

maximum of two numbers   FAILED

Condition not satisfied:

Math.max(a, b) == c
    |    |  |  |  |
    |    7  0  |  7
    42         false
The obvious question is: Which iteration failed, and what are its data values? In our example, it isn’t hard to figure out that it’s the second iteration that failed. At other times this can be more difficult or even impossible. [2] In any case, it would be nice if Spock made it loud and clear which iteration failed, rather than just reporting the failure. This is the purpose of the @Unroll annotation.        

让我们设想我们实现最大方法有一个错误，其中一个迭代失败了。

两个数字中最大的数字 失败

明显的问题是：迭代失败，数据值时什么。在我们的例子里，它非常难指出是第二个迭代失败。在其他时候可能是更加困难甚至是不可能。在任何用例里，如果spock能大声并清晰哪次失败是非常好的。
超过只是报告失败。这是@Unroll注解的目标

###Method Unrolling
A method annotated with @Unroll will have its iterations reported independently:

    @Unroll
    def "maximum of two numbers"() { ... }
Why isn’t @Unroll the default?
One reason why @Unroll isn’t the default is that some execution environments (in particular IDEs) expect to be told the number of test methods in advance, and have certain problems if the actual number varies. Another reason is that @Unroll can drastically change the number of reported tests, which may not always be desirable.
Note that unrolling has no effect on how the method gets executed; it is only an alternation in reporting. Depending on the execution environment, the output will look something like:

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
This method name uses placeholders, denoted by a leading hash sign (#), to refer to data variables a, b, and c. In the output, the placeholders will be replaced with concrete values:

这个告诉我们第二个迭代失败，索引为1，随着一点点努力，我们能做更好。
这个方法名使用占位符，表示通过一前置一个＃符号，关联数据变量a b c,在输出，占位符将会被替换使用具体的值。




    maximum of 3 and 5 is 5   PASSED
    maximum of 7 and 0 is 7   FAILED

    Math.max(a, b) == c
        |    |  |  |  |
        |    7  0  |  7
        42         false

    maximum of 0 and 0 is 0   PASSED
Now we can tell at a glance that the max method failed for inputs 7 and 0. See More on Unrolled Method Names for further details on this topic.

The @Unroll annotation can also be placed on a spec. This has the same effect as placing it on each data-driven feature method of the spec.

现在我们一眼能看出是max 方法失败在输入7与0。看主题上的更多细节在on Unrolled Method Names 这小节

Data Pipes
Data tables aren’t the only way to supply values to data variables. In fact, a data table is just syntactic sugar for one or more data pipes:

...
    where:
    a << [3, 7, 0]
    b << [5, 0, 0]
    c << [5, 7, 0]
A data pipe, indicated by the left-shift (<<) operator, connects a data variable to a data provider. The data provider holds all values for the variable, one per iteration. Any object that Groovy knows how to iterate over can be used as a data provider. This includes objects of type Collection, String, Iterable, and objects implementing the Iterable contract. Data providers don’t necessarily have to be the data (as in the case of a Collection); they can fetch data from external sources like text files, databases and spreadsheets, or generate data randomly. Data providers are queried for their next value only when needed (before the next iteration).

Multi-Variable Data Pipes
If a data provider returns multiple values per iteration (as an object that Groovy knows how to iterate over), it can be connected to multiple data variables simultaneously. The syntax is somewhat similar to Groovy multi-assignment but uses brackets instead of parentheses on the left-hand side:    
@Shared sql = Sql.newInstance("jdbc:h2:mem:", "org.h2.Driver")

数据表驱动，

def "maximum of two numbers"() {
    ...
    where:
    [a, b, c] << sql.rows("select a, b, c from maxdata")
}
Data values that aren’t of interest can be ignored with an underscore (_):

...
where:
[a, b, _, c] << sql.rows("select * from maxdata")

###Data Variable Assignment
A data variable can be directly assigned a value:
一个数据变量呗直接分配一个值
...
where:
a = 3
b = Math.random() * 100
c = a > b ? a : b
Assignments are re-evaluated for every iteration. As already shown above, the right-hand side of an assignment may refer to other data variables:

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
The number of iterations depends on how much data is available. Successive executions of the same method can yield different numbers of iterations. If a data provider runs out of values sooner than its peers, an exception will occur. Variable assignments don’t affect the number of iterations. A where: block that only contains assignments yields exactly one iteration.

迭代间的数量依赖多少数据时可变的。连续执行相同的方法能产生不同数量的迭代。如果一个数据提供者运行出值比他的同行快，一个异常将产生。多个变量分配不能影响迭代数量。一个where：block只包含分配确切产生一个迭代。

###Closing of Data Providers
After all iterations have completed, the zero-argument close method is called on all data providers that have such a method.

More on Unrolled Method Names
An unrolled method name is similar to a Groovy GString, except for the following differences:

Expressions are denoted with # instead of $ [3], and there is no equivalent for the ${…​} syntax.

Expressions only support property access and zero-arg method calls.

Given a class Person with properties name and age, and a data variable person of type Person, the following are valid method names:

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
1. The idea behind allowing method parameters is to enable better IDE support. However, recent versions of IntelliJ IDEA recognize data variables automatically, and even infer their types from the values contained in the data table.
2. For example, a feature method could use data variables in its setup: block, but not in any conditions.
3. Groovy syntax does not allow dollar signs in method names.

所有迭代完成，






































