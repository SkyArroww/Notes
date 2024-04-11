> 闭包作为前端面试的必考题目，常让1-3年工作经验的Javascripter感到困惑，其实C#语言也有闭包。

今天我们深入聊一聊[闭包]， 查缺补漏！

1. C#闭包的实现 · 庖丁解牛
2. 跨语言 · 追本溯源
    - 一等函数
    - 自由变量
    - 词法作用域
3. 闭包与线程结合

---

# 1. C#闭包： 庖丁解牛

一个闭包就是一个“捕获”或“携带”了其生成的环境中、所引用的自由变量的函数。  
这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。

ini

复制代码

 ```c#
 static void Closure()
 {
      var x = 1;
      Action action= () =>
         {
             var y = 1;
             var result = x + y;
             Console.WriteLine(result);
             x++;
         };
      action();
      action();
}
 //  调用函数输出
  2
  3

```
 
我们首先定义了一个委托action，它引用了“x”变量（x变量既不是入参，也不是委托函数内的局部变量）， 这个变量将被action"捕获”，被自动添加到action 的运行环境中了。

当我们执行action时，原始的“x”已经脱离了它被引用时的作用域环境，但是两次执行能输出2,3 说明它脱离原引用环境仍然能用。

---

当你在代码调试器（debugger）里观察“action”时，会发现很有趣的事情，可以看到一个**Target属性**，里面封装了捕获的x变量：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/770d1eb762a14d55aea3f349e51471aa~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1276&h=381&s=90389&e=png&b=f5f4f4)

> 实际上，委托，匿名函数和lambda都是继承自[Delegate类](https://link.juejin.cn?target=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fdotnet%2Fapi%2Fsystem.delegate%3Fview%3Dnet-6.0 "https://docs.microsoft.com/en-us/dotnet/api/system.delegate?view=net-6.0")，只有两个属性
> 
> - Method：方法执行体
> - Target：当前委托执行的对象，包含了关键的自由变量. ( 也就是将捕获自由变量的过程封装成了对象 )。 Action函数的执行时空和 Action捕获的自由变量所在的作用域 不是一个时空。

### 至此可以窥见“闭包”的实质：

- 在实现了Delagate的封装类上执行方法体， 我们每次执行委托，实际是是执行封装类上的实例方法（Method属性），提前捕获的自由变量被存储在封装类的Target属性。
    
- **从我们打debugger端点的时机看，Action执行前已经捕获到了自由变量， 这个观点也很重要**。
    

---

闭包是跨越语言的设计， 至少我知道 Javascript C# 都有闭包。

# 2. 追本溯源

闭包是词法闭包的简称，维基百科上是这样定义的：  
“**在计算机科学中，闭包是在词法环境中绑定自由变量的头等函数**”。

### 头等函数

头等函数( First Class)意味着**语言将其视为第一类数据类型的函数**, 意味着你可以将函数分配给一个变量(或作为参数传递），然后像正常函数一样调用。

很明显，在C#中我们常使用的匿名函数、lambda表达式都是头等函数。

typescript

```c#
Func<string,string> myFunc = delegate(string var1)
                                {
                                    return "some value";   
                                };
Func<string,string> myFunc = var1 => "some value";  

string myVar = myFunc("something");

```

### 自由变量

自由变量只是一个在函数中被引用的变量，它不是函数的参数也不是函数的局部变量。

```c#
var myVar = "this is good";

Func<string,string> myFunc = delegate(string var1)
                                {
                                    return var1 + myVar;   
                                };

```

词法作用域引用的自由变量，**注意，是引用自由变量，并不是使用当时自由变量的值**。

☺️通俗点， 就是告知这个变量环境，我这个匿名函数等会执行时要用到这个变量；如果我没被销毁，你不能销毁我引用的自由变量。

C#自由变量会在原词法作用域被捕获进 闭包。

我们再回过头来看一个结合了线程的闭包面试题。

# 3. 闭包结合线程

```c#
 static void Closure1()
{
    for (int i = 0; i < 5; i++)
    {                 
         Task.Run(()=> Console.WriteLine(i));
    }
 }
//  输出数字是不固定的：
3
3
3
5
5
 

```
 

也不一定是
```c#
5
5
5
5
5

```

并不是预期的 0.1.2.3.4.

for循环，快速开启了5个Task任务，每个任务引用的i就是自由变量； i相对于这5个任务就成了 并发访问的全局变量。

5个任务捕获的值是不固定的。

**加上临时变量就能输出乱序的0,1,2,3,4。**

这是因为在for循环内，每次都有一个局部变量j（拷贝了i的值），这样每个任务执行环境均维护了一个变量j， 这个j不是全局变量；

任务乱序执行时依旧能获取本任务绑定的自由变量值j。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d40074172b7c40058fbcd7986d885f0f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=598&h=293&s=94621&e=png&b=fcf3f0)

---

或者可以换成foreach，foreach相当于内部给你整了一个局部变量。

```c#
        var ss = new int[] { 0, 1, 2, 3, 4 };
        foreach (var item in ss)
        {
            Task.Run(() => Console.WriteLine(item));
        }

```

## 总结

本文屏蔽语言差异，理清了[闭包]的概念核心： 头等函数、自由变量，不仅能帮助我们应对多语种有关闭包的面试题， 也帮助我们了解[闭包]在通用语言中的设计初衷。

另外我们通过C# 调试器巩固了Delegate 抽象类，这是lambda表达式,委托，匿名函数的**底层抽象数据结构类**，包含两个重要属性 Method Target，分别表征了方法执行体、当前委托作用的类对象，

可想而知，其他语言也是通过这个机制捕获闭包当中的自由变量。

## 20231206 补充一个golang gin框架闭包的应用

gin 框架中中间件的默认形态是：

go

复制代码

`package middleware func AuthenticationMiddleware(c *gin.Context) {    ...... } //  Use方法的参数签名是这样：  type HandlerFunc func(*Context)， 不支持入参 router.Use(middleware.AuthenticationMiddleware)`    

这种形态貌似不支持给中间件传参数。 闭包提供了这一能力。

go

复制代码

`func Authentication2Middleware(log *zap.Logger) gin.HandlerFunc  {      return func(c *gin.Context) {           ...    这里面可以利用log 参数。      } } var logger  *zap.Logger api.Use(middleware.Authentication2Middleware(logger))`

  

作者：掘金码甲哥  
链接：https://juejin.cn/post/7167641621515730981  
来源：稀土掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。