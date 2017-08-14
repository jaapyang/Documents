# 5.处理数据变更
现在是时候对我们的模型进行一些更改，当我们进行这些更改时，我们还需要同步更新数据库结构。为此，我们将使用一个名为Code First Migrations的功能，或简称为**Migrations**。
首先，为我们定义的BloggingContext启用Code First Migrations.
- **工具** -> **NuGet管理器** -> **程序包控制台**- 选择你定义的DbContext(BloggingContext)所在的项目/工程(Project)为默认项目- 输入命令: ```Enable-Migrations```- **程序包控制台**会在DbContext所在项目自动为我们生成一个名为**Migrations**的文件夹，并自动生成了两个文件:
    - **Configuration.cs** : 这个文件定义了Migrations将用于迁移BloggingContext的设置。 一般情况下我们不需要为此做任何更改，但是您可以在此指定种子数据(数据库初始化数据).    - **<时间字符串>_InitialCreate.cs** : 这里定义的是已经应用了到目前为止的数据库变更的第一次Migration,它将会在一个空的数据库中创建"Blogs"和Posts两个数据表。在这个Migration中，Code First将代码转换成了从对象模型到数据模型的迁移，并且，Code First还在数据库中记录了迁移版本，这些版本是以时间排序的。    现在我们往 **Blog** 类中添加一个 **Url** 属性:
```public class Blog {     public int BlogId { get; set; }     public string Name { get; set; }     public string Url { get; set; }      public virtual List<Post> Posts { get; set; } }```
