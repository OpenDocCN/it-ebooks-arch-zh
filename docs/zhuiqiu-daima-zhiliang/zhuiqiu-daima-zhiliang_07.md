# 追求代码质量: 可重复的系统测试

# 追求代码质量: 可重复的系统测试

*用 Cargo 进行自动容器管理*

在测试加入到 servlet 容器的 Web 应用程序时，编写符合逻辑的可重复的测试尤其需要技巧。在 Andrew Glover 的提高代码质量的这个续篇中，他介绍了 Cargo，这是一个以通用方式自动化容器管理的开源框架，有了这个框架，您可以随时编写符合逻辑的可重复的系统测试。

在本质上，像 JUnit 和 TestNG 一样的测试框架方便了可重复性测试的创建。由于这些框架利用了简单 Boolean 逻辑（以 *assert* 方法的形式）的可靠性，这使得无人为干预而运行测试成为可能。事实上，自动化是测试框架的主要优点之一 —— 我能够编写一个用于断言具体行为的相当复杂的测试，且一旦这些行为有所改变，框架就会报告一个人人都能明白的错误。

利用成熟的测试框架会带来*框架* 可重复性的优点，这是显而易见的。但*逻辑的* 可重复性却取决于您。例如，考虑创建用于验证 Web 应用程序的可重复测试的情况，一些 JUnit 扩展框架（如 JWebUnit 和 HttpUnit）在协助自动化的 Web 测试方面非常好用。但是，使测试的 *plumbing* 可重复则是开发人员的任务，而这在部署 Web 应用程序资源时很难进行。

实际的 JWebUnit 测试的构造过程相当简单，如清单 1 所示：

##### 清单 1\. 一个简单的 JWebUnit 测试

```
package test.come.acme.widget.Web;

import net.sourceforge.jwebunit.WebTester;
import junit.framework.TestCase;

public class WidgetCreationTest extends TestCase {
 private WebTester tester;

 protected void setUp() throws Exception {
  this.tester = new WebTester();
  this.tester.getTestContext().
   setBaseUrl("http://localhost:8080/widget/");    
 }

 public void testWidgetCreation() {
  this.tester.beginAt("/CreateWidget.html");        
  this.tester.setFormElement("widget-id", "893-44");
  this.tester.setFormElement("part-num", "rt45-3");

  this.tester.submit();        
  this.tester.assertTextPresent("893-44");
  this.tester.assertTextPresent("successfully created.");
 }
} 
```

这个测试与一个 Web 应用程序通信，并试图创建一个基于该交互的小部件。该测试随后校验此部件是否被成功创建。读过本系列之前部分的读者们也许会注意到该测试的一个微妙的可重复性问题。您注意到了吗？如果这个测试用例*连续* 运行两次会怎样呢？

## 改进代码质量

