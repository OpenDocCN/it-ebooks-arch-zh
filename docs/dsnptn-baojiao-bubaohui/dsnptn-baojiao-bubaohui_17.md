# 抽象工厂模式

# 抽象工厂模式(Abstract factory Pattern)

## 简介

抽象工厂模式是一种软件开发设计模式。抽象工厂模式提供了一种方式，可以将一组具有同一主题的单独的工厂封装起来。在正常使用中，客户端程序需要创建抽象工厂的具体实现，然后使用抽象工厂作为接口来创建这一主题的具体对象。客户端程序不需要知道（或关心）它从这些内部的工厂方法中获得对象的具体类型，因为客户端程序仅使用这些对象的通用接口。抽象工厂模式将一组对象的实现细节与他们的一般使用分离开来。

## 简例

有个项目原来是依赖于 access 数据库，现在要用 sqlserver，怎么更改呢(解耦)？

### 用工厂方法模式的数据访问程序

IUser 接口，用于客户端访问，解除与具体数据库访问的耦合。

```
interface IUser
{
    void Insert(User user);

    User GetUser(int id);
} 
```

Sqlserver 类，用于访问 SQL Server 的 User。

```
class SqlserverUser : IUser
{
    public void Insert(User user)
    {
        Console.WriteLine("在 SQL Server 中给 User 表增加一条记录");
    }

    public User GetUser(int id)
    {
        Console.WriteLine("在 SQL Server 中给 User 表获取一条记录");
        return null;
    }

} 
```

AccessUser 类，用于访问 Access 的 User。

```
class AccessUser : IUser
{
    public void Insert(User user)
    {
        Console.WriteLine("在 access 中给 User 表增加一条记录");
    }

    public User GetUser(int id)
    {
        Console.WriteLine("在 access 中给 User 表获取一条记录");
        return null;
    }

} 
```

IFactory 接口，定义一个创建访问 User 表对象的抽象的工厂接口。

```
interface Ifactory
{
    IUser CreateUser();
} 
```

sqlServerFactory 类，实现 IFactory 接口，实例化 SqlserverUser。

```
class sqlServerFactory:  IFactory
{
    public IUser CreateUser()
    {
        return new SqlserverUser();
    }
} 
```

AccessFactory 类，实现 IFactory 接口，实例化 AccessUser.

```
class AccessFactory:  IFactory
{
    public IUser CreateUser()
    {
        return new AccessUser();
    }
} 
```

客户端代码

```
static void main(string[] args)
{
    User user = new User();

    IFactory factory = new SqlServerFactory();

    IUser iu = factory.CreateUser();

    iu.Insert(user);
    iu.GetUser(1);
    Console.Read();
} 
```

但是数据里面不可能只有一个 User 表，比如说增加部门表，此时怎么办呢。

### 抽象工厂类

IUser 接口，用于客户端访问，解除与具体数据库访问的耦合。

```
interface IDepartment
{
    void Insert(Department department);
    Department GetDepartment(int id);
} 
```

SqlserverDepartment 类，用于访问 SQL Server 的 Department。

```
class SqlserverDepartment : IDepartment
{
    public void Insert(Department department)
    {
        Console.WriteLine("在 SQL Server 中给 Department 表增加一条记录");
    }

    public Department GetDepartment(int id)
    {
        Console.WriteLine("在 SQL Server 中给 Department 表获取一条记录");
        return null;
    }

} 
```

AccessDepartment 类，用于访问 Access 的 Department。

```
class AccessDepartment : IDepartment
{
    public void Insert(Department department)
    {
        Console.WriteLine("在 Access 中给 Department 表增加一条记录");
    }

    public Department GetDepartment(int id)
    {
        Console.WriteLine("在 Access 中给 Department 表获取一条记录");
        return null;
    }

} 
```

* * *

```
interface IUser
{
    void Insert(User user);

    User GetUser(int id);
} 
```

Sqlserver 类，用于访问 SQL Server 的 User。

```
class SqlserverUser : IUser
{
    public void Insert(User user)
    {
        Console.WriteLine("在 SQL Server 中给 User 表增加一条记录");
    }

    public User GetUser(int id)
    {
        Console.WriteLine("在 SQL Server 中给 User 表获取一条记录");
        return null;
    }

} 
```

AccessUser 类，用于访问 Access 的 User。

```
class AccessUser : IUser
{
    public void Insert(User user)
    {
        Console.WriteLine("在 access 中给 User 表增加一条记录");
    }

    public User GetUser(int id)
    {
        Console.WriteLine("在 access 中给 User 表获取一条记录");
        return null;
    }

} 
```

IFactory 接口，定义一个创建访问 User 表对象的抽象的工厂接口。

```
interface Ifactory
{
    IUser CreateUser();

    IDepartment CreateDepartment();
} 
```

sqlServerFactory 类，实现 IFactory 接口，实例化 SqlserverUser。

```
class sqlServerFactory:  IFactory
{
    public IUser CreateUser()
    {
        return new SqlserverUser();
    }

    public IDepartment CreateDepartment()
    {
        return new SqlserverDepartment();
    }

} 
```

AccessFactory 类，实现 IFactory 接口，实例化 AccessUser.

```
class AccessFactory:  IFactory
{
    public IUser CreateUser()
    {
        return new AccessUser();
    }

    public IDepartment CreateDepartment()
    {
        return new AccessDepartment();
    }
} 
```

客户端代码

