###Spock Primer

这一章假设你有关于groovy与单元测试基本知识。如果你是java开发者，但没听说过groovy，别担心，groovy对java开发者来说非常友好。事实上，groovy相关抓哟设计目标是与java共存脚本语言。因此只需要继续并查看groovy doc

这一章目标是教会你足够写真实spec的spock，并且激起你的食欲

学习更多关于groovy，点http://groovy-lang.org/.

学些更多关于单元测试，点 http://en.wikipedia.org/wiki/Unit_testing.
###术语
让我们开始一些定义：spock 让你写描述期望特性展示spec作为系统关注点，系统感兴趣关注点可以是一个简单的类和整个系统中的任何事，并且被称为spec下的系统（sus）。一个特性描述开始于spec sus的临时快照并且作为一个合作者。快照被称作特性fixture

接下来的部分沿着在spock的spec可能组合构建blocks。一个常规spec能使用它们的中一部分。

###Specification
一个spec表现为继承spock.lang.Specification的一个groovy 类。spec名字通常描述系统或系统操作。例如： CustomerSpec, H264VideoPlayback, and ASpaceshipAttackedFromTwoSides是一个规范合理的名称

spec类包含一系统有用的方法，此处它含义为spec用Sputnik在junit中运行，spock的junit runner，感谢Sputnik，spock spec能运行在大多数的ides和构建工具
###Fields
    def obj = new ClassUnderSpecification()
    def coll = new Collaborator()

对象实例字段是一个好的地方去存储属于spec fixture 。它是一个好的实践在正确初始化他们在定义时候。（语义上它等价于初始化他们在setup()方法）.存储这些实例字段的对象不应该分享在feature方法之间。取而代之，每个特性方法得到他们自己的初始化对象。这个帮助在相互间独立隔离特性方法，提供一个好的目标。

    @Shared res = new VeryExpensiveResource()

一些时候你需要分享一个对象在特性方法之间。例如对象可能创建很昂贵，或者你可能想在不同特性方法指南交互。为了实现这一点，定义@Shared 字段，它最好事初始化正确字段在一个定义点（语义上等价等价与setupSpec）

    static final PI = 3.141592654
静态字段应该是常量，宁一方面shared字段是完美的，因为他的语义表现了更好的定义。
###Fixture Methods

    Fixture Methods
    def setup() {}          // run before every feature method
    def cleanup() {}        // run after every feature method
    def setupSpec() {}     // run before the first feature method
    def cleanupSpec() {}   // run after the last feature method
fixture方法负责在环境设置清理在每个特性方法运行。通常它是一个好的idea使用刷新fixture 方法在为每个fixture方法，在setup() and cleanup()  做这个。偶尔对特性方法分享是由意义的。他实现了分享使用shared字段与setupSpec() and cleanupSpec() methods。
所有fixture方法都是可选择的。

说明：setupSpec cleanupSpec 方法不能饮用实例字段
###Feature Methods
	def "pushing an element on the stack"() {
	// blocks go here
	}
Feature 方法是一个spec核心，他描述了你期待系统的在sus下的特性集。根据约定，特性方法呗命名字符串常量。试着为你的特性方法选择一个好的方法。并且随心使用你喜欢的特性。

概念，一个特性方法包括四个语义段：
设置特性的fixture
提供sus的刺激
描述系统期待的结果
清理fixture 
###Blocks

spock实现支持每个feature方法概念语法阶段作为一个特性方法。对结束来说，特性方法被构建为一个叫做blocks。blocks开始于一个label并且扩展开始下一个label或者方法结束。这个有六个blocks：setup, when, then, expect, cleanup, and where blocks.。任何语句在方法开始与第一次
隐含的setup block之间。

一个特性方法必须至少一个清晰的label，实际上，一个直接的block表现。Blocks 决定一个方法不同部分并且不能嵌套。

