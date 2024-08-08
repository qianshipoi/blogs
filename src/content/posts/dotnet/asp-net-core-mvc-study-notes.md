---
title: Asp.Net Core MVC Study Notes
published: 2022-06-13
description: ''
image: ''
tags: ['Dotnet']
category: 'Develop'
draft: false
---

# 1 在ASP.NET Core 中使用MVC

    1.1 在项目中引入MVC模块 在 Startup类 ConfigureServices 方法中 service.AddMvc() 引入模块 或者 services.AddMvcCore() 引入模块
    1.2 在管道中配置MVC中间件 在 startup类 Configure 方法中 app.UseFileServer(); 允许加载静态文件 app.UseMvcWithDefaultRoute()  添加默认路由中间件
    1.3 默认为 hemo控制器 index方法 id可选参数
      1.3.1 AddMvc 和 AddMvcCore 的区别：AddMvcCore()方法只会添加最核心的MVC服务，AddMvc()方法添加了所有必须的MVC服务，且在内部会调用AddMvcCore()方法

# 2 依赖注入

    2.1 service.AddSingleton() service.AddTransient() service.AddScored() 三种方式
    2.2 依赖注入的好处  定义接口实现方法时 在使用构造函数处理数据时 使用依赖注入的方式 查询到使用当前接口则调用 指定的类 低耦合
    2.3 service.AddSingleton() 在同一个或多个HTTP请求范围内使用同一个实例
    2.4 service.AddScored() 在同一个HTTP请求范围内使用同一个实例 多个HTTP请求时分别创建新实例
    2.5 service.AddTransient() 每次进行HTTP请求都会生成新实例

# 3 View视图

    3.1 View() 或 View(object model) ：查找与操作方法同名的视图文件
    3.2 View(string viewName) ：可以查询自定义名称的视图文件 可以指定视图名称或者路径 绝对路径需要加上.cshtml 相对路径不需要加
    3.3 View(string viewName, object model)

# 4 _Layout布局页

    4.1 布局页的使用
      4.1.1 在Views文件夹下面创建Shared文件夹 里面添加_Layout.cshtml布局页
      4.1.2 .cshtml顶端引入布局页 @{Layout="布局页路径";ViewBag.Title="标题名称"}
    4.2 布局页按需导入js文件
      4.2.1 @RenderSection("需要导入名称",requried:false)  是否必填
      4.2.2 在需要使用js文件的页面使用 @Section 需要导入名称{ <script src="路径"></script> }
      4.2.3 当页面没有 @Section时就不会加载js文件 存在 会自动加载js文件
      4.2.4 该方法不止可以按需导入js文件  还可以指定某些页面的某个模块在布局页指定位置显示

# 5 _ViewStart开始视图

    5.1 使用_ViewStart开始页批量设置页面布局页路径  支持文件分层
    5.2 _ViewStart可以存在多个 优先级问题：越靠近要引入布局页的页面 优先级越高（在文件夹内的开始页只会影响到文件夹内视图页内布局页路径）
    5.3 _ViewStart可以使用简单的逻辑运算符判断需要加载指定布局页

# 6 _ViewImports视图导入

    6.1 @using 引入公共命名空间 在视图页面就不需要指定命名空间了
    6.2 该页面也支持文件分成 最靠近视图的文件才能生效
    6.3 该页还有其他指令 @addTagHelper @removeTagHlper @tagHelperPrefix @modle @inherits @inject

# 7 路由

    7.1 默认路由 app.UseMvcWithDefaultRoute();   中间件 默认规则 {controller=Home}/{action=Info}/{id?}
    8.1 自定义路由 app.UseMvc(routes=> { routes.MapRoute("default","{controller=Home}/{action=Info}/{id?}"); });  参1 路由名称 参2 路由规则
    7.3 属性路由 在控制器方法上定义 [Route()] 属性  / 或 ~/ 表示默认访问的方法

