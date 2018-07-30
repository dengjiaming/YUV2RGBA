# 接入VA模块
Androidmanifest声明，继承BaseFloatViewService，添加悬浮窗。
![](http://p66upaccy.bkt.clouddn.com/15325721450435.jpg)

## 启动游戏。
![](http://p66upaccy.bkt.clouddn.com/15325903454253.jpg)

## 唤起x进程。
![](http://p66upaccy.bkt.clouddn.com/15325935367789.jpg)

## 极客模式script、float、coolplay三大进程之间的通信

#### float
FloatProcessContentProvider（float进程的组件）保存IFloatClient的实现对象BaseFloatClientImpl，通过FloatProcessContentProvider获取BaseFloatClientImpl（client对象）即可与float进程通信。
![](http://p66upaccy.bkt.clouddn.com/15326574189509.jpg)

#### script
ScriptProcessContentProvider（script进程的组件）保存IScriptClient的实现对象ScriptClientManager，通过ScriptProcessContentProvider获取ScriptClientManager（client对象）即可与script进程通信。
![](http://p66upaccy.bkt.clouddn.com/15326622226791.jpg)

### 通信例子

#### coolplay -->> float
主进程通过FloatProcessContentProvider获取BaseFloatClientImpl（client对象）与悬浮窗进程通信。
![](http://p66upaccy.bkt.clouddn.com/15326629966238.jpg)

#### coolplay -->> script
主进程通过点击事件生成FloatScriptDetailView，调用onRequestScriptInfo方法。
![](http://p66upaccy.bkt.clouddn.com/15326640723837.jpg)

最后通过ScriptProcessContentProvider获取BaseFloatClientImpl（client对象）与悬浮窗进程通信。
![](http://p66upaccy.bkt.clouddn.com/15326639795513.jpg)


#### script -->> float
script进程通过FloatProcessContentProvider获取BaseFloatClientImpl（client对象）与悬浮窗进程通信。

![](http://p66upaccy.bkt.clouddn.com/15326632038526.jpg)
![](http://p66upaccy.bkt.clouddn.com/15326632123917.jpg)

#### float -->> script
float进程通过ScriptProcessContentProvider获取ScriptClientManager（client对象）与script进程通信。
![](http://p66upaccy.bkt.clouddn.com/15326627762620.jpg)

## 通用模式pN进程调用悬浮窗、脚本

#### pN -->> x
在pN进程创建之后通过BinderContentProvider（x进程的组件）与x进程进行通信，与IFloatClient、IScriptClient的实例进行绑定。
![](http://p66upaccy.bkt.clouddn.com/15326569557532.jpg)

#### 脚本
BinderContentProvider（x进程的组件）保存IScriptClient的实现对象ScriptClientManager，通过BinderContentProvider获取ScriptClientManager（client对象）即可调用脚本。
![](http://p66upaccy.bkt.clouddn.com/15326643095020.jpg)

#### 悬浮窗
BinderContentProvider（x进程的组件）保存IFloatClient的实现对象CoolplayFloatClientImpl，通过BinderContentProvider获取CoolplayFloatClientImpl（client对象）即可调用悬浮窗。
![](http://p66upaccy.bkt.clouddn.com/15326646055267.jpg)

### 调用例子

#### 脚本 -->> 悬浮窗
当脚本需要与悬浮窗通信时，通过BinderContentProvider（x进程的组件）与x进程进行通信，获取IClientClient的实例进行通信。

![](http://p66upaccy.bkt.clouddn.com/15326632038526.jpg)
![](http://p66upaccy.bkt.clouddn.com/15326632123917.jpg)

#### 悬浮窗 -->> 脚本
当悬浮窗需要与脚本通信时，通过BinderContentProvider（x进程的组件）与x进程进行通信，获取IScriptClient的实例进行通信。

![](http://p66upaccy.bkt.clouddn.com/15326627762620.jpg)


## 极客模式在主进程创建script进程与float进程

极客模式多开APP时会做一次保护，检查是否已经存在script进程和float进程，有的话将其杀死。
![](http://p66upaccy.bkt.clouddn.com/15326033268780.jpg)

FloatProcessContentProvider（float进程的组件）保存IFloatClient的实现对象BaseFloatClientImpl，主进程通过FloatProcessContentProvider获取BaseFloatClientImpl（client对象）与float进程通信。打开悬浮窗，在BaseFloatViewService。onStartCommand添加悬浮窗。

![](http://p66upaccy.bkt.clouddn.com/15325948284402.jpg)
![](http://p66upaccy.bkt.clouddn.com/15325952891460.jpg)

添加悬浮窗之后，SmallPointPresenter。start会在float进程中重新创建script进程。
![](http://p66upaccy.bkt.clouddn.com/15326032659589.jpg)

## 通用模式在pN进程创建悬浮窗与脚本

通用模式在APP多开之后在pN进程添加悬浮窗，启动脚本。
![](http://p66upaccy.bkt.clouddn.com/15325978665457.jpg)
