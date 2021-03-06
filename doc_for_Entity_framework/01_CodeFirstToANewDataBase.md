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


# 5.处理数据变更

现在是时候对我们的模型进行一些更改，当我们进行这些更改时，我们还需要同步更新数据库结构。
为此，我们将使用一个名为Code First Migrations的功能，或简称为**Migrations**。

首先，为我们定义的BloggingContext启用Code First Migrations.

- **工具** -> **NuGet管理器** -> **程序包控制台**
- 选择你定义的DbContext(BloggingContext)所在的项目/工程(Project)为默认项目
- 输入命令: ```Enable-Migrations```
- **程序包控制台**会在DbContext所在项目自动为我们生成一个名为**Migrations**的文件夹，并自动生成了两个文件:

    - **Configuration.cs** : 这个文件定义了Migrations将用于迁移BloggingContext的设置。一般情况下我们不需要为此做任何更改，但是您可以在此指定种子数据(数据库初始化数据).
    - **<时间字符串>_InitialCreate.cs** : 这里定义的是已经应用了到目前为止的数据库变更的第一次Migration,它将会在一个空的数据库中创建"Blogs"和Posts两个数据表。在这个Migration中，Code First将代码转换成了从对象模型到数据模型的迁移，并且，Code First还在数据库中记录了迁移版本，这些版本是以时间排序的。
    
现在我们往 **Blog** 类中添加一个 **Url** 属性:

```
public class Blog 
{ 
    public int BlogId { get; set; } 
    public string Name { get; set; } 
    public string Url { get; set; } 
    public virtual List<Post> Posts { get; set; } 
}
```


-  在**程序包控制台**输入命令 ```Add-Migration AddUrl```,并执行. **Add-Migration** 命令会检查之前所有的迁移与现有实体模型之间的差异，将在从上一次迁移到现在的所有变更应用到本次迁移中(即生成到 yyyyMMdd..._**AddUrl**.cs文件中)。命令(```Add-Migration AddUrl```)中 '**AddUrl**'是命令的参数，即指定迁移的命名为 'AddUrl'.生成的代码如下:

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

- 在**程序包控制台**输入并执行命令 ```Update-Database```.这个命令会将所有待更新的迁移更新到数据库。因为之前```InitialCreate```的迁移已经更新到数据库，所以这一次更新只会应用我们刚刚生成的```AddUrl``的迁移。
> Tip: 你可以附加命令参数 ```-Verbose``` 用于查看将要执行的更新的SQL脚本
	
## 6. 数据模型标识(对于'**Data** **Annotations**'这个词的翻译我个人认为是数据模型标识，如果有更好的翻译，请帮忙指正，谢谢)

到目前为止，我们只是依赖EF的默认命名约定来定义模型，但是对于有一些场景我们无法去遵循这个约定，这个时候我们就要进行进一步的配置了。关于这部分的知识，可以在本文中查看数据模型标识的相关Attribute支持，在下一部分中会有详细介绍.

- 我们添加一个新的实体模型 ```User```

```
public class User 
{ 
	public string Username { get; set; } 
	public string DisplayName { get; set; } 
}
```

- 当然，我们相应的也要在DbContext中添加一个User的强类型DbSet<User>

```
public class BloggingContext : DbContext 
{ 
	public DbSet<Blog- Blogs { get; set; } 
	public DbSet<Post- Posts { get; set; } 
	public DbSet<User- Users { get; set; } 
}
```

- 当我们尝试使用 ```Add-Migration ...```命令去生成新的数据迁移时，会发现一个错误：**"EntityType 'User' has no key defined, Define the key for this EntityType"**, 即实体模型 'User'没有定义主键，请为实体模型'User'定义主键。 这是因为EF无法主动发现 ```Username```是 ```User```的主键。
- 这个时候我们就需要给```User```的```Username```属性添加一个```KeyAttribute```的特性，用于告诉EF，我们将具备```KeyAttribute```特性的属性定义为模型的主键,即 ```Username```.

引用命名空间如下：

```
using System.ComponentModel.DataAnnotations;
```

- 现在我们用```Key```来标识```Username```为主键

```
public class User 
{ 
	[Key] 
	public string Username { get; set; } 
	public string DisplayName { get; set; } 
}
```

- 接下来,用```add-migration AddUser```命令来创建这些更新的迁移文件
- 执行```Update-Database```命令，将迁移更新到数据库。

这个时候，我们发现数据库中已经创建好了新的数据表 **Users**

以下是EF提供的数据模型标识特性(Attribute)清单:

| 特性名 | 特性描述 |
|:--------|:---------|
| [KeyAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.keyattribute) |  用于指定实体的唯一标识，即主键 |
| [StringLengthAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.stringlengthattribute) | 指定字段的最大字符数和最小字符数 |
| [MaxLengthAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.maxlengthattribute) | 用于指定字段的最大长度 |
| [ConcurrencyCheckAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.concurrencycheckattribute) | 指定属性参与乐观并发检查。  |
| [RequiredAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.requiredattribute) | 标记为非空(必填)字段 |
| [TimestampAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.timestampattribute) | 行版本类型 （也称为序列号） 是保证可在数据库中唯一一个二进制数字。 它不表示的实际时间。 行版本数据不是以可视方式有意义的。 因此，当 TimestampAttribute 属性用于动态数据字段，则将不显示列除非 ScaffoldColumnAttribute 列的属性显式设置为 true。 |
| [ComplexTypeAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.complextypeattribute) | 表示该类是复杂类型。 复杂类型是实体类型的非标量属性，实体类型允许在实体内组织标量属性。 复杂类型没有键，并且实体框架不能脱离父对象来管理复杂类型。 |
| [ColumnAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.columnattribute) | 表示属性将映射到的数据库列 |
| [TableAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.tableattribute) | 指定类将映射到的数据库表。 |
| [InversePropertyAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.inversepropertyattribute) | 指定表示同一关系的另一端的导航属性的反向属性。  |
| [ForeignKeyAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.foreignkeyattribute) | 外键标识  |
| [DatabaseGeneratedAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.databasegeneratedattribute) | 指定数据库生成属性值的方式。一般用于指定标识列的生成方式 |
| [NotMappedAttribute](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.notmappedattribute) | 表示应从数据库映射中排除属性或类。  |




## 7. Fluent API

在上一节中我们研究了通过数据模型标识特性来补充或者覆盖默认情况下EF检测到的模型。那么，Fluent API其实还提供了另一种方法来配置模型

我们知道，大多数模型可通过简单的数据模型标识来完成定义或配置，但Code First提供了一种更先进的试来指定模型配置，即**Fluent API**，除了涵盖数据标识可以实现的所有操作外，还提供了一些数据模型标识所不可能实现的一些更高级配置方式。此外，数据模型标识和**Fluent API**这种高级配置方式是可以一起使用的。

我们可以通过Fluent API提供的支持在DbContext中重写```OnModelCreating```方法。我们一起来看一下，怎么样在更新数据模型时将在**User**实体中定义为 **DisplayName** 的属性在数据库中映射为字段名 **display_name**.


- 在 **BloggingContext**中重写**OnModelCreating**方法，代码如下：

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

- 在 **程序包控制台** 执行命令 ```Add-Migration ChangeDisplayName```，生成对于此变更的新的迁移
- 执行 ```Update-Database```命令将此迁移更新到数据库

	
## 小结

- 使用Code First 定义一个新的数据模型
- 使用DbContext检索或存储数据
- 将模型的变更通过数据迁移命令(Migrations)更新到数据库
- 使用Fluent API配置模型
	
