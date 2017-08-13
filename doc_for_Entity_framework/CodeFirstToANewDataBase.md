# 在Entity Framework 中用 Code First 创建新的数据库 （[原文链接](https://msdn.microsoft.com/en-us/library/jj193542(v=vs.113).aspx)）

>   本文将逐步介绍怎样用Code First 创建新数据库,使用在代码中定义类和API中提供的特性(Attribute)的方式来流畅地创建数据模型到新数据库或已经存在的空数据库。
		
## 1. 新建应用程序()

为了简单起见，我们将新建一个使用Code First执行数据访问的控制台应用程序。
- 打开VS
- 新建项目为 **控制台应用程序**

	
## 2. 新建数据模型

我们先新建两个简单的实体类. 参考代码如下:
	
	```
	public class Blog 
	{ 
		public int BlogId { get; set; } 
		public string Name { get; set; } 
	 
		public virtual List<Post- Posts { get; set; } 
	} 
	 
	public class Post 
	{ 
		public int PostId { get; set; } 
		public string Title { get; set; } 
		public string Content { get; set; } 
	 
		public int BlogId { get; set; } 
		public virtual Blog Blog { get; set; } 
	}
	```
	
您会注意到，我们将两个导航属性（Blog.Posts和Post.Blog）定义为虚属性(关键字```virtual```)。 这种定义实现了EF框架的延迟加载功能。
延迟加载的意思是当你尝试访问这些属性时，这些属性的内容将自动从数据库加载。

## 3. 创建一个 DbContext

现在定义一个新的数据库上下文(DbContext),它继承自System.Data.Entity.DbContext，并为我们的模型中的每个类在这个DbContext中定义一个强类型的DbSet <TEntity>,比如: ```public DbSet<Post> Posts{get;set;}```。
DbContext代表与数据库的会话，我们可以通过DbContext查询和保存数据。

在此之前，我们要通过 NuGet 安装 EntityFramework.
	
