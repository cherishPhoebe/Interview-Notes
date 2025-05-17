## **ASP.NET 4.X 与 ASP.NET Core的对比**  
** .NET Framework 和 .NET Core的主要区别是什么**  
- **跨平台性**：.NET Core支持跨平台(Windows/Linux/macOS)，.NET Framework只支持Windows  
- **依赖注入**：.NET Core内置轻量级DI容器，.NET Framework需要第三方库支持(如Autofac)  
- **性能优化**: .NET Core 的运行时（CoreCLR）和 ASP.NET Core 性能更优，支持微服务场景。  

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


**依赖注入的方式有哪些？**  
- 构造器注入：通过构造器参数来传递依赖对象，由DI容器自动解析 
``` csharp
// 定义接口和实现
public interface IMessageService {
    string Send(string message);
}

public class EmailService : IMessageService {
    public string Send(string message) => $"Email Sent: {message}";
}

// 注册服务（在Startup.cs 或 Program.cs）
builder.Services.AddTransient<IMessageService, EmailService>();

// 在类中通过构造函数注入
public class NotificationController : Controller {
    private readonly IMessageService _messageService;

    // 构造函数注入
    public NotificationController(IMessageService messageService) {
        _messageService = messageService;
    }

    public IActionResult SendMessage(string msg) {
        var result = _messageService.Send(msg);
        return Content(result);
    }
}
 ```  
 - 属性注入：默认DI容器不支持，需要借助第三方库（如Autofac）。适用于无法通过构造函数注入的场景（如基类属性） `PropertiesAutowired`
``` csharp
var builder = WebApplication.CreateBuilder(args);

// 使用 Autofac 作为容器
builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory());
builder.Host.ConfigureContainer<ContainerBuilder>(containerBuilder => {
    containerBuilder.RegisterType<EmailService>().As<IMessageService>();
    containerBuilder.RegisterType<NotificationService>()
                   .PropertiesAutowired(); // 启用属性注入
});

public class NotificationService {
    // 属性注入（需 public 修饰）
    public IMessageService MessageService { get; set; }

    public void Notify(string message) {
        var result = MessageService.Send(message);
        Console.WriteLine(result);
    }
}
```  
- 方法注入：通过方法参数传递依赖项，通常用于单一方法需要特定服务的场景，不依赖容器自动解析
``` csharp
// 在方法参数中直接注入【仅限 Controller 或者 Minimal API】
[ApiController]
[Route("api/[controller]")]
public class DemoController : ControllerBase {
    // 方法参数注入（需 [FromServices] 注解）
    [HttpGet("send")]
    public IActionResult Send([FromServices] IMessageService messageService, string msg) {
        var result = messageService.Send(msg);
        return Ok(result);
    }
}

// 普通类中的方法注入（手动传递依赖项）
public class Client {
    public void ProcessMessage(IMessageService messageService, string msg) {
        var result = messageService.Send(msg);
        Console.WriteLine(result);
    }
}

// 使用
var client = new Client();
client.ProcessMessage(new EmailService(), "Hello");
```
- 其他方式： 从容器中手动解析服务
``` csharp
// 通过 HttpContext 解析（Controller 内）
var service = HttpContext.RequestServices.GetService<IMessageService>();

// 通过 IServiceProvider 解析
var provider = app.Services;
using var scope = provider.CreateScope();
var service = scope.ServiceProvider.GetRequiredService<IMessageService>();
```  

**ASP.NET Core 内置容器的特点**  
- 一个轻量级的IOC容器，支持构造器注入，可以方便的实现面向接口编程和依赖倒置
- 提供三种生命周期模式：瞬时，作用域，单例
- 可在 Startup.cs 的 ConfigureServices 方法里使用对应生命周期的注入，也可以使用特性（如 ServiceFilterAttribute 和 TypeFilterAttribute）来注入过滤器
- 可通过扩展方法或第三方库来替换成其他的 IOC 容器，如 Autofac、Unity 等  

**AOP的通知是什么？**  
通知（Advice）是指切面在连接点处执行的具体代码，它定义了切面要做什么和何时做。具有：  
- 前置通知（Before Advice）：在连接点之前执行的通知，例如在方法开始前记录日志
- 后置通知（After Advice）：在连接点之后执行的通知，例如在方法结束后清理资源
- 返回通知（After Returning Advice）：在连接点正常返回后执行的通知，例如在方法返回后缓存结果
- 异常通知（After Throwing Advice）：在连接点抛出异常后执行的通知，例如在方法异常后记录错误信息
- 环绕通知（Around Advice）：在连接点前后都执行的通知，例如在方法前后开启和提交事务

**ASP.NET Core Filter有哪几种？** 
ASP.NET Core Filter有五种，包括：  
- 授权筛选器（Authorization Filter）：用于确定当前请求用户是否被授权。如果请求未获授权，可以让管道短路
- 资源筛选器（Resource Filter）：在授权之后第一个处理请求的过滤器，也是最后一个在请求离开过滤管道时接触请求的过滤器。在性能方面，对实现缓存或者对过滤管道进行短路特别有用
- 操作筛选器（Action Filter）：包装对单个操作方法的调用，并且可以处理传递到操作的参数以及从操作返回的操作结果
- 异常筛选器（Exception Filter）：用于对 MVC 应用程序中未处理的异常应用全局策略
- 结果筛选器（Result Filter）：包装单个操作结果的执行，并且只在操作执行成功时运行。对于必须围绕视图执行或格式化程序执行的逻辑，会很有用  

**中间件是什么？**  
中间件是一种软件组件，它可以处理HTTP请求和响应，并且可以与其他中间件协作形成一个请求管道。每个中间件都有机会对请求进行操作或短路，并且可以决定是否调用下一个中间件。中间件可以实现各种功能，例如静态文件服务，异常处理，身份认证，路由，MVC等。  

在ASP.NET Core中，中间件通常以委托（Delegate）或类（Class）的形式编写，并在Startup类的Configure方法中注册到请求管道中  

**ASP.NET Core中的中间件和过滤器有什么区别？**  
中间件是一种装配到应用管道以处理请求和响应的软件，每个中间件可以选择是否将请求传递给下一个中间件，也可以在下一个中间件前后执行一些操作。  

过滤器是一种应用于控制器或动作的特性，它可以在动作执行之前或之后执行一些操作，例如输出结果的格式化，数据验证，异常处理等。  

作用域不同：中间件作用于整个应用系统，关注于通用的功能，例如身份验证，日志记录等；过滤器作用于具体的控制器或动作，关注于业务相关的内容。
使用方式不同，中间件需要在 Startup 类的 Configure 方法中使用 Run、Use、Map 等扩展方法来注册；过滤器需要在控制器或方法上使用特性来标注，或者在 Startup中全局注册。  

**JWT是什么？**  
JWT（JSON Web Token）是一种认证协议，它可以用来生成和验证接入令牌（Access Token），令牌中包含了一些用户或应用的信息（称为声明）。它由三部分组成：头部（Header），负载（Payload）和签名（Signature）  
- 头部包含了Token的类型和加密算法
- 负载包含了Token的有效信息，如过期时间，颁发者，角色等
- 签名是由头部和负载经过加密算法和密钥生成的，用于验证Token的完整性和可信性。  

JWT可以作为一种无状态的身份验证机制，不需要在服务器端存储Session或Cookie，只需要在客户端保存Token，并在每次请求时携带Token即可