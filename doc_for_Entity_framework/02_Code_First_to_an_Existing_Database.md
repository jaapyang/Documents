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

- Enter **BloggingContext** as the name and click **OK**

- This launches the **Entity Data Model Wizard**

- Select **Code First from Database** and click **Next**

![](https://i-msdn.sec.s-msft.com/dynimg/IC716048.jpeg)

- Select the connection to the database you created in the first section and click **Next**

![](https://i-msdn.sec.s-msft.com/dynimg/IC716050.jpeg)

- Click the checkbox next to **Tables** to import all tables and click **Finish**

![](https://i-msdn.sec.s-msft.com/dynimg/IC716049.jpeg)

Once the reverse engineer process completes a number of items will have been added to the project, let's take a look at what's been added.

**Configuration file**

An App.config file has been added to the project, this file contains the connection string to the existing database.

```
<connectionStrings> 
  <add  
    name="BloggingContext"  
    connectionString="data source=(localdb)\v11.0;initial catalog=Blogging;integrated security=True;MultipleActiveResultSets=True;App=EntityFramework"  
    providerName="System.Data.SqlClient" /> 
</connectionStrings>
```

You’ll notice some other settings in the configuration file too, these are default EF settings that tell Code First where to create databases. Since we are mapping to an existing database these setting will be ignored in our application.


**Derived Context**

A **BloggingContext** class has been added to the project. The context represents a session with the database, allowing us to query and save data. The context exposes a **DbSet\<TEntity\>** for each type in our model. You’ll also notice that the default constructor calls a base constructor using the **name=** syntax. This tells Code First that the connection string to use for this context should be loaded from the configuration file.

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

You should always use the **name=** syntax when you are using a connection string in the config file. This ensures that if the connection string is not present then Entity Framework will throw rather than creating a new database by convention.

**Model classes**

Finally, a **Blog** and **Post** class have also been added to the project. These are the domain classes that make up our model. You'll see Data Annotations applied to the classes to specify configuration where the Code First conventions would not align with the structure of the existing database. For example, you'll see the **StringLength** annotation on **Blog.Name** and **Blog.Url** since they have a maximum length of **200** in the database (the Code First default is to use the maximun length supported by the database provider - **nvarchar(max)** in SQL Server).

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

## 4.Reading & Writing Data

Now that we have a model it’s time to use it to access some data. Implement the **Main** method in **Program.cs** as shown below. This code creates a new instance of our context and then uses it to insert a new **Blog**. Then it uses a LINQ query to retrieve all **Blogs** from the database ordered alphabetically by **Title**.

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

You can now run the application and test it out.

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







