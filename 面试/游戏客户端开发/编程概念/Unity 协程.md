## **一. Unity中使用协程**

### **1. 什么是协程**

游戏里面经常会出现类似的淡入淡出效果：“一个游戏物体的颜色渐渐变淡，直至消失。” 当你碰上了类似的需求，也许需要实现一个`Fade()`来实现这样的淡入淡出效果。

一个错误的实现是像这个样子的

```csharp
//错误实现
void Fade() 
{
    float alpha = 1.0f;
    while(alpha > 0)
    {
        alpha -= Time.deltaTime;
        Color c = renderer.material.color;
        c.a = alpha;
        renderer.material.color = c;
    }
}
```

这段代码的思路很简单：在函数中写一个循环，让渲染对象的alpha从1开始不断递减直至降为0。 但是，函数被调用后将运行到完成状态然后返回，函数内的一切逻辑都会在同一帧内完成。也就是说，上述`Fade`函数中的alpha从1递减到0的过程会在一帧内完成，如果你调用这个`Fade`函数，看到的将会是游戏物体瞬间消失，而不会渐渐淡出。

要解决这个错误也很简单，只需将函数改写成协程即可。

```csharp
//正确实现
IEnumerator Fade()
{
    float alpha = 1.0f;
    while(alpha > 0)
    {
        alpha -= Time.deltaTime;
        Color c = renderer.material.color;
        c.a = alpha;
        renderer.material.color = c;
        yield return null;//协程会在这里被暂停，直到下一帧被唤醒。
    }
}
​
//Fade不能直接调用，需要使用StartCoroutine(方法名(参数列表))的形式进行调用。
StartCoroutine(Fade());
```

像这种以`IEnumerator`为返回值的函数，在C#里面被称之为迭代器函数，在Unity里面也可以被称为协程函数。 当协程函数运行到`yield return null`语句时，协程会被暂停，unity继续执行其它逻辑，并在下一帧唤醒协程。

现在来说说什么是协程，Unity官方对协程的定义是这样的：

