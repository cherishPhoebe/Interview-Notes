**1.C#中的委托是什么？事件是不是一种委托**  
委托的本质是一个类，委托是将一种方法作为参数代入到另一种方法。  
事件是委托的实例，事件是一种特殊的委托。  
//比如：onclick 事件中的参数就是一种方法。  

**C#静态构造行数的特点**  
最先被执行的构造函数，且在一个类里只允许有一个无参的静态构造函数  
执行顺序：静态变量>静态构造函数>实例变量>实例构造函数  

**C#中值类型（Value Type）和引用类型（Reference Type）的底层存储区别是什么？如何避免装箱（Boxing）带来的性能问题**  
- **存储区别：**
    - 值类型存储在栈（Stack）或内联在包含它的对象中
    - 引用类型存储在堆（Heap），栈中仅存储其引用地址
- **如何避免装箱：**  
    - 使用泛型集合（如List<int>替代ArrayList）
    - 使用Span<T>或Memory<T>处理内存连续数据
    - 通过Where T : struct 约束泛型类型参数为值类型
示例：  
``` C#
// 装箱 性能差
int num = 42;
object boxed = num; // 隐式装箱

// 避免装箱（使用泛型）
List<int> numbers = new List<int>{1,2,3};
```

**C#中值类型与引用类型，类(class)与结构体(struct)的区别**  
值类型：**struct**、enum、int、float、char、bool、decimal
引用类型：class、delegate、interface、array、object、**string**  
|维度|类(class)|结构体(struct)|  
|----|----|----|  
|数据类型|类中定义的数据作为引用存储在内存中，是一种引用类型|结构体数据成员的值直接存储在堆栈中，是一种值类型|  
|构造函数|可以使用构造函数以及反构造函数来定义类|结构体不能具有构造函数或反构造函数;C# 10支持声明无参构造函数，但需要初始化该类型的所有实例字段，C#11后则无此限制|  
|继承|类支持继承功能|结构体不支持继承|  
|保护修饰符|对类中定义的数据成员可以使用protected修饰符|结构体不支持受保护的修饰符|  

**C#中readonly和const有什么区别？他们在内存中的行为有什么不同？**  
- **const：**
  - 编译时常量，必须初始化且值不可变；
  - 内联到代码中（编译时替换），存储在元数据中。
- **readonly：**
  - 运行时常量，可以在构造函数或声明时初始化；
  - 存储在堆或栈中，依赖对象生命周期。
示例：  
``` csharp
public class Example{
    public const int MaxValue = 100; // 编译时常量
    public readonly int MinValue = 0; // 运行时常量
    public readonly DateTime Timestamp;

    public Example(){
        Timestamp = DateTime.Now; // 构造函数中初始化
    }

}
```

**线程与进程的区别**  
线程(Thread)与进程(Process)二者都定义了某种边界，**进程**定义的是应用程序与应用程序之间的边界，不同的进程之间不能共享代码和数据空间；**线程**定义的是代码执行堆栈和执行上下文的边界。  
一个进程可以包括若干个线程，同时创建多个线程来完成某项任务便是多线程

**using关键字的作用？跟IDisposable有什么关系**  
using 可以申明namespace的引入；还可以实现非托管资源的释放。如果实现了IDisposable接口的类在using中创建，using结束后会自动调用该对象的Dispose方法，释放资源。  

## **Task和Thread的区别**  
Task和Thread都能创建用多线程的方式执行代码，但他们有较大的却别：  
Task较新，发布于.Net 4.5，能结合新的async/await 代码模型写代码，他不止能创建新线程，还能使用线程池(默认)、单线程等方式编程，在UI编程领域，Task还能自动返回UI线程上下文，还提供了许多便利API以管理多个Task。

## **多线程和异步的区别与联系**  
多线程是实现异步的主要方式之一，异步并不等同于多线程。实现异步的方式还有很多，比如利用硬件的特性、使用进程或线程等。  

