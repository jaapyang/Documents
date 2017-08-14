# EF对于已有数据库的Code First支持

> [原文链接](https://msdn.microsoft.com/en-us/library/jj200620(v=vs.113).aspx)

> 本文将逐步介绍怎样用Code First的方式基于已有数据库进行开发。Code First支持你使用C#或者VB.Net定义类.并使用数据模型标识和Fluent API定义与配置模型。

## 前提

已经安装  **Visual Studio 2012** 或者 **Visual Studio 2013**

同时，你还需要安装**Entity Framework Tools for Visual Studio**的**6.1**或更高版本。请参考[Get Entity Framework](https://msdn.microsoft.com/en-us/library/ee712906(v=vs.113).aspx)


## 1. 创建一个数据库

因为本文是研究基于已有数据库进行Code First,所以，我们先创建一个数据库作为我们的操作目标。(至于创建数据库的过程，可以去 [*原文链接*](https://msdn.microsoft.com/en-us/library/jj200620(v=vs.113).aspx) 看图操作，这部分内容我就不翻译了)

Let's go ahead and generate the database.

- Open Visual Studio

- **View -> Server Explorer**

- Right click on **Data Connections -> Add Connection…**

- If you haven’t connected to a database from **Server Explorer** before you’ll need to select **Microsoft SQL Server** as the data source

![](https://i-msdn.sec.s-msft.com/dynimg/IC716053.jpeg)

- Connect to your LocalDb instance (**(localdb)\v11.0**), and enter **Blogging** as the database name

![](https://i-msdn.sec.s-msft.com/dynimg/IC716052.jpeg)

- Select **OK** and you will be asked if you want to create a new database, select **Yes**

![](https://i-msdn.sec.s-msft.com/dynimg/IC716051.jpeg)

- The new database will now appear in Server Explorer, right-click on it and select New Query

- 复制以下代码，到**数据库管理器**中执行，以生成新的数据表

```
CREATE TABLE [dbo].[Blogs] ( 
    [BlogId] INT IDENTITY (1, 1) NOT NULL, 
    [Name] NVARCHAR (200) NULL, 
    [Url]  NVARCHAR (200) NULL, 
    CONSTRAINT [PK_dbo.Blogs] PRIMARY KEY CLUSTERED ([BlogId] ASC) 
); 
 
CREATE TABLE [dbo].[Posts] ( 
    [PostId] INT IDENTITY (1, 1) NOT NULL, 
    [Title] NVARCHAR (200) NULL, 
    [Content] NTEXT NULL, 
    [BlogId] INT NOT NULL, 
    CONSTRAINT [PK_dbo.Posts] PRIMARY KEY CLUSTERED ([PostId] ASC), 
    CONSTRAINT [FK_dbo.Posts_dbo.Blogs_BlogId] FOREIGN KEY ([BlogId]) REFERENCES [dbo].[Blogs] ([BlogId]) ON DELETE CASCADE 
); 
 
INSERT INTO [dbo].[Blogs] ([Name],[Url]) 
VALUES ('The Visual Studio Blog', 'http://blogs.msdn.com/visualstudio/') 
 
INSERT INTO [dbo].[Blogs] ([Name],[Url]) 
VALUES ('.NET Framework Blog', 'http://blogs.msdn.com/dotnet/')
```

## 2.创建应用程序

依然基于简单原则，我们还是创建一个控制台应用程序，并用Code First进行数据库操作：

- 打开 Visual Studio
- **文件 -> 新建 -> 项目…**
- 选择**控制台应用程序**
- 输入 **CodeFirstExistingDatabaseSample** 作为项目名
- 点击**确定**

## 3.逆向工程

我们将使用Entity framework Tools for Visual Studio来生成一些基于数据库的初始化代码。当然，你也可以手动创建这些代码。

- **项目 -> 添加新项…**

- 从左侧菜单的**数据**(**Data**)分类中选择**Ado.net 实体模型**(**Ado.net Entity Data Model**)(*目前我的电脑上没装中文版，在菜单中具体显示的选项是什么 可能会不太精确，这里我只能是把大概意思译一下，请见谅，谢谢。*)

- 将新文件命名为 **BloggingContext** 并点击**确定**

- 这时会启动**实体模型创建向导**

- 选择**从数据库创建模型**(Code First from DataBase) 并点击**下一步**

![](https://i-msdn.sec.s-msft.com/dynimg/IC716048.jpeg)

- 创建连接并点击**下一步**

![](https://i-msdn.sec.s-msft.com/dynimg/IC716050.jpeg)

- 选择要导入的数据表并点击**完成**

![](https://i-msdn.sec.s-msft.com/dynimg/IC716049.jpeg)

当逆向工程完成后，项目中会自动生成一些文件，现在让我们看看到底添加了一些什么内容吧。

**配置文件**

在项目根目录下添加了一个```App.config```的文件，这个文件中包含了一个有关目标数据库的连接字符串.

```
<connectionStrings> 
  <add  
    name="BloggingContext"  
    connectionString="data source=(localdb)\v11.0;initial catalog=Blogging;integrated security=True;MultipleActiveResultSets=True;App=EntityFramework"  
    providerName="System.Data.SqlClient" /> 
</connectionStrings>
```

你还会注意到配置文件中的其他一些设置，这些是默认的EF设置，它告诉Code First在哪里创建数据库。 由于我们正在映射到现有数据库，因此我们的应用程序将忽略这些设置。


**衍生(派生)DbContext**

在项目目录下添加了一个名为**BloggingContext**的文件。这是与数据交互的上下文，通过它我们可以从数据库中查询数据或保存数据到数据库。同时，**BloggingContext**为每一个表公开了一个**DbSet\<TEntity\>**属性。 您还会注意到，默认构造函数使用** name = syntax**语法调用基础构造函数。 这是告诉Code First，应该从配置文件基于连接字符串名称加载用于此上下文的连接字符串。

```
public partial class BloggingContext : DbContext 
{ 
    public BloggingContext() 
        : base("name=BloggingContext") 
    { 
    } 

    public virtual DbSet<Blog> Blogs { get; set; } 
    public virtual DbSet<Post> Posts { get; set; } 

    protected override void OnModelCreating(DbModelBuilder modelBuilder) 
    { 
    } 
}
```

当您在配置文件中使用连接字符串时，应始终使用**name = syntax**。 这样可以确保如果连接字符串不存在，那么Entity Framework将抛出异常而不是按照惯例创建一个新的数据库。

**实体模型类**

最后，一个**Blog**和**Post**类也被添加到项目中。 这些是构成我们模型的实体类。 你将看到应用于类的数据模型标识配置，其中**Code First**约定与现有数据库的结构不一致。 例如，你在**Blog.Name**和**Blog.Url**上看到**StringLength**特性，因为它们在数据库中的最大长度为**200**（Code First 默认是使用数据库提供程序支持的最大长度 - **nvarchar(max)**）(你可以尝试一下用Code First创建一个新的数据表，其中的string类型字段不用**StringLength**或**MaxLength**进行标识，你会发现，在数据库中自动生成的这个字段的类型就是**nvarchar(max)**)。


```
public partial class Blog 
{ 
    public Blog() 
    { 
        Posts = new HashSet<Post>(); 
    } 
 
    public int BlogId { get; set; } 
 
    [StringLength(200)] 
    public string Name { get; set; } 
 
    [StringLength(200)] 
    public string Url { get; set; } 
 
    public virtual ICollection<Post> Posts { get; set; } 
}
```

## 4.读写数据

现在，数据模型的逆向工程完成了，是时候与数据库交互进行一些数据访问了。在**Main**函数中实现如下代码:

> 这段代码的意思是创建一个新的 DbContext的对象实例，再用它去插入一个新的Blog对象到数据库.再用Linq to Sql 从数据库中查询所有的Blogs，并按 Name 属性排序。 在此过程中你完全感觉不到你在操作数据库，是的，你就是在操作面向对象语言，EF帮你实现了与数据库的所有交互

```
class Program 
{ 
    static void Main(string[] args) 
    { 
        using (var db = new BloggingContext()) 
        { 
            // Create and save a new Blog 
            Console.Write("Enter a name for a new Blog: "); 
            var name = Console.ReadLine(); 
 
            var blog = new Blog { Name = name }; 
            db.Blogs.Add(blog); 
            db.SaveChanges(); 
 
            // Display all Blogs from the database 
            var query = from b in db.Blogs 
                        orderby b.Name 
                        select b; 
 
            Console.WriteLine("All blogs in the database:"); 
            foreach (var item in query) 
            { 
                Console.WriteLine(item.Name); 
            } 
 
            Console.WriteLine("Press any key to exit..."); 
            Console.ReadKey(); 
        } 
    } 
}
```

现在请运行这个程序，并查看运行结果，结果如下：

```
Enter a name for a new Blog: ADO.NET Blog
All blogs in the database:
.NET Framework Blog
ADO.NET Blog
The Visual Studio Blog
Press any key to exit...
```

## Customizing the Scaffolded Code

For information on customizing the code that is generated by the wizard, see [Customizing Code First to an Existing Database](https://msdn.microsoft.com/en-us/library/dn753860(v=vs.113).aspx).

## What if My Database Changes?

The Code First to Database wizard is designed to generate a starting point set of classes that you can then tweak and modify. If your database schema changes you can either manually edit the classes or perform another reverse engineer to overwrite the classes.

## Using Code First Migrations with an Existing Database

If you want to use Code First Migrations with your existing database, see [Code First Migrations with an existing database](https://msdn.microsoft.com/en-us/library/dn579398(v=vs.113).aspx).

## Summary

In this walkthrough we looked at Code First development using an existing database. We used the Entity Framework Tools for Visual Studio to reverse engineer a set of classes that mapped to the database and could be used to store and retrieve data.







