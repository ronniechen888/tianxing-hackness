#导语

笔者最近一直在研究iOS逆向工程，非常感谢狗神的小黄书，让我领略到了别样的风光。客官们也许会问：你研究的是iOS，为啥却搞起了mac os x软件的破解？原因很简单，ios是在mac os基础上衍生出来的，它们有很多相似处，而且在ios逆向中，掌握Hopper是尤为的重要，而且也是一大难点，所以笔者的本意正是为了研究Hopper。

#工具

接下来介绍下我所使用的工具：

1.[Hopper](https://www.hopperapp.com)-这是一款和IDA同样价值的工具，没用IDA是因为小黄书上调试的是32位程序，而这里调试的是64位程序，IDA免费版目前不支持64位。在笔者看来Hopper在mac下威力比IDA有过之而无不及，所以这也正是笔者这次尝试破解的目的，搞定它，重中之重。

2.[Cycript](http://www.cycript.org)-有过逆向iOS经验的同学肯定知道这款大名鼎鼎的进程注入利器，在这一次破解之旅中，它是关键，之所以能想到用它，完全是笔者个人的举一反三，大家想一想这款工具能这么简单的用在iOS上，对待权限更开放的OS X岂不是更Easy。

3.[Class-dump](http://stevenygard.com/projects/class-dump/)-一款能将可执行文件破壳而出，导出头文件的利器。

#目标

工具介绍完毕，那么我们今天hack的目标是哪一款软件呢？年费价值170元的-天行VPN，版本是V1(0)!如图0-1。可以看到，在时间过期的情况下点击连接，是无法连接vpn的，那么我们的目标就是能够免费无限制使用。神秘的逆向技术比魔术更为神奇，更为吸引人，那我现在就开始给大家讲解我破解这款vpn的思路。

![图0-1](http://upload-images.jianshu.io/upload_images/2197489-3190cf5188f1b64c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#观察与分析

#####思路1:

通过使用并且观察这款VPN,如果我们能无限免费领取时间，那么我们也可以无限制的去使用这款软件了。大家可以试一下，首次使用这款软件，大家可以点选【免费领取】，每天可以领一个小时，这一个小时你必须马上使用掉，不然1个小时后你都是不能再使用了。所以最初我尝试的方式是将【免费领取】这个按钮永远处于高亮状态，也就是当你免费领取一次时间以后，按钮依然处于高亮状态，你可以一直点一直点，我用Hopper把这个按钮setEnabled:NO，修改成了NOP，即这个按钮永远不可能处于不可用的状态。我很幸运的修改成功，但是可惜的是，时间并不能一直累加！原因是什么？增加时间的逻辑会在后台校验，也就是当我在获取一次时间后，后台会记录我的当前帐号今天是否增加过时间，如果有，后台会永远返回一个错误信息告诉客户端，这个人已经增加过时间了，不能再增加！所以很遗憾，这样的方式没有突破成功。这个思路我就不做截图演示了，大家可以在看完本文后自行尝试！

#####思路2:

我们能否直接修改点击【连接】按钮后这段回调逻辑，达到连接成功呢？有同学会说，如果服务器同样做了校验怎么办，确实那就看似无法突破成功了，但是无论成与不成，我们都应该去尝试，一个优秀的逆向工程师应该具备这样敢于尝试的勇气，无法走出这一步，你就无法成为一名Hacker！而恰恰这个思路正是我破解这款vpn的关键，接着我们该怎么走！

#实战操刀

如何才能找到【连接】按钮的这段回调逻辑呢？事实上就是我们得找到点击按钮的回调函数，如何去寻找呢，最初我是直接把可执行文件拖到Hopper 里面，看左边菜单找关键字硬猜，事实上这并不是一个明智的办法，更好的方法应该是先把可执行文件破头文件而出。接下来大家可以跟着我一起操作。

- **导出头文件**

右键点击天行客户端，点选【显示包内容】，找到图1-2的可执行文件。接下来我们执行class-dump命令，至于class-dump的安装，大家可以谷歌搜索其安装方法。将可执行文件的路径和头文件输出路径写入命令行。点击回车，我们就可以看到在目标路径中已输出图1-3所示的头文件。**神马？**上面的不是大名鼎鼎的AFNetworking框架里的头文件吗！你没看错，就是这么神奇，这里要说明下，因为这个客户端是从官网客户端所下载，所以不需要砸壳，如果是经过appstore所下载的mac应用包，它的可执行文件会经过appstore加密，你可以理解给它加上了一层厚厚的外壳，在这种情况下还需要用到砸壳技术，这里不是本文的重点，笔者就不详细说明了。
```

class-dump -S -s -H  /Users/Ronnie/Desktop/tianxingmac/tianxingmac.app/Contents/MacOS/tianxingmac -o /Users/Ronnie/Desktop/tianxingHeaders/

```
![图1-1](http://upload-images.jianshu.io/upload_images/2197489-d8eae23923242717.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![图1-2](http://upload-images.jianshu.io/upload_images/2197489-915ec2494959151c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![图1-3](http://upload-images.jianshu.io/upload_images/2197489-948b77096ff6d1e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **挖掘头文件里的蛛丝马迹**
在拿到这些头文件后，大家可以好好观察下这些头文件里的内容，如果你是一名合格的iOS开发者，你应该很快就能找到一些很有价值的线索，笔者在一番探索之后，最后锁定住了`MainViewController.h`这个文件，很显然，在我们平时开发中，这个控制器的名字常常被用于主界面的控制器，打开这个文件，浏览一番后，我们发现如下两个对象很是可疑啊，`NSButton *_connectBtn;`和`NSButton *_disconnectBtn;`,大家已经看出端倪了吧，这两个对象不就是上面图0-1里的【连接】和【断开】按钮吗？没错，那就说明我们所寻找的头文件是没问题的，那么接下来是否可以大胆的猜测一下，【连接】按钮的回调函数是否也在这个头文件里，如果在，我们该怎么确定该函数就是所要寻找的目标函数呢？
![图2-1](http://upload-images.jianshu.io/upload_images/2197489-3bdb38f0613d1036.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **神奇的Cycript协助我们找到元凶**
Cycript是大神saurik开发的一个非常强大的工具,可以让开发者在命令行下和应用交互,在运行时查看和修改应用。让我们看看它是怎么帮助笔者锁定深一层目标的。继续浏览`MainViewController.h`，笔者发现`connectCallBack:`这个函数很有可能就是【连接】按钮的回调函数。
![图3-1](http://upload-images.jianshu.io/upload_images/2197489-0e1f946bd7ad4572.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
为了进一步确定该函数，拿出我们的利刃来做如下操作：
1.打开我们的控制台程序，输入命令：
```
ps -e | grep tianxing
```
回车后，我们拿到了下图两个进程，我们关注的是1595那个进程，后面显示了这个进程可执行文件的路径。
![图3-2](http://upload-images.jianshu.io/upload_images/2197489-2ed6a97bd9945297.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.继续输入命令,这里我们可以既输入进程的可执行文件名字，也可以输入进程的pid号，任选其一：
```
/Users/Ronnie/Downloads/cycript_0/cycript -p tianxingmac
/Users/Ronnie/Downloads/cycript_0/cycript -p 1595
```
我们敲下回车后，系统会提示我们授权，因为我们利用cycript注入进程内存是需要得到系统授权的，注入成功后界面会变成这样：
![图3-3](http://upload-images.jianshu.io/upload_images/2197489-e1431ba21c069d9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这是cycript脚本界面，要了解cycript具体语法和使用要领可以前去[cycript官网](http://www.cycript.org)查阅，这里我只讲解我用到的命令。
3.在脚本控制器中输入：
```
choose(MainViewController)
```
choose这个函数会列出当前进程中MainViewController类的所有对象，当然我们当前进程只有一个此类对象，如下图：
![图3-4](http://upload-images.jianshu.io/upload_images/2197489-9afbe5bf9cdc47ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
0x7fd846549820就是MainViewController的对象地址，我们可以直接把它看作是当前内存中的MainViewController类对象。
4.复制该对象地址，输入下一个命令：
```
[#0x7fd846549820 connectCallBack:nil]
```
`connectCallBack`既是我们要测试的回调函数，对象地址前记得输入#号，中扩号正是我们熟悉的OC调用方式，输入nil的原因是，在这个函数里我们应该是不需要用到参数的，(id)sender大家一定知道，如果非要传入参数，我们可以猜测这个对象应该是【连接】按钮对象，对象地址也可以用前面的方法获取到。理解这串命令含义后，我们敲下回车，神奇的事出现了，我们居然进行了隔空操作，天行vpn居然弹出了帐号已过期的提示框，如下动态图。到了这里，大家应该理解，我们也能确定了，函数`connectCallBack`就是我们要寻找的突破口。
![3-5](http://upload-images.jianshu.io/upload_images/2197489-4feb90f4864de6c4.gif?imageMogr2/auto-orient/strip)

- **神秘的Hopper犹如瑞士军刀直刺目标心脏**
1.将天行vpn中的可执行文件-tianxingmac直接拖入已打开的Hopper界面,完成加载后整个界面如下：
![图4-1](http://upload-images.jianshu.io/upload_images/2197489-895f8b7179987c5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
有没有被晕倒了？乱七八糟的汇编代码闪入眼前，蛋疼，我该怎么做，难道就此放弃？莫慌，笔者也是汇编战斗力不到5的渣渣，但即使我们很渣，也得鼓起勇气告诉自己其实我并不那么渣，我应该还是能搞出点名堂的，想要踏破汇编的门槛，勇气是第一要素。
2.这里关于Hopper的使用指南大家可以参考官方的帮助文档，这里我就不做具体介绍了，让我们快快进入战斗模式吧。首先我们在左边的搜索栏里直接输入`connectCallBack`，按下回车，我们可以看到我们直接把这个回调给索引出来了，单机这个回调，我们可以看到中间的汇编窗口已经直接切换到这个回调的入口处，如下图：
![图4-2](http://upload-images.jianshu.io/upload_images/2197489-44a23f00c55689be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看不懂啊，都是寄存器，这个函数到底啥逻辑，哥已经被弄晕了，别慌，看Hopper顶部有四个模式按钮，如下图，我们可以切换不同的模式看看。
![图4-3](http://upload-images.jianshu.io/upload_images/2197489-c7a759c0b81f3ae0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们当前处于汇编模式状态，我们可以切换到流程跳转模式看看，如下图，大家可以试试滚轮或者触摸板缩小放大页面看看，大家会发现有很多箭头，而这些箭头都是跳转逻辑，这里笔者圈出了这个回调的第一个跳转代码,汇编cmp指令表示将al寄存器中的值与0x0作比较，而je这条指令的意思是jump if equal,也就是当al中的值为0时，流程会往绿色箭头处跳转，大家可以自己分析看看。
![4-4](http://upload-images.jianshu.io/upload_images/2197489-e9337018f07e863d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
接下来再点点看第三个按钮，我们会看到窗口中出现一些伪oc代码，可读性一下提高了不少唉！如下图，为了更好的讲解清楚，笔者直接把代码粘出来。这些伪代码，看似很难懂，你想理解透彻每一行代码，似乎不可能，但是我们可以看到关键点，笔者很快的发现到关键点在`alert WithTitle`这样一个方法，仔细看一下，我们不难发现，这个里面有多处校验，如果校验失败则会发送这个弹框处理，此处笔者找到了三个`alert WithTitle`，大家仔细阅读下代码。
![图4-5](http://upload-images.jianshu.io/upload_images/2197489-9aaffde2eab91ec7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
void -[MainViewController connectCallBack:](void * self, void * _cmd, void * arg2) {
    var_8 = self;
    objc_storeStrong(0x0, arg2);
    rax = [var_8 ping];
    rax = [rax retain];
    var_30 = rax;
    [rax stop];
    [var_30 release];
    rax = var_8 + *_OBJC_IVAR_$_MainViewController._daylabel;
    rax = objc_loadWeakRetained(rax);
    var_38 = rax;
    rax = [rax stringValue];
    rax = [rax retain];
    var_41 = [rax isEqualToString:cfstring______g];
    [rax release];
    [var_38 release];
    if (var_41 != 0x0) {
            NSLog(cfstring______g);
            rax = [HS_control alertWithTitle:cfstring__c_y message:cfstring____v___S_____g_ btnTitle:@"" cancelBtn:cfstring_nx__ finish:0x0];
            rax = [rax retain];
            [rax release];
            var_1C = 0x1;
    }
    else {
            if ([var_8->_cnArr count] == 0x0) {
                    rax = [HS_control alertWithTitle:cfstring__c_y message:cfstring______b____ btnTitle:@"" cancelBtn:cfstring_nx__ finish:0x0];
                    rax = [rax retain];
                    [rax release];
                    var_1C = 0x1;
            }
            else {
                    rax = [var_8->netinfo objectForKeyedSubscript:@"canuse"];
                    rax = [rax retain];
                    var_90 = [rax integerValue];
                    [rax release];
                    if (var_90 == 0x1) {
                            rax = [ServerListInfo shareInfo];
                            rax = [rax retain];
                            var_A0 = rax;
                            rax = [rax canuse];
                            rax = [rax retain];
                            var_C0 = rax;
                            rax = [HS_control alertWithTitle:cfstring__c_y message:rax btnTitle:@"" cancelBtn:cfstring_sQ__ finish:0x0];
                            rax = [rax retain];
                            [rax release];
                            [var_C0 release];
                            [var_A0 release];
                            var_1C = 0x1;
                    }
                    else {
                            [var_8 connectStatus:0x1];
                            var_28 = 0x0;
                            if (var_8->netinfo == 0x0) {
                                    var_C8 = var_8->_cnArr;
                                    var_D0 = arc4random();
                                    rax = [var_8->_cnArr count];
                                    temp_3 = var_D0 % rax;
                                    rcx = var_28;
                                    var_28 = [[var_C8 objectAtIndex:temp_3] retain];
                                    [rcx release];
                            }
                            else {
                                    var_28 = 0x0;
                                    objc_storeStrong(var_28, var_8->netinfo);
                            }
                            var_E0 = [[var_28 objectForKeyedSubscript:@"ip"] retain];
                            var_E8 = [[var_28 objectForKeyedSubscript:@"port"] retain];
                            rax = [var_28 objectForKeyedSubscript:@"sskey"];
                            rax = [rax retain];
                            rcx = rax;
                            var_F0 = rax;
                            NSLog(@"%@  %@   %@", var_E0, var_E8, rcx);
                            [var_F0 release];
                            [var_E8 release];
                            [var_E0 release];
                            [ShadowsocksRunner stopserver];
                            rax = [var_28 objectForKeyedSubscript:@"ip"];
                            rax = [rax retain];
                            var_110 = rax;
                            [ShadowsocksRunner saveConfigForKey:@"proxy ip" value:rax];
                            [var_110 release];
                            rax = [var_28 objectForKeyedSubscript:@"port"];
                            rax = [rax retain];
                            var_128 = rax;
                            [ShadowsocksRunner saveConfigForKey:@"proxy port" value:rax];
                            [var_128 release];
                            [ShadowsocksRunner reloadConfig];
                            rax = var_8 + *_OBJC_IVAR_$_MainViewController.leftSelect;
                            rax = objc_loadWeakRetained(rax);
                            var_138 = [rax selectedColumn];
                            [rax release];
                            if (var_138 == 0x1) {
                                    [var_8 enableGlobal];
                            }
                            else {
                                    [var_8 enableAutoProxy];
                            }
                            [var_8 toggleRunning];
                            var_148 = objc_loadWeakRetained(var_8 + *_OBJC_IVAR_$_MainViewController._acclerate);
                            rax = [var_28 objectForKeyedSubscript:@"name"];
                            rax = [rax retain];
                            var_150 = rax;
                            [var_148 setStringValue:rax];
                            [var_150 release];
                            [var_148 release];
                            [var_8 connectStatus:0x2];
                            objc_storeStrong(var_28, 0x0);
                            var_1C = 0x0;
                    }
            }
    }
    var_18 = 0x0;
    rsi = 0x0;
    rdi = var_18;
    objc_storeStrong(rdi, rsi);
    rax = var_1C - 0x1;
    if (rax > 0x0) {
            stack[2002] = rbp;
            objc_storeStrong(var_18, 0x0);
            rdi->isRunning = 0x1;
            [rdi toggleRunning];
            [rdi connectStatus:0x0];
            objc_storeStrong(0x0, 0x0);
    }
    return;
}
```
为了让大家更容易的去理解，笔者将当前该回调的流程逻辑简化如下：
```
void -[MainViewController connectCallBack:](void * self, void * _cmd, void * arg2) {
   //一些用户信息的判断与读取
    if (var_41 != 0x0) {
            //var_41不等于0时，帐号有异常，执行该段逻辑弹出报错对话框
    }
    else {
            if ([var_8->_cnArr count] == 0x0) {
                    //var_8->_cnArr这个数组中数据量为0时，执行该段逻辑报错弹出对话框
            }
            else {
                    //读取用户信息是否可以使用vpn
                    if (var_90 == 0x1) {
                            //如果var90等于1，执行该顿逻辑报错弹出对话框
                    }
                    else {
                            //VPN信息初始化逻辑，想让vpn连接成功，这一段逻辑，程序必须能够执行到！
                    }
            }
    }
   //准备连接vpn一些工作
    if (rax > 0x0) {
           //当rax满足大于0时执行该段逻辑，连接目标vpn服务器，大家可以看到toggleRunning和connectStatus两个方法调用，从字面意思即可以看出，此段逻辑是跑动vpn连接的最核心！
            stack[2002] = rbp;
            objc_storeStrong(var_18, 0x0);
            rdi->isRunning = 0x1;
            [rdi toggleRunning];
            [rdi connectStatus:0x0];
            objc_storeStrong(0x0, 0x0);
    }
    return;
}
```
经过笔者的逻辑简化处理，相信大家应该能够一目了然了，想要成功破解这个vpn，成功的关键是：
1.能够翻过三个校验弹框顺利进入VPN初始化逻辑信息处
2.顺利的开启`toggleRunning`
理清思路后，切换到汇编窗口，我们来修改以下几处代码：
![图4-6](http://upload-images.jianshu.io/upload_images/2197489-f09c3016e4af4882.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![图4-7](http://upload-images.jianshu.io/upload_images/2197489-41e96a1728cee625.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![图4-8](http://upload-images.jianshu.io/upload_images/2197489-566703d02cab1764.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
修改的时候大家要注意，先点中目标代码行，然后菜单栏选择Modify,在对话框里输入修改以后的代码，这边大家注意，不能在对话框里直接输入`loc_10009422d`这样的地址，可能这是Hopper生成的假标签，像宏定义一样的概念，我们应该直接输入跳转目标的16进制地址，可以在代码行前面蓝色部分进行copy。
![图4-9](http://upload-images.jianshu.io/upload_images/2197489-6d7a04b6d29a97d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![图4-10](http://upload-images.jianshu.io/upload_images/2197489-3f9a31a62645d677.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
对照上面三张图的位置点，我们依次将目标代码处三行代码修改为：
`jmp 0x000000010009422d`
`jmp 0x00000001000942cd`
`jmp 0x000000010009442c`
修改完以后，我们再看一下现在的伪代码：
```
void -[MainViewController connectCallBack:](void * self, void * _cmd, void * arg2) {
    var_8 = self;
    objc_storeStrong(0x0, arg2);
    rax = [var_8 ping];
    rax = [rax retain];
    var_30 = rax;
    [rax stop];
    [var_30 release];
    rax = var_8 + *_OBJC_IVAR_$_MainViewController._daylabel;
    rax = objc_loadWeakRetained(rax);
    var_38 = rax;
    rax = [rax stringValue];
    rax = [rax retain];
    var_41 = [rax isEqualToString:cfstring______g];
    [rax release];
    [var_38 release];
    CMP(var_41, 0x0);
    CMP([var_8->_cnArr count], 0x0);
    rax = [var_8->netinfo objectForKeyedSubscript:@"canuse"];
    rax = [rax retain];
    var_90 = [rax integerValue];
    [rax release];
    CMP(var_90, 0x1);
    [var_8 connectStatus:0x1];
    var_28 = 0x0;
    if (var_8->netinfo == 0x0) {
            var_C8 = var_8->_cnArr;
            var_D0 = arc4random();
            rax = [var_8->_cnArr count];
            temp_23 = var_D0 % rax;
            rcx = var_28;
            var_28 = [[var_C8 objectAtIndex:temp_23] retain];
            [rcx release];
    }
    else {
            var_28 = 0x0;
            objc_storeStrong(var_28, var_8->netinfo);
    }
    var_E0 = [[var_28 objectForKeyedSubscript:@"ip"] retain];
    var_E8 = [[var_28 objectForKeyedSubscript:@"port"] retain];
    rax = [var_28 objectForKeyedSubscript:@"sskey"];
    rax = [rax retain];
    rcx = rax;
    var_F0 = rax;
    NSLog(@"%@  %@   %@", var_E0, var_E8, rcx);
    [var_F0 release];
    [var_E8 release];
    [var_E0 release];
    [ShadowsocksRunner stopserver];
    rax = [var_28 objectForKeyedSubscript:@"ip"];
    rax = [rax retain];
    var_110 = rax;
    [ShadowsocksRunner saveConfigForKey:@"proxy ip" value:rax];
    [var_110 release];
    rax = [var_28 objectForKeyedSubscript:@"port"];
    rax = [rax retain];
    var_128 = rax;
    [ShadowsocksRunner saveConfigForKey:@"proxy port" value:rax];
    [var_128 release];
    [ShadowsocksRunner reloadConfig];
    rax = var_8 + *_OBJC_IVAR_$_MainViewController.leftSelect;
    rax = objc_loadWeakRetained(rax);
    var_138 = [rax selectedColumn];
    [rax release];
    if (var_138 == 0x1) {
            [var_8 enableGlobal];
    }
    else {
            [var_8 enableAutoProxy];
    }
    var_18 = 0x0;
    [var_8 toggleRunning];
    var_148 = objc_loadWeakRetained(var_8 + *_OBJC_IVAR_$_MainViewController._acclerate);
    rax = [var_28 objectForKeyedSubscript:@"name"];
    rax = [rax retain];
    var_150 = rax;
    [var_148 setStringValue:rax];
    [var_150 release];
    [var_148 release];
    [var_8 connectStatus:0x2];
    objc_storeStrong(var_28, 0x0);
    rsi = 0x0;
    rdi = var_18;
    objc_storeStrong(rdi, rsi);
    rax = 0xffffffffffffffff;
    if (rax > 0x0) {
            stack[2002] = rbp;
            objc_storeStrong(var_18, 0x0);
            rdi->isRunning = 0x1;
            [rdi toggleRunning];
            [rdi connectStatus:0x0];
            objc_storeStrong(0x0, 0x0);
    }
    return;
}
```
可以看到vpn初始化信息和toggleRunning已经成为了必定执行的代码段了，这么说我们应该能成功了，但笔者也是猜测应该能够！那么我们就得测一下咯。
1.将修改后的可执行文件导出来，点选菜单`File`, 选择`Produce New Executable...`,选择路径保存。
![图4-11](http://upload-images.jianshu.io/upload_images/2197489-55ecacee51f6745b.gif?imageMogr2/auto-orient/strip)
2.将路径`/Users/Ronnie/Desktop/tianxingmac/tianxingmac.app/Contents/MacOS/tianxingmac`里的可执行文件替换成刚才保存的那个文件。
3.我们来测试一下，操作如下图：
![图4-12](http://upload-images.jianshu.io/upload_images/2197489-92d39e41abdc31d4.gif?imageMogr2/auto-orient/strip)
Oh,amazing!We got it!我们真的完成了此次破解之旅，天行VPN很不幸的沦为了砧板上的鱼肉！成功的一刹那，笔者确实很兴奋，当时情不自禁地骂了一句：靠，哥果然是个牛逼的Hacker。但是冷静下来后，反而觉得这并没有什么，因为真的是强中自有强中手，一山还比一山高，想想看我所使用的这几款工具作者，感觉个个都是天神一般的存在😂！！

#后续思考
完成破解任务后就结束了吗？No,我相信大部分小伙伴还有不少疑问。有些技术不错的小伙伴会质疑这个厂商的代码居然写的如此不安全，这么容易即被破解。其实笔者最初也有这样的困惑，但是很多事并没有我们想象的那么简单，笔者也是一名一线互联网工程师，其实我们在开发过程中总会有各种因素而忽略省略我们并不觉得那么严重的细节，即使是很细心的开发者也会有疏忽，有计划叫百密一疏呀，哈哈！逆向最重要的意义并不在结果，而是在整个逆向分析的过程中，通过分析我们能了解到那些我们并不熟悉的类型软件架构，从中学习到新知识，或者规避一些安全性问题！
经验不错的同学肯定会说，如果它能够在连接的时候服务端作校验，你的破解应该就白搭了，但笔者却觉得未必，为什么？我们来看下传统的http客户端应用交互过程：
![图5-1](http://upload-images.jianshu.io/upload_images/2197489-17c38e5d832ea758.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在这种情况下，想破解vip权限，似乎显的不太可能了，因为url是在服务端验证你是否是vip以后才返回过来，所以客户端这边无法破解绕过权限拿到url，因为关键校验在后台。我们再来看一下这款vpn的架构，它与上述架构存在区别，区别在哪，看下图：
![图5-2](http://upload-images.jianshu.io/upload_images/2197489-3fe27e93598ab714.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
大家可以看到两者区别了吗，在登陆以后我们就已经拥有服务器ip地址了，这是其一，最关键的一点是在于，客户端连接vpn的那个过程是通过socket流直连登入的，在这个过程中拥有用户tokenId信息的服务器无权查询用户是否合法，如果想用通过服务器来间接登入上述VPN服务器的话，貌似可以解决这个问题，但是呢，速度一下子就变慢了，因为中间中转了一层服务器。所以说在这种架构模式下，被破解似乎也情有可原。
但是到底怎样才能预防呢？办法总能有。
1.严格用户信息机制，只有合法用户才能看到这批服务器列表。
2.严格控制客户端release版本打印日志问题，因为笔者浏览过该vpn的一些本地日志，确实能给到笔者一些关键信息。
3.可以将软件上传app store，因为app store会给软件加密，就没那么容易dump出头文件了，当然也可以自己想办法上密。
4.可以自行在可执行文件里增加一些汇编花指令，扰乱破解者的思路。
5.在服务端严格管控客户端版本，如果低版本有这方面破解安全性问题，老接口就不该被低版本成功调用，可以给出提示用户升级到新版本的弹框。
6.管理好服务器的安全性。
可能还有好多好多办法，来提升安全性，但还是那句话，想要一套软件完全的安全，几乎不可能，因为这么多年过去了windows 0day漏洞依然存在，iOS每一代操作系统几乎都被成功破解越狱。攻与防永远是一场没有硝烟的战斗,看看pwn2own的新闻吧，不管是操作系统还是浏览器，几乎都无法逃过被顶尖hacker破解的命运！这篇文章讨论到这里差不多可以结束了,逆向真的是一门有趣且匪夷所思的学问，这种快乐并不是金钱所能够买来的！
#申明
笔者并没有利用技术进行非法盈利，这里只做学术研究，如果对天行vpn产生损失还请见谅，之所以选择天行作实验目标，也是因为比较认可它。当一款产品被认可的同时，必然会带来这些问题，所以请多多包涵，也希望天行早日修复此问题！顺便说一下，笔者目前在一家HR Cloud厂商负责移动端项目，我们的架构师很牛逼也很嚣张，说这边服务器架构很牛逼，安全性很高，这边笔者恳请各路大侠去挑战一下吧，我们的官网地址：[才到云](https://www.52emp.com/open_idp/)

对读者有帮助的文章罗列下：
[Hopper中文手册](http://sharex.win:878/?p=125)
[mac迅雷破解](http://iosre.com/t/hopper-mac/1428)

天行vpn破解版下载地址在我的github上,没有vpn的小伙伴可以抓紧时间去体验下，相信不久后漏洞会被堵上！
[Ronnie Chen](https://github.com/ronniechen888/tianxing-hackness)

![原创请支持](http://upload-images.jianshu.io/upload_images/2197489-2b68b89c7f1adad4.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

版权声明：本文为作者原创文章，转载请标明出处。
