# 追求代码质量: 用 JUnitPerf 进行性能测试

# 追求代码质量: 用 JUnitPerf 进行性能测试

*监控可伸缩性和性能的两个简单测试*

在应用程序的开发周期中，性能测试常被放到最后考虑，这并不是因为它不重要，而是因为存在这么多未知变量，很难有效地测试。在本月的 *追求代码质量* 系列中，Andrew Glover 使性能测试成为开发周期的一部分，并介绍了两种简单的实现方法。

在应用程序的开发中，验证应用程序的性能几乎总处于次要的地位。请注意，我强调的是*验证* 应用程序的性能。应用程序的性能*总是* 首要考虑的因素，但开发周期中却很少包含对性能的验证。

由于种种原因，性能测试常被延迟到开发周期的后期。以我的经验，企业之所以在开发过程中不包含性能测试是因为，他们不知道对于正在进行开发的应用程序要期待什么。提出了一些（性能）指数，但这些指数是基于预期负载提出的。

发生下列两种情况之一时，性能测试就成为头等大事：

*   生产中出现显而易见的性能问题。
*   在同意付费*之前* ，客户或潜在客户询问有关性能指数的问题。

本月，我将介绍两种简单的性能测试技术，在上述两种情况中的任何一种发生前进行测试。

## 改进代码质量

