# 追求代码质量: JUnit 4 与 TestNG 的对比

# 追求代码质量: JUnit 4 与 TestNG 的对比

*为什么 TestNG 框架依然是大规模测试的较好选择？*

JUnit 4 具有基于注释的新框架，它包含了 TestNG 一些最优异的特性。但这是否意味着 JUnit 4 已经淘汰了 TestNG？Andrew Glover 探讨了这两种框架各自的独特之处，并阐述了 TestNG 独有的三种高级测试特性。

经过长时间积极的开发之后，JUnit 4.0 于今年年初发布了。JUnit 框架的某些最有趣的更改 —— 特别是对于本专栏的读者来说 —— 正是通过巧妙地使用注释实现的。除外观和风格方面的显著改进外，新框架的特性使测试用例的编制从结构规则中解放出来。使原来僵化的 fixture 模型更为灵活，有利于采取可配置程度更高的方法。因此，JUnit 框架不再强求把每一项测试工作定义为一个名称以 `test` 开始的方法，并且现在可以只运行一次 fixture，而不是每次测试都需要运行一次。

虽然这些改变令人欣慰，但 JUnit 4 并不是第一个提供基于注释的灵活模型的 Java™ 测试框架。在修改 JUnit 之前很久，TestNG 就已建立为一个基于注释的框架。

事实上，是 TestNG 在 Java 编程中*率先* 实现了利用注释进行测试，这使它成为 JUnit 的有力竞争对手。然而，自从 JUnit 4 发布后，很多开发者质疑：二者之间还有什么差别吗？在本月的专栏中，我将讨论 TestNG 不同于 JUnit 4 的一些特性，并提议采用一些方法，使得这两个框架能继续互相补充，而不是互相竞争。

## 您知道吗？

在 Ant 中运行 JUnit 4 测试比预计的要难得多。事实上，一些团队已发现，惟一的解决方法是升级到 Ant 1.7。

## 表面上的相似

JUnit 4 和 TestNG 有一些共同的重要特性。这两个框架都让测试工作简单得令人吃惊（和愉快），给测试工作带来了便利。二者也都拥有活跃的社区，为主动开发提供支持，同时生成丰富的文档。

## 提高代码质量