Blocks2Phases
这个右侧图片展示block如何匹配一个特性方法语法概念。那里的block有一个特殊的角色，被简单的展现出来，但首先让我们有一个近距离查看这个block

###Setup Blocks

	setup:
	def stack = new Stack()
	def elem = "push me"
这个setup block 你能做任何你想setpup工作为你想描述的特性。他可能不能被前面其他block处理，并且不能重复。一个setup block 没有任何特殊的语义。setup: label时可选的并被提交。结果在一个直接设置的block。given: label是一个setup的别名，并且一些时候导致更多的刻的特性方法描述。

###When and Then Blocks

    when与then blocks
    when: //刺激
    then://响应
    
when与then blocks 一直一起出现。他们描述一个刺激发生并响应期待。
when blocks能包含任意代码；
then blocks被禁止使用在条件，异常条件，交互与变量定义。
一个特性方法可能匹配多个when-then 块
###Conditions

Conditions被描述为一个期待的状态，必须像junit的assertions。然而，conditions被写做一个描述布尔表达式，消除为必需的assertion API（恰恰更多的是一个条件呗处理为一个非布尔表达式，将会被作为处理根据groovy truth）让我们来看一些场景在以下
	
	stack.size() == 2
	|     |      |
	|     1      false

尽可能保持在每个小特性方法里的条件数量。一到五个是一个好的指导。如果你有更多，你得问自己，如果在一个场景下有多个不相关的特性被提及。答案如果是yes，在几个小的特性方法重构。如果你的场景时只有不同的值，考虑使用数据表驱动（spock带有这个特性）

如果一个条件没通过，spock提供什么样的反馈。让我们试并改变第二个场景

如你所见，spock在一个场景执行期间，捕获所有值处理在，并在一个容易理解的形式表现他们。干得漂亮，不是么？
###Implicit and explicit conditions
场景必须有一个 then blocks and expect blocks其中之一。出了调用空方法雨归类表达式作为交互，所有顶级表达式在这些块里面是隐含作为场景对待。在别的地方使用题哦安，你需要设计他们使用groovy的assert关键字。

	def setup() {
	stack = new Stack()
	assert stack.empty
	}
如果显试条件被触发，他将处理一些相同的诊断信息作为隐含条件。
###Exception Conditions
异常场景呗使用描述作为一个when 语句块应该抛出一个异常。他们被定义使用 thrown()方法，被传递期待的异常类型。例如，描述被弹出一个空stack 应该抛出EmptyStackException，你能像下面这样写
	
	when:
	stack.pop()

	then:
	thrown(EmptyStackException)
	stack.empty

如你所见，异常场景可能是跟别的场景相关。这个特别有用为对详细说明期待内容作为异常，访问这个异常，首先绑定一个变量。
	
	when:
	stack.pop()

	then:
	def e = thrown(EmptyStackException)
	e.cause == null

宁外，你也可以使用上面语法作为一个轻微有变化
	
	when:
	stack.pop()

	then:
	EmptyStackException e = thrown()
	e.cause == null

这个语法有两个小优势，首先，异常变量时强类型，让它非常容易被ide补全。其次，场景读起来有一点更像句子。注意，如果没有异常类型被传递在thrown方法，它从左边的变量类型被推断
 
一些时候我们需要表达一个异常不应该被抛出。例如，让我们尝试表达hashmap不应该接受null key。

	def "HashMap accepts null key"() {
	  setup:
	  def map = new HashMap()
	  map.put(null, "elem")
	}
一些时候我们需要表达一个异常不应该被抛出。例如，让我们尝试表达hashmap不应该接受null key。

这个表示不能揭示代码的含义。 是否有人离开构建之前就完成实现了这个方法。毕竟，场景在哪里？ 幸运是我们能做更好。
	
	def "HashMap accepts null key"() {
	  setup:
	def map = new HashMap()

	  when:
	  map.put(null, "elem")

	  then:
	  notThrown(NullPointerException)
	}