## **线程池的优缺点**  
优点：减小线程创建和销毁的开销，可以复用线程；从而也减少了线程上下文切换的性能损失；在GC回收时，较少的线程更有利于GC的回收效率。  
缺点：线程池无法对一个线程有更多的精确的控制，如了解其运行状态等；不能设置线程的优先级；加入到线程池的任务（方法）不能有返回值；对于需要长期运行的任务就不适合线程池。  

## **Task与并行Parallel**  
任务Task与并行Parallel本质上内部都是使用的线程池，提供了更丰富的并行编程的方式。  

**多线程并行(Parallelism)和并发(Concurrency)的区别**  
并行：同一时刻有多条指令在多个处理器上同时执行，无论从宏观还是微观上都是同时发生的。  
并发：在同一时间段内，宏观上看多个指令看起来是同时执行，微观上看是多个执行进程在快速的切换执行，同一时刻可能只有一条指令被执行。  

**C# Parallel.For 和普通 For 的区别**  
Parallel 类是.Net 4中新增的抽象线程类。Parallel.For()方法类似于C#的for循环语句，都是多次执行一个任务。但是Parallel.For()方法可以并行运行。  
对于Parallel.For、Parallel.Foreach的使用应该特别小心，他们的优势是处理列表很长，且对列表内的元素进行很复杂的业务逻辑，**且不会使用共享资源**，只针对自身的业务逻辑处理有较高的效率。  

**C#中async/await的工作原理是什么？如何避免死锁**  
- **原理**： async/await基于状态机实现异步操作，不会阻塞主线程  
- **避免死锁**：在异步方法中全程使用await，避免.Result或.Wait();配置ConfigureAwait(false)忽略同步上下文。  

**C#中的 async/await 底层是如何实现的？为什么说异步方法不会阻塞线程？**  
- **底层机制**
  - 编译器讲async方法转换为状态机（IAsyncStateMachine）
  - await 处生成代码检查异步操作是否完成，若未完成则挂起方法并释放线程
- **非阻塞原理** 
  - 异步等待时，线程返回到线程池（如Task.Delay不占用线程）
  - 仅在异步操作完成后，由线程池恢复执行后续代码  
示例：
``` csharp
public async Task<string> FetchDataAsync(){
    var httpClient = new HttpClient();
    // 发起异步请求，不阻塞线程
    string result = await httpClient.GetStringAsync("https://example.com");
    return result; // 在异步操作完成后恢复执行
}
```

**如何实现线程安全的延迟初始化（Lazy Initialization）**  
- 使用Lazy<T>类：
  - 提供线程安全的延迟初始化（默认使用 LazyThreadSafetyMode.ExecutionAndPublication）
- 双重检查锁定（Double-Check locking）
  - 手动实现需结合volatile 和 lock 确保线程安全  
示例：
``` csharp
// 使用Lazy<T>
private static readonly Lazy<MySingleton> _lazyInstance = new Lazy<MySingleton>(() => new Singleton());

public static MySingleton Instance => _lazyInstance.Value;

// 手动双重检查锁定
private static volatile MySingleton _instance;
private static readonly object _lock = new object();

public static MySingleton Instance{
    get {
        if(_instance == null){
            lock(_lock){
                if(_instance == null){
                    _instance = new MySingleton();
                }
            }
        }
        return _instance;
    }
}

```

## 详细说明一下状态机机制？   




**协变和逆变**  
在C#中，协变和逆变能够实现数组类型、委托类型和泛型类型参数的隐式引用转换。协变保留分配兼容性，逆变则与之相反。
- 协变：允许使用派生程度更高的类型（如IEnumerable<Derived> 赋值给IEnumerable<Base>）
- 逆变：允许使用派生程度更低的类型（如Action<Base> 赋值给 Action<Derived>）
- 实现方式：在泛型接口中使用out（协变）和in(逆变)关键字  

示例：
``` csharp
// 协变接口
public interface IProducer<out T>{
  T Produce();
}
```