- 在**项目**中右键选择NuGet包管理器。点击链接可以下载最新版的Nuget管理器 [最新版本的NuGet](http://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c).
- 选择**联机**标签
- 搜索 **EntityFramework** 
- 点击 **安装**（**Install**）

引用命名空间：
	
```
using System.Data.Entity;
```
	
在你新定义的 **BloggingContext** 中定义强类型DbSet，代码如下：

```
public class BloggingContext : DbContext 
{ 
	public DbSet<Blog> Blogs { get; set; } 
	public DbSet<Post> Posts { get; set; } 
}
```

下文是完整的代码.

```
using System; 
using System.Collections.Generic; 
using System.Linq; 
using System.Text; 
using System.Threading.Tasks; 
using System.Data.Entity; 
 
namespace CodeFirstNewDatabaseSample 
{ 
	class Program 
	{ 
		static void Main(string[] args) 
		{ 
		} 
	} 
 
	public class Blog 
	{ 
		public int BlogId { get; set; } 
		public string Name { get; set; } 
 
		public virtual List<Post- Posts { get; set; } 
	} 
 
	public class Post 
	{ 
		public int PostId { get; set; } 
		public string Title { get; set; } 
		public string Content { get; set; } 
 
		public int BlogId { get; set; } 
		public virtual Blog Blog { get; set; } 
	} 
 
	public class BloggingContext : DbContext 
	{ 
		public DbSet<Blog- Blogs { get; set; } 
		public DbSet<Post- Posts { get; set; } 
	} 
}
```

这就是开始存储和检索数据所需的所有代码。 显然，幕后还有很多事情，这个我们稍后再来看看，首先让我们继续往下看。

## 4. 读取 & 写入数据

在 Program.cs 中实现如下文中的 Main 方法。
创建一个新的 DbContext的对象实例，再用它去插入一个新的Blog对象到数据库.再用Linq to Sql 从数据库中查询所有的Blogs，并按 Name 属性排序。
在此过程中你完全感觉不到你在操作数据库，是的，你就是在操作面向对象语言，EF帮你实现了与数据库的所有交互。


```
class Program 
{ 
	static void Main(string[] args) 
	{ 
		using (var db = new BloggingContext()) 
		{ 
			// 创建一个新的需要保存到数据库的Blog对象 
			Console.Write("Enter a name for a new Blog: "); 
			var name = Console.ReadLine(); 
 
			var blog = new Blog { Name = name }; 
			db.Blogs.Add(blog); 
			db.SaveChanges(); 
 
			// 从数据库中查询所有的Blogs，并输出到控制台 
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

现在你可以运行程序并查看它的输出了。输出结果如下：

```
Enter a name for a new Blog: ADO.NET Blog
All blogs in the database:
ADO.NET Blog
Press any key to exit...
```

**好神奇，我的数据到底存储在哪里呢?**

其实，DbContext帮你默认创建了一个数据库.

- 在你安装VS2010及以上版本的时候，就会默认安装一个 Local Sql Express的数据库实例(它的连接方式：```.\sqlexpress```)，默认情况下(没有指定DbConnectionString的情况下)Code First就是将数据库创建在这里(你可以在配置文件中指定ConnectionString,来控制Code First将数据库创建在哪里)

- 当默认的 SQL Express 不能用或访问不到时，Code First就会尝试使用另一个实例```LocalDb```(它默认会在安装VS2012及以上版本时安装)

- 在默认情况下，Code First会按照你定义的DbContext的完整限定名来创建数据库，如 **CodeFirstNewDatabaseSample.BloggingContext**

有关更详细情况你可以查看这篇文章 [**How DbContext Discovers the Model and Database Connection**](https://msdn.microsoft.com/en-us/library/jj592674(v=vs.113).aspx)

## 5. Dealing with Model Changes

Now it’s time to make some changes to our model, when we make these changes we also need to update the database schema. To do this we are going to use a feature called Code First Migrations, or Migrations for short.

Migrations allows us to have an ordered set of steps that describe how to upgrade (and downgrade) our database schema. Each of these steps, known as a migration, contains some code that describes the changes to be applied.

The first step is to enable Code First Migrations for our BloggingContext.

	- Tools -- Library Package Manager -- Package Manager Console
	- Run the Enable-Migrations command in Package Manager Console
	- A new Migrations folder has been added to our project that contains two items:
		- Configuration.cs – This file contains the settings that Migrations will use for migrating BloggingContext. We don’t need to change anything for this walkthrough, but here is where you can specify seed data, register providers for other databases, changes the namespace that migrations are generated in etc.
		- <timestamp-_InitialCreate.cs – This is your first migration, it represents the changes that have already been applied to the database to take it from being an empty database to one that includes the Blogs and Posts tables. Although we let Code First automatically create these tables for us, now that we have opted in to Migrations they have been converted into a Migration. Code First has also recorded in our local database that this Migration has already been applied. The timestamp on the filename is used for ordering purposes.
	
	  Now let’s make a change to our model, add a Url property to the Blog class:

```
public class Blog 
{ 
	public int BlogId { get; set; } 
	public string Name { get; set; } 
	public string Url { get; set; } 
 
	public virtual List<Post- Posts { get; set; } 
}
```

	- Run the Add-Migration AddUrl command in Package Manager Console. The Add-Migration command checks for changes since your last migration and scaffolds a new migration with any changes that are found. We can give migrations a name; in this case we are calling the migration ‘AddUrl’. The scaffolded code is saying that we need to add a Url column, that can hold string data, to the dbo.Blogs table. If needed, we could edit the scaffolded code but that’s not required in this case.

```
namespace CodeFirstNewDatabaseSample.Migrations 
{ 
	using System; 
	using System.Data.Entity.Migrations; 
	 
	public partial class AddUrl : DbMigration 
	{ 
		public override void Up() 
		{ 
			AddColumn("dbo.Blogs", "Url", c =- c.String()); 
		} 
		 
		public override void Down() 
		{ 
			DropColumn("dbo.Blogs", "Url"); 
		} 
	} 
}
```

	- Run the Update-Database command in Package Manager Console. This command will apply any pending migrations to the database. Our InitialCreate migration has already been applied so migrations will just apply our new AddUrl migration. Tip: You can use the –Verbose switch when calling Update-Database to see the SQL that is being executed against the database.
	
## 6. Data Annotations

So far we’ve just let EF discover the model using its default conventions, but there are going to be times when our classes don’t follow the conventions and we need to be able to perform further configuration. There are two options for this; we’ll look at Data Annotations in this section and then the fluent API in the next section.

- Let’s add a User class to our model

```
public class User 
{ 
	public string Username { get; set; } 
	public string DisplayName { get; set; } 
}
```

- We also need to add a set to our derived context

```
public class BloggingContext : DbContext 
{ 
	public DbSet<Blog- Blogs { get; set; } 
	public DbSet<Post- Posts { get; set; } 
	public DbSet<User- Users { get; set; } 
}
```

- If we tried to add a migration we’d get an error saying “EntityType ‘User’ has no key defined. Define the key for this EntityType.” because EF has no way of knowing that Username should be the primary key for User.
- We’re going to use Data Annotations now so we need to add a using statement at the top of Program.cs

```
using System.ComponentModel.DataAnnotations;
```

- Now annotate the Username property to identify that it is the primary key

```
public class User 
{ 
	[Key] 
	public string Username { get; set; } 
	public string DisplayName { get; set; } 
}
```

- Use the Add-Migration AddUser command to scaffold a migration to apply these changes to the database
- Run the Update-Database command to apply the new migration to the database

The full list of annotations supported by EF is:

+ KeyAttribute
+ StringLengthAttribute
+ MaxLengthAttribute
+ ConcurrencyCheckAttribute
+ RequiredAttribute
+ TimestampAttribute
+ ComplexTypeAttribute
+ ColumnAttribute
+ TableAttribute
+ InversePropertyAttribute
+ ForeignKeyAttribute
+ DatabaseGeneratedAttribute
+ NotMappedAttribute

## 7. Fluent API

In the previous section we looked at using Data Annotations to supplement or override what was detected by convention. The other way to configure the model is via the Code First fluent API.

Most model configuration can be done using simple data annotations. The fluent API is a more advanced way of specifying model configuration that covers everything that data annotations can do in addition to some more advanced configuration not possible with data annotations. Data annotations and the fluent API can be used together.

To access the fluent API you override the OnModelCreating method in DbContext. Let’s say we wanted to rename the column that User.DisplayName is stored in to display_name.

- Override the OnModelCreating method on BloggingContext with the following code

```
public class BloggingContext : DbContext 
{ 
	public DbSet<Blog- Blogs { get; set; } 
	public DbSet<Post- Posts { get; set; } 
	public DbSet<User- Users { get; set; } 
 
	protected override void OnModelCreating(DbModelBuilder modelBuilder) 
	{ 
		modelBuilder.Entity<User-() 
			.Property(u =- u.DisplayName) 
			.HasColumnName("display_name"); 
	} 
}
```

- Use the Add-Migration ChangeDisplayName command to scaffold a migration to apply these changes to the database.
- Run the Update-Database command to apply the new migration to the database.
	
	
## Summary

In this walkthrough we looked at Code First development using a new database. We defined a model using classes then used that model to create a database and store and retrieve data. Once the database was created we used Code First Migrations to change the schema as our model evolved. We also saw how to configure a model using Data Annotations and the Fluent API.
	