通过使用notThrown。我们能使清晰在一个特殊的NullPointerException 不应该被抛出。（按照Map.put()方法的约定，它是正确的事情不支持null keys在map上使用。）然而，方法将会失败如果有任何逼的异常抛出。	
###Interactions
尽管条件描述了一个对象的状态，交互描述了对象互相间关系。交互被转用于一章。所以我们只快速给一个例子在这。假设我们想描述从一个发布者到订阅者的事件流。代码如下

	def "events are published to all subscribers"() {
    	def subscriber1 = Mock(Subscriber)
    	def subscriber2 = Mock(Subscriber)
    	def publisher = new Publisher()
    	publisher.add(subscriber1)
    	publisher.add(subscriber2)

    when:
    publisher.fire("event")

    then:
    1 * subscriber1.receive("event")
    1 * subscriber2.receive("event")
###Expect Blocks

一个expect block 比then block有更多的限制。他可能包含场景与变量定义。它是有用的情形当他更加自然描述刺激与期待结果响应在一个简单的表达式里。例如，比较下面两个描述 Math.max()方法。

		when:
		def x = Math.max(1, 2)
		then:
		x == 2
		expect:
		Math.max(1, 2) == 2

尽管两个片段时语义等价。第二个是更清晰选择。作为一个指引，使用when-then 去描述带有副作用。并且期待描述纯碎的函数方法。
####TIP
利用groovy jdk方法像any（） every（）去创建更多表现力与简洁的条件。
###Cleanup Blocks
  
	setup:
	def file = new File("/some/path")
	file.createNewFile()
	// ...
	cleanup:
	file.delete()
###cleanup block 
只能跟着where block后面。并且不能重复。像一个cleanup 方法，它作为释放资源使用作为一个特性方法使用，运行即使特性方法抛出异常。因此，cleanup block 必须防御编程。在最糟糕的情况下，它必须优雅的处理在特性方法里抛出异常的第一个语句。并且所有本地变量有默认值。
####TIPS
groovy的安全引用操作精简写防御是代码
对象级别的spec 通常不用cleanup方法。作为唯一资源他们消费内存，自动被回收通过垃圾收集齐。更多粗力度spec，但是，可能使用clean 块 清理文件系统关闭数据库连接，关闭网络服务
  
	def "computing the maximum of two numbers"() {
    expect:
    Math.max(a, b) == c

    where:
    a << [5, 3]
    b << [1, 9]
    c << [5, 9]
  }
This where block effectively creates two "versions" of the feature method: One where a is 5, b is 1, and c is 5, and another one where a is 3, b is 9, and c is 9.

这个where 块创建两个版本特性方法非常有效：One where a is 5, b is 1, and c is 5, and another one where a is 3, b is 9, and c is 9.


where block 将会在数据驱动测试章节解释
###Helper Methods

一些时候特性方法增长巨大在 and/or包含重复代码。在这种情形下引入一个多个帮助方法会有意义。两个好的候选者为帮助方法是setup/cleanup 逻辑与复杂场景。分解出前者是直接的，让我们看下场景


	def "offered PC matches preferred configuration"() {
    when:
    def pc = shop.buyPc()

    then:
    pc.vendor == "Sunny"
    pc.clockRate >= 2333
    pc.ram >= 4096
    pc.os == "Linux"
  }
如果碰巧你也是电脑极客，你更愿意pc配置是详细的，或者你可能想比较供应从同的商店。因此，让我们分解条件
	
	def "offered PC matches preferred configuration"() {
    when:
    def pc = shop.buyPc()

    then:
    matchesPreferredConfiguration(pc)
  }

	def matchesPreferredConfiguration(pc) {
    pc.vendor == "Sunny"
    && pc.clockRate >= 2333
    && pc.ram >= 4096
    && pc.os == "Linux"
  }
新的帮助方法matchesPreferredConfiguration 由简单的布尔表达式组成作为结果返回。
这是好的除了那些一个不充分的供应被描述。

  
    Condition not satisfied:

    matchesPreferredConfiguration(pc)
    |                             |
    false                         ...
    Not very helpful. Fortunately, we can do better:

    void matchesPreferredConfiguration(pc) {
      assert pc.vendor == "Sunny"
      assert pc.clockRate >= 2333
      assert pc.ram >= 4096
      assert pc.os == "Linux"
    }
不是很有什么帮助。幸运的事，我们可以做更好
当在一个帮助方法分解场景时，必须思考两个点：首先，隐含条件必须转换为显性条件使用assert关键字。其次，帮助方法必须返回空类型。否则，spock会解释返回值为失败条件，这并不是我们想要的。

作为猜想，改进的帮助方法告诉我们确切什么是错的。

	Condition not satisfied:

      assert pc.clockRate >= 2333
             |  |         |
             |  1666      false
             ...
条件不是满意：

最后一个建议：尽管重用代码是一个好事情，但不要是他太远。更聪明使用fixture 和帮助方法能增加耦合在特性方法间。如果你重用太多或者有错误代码，你将以脆弱和艰难改进的spec结束。

###Specifications as Documentation
写得好的spec作为一个源码信息价值点。尤其是对高级别spec目标是更广泛的受众不知是开发者（架构师 领域专家 客户等）它是有道理的提供更多信息使用自然语言比只是使用规格与特性的名称。因此，spock提供了一种方式附属文本话的描述使用块。

	setup: "open a database connection"
      // code goes here
      Individual parts of a block can be described with and::

      setup: "open a database connection"
      // code goes here

      and: "seed the customer table"
      // code goes here

      and: "seed the product table"
一个and 标签 跟着描述能插入到特性方法的任何位置，没有改变方法语义。

在行为驱动测试，面向客户的特性被描述在一个given-when-then 格式。spock直接支持这个风格的spec使用given标签
	
	given: "an empty bank account"
      // ...

      when: "the account is credited $10"
      // ...

      then: "the account's balance is $10"
      // ...
      As noted before, given: is just an alias for setup:.
作为之前的提示，given知识setup的一个别名

块描述不应该出现在源码里。但应该在运行时可用对spock运行时。块描述用法的计划被增强诊断信息并且文本化的报告被所有利益相关者同样理解。

扩展
如所见，spock提供许多写spec功能。然而总是有这种时候，当一些别的东西被需要。因此，spock提供了一个基于拦截的扩展机制。扩展通过注解被直接激活。当前，spock附带一下指令

 
    @Timeout  
    Sets a timeout for execution of a feature or fixture method.

    设置超时时间为特性方法或者fixture方法

    @Ignore 
    Ignores a feature method.

    忽略特性方法

    @IgnoreRest 
    Ignores all feature methods not carrying this annotation. Useful for quickly running just a single method.
    不移除这个注释，忽略所有特性方法不去。对快速运行单个方法非常有用。

  期待一个特性方法去完成打断。@FailsWith有两个用例：首先，文档化我们知道的bug不需要立刻解释。其次，替换异常条件在一个确定的角落用例在后面不能使用。在所有的用例，异常条件不是优选。

学习怎样实现你自己的指令与扩展，去看扩展章节

Comparison to JUnit

尽管spock使用不同的技术，大部分概念和特性呗激活从junit。这有一个粗的比较
comparison


|Spock|JUnit| 
|--|--| 
|Specification|Test class| 
|setup()|@Before| 
|cleanup()|@After| 
|setupSpec()|@BeforeClass|
|cleanupSpec()|@AfterClass| 
|Feature|Test| 
|Feature method|Test method|
|Data-driven feature|Theory|
|Condition|Assertion|
|Exception condition|@Test(expected=…​)| 
|Interaction|Mock expectation (e.g. in Mockito)|





	
	


    