# 8 TagHelper

    8.1 在视图导入.cshtml文件加载TagHelper @addTagHelper *,Microsoft.AspNetCore.Mvc.TagHelpers
    8.2 在View页面使用TagHelper <a asp-controller="Home" asp-action="Index" asp-route-id="@item.Id">查看</a>
    8.3 TagHelper好处 当路由模板发生改变时 当前链接依然可用 它是通过映射关系调用的控制器 传统拼接方式就需要重新修改地址
    8.4 图片缓存破坏 <img src="href" asp-append-version="true">
    8.5 当服务器图片文件被替换时 浏览器会重新进行图片加载 而不是用浏览器之前缓存的图片
    8.6 引入指定外部文件
      8.6.1 视图页面使用<environment></environment> 标签 include属性表示在这些环境之下引入的文件 exclude属性表示不再这些环境之下引入的文件
      8.6.2 当使用cdn引入外文件时 cdn我发正常使用的情况下要让项目自动加载本地的文件
      8.6.3 <link/> 标签 href属性 cdn地址
      8.6.4 asp-fallback-href如果cdn访问不到则加载对应文件路径
      8.6.5 asp-fallback-文件路径-class="sr-only"属性 只读
      8.6.6 asp-fallback-文件路径-property = "position"
      8.6.7 asp=fallback-文件路径-value = "absolute"
    8.7 表单TagHelper常用属性
      8.7.1 form标签 asp-contriller属性 控制器 asp-action属性 方法
      8.7.2 lable标签 asp-for属性 与 input标签 asp-for属性同时使用 当点击lable时触发input的聚焦 sap-for为生成的name属性和id属性 可以使用强类型视图上传表单
      8.7.3 select标签 sap-for属性 与 input标签 asp-for属性相同 asp-items="Html.GetEnumSelectList<枚举类>()" 从枚举类文件中绑定下拉框选项
      8.7.4 验证相关属性
        8.7.4.1 添加验证属性 在字段上添加 [验证属性名]  属性名常用有 Required 非空 Range 指定范围 MinLength 最小长度 MaxLength 最大长度 Compare 比较两个字段的值
        8.7.4.2 RegularExpression 使用正则表达式匹配
        8.7.4.3 Display 给字段取别名 此特性在枚举类Eunm中使用
        8.7.4.4 <div asp-validation-summary="All" ></div> 标签 验证信息汇总 在form表单内
        8.7.4.5 <span asp-validation-for="字段名"></span> 验证失败错误信息 在表单项内
        8.7.4.6 服务器端使用 ModelState.IsValid 判断验证是否通过
        8.7.4.7 select下拉框标签做验证时 默认验证的是枚举对应的key 为int类型 但是在页面传入的value=""传入的为空字符串 而int类型不可能为空 无法正常验证
        8.7.4.8 解决方案 在定义的枚举字段使用可空类型 使用 ? 标识类型

# 9 EF CORE ORM对象关系映射框架

    9.1 建立数据模型
      9.1.1 新建AppDbContext类 继承 DbContext 类 本类构造函数调用父类构造函数 参数为链接字符串
      9.1.2 创建模型类关系映射 DbSet<模型类> 模型类名 {get;set;}
    9.2 连接数据库
      9.2.1 在配置文件添加链接字符串 "ConneionsStrings": {"StudentDBConnection": "server=(localdb)\\MSSQLLocalDB;database=StudentDB;Trusted_Connection=true"}
      9.2.2 在startup中链接数据库 service.AppDbContextPool<AppDBContext>(options => options.GetConnectionString("StudentDBConnection"));
    9.3 建sql处理增删改查类 继承 Student 的接口类
    9.4 在startup中修改依赖注入项 service.AppScopen<IStudentRepository,SQLStudentRepository>();
    9.5 使用EFCore数据迁移
      9.5.1 打开PM控制台  工具--> NuGet 管理器 --> 程序包管理控制器
      9.5.2 get-help about_entityframeworkcore指令 获取相关帮助
      9.5.3 add-migration指令 添加迁移记录
      9.5.4 update-database指令 将数据库更新为指定迁移
    9.6 设置种子数据
      9.6.1 在AppDbContext类重写OnModelCreating方法
        protented override void OnModelCreating(ModelBuiler modelBuiler){
          modelBuiler.Entity<Sudent>().HasData(
              new Stdent{id=1,name="栗山未来",classname="114514"},
             new Stdent{id=2,name="kuriyama",classname="114514"}
        );}
      9.6.2 添加迁移文件 add-migration 文件名 update-database 更新数据库 默认使用最新的迁移文件
    9.7 将种子数据添加在新文件
     9.7.1 新建ModelBuilerExtensions类 添加方法 public static void Seed(this ModelBuiler modelBulier)  this 给ModelBuiler 添加当前方法
     9.7.2 将种子数据放入Seed方法 在AppDBContext类中 OnModelCreating()方法中 使用 modelBulier.Seed() 调用种子文件。
