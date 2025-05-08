##ASP.NET 4.X 与 ASP.NET Core的对比**  
ASP.NET Core 是一个跨平台的开源框架，用于在Windows、macOS或Linux上开发基于云的新式Web应用。
ASP.NET Core具有以下优点：  
- 生成Web UI和Web API的统一场景
- 针对可测试性进行构建
- Blazor 允许在浏览器中使用C#和javascript。共享全部使用.NET 编写的服务器端和客户端应用逻辑  
- Razor Pages 使基于页面的编码方式更简单高效  
- 能够在Windows、macOS、Linux上进行开发和运行  
- 开源 以社区为中心  
- 支持使用gRPC托管远程过程调用  
- 内置依赖注入  
- 轻型的高性能模块化HTTP请求管道  
- 并行版本控制 可同时安装多个运行时版本  

**ASP.NET Core 中服务的生命周期**  
ASP.NET Core 支持依赖注入软件设计模式，这是一种在类及其依赖关系之间实现控制反转(IoC)的技术。.NET中的依赖注入是框架内置部分，与配置、日志记录和选项模式一样。  
依赖关系注入通过以下方式解决硬编码的依赖项问题：  
- 使用接口或基类将依赖关系实现抽象化。
- 在服务容器中注册依赖关系。 .NET 提供了一个内置的服务容器 IServiceProvider。 服务通常在应用启动时注册，并追加到 IServiceCollection。 添加所有服务后，可以使用 BuildServiceProvider 创建服务容器。  
- 将服务注入到使用它的类的构造函数中。 框架负责创建依赖关系的实例，并在不再需要时将其释放。  
服务生命周期主要有三种：
- **Transient** 瞬时  
调用 `services.AddTransient`注册瞬时服务。服务每次被请求的时候都会创建一个服务实例，适合轻量级、无状态的服务。  
- **Scoped** 作用域  
调用`services.AddScoped`注册作用域服务。此模式下，会为每一个请求创建一个服务的实例，所有同一个请求中的中间件、MVC控制器等，都会获取同一个相同的服务实例。使用Entity Framework Core时，默认情况下`AddDbContext`扩展方法使用Scoped生命周期来注册`DbContext`类型。  
- **Singleton** 单例  
调用`services.AddSingleton`注册单例服务。单一实例服务必须是线程安全的，并且通常在无状态服务中使用。
