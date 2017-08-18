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
在第二行，在我们执行 ```.ToList()```的时候,EF就会去访问数据库，将```schools```表中所有行从数据库读取到实体对象集合```List<School> schools```中,以便应用程序在后续代码可以访问这些对象```schools```的实际值，而不仅仅是从以上代码中知道将会以什么方式从数据库中找到需要的值.它将访问**schools** 中的每一行，然后填充到**```List<School> schools```**集合中。我们可以在[ANTS Performance Profiler](http://www.red-gate.com/products/dotnet-development/ants-performance-profiler/index?utm_source=simpletalk&utm_medium=publink&utm_campaign=antsperformanceprofiler&utm_content=entityframeworkperformance)看到EF生成的查询语句如下 :

![](https://www.red-gate.com/simple-talk/wp-content/uploads/imported/2325-1-6b543c4e-c10f-45f3-ae1b-443ff17d8b01.png)