别错过 Andrew 的 [讨论论坛](http://www.ibm.com/developerworks/forums/dw_forum.jsp?S_TACT=105AGX52&cat=10&S_CMP=cn-a-j&forum=812)，里面有关于代码语法、测试框架以及如何编写专注于质量的代码的帮助。

## 用 JUnitPerf 进行测试

在软件开发的早期阶段，使用 JUnit 很容易确定基本的低端性能指数。JUnitPerf 框架能够将测试快速地转化为简单的负载测试，甚至压力测试。

可使用 JUnitPerf 创建两种测试类型：`TimedTest` 和 `LoadTest`。这两种类型都基于 Decorator 设计模式并利用 JUnit 的 `suite` 机制。`TimedTest` 为测试样例创建一个（时间）上限 —— 如果超过这个时间，那么测试失败。`LoadTest` 和计时器一起运行，它通过运行所需的次数（时间间隔由配置的计时器控制），在一个特定的测试用例上创建一个人工负载。

* * *

## 恰当的时限测试

JUnitPerf `TimedTest` 让您可以编写有相关时间限制的测试 —— 如果超过了该限度，就认为测试是失败的（即便测试逻辑本身实际上是成功的）。在测试对于业务致关重要的方法时，时限测试相比其他测试来说，在确定和监控性能指数方面很有帮助。甚至可以测试得更加细致一些，可以测试一系列方法来确保它们满足特定的时间限制。

例如，假设存在一个 Widget 应用程序，其中，特定的对于业务致关重要的方法（如 `createWidget()`）是严格的性能限制的测试目标。假设需要对执行该 `create()` 方法的功能方面进行性能测试。这通常会由不同的团队使用不同的工具在开发周期的后期加以确定，这通常不能指出精确的方法。但假设决定选择*早期经常测试* 方法取而代之。

创建 `TimedTest` 首先要创建一个标准的 JUnit 测试。换言之，将对 `TestCase` 或其派生类进行扩展，并编写一个以 `test` 开头的方法，如清单 1 所示：

##### 清单 1\. 简单的 widget 测试

```
public class WidgetDAOImplTest extends TestCase {    
 private WidgetDAO dao;

 public void testCreate() throws Exception{
  IWidget wdgt = new Widget();
  wdgt.setWidgetId(1000);
  wdgt.setPartNumber("12-34-BBD");  
  try{
   this.dao.createWidget(wdgt);
  }catch(CreateException e){
   TestCase.fail("CreateException thrown creating a Widget");
  }        
 }

 protected void setUp() throws Exception {       
  ApplicationContext context = 
   new ClassPathXmlApplicationContext("spring-config.xml");      
  this.dao = (WidgetDAO) context.getBean("widgetDAO");      
 }       
} 
```

由于 JUnitPerf 是一个基于装饰器的框架，为了真正地驾驭它，必须提供一个 `suite()` 方法并将现有的测试装饰以 `TimedTest`。`TimedTest` 以 `Test` 和执行该测试的最大时间量作为参数。

也可以选择传入一个 `boolean` 标志作为第三个参数（`false`），这将导致测试快速失败 —— 意味着如果超过最大时间，JUnitPerf 将*立即* 迫使测试失败。否则，测试样例将完整运行，然后失败。区别很微妙：在一个失败的样例中，不带可选标志运行测试可以帮您了解运行总时间。传入 `false` 值却意味着得不到运行总时间。

例如，在清单 2 中，我在运行 `testCreate()` 时设定了一个两秒钟的上限。如果执行总时间超过了这个时间，测试样例将失败。由于我并未传入可选的 `boolean` 参数，该测试将完整运行，而不管运行会持续多久。

##### 清单 2\. 为生成 TimedTest 而实现的 suite 方法

```
public static Test suite() {
 long maxElapsedTime = 2000; //2 seconds 
 Test timedTest = new TimedTest(
   new WidgetDAOImplTest("testCreate"), maxElapsedTime);
 return timedTest;                
} 
```

此测试通常在 JUnit 框架中运行 —— 现有的 Ant 任务、Eclipse 运行器等等，会像运行任何其他 JUnit 测试一样运行这个测试。惟一的不同是，该测试将发生在计时器的上下文中。

* * *

## 过度的负载测试

与在测试场景中验证一个方法（或系列方法）的时间限制正好相反，JUnitPerf 也方便了负载测试。正如在 `TimedTest` 中一样，JUnitPerf 的 `LoadTest` 也像装饰器一样运行，它通过将 JUnit `Test` 和额外的线程信息绑定起来，从而模拟负载。

使用 `LoadTest`，可以指定要模拟的用户（线程）数量，甚至为这些线程的启动提供计时机制。JUnitPerf 提供两类 `Timer`：`ConstantTimer` 和 `RandomTimer`。通过为 `LoadTest` 提供这两类计时器，可以更真实地模拟用户负载。如果没有 `Timer`，所有线程都会同时启动。

清单 3 是用 `ConstantTimer` 实现的含 10 个模拟用户的负载测试：

##### 清单 3\. 为生成负载测试而实现的 suite 方法

```
public static Test suite() {
 int users = 10;
 Timer timer = new ConstantTimer(100);        
 return new LoadTest(
  new WidgetDAOImplTest("testCreate"), 
    users, timer);        
} 
```

请注意，`testCreate()` 方法运行 10 次，每个线程间隔 100 毫秒启动。未设定时间限制 —— 这些方法完整运行，如果其中任何的方法执行失败，JUnit 会相应地报告失败。

* * *

## 用样式进行装饰

装饰器并不局限于单个的装饰物。例如，在 Java™ I/O 中，可以为 `FileInputStream` 装饰上一个带 `BufferedReader` 的 `InputStreamReader`（只要记住：`BufferedReader in = new BufferedReader(new InputStreamReader(new FileInputStream("infilename"), "UTF8"))`）。

装饰可以有多个层次，JUnitPerf 的 `TimedTest` 和 `LoadTest` 也是一样。当这两个类彼此装饰时，将导致一些强制的测试场景，例如像这样的场景：在一项业务中放置了负载并应用了时间限制。或者，我们可以仅仅将之前的两个测试场景以如下方式结合起来：

*   在 `testCreate()` 方法中放置一项负载。
*   规定每个线程必须在该时间限制内结束。

我通过为一个标准 `Test` 装饰上 `LoadTest`（由 `TimedTest` 装饰）应用了上述规范，清单 4 显示了其结果。

##### 清单 4\. 经装饰的负载和时限测试

```
public static Test suite() {
 int users = 10;
 Timer timer = new ConstantTimer(100);
 long maxElapsedTime = 2000;      
 return new TimedTest(new LoadTest(
   new WidgetDAOImplTest("testCreate"), users, timer), 
     maxElapsedTime);          
} 
```

正如您所看到的那样，`testCreate()` 方法运行 10 次（每隔 100 毫秒启动一个线程），且每个线程必须在 2 秒内完成，否则整个测试场景将失败。

* * *

## 使用注意

尽管 JUnitPerf 是一个性能测试框架，但也要先大致估计一下测试要设定的性能指数。这是由于所有由 JUnitPerf 装饰的测试都通过 JUnit 框架运行，所以就存在额外的消耗，特别是在利用 fixture 时。由于 JUnit 本身用一个 `setUp` 和一个 `tearDown()` 方法装饰所有测试样例，所以要在测试场景的整个上下文中考虑执行时间。

相应地，我经常创建使用我想要的 fixture 逻辑的测试，但也会运行一个空白测试来确定性能指数基线。*这是一个大致的估计*，但它必须作为基线添加到任何想要的测试限制中。

例如，如果运行一个由 fixture 逻辑（使用 DbUnit）装饰的空白测试用时 2.5 秒，那么您想要的所有测试限制都应将这一额外时间考虑在内 —— 这可以从清单 5 中的基准测试中看到：

##### 清单 5\. JUnitPerf 基准测试

```
public class DBUnitSetUpBenchmarkTest extends DatabaseTestCase {
 private WidgetDAO dao = null;

 public void testNothing(){
  //should be about 2.5 seconds
 }

 protected IDatabaseConnection getConnection() throws Exception {        
  Class driverClass = Class.forName("org.hsqldb.jdbcDriver");
  Connection jdbcConnection = 
   DriverManager.getConnection(
     "jdbc:hsqldb:hsql://127.0.0.1", "sa", "");
  return new DatabaseConnection(jdbcConnection);
 }

 protected IDataSet getDataSet() throws Exception {        
  return new FlatXmlDataSet(new File("test/conf/seed.xml"));
 }

 protected void setUp() throws Exception {
  super.setUp();      
  final ApplicationContext context = 
   new ClassPathXmlApplicationContext("spring-config.xml");      
  this.dao = (WidgetDAO) context.getBean("widgetDAO");      
 }
} 
```

请注意，清单 5 的测试样例 `testNothing()`*什么都没做*。其惟一的目的是确定运行 `setUp()` 方法（当然，该方法也通过 DbUnit 设置了一个数据库）的总时间。

也请记住，测试时间将依赖于机器的配置而变化，同时也依赖于在执行 JUnitPerf 测试时运行的东西而变化。我经常发现，将 JUnitPerf 测试放到它们自己的分类中有助于将它们同标准测试隔离开。这意味着，在运行一个测试时不必每次都运行 JUnitPerf 测试，例如在一个 CI 环境中签入代码。我也会创建特定的 Ant 任务，从而只在精心策划的将性能测试考虑在内的场景或环境中运行这些测试。

* * *

## 试试吧！

用 JUnitPerf 进行性能测试无疑是一门严格的科学，但在开发生命周期的早期，这是确定和监控应用程序代码的低端性能的极佳方式。另外，由于它是一个基于装饰器的 JUnit 扩展框架，所以可以很容易地用 JUnitPerf 装饰*现有的* JUnit 测试。

想想您已经花了这么多时间来担心应用程序在负载下会怎样执行。用 JUnitPerf 进行性能测试可以为您减少担忧并节省时间，同时也确保了应用程序代码的质量。