不要错过了 Andrew 的附随的 [讨论论坛](http://www.ibm.com/developerworks/forums/dw_forum.jsp?S_TACT=105AGX52&cat=10&S_CMP=cn-a-j&forum=812)，获取有关代码编写方法、测试框架及编写高质量代码的帮助。

由这个小部件实例（即，`widget-id`）的验证方面可以判断出，可以安全地做出这样的假设，即此应用程序中的数据库约束很可能会阻止创建一个已经存在的额外的小部件。由于缺少了一个在运行另一个测试前删除此测试用例的目标小部件的过程，如果再连续运行两次，这个测试用例非常有可能会失败。

幸运的是，如前面文章中所探讨的那样，有一个有助于数据库-依赖性（database-dependent）测试用例可重复性的机制 —— 即 DbUnit。

## 使用 DbUnit

改进 清单 1 中的测试用例来使用 DbUnit 是非常简单的。DbUnit 只需要一些插入数据库的数据和一个相应的数据库连接，如清单 2 所示：

##### 清单 2\. 用 DbUnit 进行的数据库-依赖性测试

```
package test.come.acme.widget.Web;

import java.io.File;
import java.io.IOException;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

import org.dbunit.database.DatabaseConnection;
import org.dbunit.database.IDatabaseConnection;
import org.dbunit.dataset.DataSetException;
import org.dbunit.dataset.IDataSet;
import org.dbunit.dataset.xml.FlatXmlDataSet;
import org.dbunit.operation.DatabaseOperation;

import net.sourceforge.jwebunit.WebTester;
import junit.framework.TestCase;

public class RepeatableWidgetCreationTest extends TestCase {
 private WebTester tester;

 protected void setUp() throws Exception {
  this.handleSetUpOperation();
  this.tester = new WebTester();
  this.tester.getTestContext().
   setBaseUrl("http://localhost:8080/widget/");    
 }

 public void testWidgetCreation() {
  this.tester.beginAt("/CreateWord.html");        
  this.tester.setFormElement("widget-id", "893-44");
  this.tester.setFormElement("part-num", "rt45-3");

  this.tester.submit();        
  this.tester.assertTextPresent("893-44");
  this.tester.assertTextPresent("successfully created.");
 }

 private void handleSetUpOperation() throws Exception{
  final IDatabaseConnection conn = this.getConnection();
  final IDataSet data = this.getDataSet();        
  try{
   DatabaseOperation.CLEAN_INSERT.execute(conn, data);
  }finally{
   conn.close();
  }
 }

 private IDataSet getDataSet() throws IOException, DataSetException {
  return new FlatXmlDataSet(new File("test/conf/seed.xml"));
 }

 private IDatabaseConnection getConnection() throws 
   ClassNotFoundException, SQLException {
    Class.forName("org.hsqldb.jdbcDriver");
    final Connection jdbcConnection = 
     DriverManager.getConnection("jdbc:hsqldb:hsql://127.0.0.1", 
      "sa", "");
    return new DatabaseConnection(jdbcConnection);
 }
} 
```

加入了 DbUnit，测试用例真的是可重复的了。在 `handleSetUpOperation()` 方法中，每当运行一个测试用例时，DbUnit 对数据执行一个 `CLEAN_INSERT`。此操作本质上将一个数据库的数据清空并插入一个新的数据集，从而删除任何之前创建的小部件。

## 再一次探讨什么是 DbUnit？

DbUnit 是一个 JUnit 扩展，用于在运行测试时将数据库放入一个已知状态中。开发人员使用 XML 种子文件将特定数据插入到测试用例所依赖的数据库中。因而，DbUnit 便利了依赖于一个或多个数据库的测试用例的可重复性。

但那并不意味着已经结束了对测试用例可重复性这一话题的探讨。事实上，一切才刚刚开始。

## 重复系统测试

我喜欢将 清单 1 和 清单 2 中定义的测试用例称为*系统测试*。因为系统测试运行安装完整的应用程序，如 Web 应用程序，它们通常包含一个 servlet 容器和一个相关联的数据库。这些测试的目的在于校验那些设计为端对端操作的外部接口（如 Web 应用程序中的 Web 页面）。

## 弹性优先级

作为总体规则，应在任何可能的时候避免测试用例继承。许多 JUnit 扩展框架都提供特定的可继承测试用例，以便利于测试一个特定的架构。然而由于 Java™ 平台的单一继承范例，使得从框架中继承类的测试用例饱受缺乏弹性之苦。通常，这些相同的 JUnit 扩展框架提供了代理 API，这使得联合各种不具有严格继承结构的框架变得十分简单。

由于设计它们的目的是为了测试功能完整的应用程序，因而系统测试趋向于增加运行次数而不是减少设置测试的总时间。例如，清单 1 和 清单 2 中展示的逻辑测试在运行*前* 需要下列步骤：

1.  创建一个 war 文件，该文件包含所有相关 Web 内容，如 JSP 文件、servlet、第三方的 jar 文件、图像等。
2.  将此 war 文件部署到目标 Web 容器中。（如果该容器尚未启动，启动该容器。）
3.  启动任何相关的数据库。（如果需要更新数据库模式，在启动前进行更新。）

现在，对于一个微不足道的小测试要做大量的辅助性工作！如果证明这个过程是耗时的，那么您认为这个测试会间隔多长时间运行一次呢？面对要使系统测试在逻辑上可重复（在一个连续的集成环境中）这一需求，这个步骤列表的确令人望而生畏。

* * *

## 介绍 Cargo

好消息是可以在之前的列表中使所有主要设置步骤自动化。事实上，如果恰好从事过 Java Web 开发，可能已经用 Ant、Maven 或其他构建工具使步骤 1 自动化了。

步骤 2 却是一个有趣的障碍。自动化一个 Web 容器还是需要一定技巧的。例如，一些容器具有定制的 Ant 任务，这些任务方便了其自动部署及运行，但这些任务是特定于容器的。而且，这些任务还有一些假设，如容器的安装位置，还有更重要的是，*容器*已被安装。

Cargo 是一个致力于以通用方式自动化容器管理的创新型开源项目，因而用于将 WAR 文件部署到 JBoss 的相同的 API 也能够启动及停止 Jetty。Cargo 也能自动下载并安装一个容器。可以以不同的方式利用 Cargo 的 API，从 Java 代码到 Ant 任务，再到 Maven 目标。

运用一个如 Cargo 这样的工具，应对了在编写合乎逻辑可重复的测试用例中遇到的主要问题之一。另外，还可以构造一个构建用于驾驭 Cargo 的功能以 *自动地*完成下列任务：

1.  下载一个所期望的容器。
2.  安装该容器。
3.  启动该容器。
4.  将一个选定的 WAR 或 EAR 文件部署到该容器中。

很简单，是吧？接下来，您还能够用 Cargo 停止一个选定的容器。

### “谈谈” Cargo

在深入 Cargo 前，最好先了解一下 Cargo 的基础知识。也就是说，由于 Cargo 与容器及容器管理相关，所以要理解了容器及容器管理的有关概念。

对于新手，显然要先了解*容器* 的概念。容器是用以寄存应用程序的服务器。应用程序可以是基于 Web 的，基于 EJB 的，或基于这两者的，这就是为什么有 Web 容器和 EJB 容器的原因。Tomcat 是 Web 容器，而 JBoss 则会被认为是 EJB 容器。因此，Cargo 支持相当多的容器，但在我的例子中，我将使用 Tomcat 5.0.28 版。（Cargo 将称其为“tomcat5x”容器。）

接下来，如果尚未安装容器，可以使用 Cargo 来下载并安装一个特定的容器。为此，需要提供给 Cargo 一个下载 URL。一旦安装了容器，Cargo 也会允许使用*配置选项* 来对其进行配置。这些选项以名称-值对的形式存在。

最后，要介绍*可部署资源* 的概念，在我的例子中即 WAR 文件。请注意 EAR 文件也是一样的简单。

将这些概念记住，让我们来看一下可以用 Cargo 来完成什么任务。

* * *

## Cargo 实践

本文中的例子涉及到在 Ant 中使用 Cargo，这就必需将之前定义的系统测试和 Cargo Ant 任务包装在一起。这些任务随后安装、启动、部署并停止容器。我们将首先进行安装设置，运行测试然后停止容器。

在 Ant 构建中使用 Cargo 所需的第一步是提供一个针对所有的 Cargo 任务的任务定义。这一步允许随后在构建文件中引用 Cargo 任务。应付这一步有很多的方法。清单 3 简单地装载了来自 Cargo JAR 文件中的属性文件的任务：

##### 清单 3\. 在 Ant 中装载所有的 Cargo 任务

```
<taskdef resource="cargo.tasks">
 <classpath>
  <pathelement location="${libdir}/${cargo-jar}"/>
  <pathelement location="${libdir}/${cargo-ant-jar}"/>
 </classpath>
</taskdef> 
```

一但定义了 Cargo 的任务，真正的行动就开始了。清单 4 定义了下载、安装及启动 Tomcat 容器的 Cargo 任务。`zipurlinstaller` 任务将 Tomcat 从 `http://www.apache.org/dist/tomcat/tomcat-5/v5.0.28/bin/ jakarta-tomcat-5.0.28.zip` 中下载并安装到一个本地临时目录中。

##### 清单 4\. 下载并启动 Tomcat 5.0.28

```
<cargo containerId="tomcat5x" action="start" 
       wait="false" id="${tomcat-refid}">

 <zipurlinstaller installurl="${tomcat-installer-url}"/>

 <configuration type="standalone" home="${tomcatdir}">
  <property name="cargo.remote.username" value="admin"/>
  <property name="cargo.remote.password" value=""/>

  <deployable type="war" file="${wardir}/${warfile}"/>

 </configuration>        

</cargo> 
```

请注意要想如您所愿，从不同的任务中启动和停止一个容器，必需将容器同一个惟一的 id 联系起来，此 id 是 `cargo` 任务的 `id="${tomcat-refid}"`。

还要注意的是，Tomcat 的配置是在 `cargo` 任务内处理的。在 Tomcat 中，必需设置 `username` 和 `password` 属性。最后，使用 `deployable` 元素定义一个指向 WAR 文件的指针。

### Cargo 属性

Cargo 任务中用到的所有属性都显示在清单 5 中。例如，`tomcatdir` 定义 Tomcat 将安装的两个位置中的一个。这个特别的位置是一个镜像结构，该位置将被实际下载并安装的 Tomcat 实例（在临时目录中找到的）所引用。`tomcat-refid` 属性则帮助将容器中惟一的实例与其镜像关联起来。

##### 清单 5\. Cargo 属性

```
<property name="tomcat-installer-url" 
  value="http://www.apache.org/dist/tomcat/tomcat-5/v5.0.28/bin/
    jakarta-tomcat-5.0.28.zip"/>
<property name="tomcatdir" value="target/tomcat"/>    
<property name="tomcat.username" value="admin"/>    
<property name="tomcat.passwrd" value=""/>    
<property name="wardir" value="target/war"/>
<property name="warfile" value="words.war"/>    
<property name="tomcat-refid" value="tmptmct01"/> 
```

为停止一个容器，可以定义一个引用 `tomcat-refid` 属性的任务，如清单 6 所示。

##### 清单 6\. 按 Cargo 方式停止容器

```
<cargo containerId="tomcat5x" action="stop" 
       refid="${tomcat-refid}"/> 
```

### 用 Cargo 封装

清单 7 将 清单 4 和清单 6 中的代码联合起来，用两个 Cargo 任务封装了一个测试目标：一个用于启动 Tomcat，另一个用于停止 Tomcat。`antcall` 任务调用在清单 8 中定义的名为 `_run-system-tests` 的目标。

##### 清单 7\. 用 Cargo 封装测试目标

```
<target name="system-test" if="Junit.present" 
        depends="init,junit-present,compile-tests,war">

 <cargo containerId="tomcat5x" action="start" 
       wait="false" id="${tomcat-refid}">            
  <zipurlinstaller installurl="${tomcat-installer-url}"/>
  <configuration type="standalone" home="${tomcatdir}">
   <property name="cargo.remote.username" value="admin"/>
   <property name="cargo.remote.password" value=""/>
   <deployable type="war" file="${wardir}/${warfile}"/>
  </configuration>
 </cargo>

 <antcall target="_run-system-tests"/>

 <cargo containerId="tomcat5x" action="stop" 
       refid="${tomcat-refid}"/>            

</target> 
```

清单 8 定义测试目标，称作 `_run-system-tests`。请注意此任务*只* 运行置于 `test/system` 目录下的系统测试。例如，清单 2 中定义的测试用例就位于这个目录下。

##### 清单 8\. 通过 Ant 运行 JUnit

```
<target name="_run-system-tests">
 <mkdir dir="${testreportdir}"/>   
 <junit dir="./" failureproperty="test.failure" 
        printSummary="yes" fork="true" 
        haltonerror="true">
  <sysproperty key="basedir" value="."/>     
  <formatter type="xml"/>      
  <formatter usefile="false" type="plain"/>     
  <classpath>
   <path refid="build.classpath"/>       
   <pathelement path="${testclassesdir}"/>
   <pathelement path="${classesdir}"/>      
  </classpath>
  <batchtest todir="${testreportdir}">
   <fileset dir="test/system">
    <include name="**/**Test.java"/> 
   </fileset>
  </batchtest>
 </junit>
</target> 
```

在 清单 7 中，完整地配置了 Ant 构建文件，从而将系统测试与 Cargo 部署封装在一起。清单 7 中的代码确保了清单 8 中 `test/system` 目录下的所有系统测试都是逻辑上可重复的。可以在任何时间里在任何机器上运行这些系统测试，对于连续集成环境尤佳。该测试对容器未做任何假设 —— 未对位置做假设，甚至未对其是否运行做假设！（当然，这些测试仍做了一个假设，我没有强调，即潜在的数据库是配置良好且在运行中的。但那又是另一个要讨论的主题了。）

* * *

## 可重复的结果

在清单 9 中，可以看到工作的成果。当将 `system-test` 命令发布到 Ant 构建后，就会执行系统测试。Cargo 处理管理所选容器的所有细节，不需要对测试环境作出绝对重复性假设。

##### 清单 9\. 增强的构建

```
war:
 [war] Building war: C:\dev\projects\acme\target\widget.war

system-test:

_run-system-tests:
 [mkdir] Created dir: C:\dev\projects\acme\target\test-reports
 [junit] Running test.come.acme.widget.Web.RepeatableWordCreationTest
 [junit] Tests run: 1, Failures: 0, Errors: 0, Time elapsed: 4.53 sec
 [junit] Testcase: testWordCreation took 4.436 sec

BUILD SUCCESSFUL
Total time: 1 minute 2 seconds 
```

请记住，Cargo 也在 Maven 构建中起作用。另外，从正常的应用程序到测试用例，Cargo Java API 都有助于容器的程序化管理。且 Cargo 不仅适用于 JUnit（尽管样例代码是用 JUnit 写的），TestNG 用户将会很高兴地了解到 Cargo 对其测试套件也起作用。事实上，测试用什么编写并不重要，重要的是将它们同 Cargo 封装起来，容器管理问题就会迎刃而解！

* * *

## 结束语

您的测试是否在逻辑上可重复由您来决定，但是通过本文您确实看到 Cargo 的确很有用处。Cargo 管理容器环境，所以您就可以不用管理。将 Cargo 包含到您的测试例程中 —— 这毫无疑问会减轻您构造用于验证 Web 应用程序的可重复测试的负担。