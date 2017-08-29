# Entity Framework 性能优化指南

[原文链接](https://www.red-gate.com/simple-talk/dotnet/net-tools/entity-framework-performance-and-what-you-can-do-about-it/)

> 众所周知， Entity Framework是一个让人非常满意的基于数据库应用程序的快速开发框架。但Entity Framework在性能方面也一直让人有些诟病，那么，本文就如何进行性能优化与如何规避一些影响性能的 **坑** 进行介绍。

相对于自己写SQL语句进行数据访问来说，使用EF进行数据访问会有更高的开发效率。但悲催的是，EF有些影响性能的 **坑** 让它备受诟病；但实际上这些是可以规避的。也许EF
与编写自己的SQL来访问数据相比，通过使用实体框架（EF），您可以变得奇迹般地更高效。 不幸的是，容易陷入的几个陷阱给了它表现不佳的声誉; 但它不一定是这样的！ 实体框架的表现可能曾经本来就很差，但是如果你知道地雷在哪里，那就不再是这样了。 在这篇文章中，我们将看看这些“陷阱”在哪里隐藏，检查它们如何被发现以及你可以做些什么。

我们将通过Code first的方式，实现一个简单的学校管理系统的案例。它有一个数据库，两个数据表，分别为学校和在校学生信息表，以Winform窗体程序的方式进行呈现。在此过程中，我们将采用一些低效的数据访问方式，并尝试进行优化。

你可以从[https://github.com/bcemmett/EntityFrameworkSchoolSystem](https://github.com/bcemmett/EntityFrameworkSchoolSystem) 获取到这个案例的代码，相关的设置说明请参考 Readme.md
 
 ## 数据访问 
到目前为止，你可能遇到最大的性能问题是访问数据库，以下是最常见的几个问题：

### Being too greedy with Rows 
*Sample application: button 1*

在它的核心，EF是以一种暴露.NET对象属性而不赋值的方式来提供一种由数据表结构到.NET的映射，EF只会在程序需要时，才从数据库中获取或更新这些值到映射的对象实体中。所以，重要的是要知道什么时候EF会访问数据 ———— 实现由数据到实体的转换

假设有一个连接到 ```db.Schools``` 的DbContext，我们可能会以下面代码中的方式执行数据访问：
```
string city =  "New York";
List<School> schools = db.Schools.ToList();
List<School> newYorkSchools = schools.Where(s => s.City == city).ToList();
```
在第二行，在我们执行 ```.ToList()```的时候,EF就会去访问数据库，将```schools```表中所有行从数据库读取到实体对象集合```List<School> schools```中,以便应用程序在后续代码可以访问这些对象```schools```的实际值，而不仅仅是从以上代码中知道将会以什么方式从数据库中找到需要的值.它将访问**schools** 中的每一行，然后填充到```List<School> schools```集合中。我们可以在[ANTS Performance Profiler](http://www.red-gate.com/products/dotnet-development/ants-performance-profiler/index?utm_source=simpletalk&utm_medium=publink&utm_campaign=antsperformanceprofiler&utm_content=entityframeworkperformance)看到EF生成的查询语句如下 :

![](https://www.red-gate.com/simple-talk/wp-content/uploads/imported/2325-1-6b543c4e-c10f-45f3-ae1b-443ff17d8b01.png)

为了让SQL Server进行更高效的查询并传输更少的数据，我们可以用以下的写法：
```
List<School>  newYorkSchools = db.Schools.Where(s => s.City == city).ToList();
```

或者像这样:

```
IQueryable<School> schools = db.Schools;
List<School> newYorkSchools = schools.Where(s => s.City == city).ToList();
```

### "N+1查询"问题：最小化到数据库的行程

案例: *button 2*

这是一个对象实体化时另一个常见的**坑**.

在我们的数据库中，```Pupil```数据表中有一个对```School```表的外键引用（```SchoolId```）.而在EF实体模型对象结构中，它表现为在```Schools```对象中的一个```virtual```修饰的属性```Puplils```（我们姑且称之为"虚属性"）.

我可以通过以下代码将每个```School```的```Pupils```输出：
```
	string city =  "New York";
	List<School> schools = db.Schools.Where(s => s.City == city).ToList();
	var sb =  new  StringBuilder();
	foreach(var school  in schools)
	{
	    sb.Append(school.Name);
	    sb.Append(": ");
	    sb.Append(school.Pupils.Count);
	    sb.Append(Environment.NewLine);
	}
```

我们可以在 ANTS中观察到正在运行的代码到底做了什么,我们发现有一个查询请求在查询一个在**'New York'**的**Schools**列表，但同时，也有另一个获取**Pupil**信息的查询执行了500次。

![](https://www.red-gate.com/simple-talk/wp-content/uploads/imported/2325-1-2fbc0f9f-774f-441e-a91c-acdd4a812ba7.png)

发生这种情况是因为在默认情况下，EF使用一种叫做[**Lazy Loading**],当第一次查询**Schoo**对象的时候，它不会获取与**School**关系的**Puppies**的数据。在后面的过程中，当你尝试去读取**School**对应的**Pupils**的数据时，程序就只能从数据库查询了。在大多数时候，这是一种很好的解决方案，否则你访问**School**对象时，会同时连带查询**Pupils**的数据，无论你是否需要**Pupils**的数据。但是在上面这个案例中，EF首先要查询所有符合条件的**SChool**，然后再查询与这些**School**关联的**Pupils**的数据。

这就是取名“N + 1”的原因，其中N是原始查询返回的对象数。 如果你知道你肯定会想要学生数据，那么你应该采用不同的做法，特别是如果你想要大量的School对象。 如果应用程序和数据库服务器之间存在高延迟，这一点尤为重要。

有几种不同的方法可以解决这个问题。 第一个是使用贪婪加载（Eager Loading）数据访问策略，当您使用Include（）语句时，它将在单个查询中获取指定的相关数据。这样就会在一个查询中把**Pupils**数据加载到内存中，所以EF不需要再次访问数据库。 为了做到这一点，你的第一行将是：

```
List<School> schools = db.Schools.Where(s => s.City == city)
	    .Include(x => x.Pupils) //贪婪加载
	    .ToList();
```

这种方法相对之前是一个改进，至少我们没有执行额外的500次查询，但这样还是有个缺点，那就是有时候我们加载这些**Pupils**数据只是为了看看有多少个**Pupils**，但不需要这些**Pupils**的详细数据。例如，如果我们想要查询在**New York**在校生超过100(More than 100 Pupils)的**School**(但并不关心究竟有多少个学生)，那么常见的一种错误写法就是：

```
List<School> schools = db.Schools
	    .Where(s => s.City == city)
	    .ToList();
	 
	List<School> popularSchools = schools
	    .Where(s => s.Pupils.Count > 100)
	    .ToList();
```

我们同样只要使用之前提过的办法解决这个问题，即添加一个**Include()**声明,像下面这段代码一样：

```
List<School> schools = db.Schools
	    .Where(s => s.City == city)
	    .Include(x => x.Pupils)
	    .ToList();
	 
List<School> popularSchools = schools
    .Where(s => s.Pupils.Count > 100)
    .ToList();
```

但实际不我们并不需要知道具体每个学校有多少学生，所以，我们有更高效的办法：

```
List<School> schools = db.Schools
	    .Where(s => s.City == city
	        && s.Pupils.Count > 100)
	    .ToList();
```

现在我将介绍通过选择我们需要的特殊列来进一步提高性能。

### Being too grady with Columns

案例程序:Button 3

如果我们要想获取某些学校中每个学生的name，我们可以这样做：
```
int schoolId = 1;

List<Pupil> pupils = db.Pupils
 .Where(p => p.SchoolId == schoolId)
 .ToList();

foreach(var pupil  in pupils)
{
 textBox_Output.Text += pupil.FirstName +  " " + pupil.LastName;
 textBox_Output.Text +=  Environment.NewLine;
}
```

我们可以在 ANTS 看到我们正在执行的查询，我们会发现在除了我们需要的FirstName和LastName以外，同时还检索出很多其它我们不需要的数据。
![](https://www.red-gate.com/simple-talk/wp-content/uploads/imported/2325-1-749f6160-0d43-4f86-84ee-f68b31055a63.png)

### Missing indexes 

### Overly-generic queries 

### Bloating the plan cache 

### Inserting data 

## Extra work in the client 

### Detecting Changes 

### Change tracking 


## Startup Performance 

### Precompiled views 

### Giant contexts 

### NGen everything 

### Unnecessary queries 


## Other tips 

### Disposing 

### Multiple result sets 

### Caching 

### Consider Using Async 

### Upgrade 

### Test with Realistic Data 

### Occasionally, EF isn’t the answer 

## Summary 

