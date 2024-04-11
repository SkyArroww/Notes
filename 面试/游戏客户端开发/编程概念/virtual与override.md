在函数的声明中，当有“virtual”修饰的时候，和没有virtual有什么区别呢？最重要的一点就是调用实例的函数是在编译的时候确定还是在运行的时候确定，virtual函数是在运行的时候来确定具体调用哪个类。这个特性是和父子类继承息息相关的。

这儿有个例子，在网上很多地方被转载，我稍微扩展了一下：
```c
1. using System;  
2. namespace Smz.Test  
3. {  
4.     class A  
5.     {  
6.         public virtual void Func() // 注意virtual,表明这是一个虚拟函数  
7.         {  
8.             Console.WriteLine("Func In A");  
9.         }  

11.         public void Non_virtual()  
12.         {  
13.             Console.WriteLine("Non virtual func in A");  
14.         }  
15.     }  
16.     class B : A // 注意B是从A类继承,所以A是父类,B是子类  
17.     {  
18.         public override void Func() // 注意override ,表明重新实现了虚函数  
19.         {  
20.             Console.WriteLine("Func In B");  
21.         }  

23.         public void Non_virtual()  
24.         {  
25.             Console.WriteLine("Non virtual func in B");  
26.         }  
27.     }  
28.     class C : B // 注意C是从A类继承,所以B是父类,C是子类  
29.     {  
30.         public void Non_virtual()  
31.         {  
32.             Console.WriteLine("Non virtual func in C");  
33.         }  
34.     }  
35.     class D : A // 注意D是从A类继承,所以A是父类,D是子类  
36.     {  
37.         public new void Func() // 注意new ，表明覆盖父类里的同名类，而不是重新实现  
38.         {  
39.             Console.WriteLine("Func In D");  
40.         }  

42.         public new void Non_virtual()  
43.         {  
44.             Console.WriteLine("Non virtual func in D");  
45.         }  
46.     }  
47.     class program  
48.     {  
49.         static void Main()  
50.         {  
51.             A a;         // 定义一个a这个A类的对象.这个A就是a的申明类  
52.             A b;         // 定义一个b这个A类的对象.这个A就是b的申明类  
53.             A c;         // 定义一个c这个A类的对象.这个A就是c的申明类  
54.             A d;         // 定义一个d这个A类的对象.这个A就是d的申明类  
55.             a = new A(); // 实例化a对象,A是a的实例类  
56.             b = new B(); // 实例化b对象,B是b的实例类  
57.             c = new C(); // 实例化c对象,C是c的实例类  
58.             d = new D(); // 实例化d对象,D是d的实例类  
59.             a.Func();    // 执行a.Func：1.先检查申明类A 2.检查到是虚拟方法 3.转去检查实例类A，就为本身 4.执行实例类A中的方法 5.输出结果 Func In A  
60.             b.Func();    // 执行b.Func：1.先检查申明类A 2.检查到是虚拟方法 3.转去检查实例类B，有重写的 4.执行实例类B中的方法 5.输出结果 Func In B  
61.             c.Func();    // 执行c.Func：1.先检查申明类A 2.检查到是虚拟方法 3.转去检查实例类C，无重写的 4.转去检查类C的父类B，有重载的 5.执行父类B中的Func方法 5.输出结果 Func In B  
62.             d.Func();    // 执行d.Func：1.先检查申明类A 2.检查到是虚拟方法 3.转去检查实例类D，无重写的（这个地方要注意了，虽然D里有实现Func()，但没有使用override关键字，所以不会被认为是重写） 4.转去检查类D的父类A，就为本身 5.执行父类A中的Func方法 5.输出结果 Func In A  
63.             D d1 = new D();  
64.             d1.Func(); // 执行D类里的Func()，输出结果 Func In D  

66.             a.Non_virtual(); //执行A类的Non_virtual  
67.             b.Non_virtual(); //执行A类的Non_virtual  
68.             c.Non_virtual(); //执行A类的Non_virtual  
69.             d.Non_virtual(); //执行A类的Non_virtual  
70.             d1.Non_virtual(); //执行D类的Non_virtual  

72.             Console.ReadLine();  
73.         }  
74.     }  
75. }
```

 

具体在检查的时候，遵循的规则，已经有人在网上做了很详细的阐述，我在这里引用一下：

虚拟函数从C#的程序编译的角度来看，它和其它一般的函数有什么区别呢？一般函数在编译时就静态地编译到了执行文件中，其相对地址在程序运行期间是不发生变化的，也就是写死了的！而虚函数在编译期间是不被静态编译的，它的相对地址是不确定的，它会根据运行时期对象实例来动态判断要调用的函数，其中那个申明时定义的类叫申明类，那个执行时实例化的类叫实例类。

如：飞禽 bird = new 麻雀();  
那么飞禽就是申明类，麻雀是实例类。

具体的检查的流程如下

1、当调用一个对象的函数时，系统会直接去检查这个对象申明定义的类，即申明类，看所调用的函数是否为虚函数；

2、如果不是虚函数，那么它就直接执行该函数。而如果有virtual关键字，也就是一个虚函数，那么这个时候它就不会立刻执行该函数了，而是转去检查对象的实例类。

3、在这个实例类里，他会检查这个实例类的定义中是否有重新实现该虚函数（通过override关键字），如果是有，那么OK，它就不会再找了，而马上执行该实例类中的这个重新实现的函数。而如果没有的话，系统就会不停地往上找实例类的父类，并对父类重复刚才在实例类里的检查，直到找到第一个重载了该虚函数的父类为止，然后执行该父类里重载后的函数。

在上面的规则中，可以看到，如果子类没有override的修饰，那么就算父类是virtual的方法，子类的方法也无法被调用，而会去它的父类中找override的方法，直到找到祖先类。所以在面向对象的开发过程中，如果要实现Dependency Injection、IoC等设计模式，就必须非常留意类设计中继承方法的声明，否则很可能导致实际的程序运行与预期不符。