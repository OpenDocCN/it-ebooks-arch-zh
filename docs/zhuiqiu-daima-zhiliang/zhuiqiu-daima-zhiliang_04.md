# 追求代码质量: 探究 XMLUnit

# 追求代码质量: 探究 XMLUnit

*一种用于测试 XML 文档的 JUnit 扩展框架*

Java™ 开发人员一般都很善于解决问题，所以由 Java 开发人员提出更容易的方法用以验证 XML 文档是很自然的事。本月，Andrew 将向您介绍 XMLUnit，一个能满足您所有的 XML 验证需求的 JUnit 扩展框架。

在软件开发周期中，需要不时地验证 XML 文档的结构或内容。不管构建的是何种应用程序，测试 XML 文档都具有一定的挑战性，尤其是在没有相关工具的情况下就更是如此。

本月，我将首先向您说明为何*不能* 使用 `String` 比较来验证 XML 文档的结构和内容。之后，我会介绍 XMLUnit，一个由 Java 开发人员创建并可服务于 Java 开发人员的 XML 验证工具，向您展示如何使用它来验证 XML 文档。

## 古典的 String 比较

首先，假设您已经构建了一个应用程序，该应用程序可以输出代表对象依赖性报告的 XML 文档。对于给定的类和对应的过滤器的集合，会生成一个报告来输出类和类的依赖项（想象一下导入）。

清单 1 显示了用于给定类列表（`com.acme.web.Widget` 和 `com.acme.web.Account`）的报告，过滤器被设为忽略外部类，比如 `java.lang.String`：

##### 清单 1\. 一个示例依赖性 XML 报告

```
<DependencyReport date="Sun Dec 03 22:30:21 EST 2006">
  <FiltersApplied>
    <Filter pattern="java|org"/>
    <Filter pattern="net."/>
  </FiltersApplied>
  <Class name="com.acme.web.Widget">
    <Dependency name="com.acme.resource.Configuration"/>
    <Dependency name="com.acme.xml.Document"/>
  </Class>
  <Class name="com.acme.web.Account">
    <Dependency name="com.acme.resource.Configuration"/>
    <Dependency name="com.acme.xml.Document"/>
  </Class>
</DependencyReport> 
```

清单 1 很明显是由应用程序生成的；因而，第一层测试就是验证应用程序是否真能生成一个文档。一旦验证了这一点，就可以继续测试指定文档的其他三个方面：

*   结构
*   内容
*   指定内容

可以通过单独使用 JUnit 利用 `String` 比较处理上述前两个方面，如清单 2 所示：

##### 清单 2\. 硬性验证 XML

```
public class XMLReportTest extends TestCase {

 private Filter[] getFilters(){
  Filter[] fltrs = new Filter[2];
  fltrs[0] = new RegexPackageFilter("java|org");
  fltrs[1] = new SimplePackageFilter("net.");
  return fltrs;
 }

 private Dependency[] getDependencies(){
  Dependency[] deps = new Dependency[2];
  deps[0] = new Dependency("com.acme.resource.Configuration");
  deps[1] = new Dependency("com.acme.xml.Document");
  return deps;
 }

 public void testToXML() {
  Date now = new Date();
  BatchDependencyXMLReport report = 
   new BatchDependencyXMLReport(now, this.getFilters());

  report.addTargetAndDependencies(
    "com.acme.web.Widget", this.getDependencies());
  report.addTargetAndDependencies(
    "com.acme.web.Account", this.getDependencies());

  String valid = "<DependencyReport date=\"" + now.toString() + "\">"+
    "<FiltersApplied><Filter pattern=\"java|org\" /><Filter pattern=\"net.\" />"+
    "</FiltersApplied><Class name=\"com.acme.web.Widget\">" +
    " <Dependency name=\"com.acme.resource.Configuration\" />"+
    "<Dependency name=\"com.acme.xml.Document\" /></Class>"+
    "<Class name=\"com.acme.web.Account\">"+
    "<Dependency name=\"com.acme.resource.Configuration\" />"+
    "<Dependency name=\"com.acme.xml.Document\" />"+
    "</Class></DependencyReport>";

   assertEquals("report didn't match xml", valid, report.toXML());
 }
} 
```

清单 2 中的测试有其他一些重大的缺陷 —— 而不仅仅是硬编码 `String` 比较那么简单。首先，测试并不真正可读。第二，它惊人的脆弱；一旦 XML 文档的格式改变（包括添加空格），与其尝试修复 `String` 本身，还不如粘贴进一个新的文档副本。最后，测试的本性会迫使您必须应付 `Date` 方面，虽然您并不想如此。

若想确保文档中第二个 `Class` 元素的 `name` 值是 `com.acme.web.Account` 又该如何呢？当然，您可以使用常规表达式或 `String` 搜索，但所需的工作量太大。这样看来，通过一个解析框架来操纵此 DOM 不是更有意义么？

## XMLUnit 能否用于 TestNG？

