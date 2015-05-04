前言
---
本次在QCon上我讲解了Hybrid应用的质量保证实践。现在Hybrid应用被越来越多的企业所使用，虽然Hybrid应用给研发以及用户带来比较大的便利，不过对于保证Hybrid应用的质量却也是随之而来比较大的问题，所以我这里也是结合自己的实践来和大家讨论下这个问题。

常用工具
---
如下图所示我列了些在日常工作中使用的比较多的工具。
![Image text](https://raw.githubusercontent.com/monkeytest15/QCon_Docs/master/img/1.png)

这里着重说下`Charles`和`AnyProxy`。在日常的工作中我们测试`Hybrid`应用很大的一个重点就是不同网络状态的测试，虽然的确有部分问题是存在于不同的运营商网络，但大部分的测试还是通过模拟不同网速来进行测试。我们通常使用`Charles`来对应用进行抓包测试，关注的点有如下几个：
>单个场景对应的接口消耗时间

>单个场景对应的第一个接口开始调用的时间与最后一个接口开始调用的时间差

>单个场景中流量消耗数据

![Image text](https://raw.githubusercontent.com/monkeytest15/QCon_Docs/master/img/2.png)
>`Duration`即是第一个接口调用和最后一个接口调用之间的时间差

>`Durations`即是我们应用与服务器交互的接口所消耗的时间和

>`Combined`即是所有接口流量的消耗值
同时我们在测试这些场景的时候也会再去区分两个维度。分别是 **首次启动（无缓存)** 和 **非首次启动** ，接着我们可以给出如下的报告。
![Image text](https://raw.githubusercontent.com/monkeytest15/QCon_Docs/master/img/3.png)

Charles也能够很好的进行网络上的模拟。接入PC的代理之后，可以选择`throttle`的限速设置。
![Image text](https://raw.githubusercontent.com/monkeytest15/QCon_Docs/master/img/4.png)
>`Bandwidth`：设置网络的上下行速度，kbps与kb/s是8:1的关系

>`Utilisation`：设置之后可以模拟与丢包率相似的功能

>`Round-trip Latency`：模拟网络延迟

>`MTU`：限制每秒的传输的字节值

接着我还介绍了另外一个工具——`AnyProxy`，这是阿里巴巴开源的一个网络代理工具，该工具几乎包括很多抓包工具的功能并且能够在终端运行，`AnyProxy`很好的运用在了自动化测试中。我们来看下功能和优势
![Image text](https://raw.githubusercontent.com/monkeytest15/QCon_Docs/master/img/5.png)

API Automation
---
`Hybrid`应用一大特点就是客户端与服务端有着很多的交互，那接口着一层的测试就不可避免。服务端的接口测试目前公司里的做法已经非常成熟了，有一套自己编写的测试框架ATS，其核心是使用了testng并且与公司内的自动化平台，缺陷管理系统很好的结合在了一起。通过数据驱动的方式进行接口的自动化。但现在我们发现单纯的服务端的接口的质量保证无法满足我们的需求。因为服务端的接口自动化测试其本质是我们通过技术的手段模拟不同的数据进行网络请求，从而查看接口返回的数据是否正确等。而仅仅是这样的话，我们覆盖了服务端的逻辑，却没有覆盖客户端的逻辑。所以现在我们分别通过了`Android Junit Test`以及`OCUnit`等框架直接去调用客户端中发出请求的方法，从而验证服务端的返回以及客户端对于返回的数据解析是否正确。虽然也同样都是数据驱动的方式，但这样既保证了服务端的逻辑也完全的是从客户端的逻辑发出请求，两端都得到了保障

头疼的线上故障
---
现在很多的应用中除了自己的服务以外，还会接入很多其他第三方的服务。在应用研发过程中测试的时间包括灰度发布的时间也都是有限的，我们经常会碰见一些问题是在测试过程中并没有暴露出来，而等版本一上线之后，各式各样的问题就如潮水般出现，其中我们发现很大一部分是存在于第三方服务中。其实很多人会很头疼，原因是第三方服务并非自己公司的服务，并不能很好的进行控制，但一旦出了问题，对于用户而言用户只会认为是你应用的问题，而不是第三方的问题，很多时候`黑锅`就背起来了。所以公司测试团队就研发了一套巡检平台，针对应用中的第三方服务质量。
![Image text](https://raw.githubusercontent.com/monkeytest15/QCon_Docs/master/img/6.png)
>保证第三方服务是可用的

>能够自定义巡检时间

>支持邮件、短信等各种形式的报警

>对返回的html进行针对性的Dom解析
其实巡检本身是无法对服务质量进行提升的，但巡检得出的数据能够很好的作为我们去让第三方提升质量的依据。

动态更新
---
其实这已经在很多公司的应用中被使用了。比如点评很早就用来做线上的A B Test，其他很多公司也会使用这种动态更新的方式做hotfix。这里举个iOS中的例子，我们从服务端推送如下的一个脚本，那么应用的本地控件的实现就会被更新。详细可见：http://testerhome.com/topics/2041
```java
waxClass{"ViewController", UIViewController}

function viewDidLoad(self)
self.super:viewDidLoad(self)

local label = UILabel:initWithFrame(CGRect(0, 120, 320, 40))
label:setColor(UIColor:blackColor())
label:setText("Hello Wax!")
label:setTextAlignment(UITextAlignmentCenter)
local font = UIFont:fontWithName_size("Helvetica-Bold",50)
label:setFont(font)
self:view():addSubview(label)
end
```
不过类似的方式其目前也只能更新部分Native的控件，作为hotfix而言其稳定性也没有得到很好的保证。和鬼道等朋友交流了对react native的理解，我也总结下：
>react native相对来讲，目前看来是一个很好的解决方案，但是并非一定要和html5比谁好谁坏。

>react native并非银弹，总体认为，是否能够大范围使用还是要看产品架构，业务复杂程度等上下文。最终应该是native、html、react >native三者并行的状态，只不过是侧重的功能不同

>当然，由于现在react native的经验并非很多。但是按照lua之前的经验。动态的去调用生成本地的控件还是有一定的风险和不稳定性，而且也是针对特定的一些场景和控件。当然，我更多的觉得对于测试会是一个新的考验。

>同样的，react native本身的js其实是打在整个安装包内的，那么关于安全性的话。其实本身目前针对js就没有一个完全100%安全的方法，所以代码安全性上面可能也没有办法完全的保证。

>最后一点就是流量。怎么说呢。推脚本也好，或者以后动态更新也罢，其实都有流量上的消耗。这个其实我觉得如果运用频繁之后，流量上的消耗可能也是一个需要关注的点。

UI自动化
---
最终其实还是要回到这样一个老话题上——自动化。`Hybrid`应用的UI自动化可谓是非常难做，我在大会上也谈到了一个框架——`Appium`，虽然`Appium`对于`Hybrid`应用已经是目前行业中支持的很好的框架之一了，但是从UI自动化角度而言，依然存在着不稳定性以及性能低的问题。所以为了提升自动化测试的投入产出比，我们采用了容器直接跳转业务界面，并通过界面截图对比的方式来保证质量。一方面在自动化中不需要再去实现很复杂的业务流程代码，提升了自动化稳定性。另外一方面截图的对比也能够最大程度的去验证界面上的准确性。
![Image text](https://raw.githubusercontent.com/monkeytest15/QCon_Docs/master/img/7.png)

从框架上来讲的话我们除了做自动化以外，我们也希望能够同时获取到性能数据，所以最终使用了`Agent`+`AnyProxy`的方式来获取数据，部分性能数据还是编写了辅助的Apk来获取，其余使用`Android SDK`自带的方式进行获取。
![Image text](https://raw.githubusercontent.com/monkeytest15/QCon_Docs/master/img/8.png)

总结
---
移动互联网应用的技术可以说是日新月异，Hybrid模式肯定是一个过度。现在越来越多的人也开始关注应用的非功能性测试，详细可见我写的这份PPT：专项测试详细PPT可见：http://pan.baidu.com/s/1kTJzAmj。 
本次大会认识了很多移动互联网的朋友，得出一个结论——测试人员不能仅仅通过单纯的测试活动来保证质量了。其实很简单，我这次提出了性能监测，`Hybrid`的巡检，手Q的同学也是说到了性能的持续集成和分析等等。那么我们的测试已经变的多样性了，测试活动也不仅仅存在于研发过程中，甚至渗透到了每时每刻。那么其实移动无线现在产品的质量怎么做？就应该是各个方面去保证，通过我们想得到的各种手段，持续集成，自动化的用户舆情反馈分析，线下线上的数据搜集分析，专项测试，安全分析，风险评估等等等等，这一系列看似很多和测试没有关系的事情，其实对最终的质量都有着至关重要的效果