```
static void main(string[] args)
{
    User user = new User();

    IFactory factory = new SqlServerFactory();

    IUser iu = factory.CreateUser();

    iu.Insert(user);
    iu.GetUser(1);
    Console.Read();
} 
```

抽象工厂模式：提供一个创建一系列相关或者相互依赖的对象的接口，而无需指定它们具体的类。

### 用简单工厂来改进抽象工厂

```
class DataAccess
{
    private static readonly string db = "Sqlserver";
    //    private static readonly string db = "access";

    public static IUser CreateUser()
    {
        IUser result = null;

        switch (db)
        {
            case:"Sqlserver":
                result = new SqlserverUser();
                break;
            case:"Acesss":
                result = new AcesssUser();
                break;
        }
        return result;
    }

    publci static IDepartment CreateDepartment()
    {
        IDepartment result = null;
                switch (db)
        {
            case:"Sqlserver":
                result = new SqlserverDepartment();
                break;
            case:"Acesss":
                result = new AcesssDepartment();
                break;
        }
        return result;
    }

} 
```

客户端代码

```
static void Main(string[] args)
{
    User user = new User();
    Department dept = new Department();

    IUser iu = DataAccess.createUser();
    iu.Insert(user);
    iu.GetUser(1);

    IDepartment id = DataAccess.createDepartment();
    id.Inert(dept);
    id.GetDepartment(1);

    Console.Read();
} 
```

利用反射或者配置文件都可以减少在所有简单工厂的 switch 和 if，解除分支判断带来的耦合。

“工厂”是创建产品（对象）的地方，其目的是将产品的创建与产品的使用分离。抽象工厂模式的目的，是将若干抽象产品的接口与不同主题产品的具体实现分离开。这样就能在增加新的具体工厂的时候，不用修改引用抽象工厂的客户端代码。

使用抽象工厂模式，能够在具体工厂变化的时候，不用修改使用工厂的客户端代码，甚至是在运行时。然而，使用这种模式或者相似的设计模式，可能给编写代码带来不必要的复杂性和额外的工作。正确使用设计模式能够抵消这样的“额外工作”。

## 从 NSArray 看类簇

Class Clusters（类簇）是抽象工厂模式在 iOS 下的一种实现，众多常用类，如 NSString，NSArray，NSDictionary，NSNumber 都运作在这一模式下，它是接口简单性和扩展性的权衡体现，在我们完全不知情的情况下，偷偷隐藏了很多具体的实现类，只暴露出简单的接口。

虽然官方文档中拿 NSNumber 说事儿，但 Foundation 并没有像图中描述的那样为每个 number 都弄一个子类，于是研究下 NSArray 类簇的实现方式。

### __NSPlacehodlerArray

熟悉这个模式的同学很可能看过下面的测试代码，将原有的 alloc+init 拆开写：

```
id obj1 = [NSArray alloc]; // __NSPlacehodlerArray *
id obj2 = [NSMutableArray alloc];  // __NSPlacehodlerArray *
id obj3 = [obj1 init];  // __NSArrayI *
id obj4 = [obj2 init];  // __NSArrayM * 
```

发现+ alloc 后并非生成了我们期望的类实例，而是一个**NSPlacehodlerArray 的中间对象，后面的- init 或- initWithXXXXX 消息都是发送给这个中间对象，再由它做工厂，生成真的对象。这里的**NSArrayI 和 __NSArrayM 分别对应 Immutable 和 Mutable（后面的 I 和 M 的意思）

于是顺着思路猜实现，__NSPlacehodlerArray 必定用某种方式存储了它是由谁 alloc 出来的这个信息，才能在 init 的时候知道要创建的是可变数组还是不可变数组

经过研究发现，Foundation 用了一个很贱的比较静态实例地址方式来实现，伪代码如下：

```
static __NSPlacehodlerArray *GetPlaceholderForNSArray() {
    static __NSPlacehodlerArray *instanceForNSArray;
    if (!instanceForNSArray) {
        instanceForNSArray = [[__NSPlacehodlerArray alloc] init];
    }
    return instanceForNSArray;
}

static __NSPlacehodlerArray *GetPlaceholderForNSMutableArray() {
    static __NSPlacehodlerArray *instanceForNSMutableArray;
    if (!instanceForNSMutableArray) {
        instanceForNSMutableArray = [[__NSPlacehodlerArray alloc] init];
    }
    return instanceForNSMutableArray;
}
// NSArray 实现
+ (id)alloc {
    if (self == [NSArray class]) {
        return GetPlaceholderForNSArray()
    }
}
// NSMutableArray 实现
+ (id)alloc {
    if (self == [NSMutableArray class]) {
        return GetPlaceholderForNSMutableArray()
    }
}
// __NSPlacehodlerArray 实现
- (id)init {
    if (self == GetPlaceholderForNSArray()) {
        self = [[__NSArrayI alloc] init];
    }
    else if (self == GetPlaceholderForNSMutableArray()) {
        self = [[__NSArrayM alloc] init];
    }
    return self;
} 
```

Foundation 不是开源的，所以上面的代码是猜测的，思路大概就是这样，可以这样验证下：

```
id obj1 = [NSArray alloc];
id obj2 = [NSArray alloc];
id obj3 = [NSMutableArray alloc];
id obj4 = [NSMutableArray alloc];
// 1 和 2 地址相同，3 和 4 地址相同，无论多少次都相同，且地址相差 16 位 
```