XMLUnit 是一个 JUnit 扩展，但这并不意味着不能在 TestNG 使用它。只要它具有 API 而且此 API 支持委托同时不基于修饰器，那么您可以将几乎任何框架整合进 TestNG。

## 用 XMLUnit 进行测试

当您感觉自己为完成一项任务而努力过了头，您就可以想想解决此问题是否还有更容易的捷径可寻。如果所要解决的问题涉及的是编程式地验证 XML 文档，那么所应想到的解决方案就是 XMLUnit。

XMLUnit 是一种 JUnit 扩展框架，有助于开发人员测试 XML 文档。实际上，XMLUnit 是一种真正的 XML 测试的“多面手”：可以使用它来验证 XML 文档的结构、内容甚至该文档的指定部分。

最简单的做法是使用 XMLUnit 在逻辑上对比运行时 XML 文档和预定义的有效控制文件。本质上讲，这就是一种*差异测试*：假定一个 XML 文档是正确的，那么此应用程序在运行时是否会生成同样的东西？它是相对简单的一种测试，但也可以使用它来验证 XML 文档的结构和内容。也可以通过 XPath 的一点帮助来验证特定内容。

## 委托而非继承

首要原则是尽量避免测试用例继承。许多 JUnit 扩展框架，包括 XMLUnit，都提供可以通过继承得到的专门的测试用例来协助测试某一特定的架构。从框架继承来的测试用例都缺乏灵活性，这是 Java 平台的单一继承的范型所致。更多的时候，这些相同的 JUnit 扩展框架提供一个委托 API，此 API 可以更易于组合不同的框架，而无需采用严格的继承结构。

## 验证内容

可以通过委托或继承的方式使用 XMLUnit。作为最佳策略，我建议避免测试用例继承。另一方面，从 XMLUnit 的 `XMLTestCase` 继承确实可以提供一些方便的声明方法（这些方法不是`静态` 的，因而也就不能像 JUnit 的 `TestCase` 声明一样被静态引用）。

不管您如何选择使用 XMLUnit，都必须实例化 XMLUnit 的解析器。您可以通过 `System.setProperty` 调用实例化它们，也可以通过 `XMLUnit` 核心类上的一些方便的 `static` 方法对它们进行实例化。

一旦用所需要的不同的解析器实例化 XMLUnit 之后，就可以使用 `Diff` 类，这是从逻辑上对比两个 XML 文档所需的中心机制。在清单 3 中，我利用 XMLUnit 对 >testToXML test 做了一些改进：

##### 清单 3\. 改进后的 testToXML 测试

```
public class XMLReportTest extends TestCase {

 protected void setUp() throws Exception {         
  XMLUnit.setControlParser(
    "org.apache.xerces.jaxp.DocumentBuilderFactoryImpl");
  XMLUnit.setTestParser(
    "org.apache.xerces.jaxp.DocumentBuilderFactoryImpl");
  XMLUnit.setSAXParserFactory(
    "org.apache.xerces.jaxp.SAXParserFactoryImpl");
  XMLUnit.setIgnoreWhitespace(true);   
 }

 private Filter[] getFilters(){
  Filter[] fltrs = new Filter[2];
  fltrs[0] = new RegexPackageFilter("java|org");
  fltrs[1] = new SimplePackageFilter("net.");
  return fltrs;
 }

 private Dependency[] getDependencies(){
  Dependency[] deps = new Dependency[2];
  deps[0] = new Dependency("com.acme.resource.Configuration");
  deps[1] = new Dependency("com.acme.xml.Document");
  return deps;
 }

 public void testToXML() {
  BatchDependencyXMLReport report = 
    new BatchDependencyXMLReport(new Date(1165203021718L), 
      this.getFilters());

  report.addTargetAndDependencies(
    "com.acme.web.Widget", this.getDependencies());
  report.addTargetAndDependencies(
    "com.acme.web.Account", this.getDependencies());

  Diff diff = new Diff(new FileReader(
    new File("./test/conf/report-control.xml")),
    new StringReader(report.toXML()));

  assertTrue("XML was not identical", diff.identical());        
 }
} 
```

注意一下我是如何实例化 XMLUnit 的 `setControlParser`、`setTestParser` 和 `setSAXParserFactory` 方法的。您可以为这些值使用任何兼容 JAXP 的解析器。还要注意我是用 `true` 调用 `setIgnoreWhitespace` 的 —— 这是一根救命稻草，相信我！否则，不一致的空白会导致很多故障。

* * *

## 用 Diff 比较

`Diff` 类支持两种比较：`identical` 和 `similar`。如果所比较的文档在结构和值（如果设置了标志就忽略空白）方面都完全相同，那么它们就被认为是 *identical*；如果两个文档是完全相同的，那么它们也就很自然的是 *similar* 的。反之，却不一定。

例如，清单 4 是与清单 5 相似的一个简单的 XML 代码片段，但二者并不相同：

##### 清单 4\. 一个帐号 XML 片段

```
<account>
 <id>3A-00</id>
 <name>acme</name>
</account> 
```

