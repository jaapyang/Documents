# Entity Framework Code First to an Existing Database

> This video and step-by-step walkthrough provide an introduction to Code First development targeting an existing database. Code First allows you to define your model using C# or VB.Net classes. Optionally additional configuration can be performed using attributes on your classes and properties or by using a fluent API.

## Pre-Requisites

You will need to have **Visual Studio 2012** or **Visual Studio 2013** installed to complete this walkthrough.

You will also need version **6.1** (or later) of the **Entity Framework Tools for Visual Studio** installed. See [Get Entity Framework](https://msdn.microsoft.com/en-us/library/ee712906(v=vs.113).aspx) for information on installing the latest version of the Entity Framework Tools.

# 1. Create an Existing Database

Typically when you are targeting an existing database it will already be created, but for this walkthrough we need to create a database to access.

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

- Copy the following SQL into the new query, then right-click on the query and select Execute

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

# 2.Create the Application

To keep things simple we’re going to build a basic console application that uses Code First to perform data access:

- Open Visual Studio
- **File -> New -> Project…**
- Select **Windows** from the left menu and **Console Application**
- Enter **CodeFirstExistingDatabaseSample** as the name
- Select **OK**

# 3.Reverse Engineer Model

We’re going to make use of the Entity Framework Tools for Visual Studio to help us generate some initial code to map to the database. These tools are just generating code that you could also type by hand if you prefer.

- **Project -> Add New Item…**

- Select **Data** from the left menu and then **ADO.NET Entity Data Model**

- Enter **BloggingContext** as the name and click **OK**

- This launches the **Entity Data Model Wizard**

- Select **Code First from Database** and click **Next**

![](https://i-msdn.sec.s-msft.com/dynimg/IC716048.jpeg)

- Select the connection to the database you created in the first section and click **Next**

![](https://i-msdn.sec.s-msft.com/dynimg/IC716050.jpeg)








