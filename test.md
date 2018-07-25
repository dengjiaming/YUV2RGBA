# 简析分析VA启动APP的逻辑
#### 首先我们分析Application所做的准备工作。
![](http://p66upaccy.bkt.clouddn.com/15324328819885.jpg)

如上图所示，ContextImpl也继承于Context，而Activity、Application、Service的父类ContextWrappper是使用了装饰模式，ContextWrappper（包括Activity、Application、Service）的多种方法（比如getApplicationContext）其实都是由ContextImpl去实现的。但是ContextImpl!=applicationContext。


![Free-Converter.com-pasted_graphic_1-3115234](http://p66upaccy.bkt.clouddn.com/Free-Converter.com-pasted_graphic_1-3115234.jpg)
上图中，第一个参数为ContextImpl的值，第二个参数为applicationContext值，第三个参数为应用所在进程的名字。由上图可知，酷玩中所有进程的ContextImpl都是同一个值，applicationContext也基本相同（有时候会出现不同的Context，不太清楚）。


![Pasted Graphic 3](http://p66upaccy.bkt.clouddn.com/3.jpg)

在这一步中会调用VirtualCore，并把ContextImpl、酷玩的包名等信息保存在Virtual Core。每个进程都有对应的VirtualCore，调用VirtualCore时判断当前是什么进程。x进程调用各种系统的SystemService所需的上下文均由ContextImpl提供。


![Pasted Graphic 5](http://p66upaccy.bkt.clouddn.com/5.jpg)

MethodInvocationProxy类负责获取要hook的对象以及通过动态代理获取要hook的新对象做相应的业务逻辑，而MethodInvocationStub实现了动态代理所需要的HookInvocationHandler，具体的动态代理业务逻辑交给MethodProxy处理。这就是InvocationStubManager初始化所做的内容。另外，IPCBus将跨进程的binder对象保存在map中。


![Pasted Graphic 6](http://p66upaccy.bkt.clouddn.com/6.jpg)

VirtualApp为多开的APP做监听事件，在APP打开时做相应的初始化操作。


![Free-Converter.com-pasted_graphic_2-61585356](http://p66upaccy.bkt.clouddn.com/Free-Converter.com-pasted_graphic_2-61585356.jpg)

BinderContentProvider用于跨进程通信，将binder对象传过去。另外因为BinderContentProvider的x进程中的组件，因此就会打开新的进程x。

#### 下面是就是启动APP的逻辑。

![Pasted Graphic 7](http://p66upaccy.bkt.clouddn.com/7.jpg)

启动APP的入口在ScriptRuntimeManager类中，如果是极客模式就打开本地的APP，其他模式就在VA中打开APP。


openAppAutomatically最后会调用到这里，
![Pasted Graphic 8](http://p66upaccy.bkt.clouddn.com/8.jpg)

ServerInterface通过反射把类的方法全部保存在map中，保存是在x进程中进行的，IPCBus对其进行管理。因此，通过IPC跳转到x进程进行startActivity。


![Pasted Graphic 9](http://p66upaccy.bkt.clouddn.com/9.jpg)

在这方法中调用了三次startActivityProcess方法，第一次调用startActivityProcess就是第一次打开APP的时候，即使当前class的TaskAffinity和要打开class的TaskAffinity相同，仍然会新建新的Task，并且StubActivityRecord把实际要打开的intent保存起来。接下来APP打开新的activity会先通过第二次startActivityProcess把实际要打开的intent保存起来，第三次startActivityProcess判断destIntent是否为空值，然后打开新的activity。


![Pasted Graphic 10](http://p66upaccy.bkt.clouddn.com/10.jpg)
StubActivityRecord把Intent保存的步骤。

接下来要通过IInjector类进行Hook，新开Activity的逻辑主要涉及三个IInjector类：ActivityManagerStub、HCallBackStub、AppInstrumentation。
![Pasted Graphic 11](http://p66upaccy.bkt.clouddn.com/11.jpg)

又上文所知，ActivityManagerStub通过MethodInvocationStub实现动态代理，而业务逻辑在MethodProxy中实现。而打开Activity的逻辑在ActivityManagerStub\$StartActivity类中。ActivityManagerStub替换AMS，通过ActivityManagerStub\$StartActivity类检验当前intent包名是否与酷玩的包名一样。


![Pasted Graphic 12](http://p66upaccy.bkt.clouddn.com/12.jpg)

而HCallBackStub替换mH，修改mH的handleMessage，把StubActivityRecord保存的intent替换为当前intent，并且通过bindApplication调用hAppInstrumentation。


![Pasted Graphic 14](http://p66upaccy.bkt.clouddn.com/14.jpg)
![Pasted Graphic 13](http://p66upaccy.bkt.clouddn.com/13.jpg)

AppInstrumentation替换mInstrumentation，没有修改mInstrumentation的callApplicationOnCreate方法，修改mInstrumentation的callActivityOnCreate，并且调用监听APP状态的方法。

