# Entity Framework 性能优化指南

[原文链接](https://www.red-gate.com/simple-talk/dotnet/net-tools/entity-framework-performance-and-what-you-can-do-about-it/)

> 众所周知， Entity Framework是一个让人非常满意的基于数据库应用程序的快速开发框架。但Entity Framework在性能方面也一直让人有些诟病，那么，本文就如何进行性能优化与如何规避一些影响性能的 **坑** 进行介绍。

Compared to writing your own SQL to access data, you can become miraculously more productive by using Entity Framework (EF). Unfortunately, several traps that are easy to fall into have given it a reputation for performing poorly; but it doesn’t have to be this way! The performance of Entity Framework may once have been inherently poor but isn’t any more if you know where the landmines are. In this article we’ll look at where these ‘traps’ are hiding, examining how they can be spotted and what you can do about them.

We’ll use examples from a simplified school management system. There’s a database with two tables for Schools and their Pupils, and a WinForms app using an EF code-first model of this database to fetch data in a variety of inefficient ways.

To play along at home, you can grab the code for most of these examples from
 https://github.com/bcemmett/EntityFrameworkSchoolSystem – setup instructions are included in the readme. 
 
 ## Database access 
By far the biggest performance issues you’re likely to encounter are of course around accessing the database. These few are the most common.

### Being too greedy with Rows 
*Sample application: button 1*

At its heart, Entity Framework is a way of exposing .NET objects without actually knowing their values, but then fetching / updating those values from the database behind the scenes when you need them. It’s important to be aware of when EF is going to hit the database – a process called materialization.

Let’s say we have a context db with an entity db.Schools. We might choose to write something like:
```
string city =  "New York";
List<School> schools = db.Schools.ToList();
List<School> newYorkSchools = schools.Where(s => s.City == city).ToList();
```
On line 2 when we do .ToList(), Entity Framework will go out to the database to materialize the entities, so that the application has access to the actual values of those objects, rather than just having an understanding of how to look them up from the database. It’s going to retrieve every row in that Schools table, then filter the list in .NET. We can see this query in [ANTS Performance Profiler](http://www.red-gate.com/products/dotnet-development/ants-performance-profiler/index?utm_source=simpletalk&utm_medium=publink&utm_campaign=antsperformanceprofiler&utm_content=entityframeworkperformance):
![](https://www.red-gate.com/simple-talk/wp-content/uploads/imported/2325-1-6b543c4e-c10f-45f3-ae1b-443ff17d8b01.png)


