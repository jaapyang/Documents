# [Entity Framework Performance and What You Can Do About It](https://www.red-gate.com/simple-talk/dotnet/net-tools/entity-framework-performance-and-what-you-can-do-about-it/)

>Without a doubt, Entity Framework is a quick and satisfactory way of producing a database-driven web application. As performance becomes more important, it does, however, require some knowledge of the traps that you need to avoid, and of the wrinkles that impact performance. Ben Emmett gives a practical guide.

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
