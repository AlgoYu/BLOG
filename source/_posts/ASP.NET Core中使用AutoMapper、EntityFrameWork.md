---
title: 'ASP.NET Core中使用AutoMapper、EntityFrameWork'
id: AutoMapperAndEntityFrameWorkAreUsedInASP.NETCore
directory: Guide
tags:
  - ASP.NET Core
  - AutoMapper
  - EntityFrameworkCore
  - C#
cover: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1542093920&di=7f8e1193593a9caec4b24aeb3f23d631&imgtype=jpg&er=1&src=http%3A%2F%2Fold.bz55.com%2Fuploads%2Fallimg%2F121218%2F1-12121P92314.jpg
date: 2018-11-05 16:25:25
---
> ## 什么是ASP.NET Core ##
.NET Core 是.NET Framework的新一代版本，是微软开发的第一个官方版本，具有跨平台 (Windows、Mac OSX、Linux) 能力的应用程序开发框架 (Application Framework)，未来也将会支持 FreeBSD 与 Alpine 平台，也是微软在一开始发展时就开源的软件平台，它经常也会拿来和现有的开源 .NET 平台 Mono 比较。
> ## 什么是AutoMapper？##
AutoMapper是一个简单的小型库，用于解决一个看似复杂的问题 - 摆脱将一个对象映射到另一个对象的代码。这种类型的代码是相当沉闷和无聊的写，所以为什么不发明一个工具来为我们做？
> ## 什么是Entity Framework Core ##
Entity Framework (EF) Core 是轻量化、可扩展和跨平台版的常用 Entity Framework 数据访问技术。
EF Core 可用作对象关系映射程序 (O/RM)，以便于 .NET 开发人员能够使用 .NET 对象来处理数据库，这样就不必经常编写大部分数据访问代码了。

创建一个ASP.NET Core Web工程，选择MVC的模板，
在程序包管理控制台中安装EntityFramework。
```xml
Install-Package Microsoft.EntityFrameworkCore
```
安装AutoMapper。
```xml
Install-Package AutoMapper
```
在appsettings.json中添加连接数据库对象
```json
"ConnectionStrings": {
    "default": "server=(主机IP);database=（数据库名称）;uid=（用户名）;pwd=（密码）.;MultipleActiveResultSets=true;"
  }
```
创建两个实体类：
```csharp
public class Book
{
    public int Id { get; set; }
    public string BookName { get; set; }
    //虚拟映射 多个Comment
    public virtual ICollection<Comment> Comments { get; set; }
}
```
```csharp
public class Comment
{
    public int Id { get; set; }
    public int BookId { get; set; }
    public string Content { get; set; }
    //虚拟映射 一个Book
    public virtual Book Book { get; set; }
}
```
写好两个实体类后，创建EF上下文：
```csharp
public class MyDB : DbContext
{
    //构造函数
    public MyDB(DbContextOptions options) : base(options)
    {
        
    }
    //Book表
    public DbSet<Book> Books { get; set; }
    //Comment表
    public DbSet<Comment> Comments { get; set; }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Book>(book =>
        {
            //设置主键
            book.HasKey(x => x.Id);
            //设置一对多关系和外键字段
            book.HasMany(x => x.Comments).WithOne(x => x.Book).HasForeignKey(x => x.BookId);
            //设置数据库中的表名
            book.ToTable("Books");
        });
        modelBuilder.Entity<Comment>(comment =>
        {
            //以下操作同上
            comment.HasKey(x => x.Id);
            comment.HasOne(x => x.Book).WithMany(x => x.Comments).HasForeignKey(x => x.BookId);
            comment.ToTable("Comments");
        });
        base.OnModelCreating(modelBuilder);
    }
}
```
在startup的ConfigureServices方法中添加一个依赖注入：
```csharp
services.AddDbContext<MyDB>(x => x.UseSqlServer(Configuration.GetConnectionString("default")));
```
创建Dto传输实体类：
```csharp
public class BookDto
{
    public int Id { get; set; }
    public string BookName { get; set; }
}
```
创建AutoMapper配置类：
```csharp
public class AutoMapperConfiguration
{
    //一个静态方法
    public static void initializeMapper()
    {
        //初始化
        Mapper.Initialize(map =>
        {
            //创建映射关系
            map.CreateMap<Book, BookDto>()
                //映射属性
                .ForMember(dest => dest.Id, opt => opt.MapFrom(src => src.Id))
                .ForMember(dest => dest.BookName,opt=>opt.MapFrom(src=>src.BookName));
        });
    }
}
```
在startup的Configure方法中加入静态方法调用：
```csharp
AutoMapperConfiguration.initializeMapper();
```
在程序包管理器控制台添加数据迁移，再更新数据库：
```xml
Add-Migration 迁移名称
Update-Database
```
在数据库中添加几条数据，运行程序，根据路由地址访问，得到相应的结果。

<b><font color="FF0000">文章内容均为本人手写；<br>禁止转载，请尊重原著；<br>本人QQ：794763733。</font></b>