> A coroutine is like a function that has the ability to pause execution and return control to Unity but then to continue where it left off on the following frame.  
> By default, a coroutine is resumed on the frame after it yields but it is also possible to introduce a time delay using [WaitForSeconds]([https://docs.unity3d.com/ScriptReference/WaitForSeconds.html](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/ScriptReference/WaitForSeconds.html))

**简单的说，协程就是一种特殊的函数，它可以主动的请求暂停自身并提交一个唤醒条件，Unity会在唤醒条件满足的时候去重新唤醒协程。**

### **2. 如何使用**

`MonoBehaviour.StartCoroutine()`方法可以开启一个协程，这个协程会挂在该`MonoBehaviour`下。

在`MonoBehaviour`生命周期的`Update`和`LateUpdate`之间，会检查这个`MonoBehaviour`下挂载的所有协程，并唤醒其中满足唤醒条件的协程。

要想使用协程，只需要以`IEnumerator`为返回值，并且在函数体里面用`yield return`语句来暂停协程并提交一个唤醒条件。然后使用`StartCoroutine`来开启协程。

下面这个实例展示了协程的用法。

```csharp
IEnumerator CoroutineA(int arg1, string arg2)
{
    Debug.Log($"协程A被开启了");
    yield return null;
    Debug.Log("刚刚协程被暂停了一帧");
    yield return new WaitForSeconds(1.0f);
    Debug.Log("刚刚协程被暂停了一秒");
    yield return StartCoroutine(CoroutineB(arg1, arg2));
    Debug.Log("CoroutineB运行结束后协程A才被唤醒");
    yield return new WaitForEndOfFrame();
    Debug.Log("在这一帧的最后，协程被唤醒");
    Debug.Log("协程A运行结束");
}
​
IEnumerator CoroutineB(int arg1, string arg2)
{
    Debug.Log($"协程B被开启了，可以传参数，arg1={arg1}, arg2={arg2}");
    yield return new WaitForSeconds(3.0f);
    Debug.Log("协程B运行结束");
}
```

### **3. 协程的应用场景**

### **创建补间动画**

补间动画指的是在给定若干个关键帧中插值来实现的动画。 如：给定两个时间点的Alpha值，可以插值出一个淡入淡出的动画效果。

创建补间动画更常用的做法是使用Dotween插件。

### **打字机效果**

很多游戏的人物对话界面中，文字并不是一开始就显示在对话框中的，而是一个一个显示出来的。这种将文本一个一个字的显示出来的效果称之为打字机(Typewriter)。

使用协程，你可以每显示一个字符后等待若干时间，从而实现打字机效果。 b站上有一个基于协程的打字机效果的简单实现（[https://www.bilibili.com/video/BV1cJ411Y7F6](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1cJ411Y7F6)）

### **异步加载资源**

资源加载指的是通过IO操作，将磁盘或服务器上的数据加载成内存中的对象。资源加载一般是一个比较耗时的操作，如果直接放在主线程中会导致游戏卡顿，通常会放到异步线程中去执行。

举个例子，当你需要从服务器上加载一个图片并显示给用户，你需要做两件事情：

1. 通过IO操作从服务器上加载图片数据到内存中。
2. 当加载完成后，将图片显示在屏幕上。

其中，2操作必须等待1操作执行完毕后才能开始执行。

在传统的互联网应用中，一般会使用回调函数来实现类似功能：

```csharp
//伪代码
​
//提供给用户的接口
void ShowImageFromUrl(string url)
{
    LoadImageAsync(url, Callback); //开启一个异步线程来加载图像，加载完成后会自动调用回调函数
}
​
//回调函数
void Callback(Image image)
{
    Show(image);
}   
```

我们也可以改写成协程的形式：

```csharp
//伪代码
​
IEnumerator ShowImageFromUrl(string url)
{
    Image image = null;
    yield return LoadImageAsync(url, image); //异步加载图像，加载完成后唤醒协程
    Show(image);
}
```

使用协程来进行异步加载在Unity中是一个很常用的写法。资源加载是一个很重的话题，有兴趣的同学可以研究研究。 这里贴两个参考链接： [Unity官方的异步加载场景的示例](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/ScriptReference/SceneManagement.SceneManager.LoadSceneAsync.html) [倩女幽魂手游中的资源加载与更新方案](https://zhuanlan.zhihu.com/p/150171940)

### **定时器操作**

当你需要延时执行一个方法或者是每隔一段时间就执行某项操作时，可以使用协程。不过对于这种情况，也可以考虑写一个`TickManager`来管理定时操作。[Electricity项目中的定时器](https://link.zhihu.com/?target=https%3A//gitee.com/Taoist_Yu/electricity/blob/master/Assets/Scripts/Engine/CTickMgr.cs)

  

**思考：协程能做的Update都能做，那为什么我们需要协程呢？** **答：使用协程，我们可以把一个跨越多帧的操作封装到一个方法内部，代码会更清晰。**

### **4. 注意事项**

1. 协程是挂在MonoBehaviour上的，必须要通过一个MonoBehaviour才能开启协程。
2. MonoBehaviour被Disable的时候协程会继续执行，只有MonoBehaviour被销毁的时候协程才会被销毁。
3. 协程看起来有点像是轻量级线程，但是本质上协程还是运行在主线程上的，协程更类似于`Update()`方法，Unity会每一帧去检测协程需不需要被唤醒。一旦你在协程中执行了一个耗时操作，很可能会堵塞主线程。这里提供两个解决思路：(1) 在耗时算法的循环体中加入`yield return null`来将算法分到很多帧里面执行；(2) 如果耗时操作里面没有使用Unity API，那么可以考虑在异步线程中执行耗时操作，完成后唤醒主线程中的协程。

## **二. Unity协程的底层原理**

协程分为两部分，协程与协程调度器：协程仅仅是一个能够中间暂停返回的函数，而协程调度是在MonoBehaviour的生命周期中实现的。 准确的说，Unity只实现了协程调度部分，而协程本身其实就是用了C#原生的”迭代器方法“。

### **1. 协程本体：C#的迭代器函数**

> 许多语言都有迭代器的概念，使用迭代器我们可以很轻松的遍历一个容器。 但是C#里面的迭代器要屌一点，它可以“遍历函数”。

C#中的迭代器方法其实就是一个协程，你可以使用`yield`来暂停，使用`MoveNext()`来继续执行。 当一个方法的返回值写成了`IEnumerator`类型，他就会自动被解析成迭代器方法_（后文直接称之为协程）_，你调用此方法的时候不会真的运行，而是会返回一个迭代器，需要用`MoveNext()`来真正的运行。看例子：

```csharp
static void Main(string[] args)
{
    IEnumerator it = Test();//仅仅返回一个指向Test的迭代器，不会真的执行。
    Console.ReadKey();
    it.MoveNext();//执行Test直到遇到第一个yield
    System.Console.WriteLine(it.Current);//输出1
    Console.ReadKey();
    it.MoveNext();//执行Test直到遇到第二个yield
    System.Console.WriteLine(it.Current);//输出2
    Console.ReadKey();
    it.MoveNext();//执行Test直到遇到第三个yield
    System.Console.WriteLine(it.Current);//输出test3
    Console.ReadKey();
}
​
static IEnumerator Test()
{
    System.Console.WriteLine("第一次执行");
    yield return 1;
    System.Console.WriteLine("第二次执行");
    yield return 2;
    System.Console.WriteLine("第三次执行");
    yield return "test3";
}
```

- 执行`Test()`不会运行函数体，会直接返回一个`IEnumerator`
- 调用`IEnumerator`的`MoveNext()`成员，会执行协程直到遇到第一个`yield return`或者执行完毕。
- 调用`IEnumerator`的`Current`成员，可以获得`yield return`后面接的返回值，该返回值可以是任何类型的对象。

这里有两个要注意的地方：

1. `IEnumerator`中的`yield return`可以返回任意类型的对象，事实上它还有泛型版本`IEnumerator<T>`，泛型类型的迭代器中只能返回`T`类型的对象。Unity原生协程使用普通版本的`IEnumerator`，但是有些项目_（比如倩女幽魂）_自己造的协程轮子可能会使用泛型版本的`IEnumerator<T>`
2. 函数调用的本质是压栈，协程的唤醒也一样，调用`IEnumerator.MoveNext()`时会把协程方法体压入当前的函数调用栈中执行，运行到`yield return`后再弹栈。这点和有些语言中的协程不大一样，有些语言的协程会维护一个自己的函数调用栈，在唤醒的时候会把整个函数调用栈给替换，这类协程被称为**有栈协程**，而像C#中这样直接在当前函数调用栈中压入栈帧的协程我们称之为**无栈协程**。关于有栈协程和无栈协程的概念我们会在后文[四. 跳出Unity看协程](https://zhuanlan.zhihu.com/p/279383752/edit#%E5%9B%9B.%20%E8%B7%B3%E5%87%BAUnity%E7%9C%8B%E5%8D%8F%E7%A8%8B)中继续讨论

> Unity中的协程是无栈协程，它不会维护整个函数调用栈，仅仅是保存一个栈帧。

### **2. 协程调度：MonoBehaviour生命周期中实现**

仔细翻阅Unity官方文档中介绍`MonoBehaviour`生命周期的部分，会发现有很多yield阶段，在这些阶段中，Unity会检查`MonoBehaviour`中是否挂载了可以被唤醒的协程，如果有则唤醒它。

通过对C#迭代器的了解，我们可以模仿Unity自己实现一个简单的协程调度。这里以`YieldWaitForSeconds`为例

```csharp
// 伪代码
void YieldWaitForSeconds()
{
    //定义一个移除列表，当一个协程执行完毕或者唤醒条件的类型改变时，应该从当前协程列表中移除。
    List<WaitForSeconds> removeList = new List<WaitForSeconds>();
    foreach(IEnumerator w in m_WaitForSeconds) //遍历所有唤醒条件为WaitForSeconds的协程
    {
        if(Time.time >= w.beginTime() + w.interval) //检查是否满足了唤醒条件
        {
            //尝试唤醒协程，如果唤醒失败，则证明协程已经执行完毕
            if(it.MoveNext();)
            {
                //应用新的唤醒条件
                if(!(it.Current is WaitForSeconds))
                {
                    removeList.Add(it);
                       //在这里写一些代码，将it移到其它的协程队列里面去
                }
            }
            else 
            {
                removeList.Add(it);
            }
        }
    }
    m_WaitForSeconds.RemoveAll(removeList);
}
```

### **3. Unity协程的架构**

```text
YieldInstructionWaitForEndOfFrameWaitForFixedUpdateCoroutineWaitForSecondsAsyncOperation
```

基类：[YieldInstruction](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/ScriptReference/YieldInstruction.html) 其它所有协程相关的类都继承自这个类。Unity的协程只允许返回继承自`YieldInstruction`的对象或者`null`。如果返回了其他对象则会被当成null处理。

协程类：[Coroutine](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/ScriptReference/Coroutine.html) 你可以通过`yield return`一个协程来等待一个协程执行完毕，所以`Coroutine`也会继承自`YieldInstruction`。 `Coroutine`仅仅代表一个协程实例，不含任何成员方法，你可以将`Coroutine`对象传到`MonoBehaviour.StopCoroutine`方法中去关闭这个协程。

遗憾的是，Unity关于协程的这套都是在C++层实现的并且几乎没有暴露出C#接口，所以扩展起来会比较麻烦。

## **三. 扩展Unity的协程**

了解了协程的底层原理后，就可以尝试去扩展协程啦！！！

**为什么要扩展协程？** 那必然是Unity自带的协程系统无法满足我们的需求了，比如有的时候我们可能需要一个WaitForSceneLoaded或者WaitForAnimationDown之类东西，Unity就没有。

扩展协程有两个思路，一种是自己另外写一套，可以有高度的定制性；另一种是对Unity现有的协程系统进行封装，可以兼容Unity现有的WaitForXXX。

### **1. 思路一：另外写一套协程**

这种其实没什么说的，其实就是自己手写一个协程调度器，有点是可以高度定制调度器行为，缺点是不能兼容Unity自带的协程。

倩女幽魂的协程轮子是将调度器写到了`MonoBehaviour`的`Update`里面，但是你也可以自己在另一个线程开一个死循环来完成协程调度。可以在GitHub上找找别人的代码看看实现，网上那些讲Unity协程的帖子也有很多都实现了简易协程调度。

[Github上的某个C#框架下的协程模块，不依赖于Unity](https://link.zhihu.com/?target=https%3A//github.com/OnClick9927/IFramework_CS/tree/master/IFramework/Modules/Coroutine)

  

### **2. 思路二：在Unity协程的基础上进行封装**

由于Unity并未暴露出协程的扩展接口，所以不能直接在Unity的协程上进行扩展。 一个折衷的方案是，使用一个代理协程，这个代理协程被Unity管理，然后所有的用户级协程被这个代理协程管理。本质上是对Unity现有协程模块进行封装。

**Electiricy项目中的协程模块：**

在Electricity项目中，有个单例`CCoroutineMgr`用来管理所有的协程。 使用`CCoroutineMgr.Inst.StartCoroutine()`即可开启协程。

```csharp
//In CCoroutineMgr.cs
​
//所有扩展的WaitForXXX的接口
public interface IWaitable
{
    bool IsReady();
}
​
public class CCoroutineMgr : CSingletonBehaviour<CCoroutineMgr>
{
    //开启一个协程，讲用户传进来的iter传入协程调度器中，协程调度器本身也是一个协程，被Unity管理
    public new WaitForCoroutine StartCoroutine(IEnumerator iter)
    {
        WaitForCoroutine wait = new WaitForCoroutine();
        //开启协程调度器，使用base.StartCoroutine可以开启Unity自带协程
        Coroutine co = base.StartCoroutine(CoScheduler(iter, wait));
        mCoroutines.Add(wait, co);
        return wait;
    }
    
    //核心：协程调度
    private IEnumerator CoScheduler(IEnumerator iter, WaitForCoroutine wait)
    {
        while (iter.MoveNext())
        {
            var res = iter.Current;
            if (res is IWaitable) //iter中返回了一个IWaitable
            {
                IWaitable waitable = res as IWaitable;
                while (!waitable.IsReady())
                    yield return null;
            }
            else //iter中返回了一个Unity自带的协程条件
            {
                yield return res;
            }
        }
        mCoroutines.Remove(wait);
    }
}
```

[完整代码 CCoroutineMgr.cs](https://link.zhihu.com/?target=https%3A//gitee.com/Taoist_Yu/electricity/blob/master/Assets/Scripts/Engine/CCoroutineMgr.cs)

`IWaitable`是所有用户自定义`WaitForXXX`的公共接口；`WaitForCoroutine`实现了这个接口，用来表示某个协程有没有结束。

当开启协程时，用户会传进来一个迭代器方法`iter`，`iter`会被传进协程调度器中被调度器管理，而调度器会作为一个Unity原生协程被Unity管理。

在协程调度器`CoScheduler`中，当调度器被唤醒，会执行一次`iter`并检查`yield return`返回的结果`res`，如果`res`是一个`IWaitable`，就每帧检测`res`是否满足恢复条件；如果`res`不是`IWaitable`，就会将`res`作为Unity自带的协程唤醒条件返回。

使用单例的协程管理器可能会导致一些问题，考虑这样一个情况： 对象A请求`CCoroutineMgr`开启了一个协程C，后来对象A被销毁了而协程C还在运行，协程C会访问已经被销毁了的对象A，从而引发错误。

在使用单例的协程管理器时，应当注意在对象被销毁的时候将所有引用了这个对象的协程一并销毁。如果每个`MonoBehaviour`都写一套这样的逻辑可能会比较麻烦，Electricity项目中对`MonoBehavioiur`做了一些封装，可以自动管理协程的销毁。

**MemoBehaviour**

`MemoBehaviour`可以自动管理协程的销毁，实现比较简单，就直接上代码了。

```csharp
public class MemoBehaviour : MonoBehaviour
{
    private List<WaitForCoroutine> mCoroutines = new List<WaitForCoroutine>();
    private int mNextCoroutineClearCount = 10;
    
    public new WaitForCoroutine StartCoroutine(IEnumerator iter)
    {
        WaitForCoroutine wait = CCoroutineMgr.Inst.StartCoroutine(iter);
        mCoroutines.Add(wait);
        if(mCoroutines.Count >= mNextCoroutineClearCount)
        {
            ClearCoroutines();
            mNextCoroutineClearCount  = mNextCoroutineClearCount * 2 + 1;
        }
        return wait;
    }
​
    //清理已经执行完毕的协程。
    private void ClearCoroutines()
    {
        mCoroutines.RemoveAll((WaitForCoroutine wait) => wait.IsReady());
    }
}
```

[完整代码 MemoBehaviour.cs](https://link.zhihu.com/?target=https%3A//gitee.com/Taoist_Yu/electricity/blob/master/Assets/Scripts/Engine/MemoBehaviour.cs)

## **四. 跳出Unity看协程**

前面都是在Unity的基础上讲协程，实际上有很多开发框架中都有协程的概念，而且都有一些区别。这一节就跳出Unity，简单的介绍一下广义上的协程。

### **1. 进程，线程与协程**

> 进程是操作系统资源分配的基本单位 线程是处理器调度与执行的基本单位

这是操作系统书上对进程与线程的抽象描述。具体一点的说，进程其实就是程序运行的实例：程序本身只是存储在外存上的冷冰冰的二进制流，计算机将这些二进制流读进内存并解析成指令和数据然后执行，程序便成为了进程。

每一个进程都独立拥有自己的指令和数据，所以称为资源分配的基本单位。其中数据又分布在内存的不同区域，我们在C语言课程中学习过内存四区的概念，一个运行中的进程所占有的内存大体可以分为四个区域：栈区、堆区、数据区、代码区。其中代码区存储指令，另外三个区存储数据。

线程是处理器调度和执行的基本单位，一个线程往往和一个函数调用栈绑定，一个进程有多个线程，每个线程拥有自己的函数调用栈，同时共用进程的堆区，数据区，代码区。操作系统会不停地在不同线程之间切换来营造出一个并行的效果，这个策略称为时间片轮转法。

那么协程在其中又处于什么地位呢？ **一切用户自己实现的，类似于线程的轮子，都可以称之为是协程。**

C#中的迭代器方法是协程； Unity在迭代器的基础上扩展出来的协程模块是协程； 你在操作系统实验中模仿线程自己写出来的"线程"也是协程； ........

协程有什么样的行为，完全由实现协程的程序员来决定（_线程和进程都是操作系统中写死的_），这就导致了不同开发框架下的协程差别很大。有的协程有自己的函数调用栈，有的协程共用线程的函数调用栈；有的协程是单线程上的，有的协程可以多线程调度；有的协程和线程是一对多的关系，有的协程和线程是多对多的关系。

> 操作系统可以有多个进程 一个进程对应一个或多个线程 线程和协程的对应关系，由具体的开发框架决定

### **2. 不同框架下协程的共同点**

虽然不同开发框架下的协程各不一样，但是这些协程基本上还是有一些共性的

### **(1) 协程有yield和resume操作**

协程可以通过`yield`操作挂起，通过`resume`操作恢复。`yield`一般是协程主动调用，`resume`一般是调度器调用。 大多数协程库都支持这两个操作，无非是可能API的名字不一样。 比如C#中，`resume`操作就是`MoveNext`

### **(2) 协程调度是非抢占式的**

线程调度是抢占式的：操作系统会主动中断当前执行中的线程，然后把CPU控制权交给别的线程，就好像有很多线程去争抢CPU的控制权一样。

协程调度是非抢占式的：协程需要主动调用`yield`来释放CPU控制权，协程的运行中间不会被系统中断打断。

### **3. 如何在C语言中造一个协程**

 两个概念：  
1. PC指针：指向CPU下一条运行的指令
2. 程序运行上下文：这是一个比较宽泛的概念，表示程序运行所需要的数据，比如当前栈帧(无栈协程)或者整个函数调用栈(有栈协程)

  

协程的本质就是一个可以暂停执行的函数，使用`yiled`挂起，使用`resume`恢复。 只要在协程挂起的时候，保存当前的PC指针和程序运行上下文，然后在协程恢复的时候将保存的数据应用上，即可自己实现一个协程。 有兴趣的话可以去GitHub上找些别人自己写的协程看看源码，直接搜coroutine就好。

[云风大神的协程库——基于Unix下的ucontext函数簇](https://link.zhihu.com/?target=https%3A//github.com/cloudwu/coroutine)  
[sylar的协程库——基于ucontext，C++实现](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV184411s7qFp%3D26)  
[我的协程库——基于Windows下的纤程API](https://link.zhihu.com/?target=https%3A//github.com/Taoist-Yu/Coroutine_Win)

  

## **五. 参考资料**

1. [Unity官方的协程文档](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/Manual/Coroutines.html)
2. [unity3d 协程的初步理解 - 支持返回值/支持异常处理/支持泛型](https://link.zhihu.com/?target=https%3A//blog.csdn.net/ybhjx/article/details/55188777)
3. [倩女幽魂手游中的资源加载与更新方案](https://zhuanlan.zhihu.com/p/150171940)
4. [IFramework-CS的协程模块](https://link.zhihu.com/?target=https%3A//github.com/OnClick9927/IFramework_CS/tree/master/IFramework/Modules/Coroutine)
5. [云风大佬的blog](https://link.zhihu.com/?target=https%3A//blog.codingnow.com/2012/07/c_coroutine.html)
6. [sylar的服务端框架中的协程模块](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV184411s7qF%3Fp%3D26)