要找到您最迫切问题的答案，请不要错过 Andrew 的 [论坛](http://www.ibm.com/developerworks/forums/dw_forum.jsp?S_TACT=105AGX52&cat=10&S_CMP=cn-a-j&forum=812)。

两个框架的不同在于核心设计。JUnit *一直* 是一个单元测试框架，也就是说，其构建目的是促进单个对象的测试，它确实能够极其有效地完成此类任务。而 TestNG 则是用来解决*更高* 级别的测试问题，因此，它具有 JUnit 中所没有的一些特性。

### 一个简单的测试用例

初看起来，JUnit 4 和 TestNG 中实现的测试非常相似。为了更好地理解我的意思，请看一下清单 1 中的代码。这是一个 JUnit 4 测试，它有一个 macro-fixture（即仅在所有测试运行前调用一次的 fixture），这个 macro-fixture 由 `@BeforeClass` 属性表示：

##### 清单 1\. 一个简单的 JUnit 4 测试用例

```
package test.com.acme.dona.dep;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNotNull;
import org.junit.BeforeClass;
import org.junit.Test;

public class DependencyFinderTest {
 private static DependencyFinder finder;

 @BeforeClass
 public static void init() throws Exception {
  finder = new DependencyFinder();
 }

 @Test
 public void verifyDependencies() 
  throws Exception {
   String targetClss = 
     "test.com.acme.dona.dep.DependencyFind";

   Filter[] filtr = new Filter[] { 
      new RegexPackageFilter("java|junit|org")};

   Dependency[] deps = 
      finder.findDependencies(targetClss, filtr);

   assertNotNull("deps was null", deps);
   assertEquals("should be 5 large", 5, deps.length);    
 }
} 
```

JUnit 用户会立即注意到：这个类中没有了以前版本的 JUnit 中所要求的一些*语法成分*。这个类没有 `setUp()` 方法，也不对 `TestCase` 类进行扩展，甚至也没有哪个方法的名称以 `test` 开始。这个类还利用了 Java 5 的一些特性，例如静态导入，很明显地，它还使用了注释。

### 更多的灵活性

在清单 2 中，您可以看到*同一个* 测试项目。不过这次是用 TestNG 实现的。这里的代码跟清单 1 中的测试代码有个微妙的差别。发现了吗？

##### 清单 2\. 一个 TestNG 测试用例

```
package test.com.acme.dona.dep;

import static org.testng.Assert.assertEquals;
import static org.testng.Assert.assertNotNull;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Configuration;
import org.testng.annotations.Test;

public class DependencyFinderTest {
 private DependencyFinder finder;

 @BeforeClass
 private void init(){
  this.finder = new DependencyFinder();
 }

 @Test
 public void verifyDependencies() 
  throws Exception {
   String targetClss = 
     "test.com.acme.dona.dep.DependencyFind";

   Filter[] filtr = new Filter[] { 
      new RegexPackageFilter("java|junit|org")};

   Dependency[] deps = 
      finder.findDependencies(targetClss, filtr);

   assertNotNull(deps, "deps was null" );
   assertEquals(5, deps.length, "should be 5 large");        
 }
} 
```

显然，这两个清单很相似。不过，如果仔细看，您会发现 TestNG 的编码规则比 JUnit 4 更灵活。清单 1 里，在 JUnit 中我必须把 `@BeforeClass` 修饰的方法声明为 `static`，这又要求我把 fixture，即 `finder` 声明为 `static`。我还必须把 `init()` 声明为 `public`。看看清单 2，您就会发现不同。这里不再需要那些规则了。我的 `init()` 方法既不是 `static`，也不是 `public`。

从最初起，TestNG 的灵活性就是其主要优势之一，但这并非它惟一的卖点。TestNG 还提供了 JUnit 4 所不具备的其他一些特性。

* * *

## 依赖性测试

JUnit 框架想达到的一个目标就是测试隔离。它的缺点是：人们很难确定测试用例执行的顺序，而这对于任何类型的依赖性测试都非常重要。开发者们使用了多种技术来解决这个问题，例如，按字母顺序指定测试用例，或是更多地依靠 fixture 来适当地解决问题。

如果测试成功，这些解决方法都没什么问题。但是，如果测试不成功，就会产生一个很麻烦的后果：*所有* 后续的依赖测试也会失败。在某些情况下，这会使大型测试套件报告出许多不必要的错误。例如，假设有一个测试套件测试一个需要登录的 Web 应用程序。您可以创建一个有依赖关系的方法，通过登录到这个应用程序来创建整个测试套件，从而避免 JUnit 的隔离机制。这种解决方法不错，但是如果登录失败，即使登录该应用程序后的其他功能都正常工作，整个测试套件依然会全部失败！

### 跳过，而不是标为失败

与 JUnit 不同，TestNG 利用 `Test` 注释的 `dependsOnMethods` 属性来应对测试的依赖性问题。有了这个便利的特性，就可以轻松指定依赖方法。例如，前面所说的登录将在某个方法*之前* 运行。此外，如果依赖方法失败，它将被*跳过*，而不是标记为失败。

##### 清单 3\. 使用 TestNG 进行依赖性测试

```
import net.sourceforge.jwebunit.WebTester;

public class AccountHistoryTest  {
 private WebTester tester;

 @BeforeClass
 protected void init() throws Exception {
  this.tester = new WebTester();
  this.tester.getTestContext().
   setBaseUrl("http://div.acme.com:8185/ceg/");
 }

 @Test
 public void verifyLogIn() {
  this.tester.beginAt("/");        
  this.tester.setFormElement("username", "admin");
  this.tester.setFormElement("password", "admin");
  this.tester.submit();        
  this.tester.assertTextPresent("Logged in as admin");
 }

 @Test (dependsOnMethods = {"verifyLogIn"})
 public void verifyAccountInfo() {
  this.tester.clickLinkWithText("History", 0);        
  this.tester.assertTextPresent("GTG Data Feed");
 }
} 
```

在清单 3 中定义了两个测试：一个验证登录，另一个验证账户信息。请注意，通过使用 `Test` 注释的 `dependsOnMethods = {"verifyLogIn"}` 子句，`verifyAccountInfo` 测试指定了它依赖 `verifyLogIn()` 方法。

通过 TestNG 的 Eclipse 插件（例如）运行该测试时，如果 `verifyLogIn` 测试失败，TestNG 将直接跳过 `verifyAccountInfo` 测试，请参见图 1：

##### 图 1\. 在 TestNG 中跳过的测试

![](img/skipped-tests-eclipse.jpg)

对于大型测试套件，TestNG 这种不标记为失败，而只是跳过的处理方法可以减轻很多压力。您的团队可以集中精力查找为什么百分之五十的测试套件被跳过，而不是去找百分之五十的测试套件失败的原因！更有利的是，TestNG 采取了只重新运行失败测试的机制，这使它的依赖性测试设置更为完善。

* * *

## 失败和重运行

在大型测试套件中，这种重新运行失败测试的能力显得尤为方便。这是 TestNG 独有的一个特性。在 JUnit 4 中，如果测试套件包括 1000 项测试，其中 3 项失败，很可能就会迫使您重新运行整个测试套件（修改错误以后）。不用说，这样的工作可能会耗费几个小时。

一旦 TestNG 中出现失败，它就会创建一个 XML 配置文件，对失败的测试加以说明。如果利用这个文件执行 TestNG 运行程序，TestNG 就*只* 运行失败的测试。所以，在前面的例子里，您只需重新运行那三个失败的测试，而不是整个测试套件。

实际上，您可以通过清单 2 中的 Web 测试的例子自己看到这点。`verifyLogIn()` 方法失败时，TestNG 自动创建一个 testng-failed.xml 文件。该文件将成为如清单 4 所示的替代性测试套件：

##### 清单 4\. 失败测试的 XML 文件

```
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite thread-count="5" verbose="1" name="Failed suite [HistoryTesting]" 
       parallel="false" annotations="JDK5">
 <test name="test.com.acme.ceg.AccountHistoryTest(failed)" junit="false">
  <classes>
   <class name="test.com.acme.ceg.AccountHistoryTest">
    <methods>
     <include name="verifyLogIn"/>
    </methods>
   </class>
  </classes>
 </test>
</suite> 
```

运行小的测试套件时，这个特性似乎没什么大不了。但是如果您的测试套件规模较大，您很快就会体会到它的好处。

* * *

## 参数化测试

TestNG 中另一个有趣的特性是*参数化测试*。在 JUnit 中，如果您想改变某个受测方法的参数组，就只能给*每个* 不同的参数组编写一个测试用例。多数情况下，这不会带来太多麻烦。然而，我们有时会碰到一些情况，对其中的业务逻辑，需要运行的测试数目变化范围很大。

在这样的情况下，使用 JUnit 的测试人员往往会转而使用 FIT 这样的框架，因为这样就可以用表格数据驱动测试。但是 TestNG 提供了开箱即用的类似特性。通过在 TestNG 的 XML 配置文件中放入参数化数据，就可以对不同的数据集重用同一个测试用例，甚至有可能会得到不同的结果。这种技术完美地避免了*只能* 假定一切正常的测试，或是没有对边界进行有效验证的情况。

在清单 5 中，我用 Java 1.4 定义了一个 TestNG 测试，该测试可接收两个参数：`classname` 和 `size`。这两个参数可以验证某个类的层次结构（也就是说，如果传入 `java.util.Vector`，则 `HierarchyBuilder` 所构建的 `Hierarchy` 的值将为 `2` ）。

##### 清单 5\. 一个 TestNG 参数化测试

```
package test.com.acme.da;

import com.acme.da.hierarchy.Hierarchy;
import com.acme.da.hierarchy.HierarchyBuilder;

public class HierarchyTest {
 /**
  * @testng.test
  * @testng.parameters value="class_name, size"
  */
 public void assertValues(String classname, int size) throws Exception{
  Hierarchy hier = HierarchyBuilder.buildHierarchy(classname);
  assert hier.getHierarchyClassNames().length == size: "didn't equal!";
 }
} 
```

清单 5 列出了一个泛型测试，它可以采用不同的数据反复重用。请花点时间思考一下这个问题。如果有 10 个不同的参数组合需要在 JUnit 中测试，您只能写 10 个测试用例。每个测试用例完成的任务基本是相同的，只是受测方法的参数有所改变。但是，如果使用参数化测试，就可以只定义*一个* 测试用例，然后，（举例来说）把所需的参数模式加到 TestNG 的测试套件文件中。清单 6 中展示了这中方法：

##### 清单 6\. 一个 TestNG 参数化测试套件文件

```
<!DOCTYPE suite SYSTEM "http://beust.com/testng/testng-1.0.dtd">
<suite name="Deckt-10">
 <test name="Deckt-10-test">

  <parameter name="class_name" value="java.util.Vector"/>
  <parameter name="size" value="2"/>     

  <classes>          
   <class name="test.com.acme.da.HierarchyTest"/>
  </classes>
 </test>  
</suite> 
```

清单 6 中的 TestNG 测试套件文件只对该测试定义了一个参数组（`class_name` 为 `java.util.Vector`，且 `size` 等于 `2`），但却具有无限的可能。这样做的一个额外的好处是：将测试数据移动到 XML 文件的无代码工件就意味着非程序员也可以指定数据。

* * *

## 高级参数化测试

尽管从一个 XML 文件中抽取数据会很方便，但偶尔会有些测试需要有复杂类型，这些类型无法用 `String` 或原语值来表示。TestNG 可以通过它的 `@DataProvider` 注释处理这样的情况。`@DataProvider` 注释可以方便地把复杂参数类型映射到某个测试方法。例如，清单 7 中的 `verifyHierarchy` 测试中，我采用了重载的 `buildHierarchy` 方法，它可接收一个 `Class` 类型的数据, 它断言（asserting）`Hierarchy` 的 `getHierarchyClassNames()` 方法应该返回一个适当的字符串数组：

##### 清单 7\. TestNG 中的 DataProvider 用法

```
package test.com.acme.da.ng;

import java.util.Vector;

import static org.testng.Assert.assertEquals;
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

import com.acme.da.hierarchy.Hierarchy;
import com.acme.da.hierarchy.HierarchyBuilder;

public class HierarchyTest {

 @DataProvider(name = "class-hierarchies")
 public Object[][] dataValues(){
  return new Object[][]{
   {Vector.class, new String[] {"java.util.AbstractList", 
     "java.util.AbstractCollection"}},
   {String.class, new String[] {}}
  };
 }

 @Test(dataProvider = "class-hierarchies")
 public void verifyHierarchy(Class clzz, String[] names) 
  throws Exception{
    Hierarchy hier = HierarchyBuilder.buildHierarchy(clzz);
    assertEquals(hier.getHierarchyClassNames(), names, 
      "values were not equal");        
 }
} 
```

`dataValues()` 方法通过一个多维数组提供与 `verifyHierarchy` 测试方法的参数值匹配的数据值。TestNG 遍历这些数据值，并根据数据值调用了两次 `verifyHierarchy`。在第一次调用时，`Class` 参数被设置为 `Vector.class` ，而 `String` 数组参数容纳 “`java.util.AbstractList` ” 和 “ `java.util.AbstractCollection` ” 这两个 `String` 类型的数据。这样挺方便吧？

* * *

## 为什么只选择其一？

我已经探讨了对我而言，TestNG 的一些独有优势，但是它还有其他几个特性是 JUnit 所不具备的。例如 TestNG 中使用了测试分组，它可以根据诸如运行时间这样的特征来对测试分类。也可在 Java 1.4 中通过 javadoc 风格的注释来使用它，如 清单 5 所示。

正如我在本文开头所说，JUnit 4 和 TestNG 在表面上是相似的。然而，设计 JUnit 的目的是为了分析代码单元，而 TestNG 的预期用途则针对高级测试。对于大型测试套件，我们不希望在某一项测试失败时就得重新运行数千项测试，TestNG 的灵活性在这里尤为有用。这两个框架都有自己的优势，您可以随意同时使用它们。