清单 5 中的 XML 片段与清单 4 中所示的 XML 片段有相同的逻辑文档。但 XMLUnit 并不认为二者是相同的，原因是二者的 `name` 和 `id` 元素是颠倒的。

##### 清单 5\. 一个相似的 XML 片段

```
<account>
 <name>acme</name>
 <id>3A-00</id>
</account> 
```

相应地，我可以编写测试用例来验证 XMLUnit 的行为，如清单 6 所示：

##### 清单 6\. 用来验证相同性和相似性的测试

```
public void testIdenticalAndSimilar() throws Exception {
 String controlXML = "<account><id>3A-00</id><name>acme</name></account>";
 String testXML = "<account><name>acme</name><id>3A-00</id></account>"; 
 Diff diff = new Diff(controlXML, testXML);
 assertTrue(diff.similar());
 assertFalse(diff.identical());
} 
```

相似和相同的 XML 文档之间的差异是很微小的；但若能验证两者却非常有用，例如在需要测试由不同应用程序或客户程序生成的文档的情况下。

* * *

## 验证结构

除了验证内容之外，您还需要验证 XML 文档的结构。在这种情况下，元素和属性的值并不重要 —— 您所关心的是结构。

还好，我还可以再次使用清单 3 中定义的测试用例来验证文档的结构，并可以有效忽略元素文本值和属性值。为实现此目的，我调用 `Diff` 类上的 `overrideDifferenceListener()` 并为它添加由 XMLUnit 提供的 `IgnoreTextAndAttributeValuesDifferenceListener`。修改后的测试如清单 7 所示：

##### 清单 7\. 无需属性值验证 XML 结构

```
public void testToXMLFormatOnly() throws Exception{
 BatchDependencyXMLReport report = 
   new BatchDependencyXMLReport(new Date(), this.getFilters());

 report.addTargetAndDependencies(
   "com.acme.web.Widget", this.getDependencies());
 report.addTargetAndDependencies(
   "com.acme.web.Account", this.getDependencies());

 Diff diff = new Diff(new FileReader(
   new File("./test/conf/report-control.xml")),
   new StringReader(report.toXML()));

 diff.overrideDifferenceListener(
   new IgnoreTextAndAttributeValuesDifferenceListener());
 assertTrue("XML was not similar", diff.similar());        
} 
```

## 相似但不相同！

当使用 `IgnoreTextAndAttributeValuesDifferenceListener` 类时，必须声明这两个文档是 `similar` 而*非* `identical`。如果错误地调用了 `identical`，那么就需要处理属性值。

当然，DTD 的模式和 XML 模式都有助于 XML 结构验证，然而，有时文档并不需要引用它们 —— 在这些场景下，结构验证可能会很有用。同样，如果需要忽略特定的一些值（例如那些 `Date` 值），就可以实现 `DifferenceListener` 接口（正如 `IgnoreTextAndAttributeValuesDifferenceListener` 所做的一样）并提供一个定制实现。

## XMLUnit 和 XPath

为实现 XML 测试的所有三个方面，XMLUnit 还可以借助 XPath 进行 XML 文档特定部分的验证。

例如，使用清单 1 所示相同的格式，我想验证由应用程序生成的第一个 `Class` 元素的 `name` 属性值是否是 `com.acme.web.Widget`。要实现此目的，我必须创建一个 XPath 表达式来导航到准确的位置；而且，XMLUnit 的 `XMLTestCase` 提供了一个方便的 `assertXpathExists()` 方法，这意味着我必须现在扩展 `XMLTestCase`。

##### 清单 8\. 使用 XPath 来验证准确的 XML 值

```
public void testToXMLFormatOnly() throws Exception{
 BatchDependencyXMLReport report = 
   new BatchDependencyXMLReport(new Date(), this.getFilters());

 report.addTargetAndDependencies(
   "com.acme.web.Widget", this.getDependencies());
 report.addTargetAndDependencies(
   "com.acme.web.Account", this.getDependencies());

 assertXpathExists("//Class[1][@name='com.acme.web.Widget']", 
  report.toXML());    
} 
```

如清单 8 所示，XMLUnit 和 XPath 一起协作提供了可以准确验证 XML 文档 的一种便捷机制，而不是进行大规模的差异测试。请记住要在 XMLUnit 内充分利用 XPath，您的测试用例必须要扩展 `XMLTestCase`。如果熟悉 XPath 也会大有帮助！

## XPath 是什么？

XPath 或 XML Path Language 是一种表达式语言，用来基于树表示定位 XML 文档的各部分。XPath 允许您导航 XML 文档并可以帮您选择文档值。

## 为何要舍近求远呢？

XMLUnit 是一种基于 Java 的开放源码工具，它使测试 XML 文档更为简单和灵活，而这是使用 `String` 比较所达不到的。使用 XMLUnit 进行差异测试所存在的惟一缺点是测试会依赖于文件系统来加载控制文档。在编写测试时，请务必考虑这一附加的依赖性。

虽然 XMLUnit 已经有段时间没有发布任何更新了，但它当前的特性集已经足够健壮来应对各种测试冲击，并且它用在这种情况下基本上是免费的！