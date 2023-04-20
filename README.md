# nsis_skinBtn

https://www.pianshen.com/article/41661316587/

NSIS进阶教程(一)
来自：http://www.pylife.net/post/2012-06-12/40027112705
自定义界面之无边框窗体移动贴图

**前言**

在Windows下，有很多人想做一个完全自己把控的安装程序，想过很多种途径去实现，有人说MFC可以实现，有人说C#可以实现，有人说Delphi可以实现，有人说VB又未尝不可呢。MFC，Delphi，VB，C#都需要自己去实现打包压缩，释放，释放过程中的业务逻辑跟界面功能，是一项比较麻烦的工作，甚至于C#程序需要运行的话，还需要装dotnet Framework的runtime。NSIS制作的安装包可以运行在Win9x下，完全是WinAPI的调用，不需要额外装任何的runtime，安装包双击就能运行，本身封装了很多Win的函数，方便调用与开放接口。功能部分也是实现了基本的安装过程所需的操作，NSIS的很多Editor做到了向导模式的脚本生成，很是方便。

这么好的工具能否定制开发呢，答案是肯定的。

本篇主要讲讲以下几点：

如何消除普通的NSIS脚本生成的窗体的边框

如何使得无边框窗体能够移动

如何给这个窗体贴上一张大小合适的背景图

所用到的NSIS插件：

nsDialogs

nsWindows

WinProc

System

**讲义**：

首先贴出一个今天教程的完整的例子（附带图片） 猛击这里

题外话，本来想用新浪爱问做文件分享平台的，发现上传后一直在审核中……练葵花宝典能力谁也比过性浪呀，用CSDN也不好，还要登录，本人就因为积分太少而不得不去做无聊的工作赢得积分，用于CSDN下载，自从CSDN把我的密码明码保存还被黑客给搞了之后，我不再上此网站。115虽然下载页广告多的一笔，但是后台上传页相当的干净，还不用审核以及无登陆下载，极致方便大家。(115被政府搞了，转性浪爱问)

使用时：把插件DLL跟头文件分别放入到你的NSIS本地对应的安装目录中，然后编译源码即可。

下文都用%NSIS_Install_DIR%来替代你本地安装路径

去除窗体Border

在去除窗体边框之前有一项工作是必须做的，那就是更改默认窗体的大小，因为每个人想做的打包窗体不可能都一样大，更改窗体大小有两种方法，也可以两种方法并用

修改NSIS内部的UI

NSIS的默认UI放在"%NSIS_Install_DIR%\Contrib\UIs"中，其中常常见到的创建自定义窗体的1018，1044都在此路径的modern.exe中。我们只要修改modern.exe里面的资源文件即可，做过MFC的都知道，VC在创建程序的时候是有Resources的，只要找到一些能更改Resources里面Dialog的工具即可，本文推荐ResHacker 。

修改的时候宁可大点，也绝不小，因为开发过程中我遇到用nsWindows命令扩大窗体的时候，出现不起作用的情况，但是默认窗体比需要的窗体小的时候可以用nsWindows命令控制。

打开ResHacker工具拖入modern.exe，操作前请备份modern.exe，拖动资源窗体或者直接修改你想要的大小。默认的1044跟1018窗体都在105分类下。




通过nsWindows命令

nsDialogs::Create 1044
    Pop $0
    ${If} $0 == error
        Abort
    ${EndIf}
    SetCtlColors $0 ""  transparent ;背景设成透明
                                                                                                        
    ${NSW_SetWindowSize} $HWNDPARENT 513 354 ;改变窗体大小
    ${NSW_SetWindowSize} $0 513 354 ;改变Page大小
该脚本添加在自定义窗体的创建Function中，创建的是1044类型窗口，修改命令是两条，分别是对$HWNDPARENT的窗体跟创建的1044page的修改，确保默认的modern.exe的窗口大小比这个要大！
修改好窗体大小后，直接在初始化的Function中直接填入以下代码即可去除边框

Function onGUIInit
    ;消除边框
    System::Call `user32::SetWindowLong(i$HWNDPARENT,i${GWL_STYLE},0x9480084C)i.R0`
    ;隐藏一些既有控件
    GetDlgItem $0 $HWNDPARENT 1034
    ShowWindow $0 ${SW_HIDE}
    GetDlgItem $0 $HWNDPARENT 1035
    ShowWindow $0 ${SW_HIDE}
    GetDlgItem $0 $HWNDPARENT 1036
    ShowWindow $0 ${SW_HIDE}
    GetDlgItem $0 $HWNDPARENT 1037
    ShowWindow $0 ${SW_HIDE}
    GetDlgItem $0 $HWNDPARENT 1038
    ShowWindow $0 ${SW_HIDE}
    GetDlgItem $0 $HWNDPARENT 1039
    ShowWindow $0 ${SW_HIDE}
    GetDlgItem $0 $HWNDPARENT 1256
    ShowWindow $0 ${SW_HIDE}
    GetDlgItem $0 $HWNDPARENT 1028
    ShowWindow $0 ${SW_HIDE}
FunctionEnd
用System::Call命令调用SetWindowLong的API函数改变GWL_STYLE的样式即可，System是NSIS官方插件用于帮助用户调用系统函数，是相当重要的自动安装程序的插件！程序中其余的代码是把创建的1004页面上其余的控件给隐藏掉，后面携带的ID都是可以通过ResHacker在105包中查询到。

贴一张大小合适的背景图

贴图需要用到nsDialogs插件的命令：

;贴背景大图
    ${NSD_CreateBitmap} 0 0 100% 100% ""
    Pop $BGImage
    ${NSD_SetImage} $BGImage $PLUGINSDIR\bg.bmp $ImageHandle
                                                                   
    ${NSD_FreeImage} $ImageHandle
${NSD_CreateBitmap}命令创建一个跟窗体一样大小的图片区域，后面的五个参数分别是x,y,width,height,text，坐标，宽高，文字。紧接着给这张图贴上一张合适的图片bg.bmp，贴图片之前需要把这个图片打包到安装程序中，这个是基本的操作，源码包中有，这里就不做说明了。最后还要通过${NSD_FreeImage}去释放该图片内存区。

无标题移动

做到无标题移动的潜台词是把原本传递给标题栏的Message通过你定义的元素回调传递给标题栏，所以只要给你添加的资源加上传递信息的回调函数就可以了。这里是通过WinProc这个插件完成的，WinProc这个插件在官方的插件库中没有，Google一下就可以查询到，这里的源码包中也有。除了WinProc，第三方插件SkinBtn也可以帮助实现。

Function onGUICallback
  ${If} $MSG = ${WM_LBUTTONDOWN}
    SendMessage $HWNDPARENT ${WM_NCLBUTTONDOWN} ${HTCAPTION} $0
  ${EndIf}
FunctionEnd
以上是回调函数，判断鼠标左键的Down事件，并且传递消息给标题栏。

GetFunctionAddress $0 onGUICallback
    WndProc::onCallback $BGImage $0 ;处理无边框窗体移动
以上是把当前的$BGImage作为回调主体，当用户左键点击$BGImage的时候，消息就传递给了窗体标题栏，实现了无边框的移动。

**结束语**

看看结果是什么样子的……哦！kugou也被我山寨了一把，有人精益求精，说，你的程序鼠标放在哪里都能移动，人家kugou只能标题栏移动……是的呀，你把图切成几份，分别贴，有的给回调函数，有的不给就实现了消息部分传递的功能。



有一个注意点：贴图的时候注意了！代码的运行是自上而下，如果要贴的图需要在另一张上面的话，需要把代码写在前面。

比如：

;贴小图
    ${NSD_CreateBitmap} 0 34 100% 100% ""
    Pop $MiddleImage
    ${NSD_SetImage} $MiddleImage $PLUGINSDIR\middle.bmp $ImageHandle
                  
    ;贴背景大图
    ${NSD_CreateBitmap} 0 0 100% 100% ""
    Pop $BGImage
    ${NSD_SetImage} $BGImage $PLUGINSDIR\bg.bmp $ImageHandle
That's all

下回继续探讨

***************************

特别感谢石头（石头的qq群号97208217），梦想吧论坛（虽然我还不是个会员，但是学到了很多）

NSIS进阶教程(二)
来自：http://www.pylife.net/post/2012-06-13/40027965518
自定义界面之Button、License窗口实现

**前言**

在上一节中我们粗略的处理一下无边框窗体、背景贴图、鼠标移动。这节主要是创建用于响应事件的Button以及能展示软件License的窗口，还能用Button控制软件协议的展示与否。

代码还是延续上一节的。

本篇主要讲讲以下几点：

如何创建一个自己的按钮

如何创建一个自己的License窗口

如何用自己的按钮控制自己的License窗口事件

所用到的插件：

nsDialogs

SkinBtn

**讲义**

首先贴出一个今天教程的完整的例子（附带图片） 猛击这里

本文还是在安装程序的首张Welcome中执行，上一节例子中的背景图需要做适当的修改，因为原本需要创建Button的地方背后有图，适当用PS去抹去就好。本文的前期工具就是PS了一部分元素。加入了几张Button的背景图。

创建一个属于自己Design的Button

;下一步
    ${NSD_CreateButton} 319 303 88 25 ""
        Pop $Btn_Next
        StrCpy $1 $Btn_Next
        Call SkinBtn_Next
        GetFunctionAddress $3 onClickNext
    SkinBtn::onClick $1 $3
原本nsDialogs插件就可以用${NSD_CreateButton}命令创建一个Button按钮出来，可惜的是此Button按钮默认的有点丑。

下面就要说到如何改观Button的样式，再接触之前我一直想找有没有用一张图片来替代Button的区域，并且给Image附上事件之类的想法。等到刘若英的后来，我发现有一个SkinBtn的插件可以完成此类工作。

SkinBtn是我在梦想吧中下载的一个第三方插件，国人开发，支持Button的五种状态：Normal，Hover，Click，Disabled，Focus，正常、鼠标悬浮、单击时、不可用时、获得焦点时。这五种状态已经完全够用了。

所要注意的就是SkinBtn的五种状态图需要竖行排列，宽高没有限制，但是高度像素最好的5的倍数（我猜是为了整除，方便取图）下面贴一张示例Button的贴图：



SkinBtn的用法是这样的：

在.onInit的初始化图片资源，并且要用SkinBtn插件的方法初始化一下：

File `/ONAME=$PLUGINSDIR\btn_next.bmp` `images\btn_next.bmp`
SkinBtn::Init "$PLUGINSDIR\btn_next.bmp"
紧接着Call以下方法，给当前的Button附上定义的图形：

Function SkinBtn_Next
  SkinBtn::Set /IMGID=$PLUGINSDIR\btn_next.bmp $1
FunctionEnd
现在窗体中的按钮已经被处理成自己想要的效果了，这时候衣服算是穿起来了，我们再试一试他的功能怎么样，我们定义一个onClickNext的function，然后调用SkinBtn插件的onClick方法进行调用：

GetFunctionAddress $3 onClickNext
    SkinBtn::onClick $1 $3
;下一步按钮事件
Function onClickNext
  MessageBox MB_OK "下一步"
  Abort
FunctionEnd
单击事件里面只做了弹出消息框这么一个临时的操作，后面的教程中会说明怎么弹到另一张page中，今天这里不谈。

这样一个自己的贴图的Button，从外观到功能都已经完成了。

创建显示License内容的控件

关于License的框如何显示的问题，NSIS的默认窗体有一个专门的协议Page，不过在我们的自定义Page中就不说那个了，在我们的Welcome页面中加入一个协议显示的窗口，这时候需要准备一个协议按钮的贴图，协议窗口。




协议的按钮样子看上去是超链接，我们可以通过Button贴图去实现，不要一味认死理去创建超链接的命令。因为有两种状态，所以有两份贴图。

图片预载入跟贴图的步骤就不说了，这里初始化该Button的时候是默认的箭头向上的图，我们只要额外的创建一个变量来控制点击的状态，并且在按钮点击事件中重新给Button贴上第二张箭头向上的图就可以实现按钮的贴图跟功能的统一：

Function SkinBtn_Agreement1
  SkinBtn::Set /IMGID=$PLUGINSDIR\btn_agreement1.bmp $1
FunctionEnd
                                                                                                  
Function SkinBtn_Agreement2
  SkinBtn::Set /IMGID=$PLUGINSDIR\btn_agreement2.bmp $1
FunctionEnd
                                                                                                  
;协议按钮事件
Function onClickAgreement
    ${IF} $Bool_License == 1
        IntOp $Bool_License $Bool_License - 1
        StrCpy $1 $Btn_Agreement
        Call SkinBtn_Agreement1
    ${ELSE}
        IntOp $Bool_License $Bool_License + 1
        StrCpy $1 $Btn_Agreement
        Call SkinBtn_Agreement2
    ${EndIf}
FunctionEnd
以上代码中，onClickAgreement方法是Button的点击响应事件，变量Bool_License是控制显示那张Button背景图的控制单元，它的初始值是0，IF判断中写的是当前状态下需要处理的事件。IntOp命令是执行加减运算。

这样，一个根据点击来更改背景图片的按钮就做好了。

用自定义的按钮去控制License窗口的显示

在做License之前首先要准备一份License的文件，NSIS命令中有读取文件，显示文件内容的命令，该文件可以是Txt，也可以是带有格式的RTF，这里示例中选用RTF文件。在源码包的RTF协议用的KuGou的。而用于显示RTF的控件则是nsDialogs插件提供的RichEdit20A，还要注意引入头文件LoadRTF.nsh

;读取RTF的文本框
    nsDialogs::CreateControl "RichEdit20A" \
    ${ES_READONLY}|${WS_VISIBLE}|${WS_CHILD}|${WS_TABSTOP}|${WS_VSCROLL}|${ES_MULTILINE}|${ES_WANTRETURN} \
    ${WS_EX_STATICEDGE} \
    5 44 500 203 ''
    Pop $Txt_License
    ${LoadRTF} '$PLUGINSDIR\license.rtf' $Txt_License
    ShowWindow $Txt_License ${SW_HIDE}
以上代码就是创建读取RTF文件的多行文本框，并且设定了一些常用的属性，具体的样式属性可以通过查询nsDialogs插件的官方文件可以得到，我们让该控件默认是不显示的状态。

这样一个，协议的显示控件就创建完毕了。

创建好了协议控件$Txt_License，那么接着就是用刚才创建的Button控制该窗口的显示与否，修改刚才的代码：

;协议按钮事件
Function onClickAgreement
    ${IF} $Bool_License == 1
        ShowWindow $MiddleImage ${SW_SHOW}
        ShowWindow $Txt_License ${SW_HIDE}
        IntOp $Bool_License $Bool_License - 1
        StrCpy $1 $Btn_Agreement
        Call SkinBtn_Agreement1
    ${ELSE}
        ShowWindow $MiddleImage ${SW_HIDE}
        ShowWindow $Txt_License ${SW_SHOW}
        IntOp $Bool_License $Bool_License + 1
        StrCpy $1 $Btn_Agreement
        Call SkinBtn_Agreement2
    ${EndIf}
FunctionEnd
在事件当中，加入隐藏中间背景图跟显示License窗口的代码，这样一个修改Button贴图跟控制协议窗口显示与否的关联就做好了。

**结束语**


图片还需修整修整。

在此文中，核心的部分是如何创建读取RTF的协议窗口还有Button控制协议窗口的显示的部分，这部分所用到的代码逻辑是每个初级Coder都可以写出来的，本文也就是抛砖引玉。

**困惑**

在源码中，开头的部分定义了两个变量：

Var MSG

Var Dialog

$MSG变量在移动回调函数中用到，$Dialog变量没有显式用到，据我的实验$Dialog变量在按钮控制MiddleImage中间图形中用到，如果不声明$Dialog变量，在Button的单击事件中控制图形的显示与否是失效的！

怀疑：$MSG是NSIS系统默认的消息变量用于记录消息信息，$Dialog是NSIS默认的对话框变量用于保存窗体中控件的信息。有待高手解答。

*********************************

That's all

下回再探讨

NSIS进阶教程(三)
来自：http://www.pylife.net/post/2012-07-13/40030360875
自定义MessageBox，自定义页跳转，自定义CheckBox样式

**前言**

上一节中我们处理了Button的自定义以及Button的事件消息、协议框的创建等等，这节中我们要更加完美的要求我们的提示框也要漂亮，CheckBox也要自定义样式。有人说MessageBox在NSIS默认情况下是带边框的API窗口，是一个比较丑雏形，但是NSIS的nsDialogs插件也没有提供一个可以创建弹出窗的命令行呀，CheckBox系统自带的只能是那个默认的皮肤，如果想把CheckBox的背景改成蓝色，或者把CheckBox的勾改成爱心，把叉改成骷髅头如何去做呢？这些问题在任何一个面向对象的编程语言中都可以很容易实现，在NSIS这样局限的环境中也可以变通实现，接下来我们处理一下这些问题。

本篇主要讲讲以下几点：

如何抛弃系统的MessageBox

如何实现自定义页面的跳转

如何用自己的贴图实现CheckBox功能

所用到的插件【新增】：

FindProcDLL（查找当前进程插件）

KillProcDLL （关闭指定进程插件）

**讲义**

首先贴出一个今天教程的完整的例子【已测试】【附带图片】 猛击这里

题外话，本节的重点是如何把系统的MessageBox替换掉自定义的窗口，一开始这个命题挺难的，原本因为nsDialogs可以创建一个弹窗，然后给它披上一层皮就可以了，在查了很多资料后，我了解到根本不行，nsDialogs插件不能完成这么简单的工作，而是通过另一个插件nsWindows来完成。另一个自定义CheckBox功能还是有点讨巧的，其实在窗体编程中，Button跟CheckBox本质上是一样的东西，CheckBox就是Button披上一层皮加上一些单击事件改变皮肤而存在的，于是就有了CheckBox的自定义的可能性。

还是要讲一点不足，此处CheckBox不包含旁边的说明文字，也就是说，当你促发CheckBox旁边的文字某些事件的时候CheckBox本事不会有任何反应，这样就不能构成一个CheckBox的整体，不过对于要求不是非常高的人来说，这点完全可以忽略。

如何抛弃系统的MessageBox

通常在使用警告框的时候，我们会调用MessageBox的NSIS命令，然而此窗口无比丑陋，简直是对我们正在做的窗体的亵渎。于是我们自己创建警告框，此警告框也要跟我们已有的窗口风格一致，包含几大问题，“无边框”、“美观贴图”、“移动”、“逻辑判断功能”，做到以上几点就可以了。上代码：

Function onCancel
    IsWindow $WarningForm Create_End
    !define Style ${WS_VISIBLE}|${WS_OVERLAPPEDWINDOW}
    ${NSW_CreateWindowEx} $WarningForm $hwndparent ${ExStyle} ${Style} "" 1018
                                                                                                                               
    ${NSW_SetWindowSize} $WarningForm 349 184
    EnableWindow $hwndparent 0
    System::Call `user32::SetWindowLong(i$WarningForm,i${GWL_STYLE},0x9480084C)i.R0`
    ${NSW_CreateButton} 148 122 88 25 ''
    Pop $R0
    StrCpy $1 $R0
    Call SkinBtn_Quit
    ${NSW_OnClick} $R0 OnClickQuitOK
                                                                                                                               
    ${NSW_CreateButton} 248 122 88 25 ''
    Pop $R0
    StrCpy $1 $R0
    Call SkinBtn_Cancel
    ${NSW_OnClick} $R0 OnClickQuitCancel
                                                                                                                               
    ${NSW_CreateBitmap} 0 0 100% 100% ""
    Pop $BGImage
  ${NSW_SetImage} $BGImage $PLUGINSDIR\quit.bmp $ImageHandle
    GetFunctionAddress $0 onWarningGUICallback
    WndProc::onCallback $BGImage $0 ;处理无边框窗体移动
  ${NSW_CenterWindow} $WarningForm $hwndparent
    ${NSW_Show}
    Create_End:
  ShowWindow $WarningForm ${SW_SHOW}
FunctionEnd
这个onCancel方法是在我们在主窗体上点击“取消”或者“关闭”的时候促发的方法，这时会弹出一个命令窗口出来。

创建跟设置样式：WS_VISIBLE  是显示的意思，不加会隐藏掉，WS_OVERLAPPEDWINDOW  是创建一个层叠窗体的属性也需要加上。NSW_CreateWindowEx  是创建窗体的主命令。

创建完窗体后，用NSW_SetWindowSize  更改一下窗体的大小。

EnableWindow $hwndparent 0 这段代码一定要加上，这是创建一个警告框模式窗体的必要条件，就是让主窗体不能操作。

接下来就是消除边框，用nsWindows命令创建按钮的一套，跟nsDialogs的创建是一致的，包括无边框移动的一套，在第一节中已经讲过，这里就不啰嗦了。

最后把这个窗体展示出来就可以了。把这个onCancel这个方法赋给“取消”按钮的单击事件，这样一个无边框模式窗体警告框就好了。



当我们点击“退出”的时候，就要关掉当前程序，所以要添加一个关闭的方法： 

Function onClickClose
    FindProcDLL::FindProc "test.exe"
    Sleep 500
    Pop $R0
    ${If} $R0 != 0
    KillProcDLL::KillProc "test.exe"
    ${EndIf}
FunctionEnd
通过FindProcDLL插件的FindProc方法找到安装进程，并且通过KillProcDLL插件的KillProc杀之。

当我们点击“取消”的时候，这时候需要关闭当前警告框，并且让主窗体能够处于Active状态

Function OnClickQuitCancel
  ${NSW_DestroyWindow} $WarningForm
  EnableWindow $hwndparent 1
  BringToFront
FunctionEnd
NSW_DestroyWindow 命令销毁掉警告框，使主窗体能活动，并且Bring到前端。

如何实现自定义页面的跳转

一开始我们就定义了两个自定义页面：

Page custom WelcomePage
Page custom InstallationPage
从WelcomePage跳转到InstallationPage，这个就是“下一步”按钮的事件。

创建一个RelGotoPage方法：

Function RelGotoPage
  IntCmp $R9 0 0 Move Move
    StrCmp $R9 "X" 0 Move
      StrCpy $R9 "120"
  Move:
  SendMessage $HWNDPARENT "0x408" "$R9" ""
FunctionEnd
详细解释在这里 http://nsis.sourceforge.net/Go_to_a_NSIS_page

If a number > 0: Goes foward that number of pages. Code of that page will be executed, not returning to this point. If it is bigger than the number of pages that are after that page, it simulates a "Cancel" click.

If a number < 0: Goes back that number of pages. Code of that page will be executed, not returning to this point. If it is bigger than the number of pages that are before that page, it simulates a "Cancel" click.

If X: Simulates a "Cancel" click. Code will go to callback functions, not returning to this point.

If 0: Continues on the same page. Code will still be running after the call.

在“下一步”的单击事件中加入以下代码，跳转就好了，而且以后的调转RelGotoPage都适用

Function onClickNext
  StrCpy $R9 1
  Call RelGotoPage
  Abort
FunctionEnd
如何用自己的贴图实现CheckBox功能

CheckBox也有系统自带的那种健全的，但是效果没有Button贴皮肤后好，所以弃用之，在第二个页面中创建几个Button跟标签：

${NSD_CreateButton} 26 150 15 15 ""
    Pop $Ck_ShortCut
    StrCpy $1 $Ck_ShortCut
    Call SkinBtn_Checked
    GetFunctionAddress $3 OnClick_CheckShortCut
    SkinBtn::onClick $1 $3
    StrCpy $Bool_ShortCut 1
    ${NSD_CreateLabel} 45 151 100 15 "添加桌面快捷方式"
    Pop $Lbl_ShortCut
    SetCtlColors $Lbl_ShortCut ""  transparent ;背景设成透明
创建CheckBox的时候要考虑周全，首先要定义该CheckBox的变量，该CheckBox的皮肤，记录该CheckBox状态的变量$Bool_ShortCut ，该CheckBox旁边的提示文字变量$Lbl_ShortCut



            


Button的贴图跟变换方式在第二节中已经介绍，这里就不啰嗦。

这里的OnClick_CheckShortCut 方法不仅仅是一个变换的实现，也通过$Bool_ShortCut记录了当前CheckBox的状态

Function OnClick_CheckShortCut
  ${IF} $Bool_ShortCut == 1
        IntOp $Bool_ShortCut $Bool_ShortCut - 1
        StrCpy $1 $Ck_ShortCut
        Call SkinBtn_UnChecked
    ${ELSE}
        IntOp $Bool_ShortCut $Bool_ShortCut + 1
        StrCpy $1 $Ck_ShortCut
        Call SkinBtn_Checked
    ${EndIf}
FunctionEnd
最后我们在完成安装的时候，可以通过$Bool_ShortCut该变量来了解用户的选择



**结束语**

其实不难，逻辑清晰，知道自己朝哪个方向去寻找答案，一切就会云开雾散。希望大家能学到一些。

***************************

有任何疑问请留言。

That's all

下回继续探讨

NSIS进阶教程(四)
来自：http://www.pylife.net/post/2012-11-23/40043392092
自定义目录选择，自定义进度条，自定义图片切换效果

**前言**

上一节中我们已经处理了有关CheckBox自定义贴图的部分，但是目录选择的部分还没有加上，这节，我们先处理一下目录的选择部分，选择完路径之后就剩下安装了，于是进度条的创建的显得很有必要，但是系统的进度条创建简单，如何改变进度条的背景色跟进度色呢，这节我们也处理掉。有关图片切换的效果的插件也有很多，但是大部分都是基于默认安装窗体进行的，如何在一个完全自定义的页面上面创建一个图片切换效果呢，这节也可以得到答案。

本篇主要讲讲以下几点：

创建目录选择按钮与文本框

创建自定义的进度条

创建图片切换效果

所用到的插件【新增】：

WebCtrl

SkinProgress

BgWorker

**讲义**

首先贴出今天教程的完整的例子【已测试】【附带图片】 猛击这里

这次的教程基本是安装界面的最后的阶段的处理了，主要是创建一个目录选择控件，用于确定安装程序安装的路径，在安装过程中等待时间也是比较长的，遇到打广告的好时机不要错过，切换多图是个不错的选择；在安装过程中，释放文件是一个主要动作。如果不是自定义页面，该动作是在系统的安装页面的Section里面完成的，如果不做多线程处理，该动作会阻塞主线程也就是主界面的消息传递，主界面没有了消息传递也就不会响应拖动、点击关闭等等操作，这个是本节主要说明的地方。最后说一下图片切换的实现，以我目前尝试的，效果最好的还算是直接放一张网页最实在，通过网页的js实现切换效果。

目录选择框

目录选择框包括一个文本框跟一个Button按钮，做过Web的人都知道，html中有一个file的控件与之相吻合，在form中则是两个不同的component，文本框很好创建，按钮的单击事件是一个重点，我们看看代码：

;更改目录控件创建
    ${NSD_CreateDirRequest} 26 79 358 25 "$INSTDIR"
    Pop $Txt_Browser
    ${NSD_OnChange} $Txt_Browser OnChange_DirRequest
                                                                                                                                                                        
    ${NSD_CreateBrowseButton} 400 79 88 25 ""
    Pop $Btn_Browser
    StrCpy $1 $Btn_Browser
    Call SkinBtn_Browser
    GetFunctionAddress $3 OnClick_BrowseButton
      SkinBtn::onClick $1 $3
这里创建了一个$Txt_Browser的文本框，一个$Btn_Browser的按钮，其中按钮的单击事件是 OnClick_BrowserButton，看看按钮的事件代码:

Function OnClick_BrowseButton
  Pop $0
                                                                                                                                                                 
  Push $INSTDIR 
  Call GetParent
  Pop $R0
                                                                                                                                                                 
  Push $INSTDIR
  Push "\"
  Call GetLastPart
  Pop $R1
                                                                                                                                                                 
  nsDialogs::SelectFolderDialog "请选择 $R0 安装的文件夹:" "$R0"
  Pop $0
  ${If} $0 == "error" 
    Return
  ${EndIf}
  ${If} $0 != ""
    StrCpy $INSTDIR "$0\$R1"
    system::Call `user32::SetWindowText(i $Txt_Browser, t "$INSTDIR")`
  ${EndIf}
FunctionEnd
                                                                                                                                                            
;得到选中目录用于拼接安装程序名称
Function GetParent
  Exch $R0
  Push $R1
  Push $R2
  Push $R3
  StrCpy $R1 0
  StrLen $R2 $R0
  loop:
    IntOp $R1 $R1 + 1
    IntCmp $R1 $R2 get 0 get
    StrCpy $R3 $R0 1 -$R1
    StrCmp $R3 "\" get
    Goto loop
  get:
    StrCpy $R0 $R0 -$R1
    Pop $R3
    Pop $R2
    Pop $R1
    Exch $R0
FunctionEnd
                                                                                                                                                                 
                                                                                                                                                           
;截取选中目录
Function GetLastPart
  Exch $0 ; chop char
  Exch
  Exch $1 
  Push $2
  Push $3
  StrCpy $2 0
  loop:
    IntOp $2 $2 - 1
    StrCpy $3 $1 1 $2
    StrCmp $3 "" 0 +3
      StrCpy $0 ""
      Goto exit2
    StrCmp $3 $0 exit1
    Goto loop
  exit1:
    IntOp $2 $2 + 1
    StrCpy $0 $1 "" $2
  exit2:
    Pop $3
    Pop $2
    Pop $1
    Exch $0 
FunctionEnd
单击按钮的时候用nsDialogs创建一个SelectFolderDialog的对话框，选择需要安装的路径。GetParent方法主要是要取得当前选中的路径，GetLastPart主要是保留当前的安装程序最底层的目录名称，最终两个部分合起来作为整体的安装路径赋值给$INSTDIR。具体的效果可以运行源码查看。             

创建自定义进度条

nsDialogs有自带的进度条，该进度条的颜色是那种系统的颜色，如果遇到自己设计的，就会出现问题，于是用SkinProgress就可以给进度条赋上两张图片，一张是底图，一张是进度图，虽然不是非常的完美，比如圆角，比如透明，比如阴影，比如……但是已经是跟NSIS快捷脚本配合的很好的了。

${NSD_CreateProgressBar} 24 265 474 7 ""
    Pop $PB_ProgressBar
    SkinProgress::Set $PB_ProgressBar "$PLUGINSDIR\loading2.bmp" "$PLUGINSDIR\loading1.bmp"
用法非常简单，用nsDialogs创建一个ProgressBar，然后用SkinProgress去set一下，后面跟上两张图片。ProgressBar的图片是有讲究的，具体的可以看源码的image文件夹中两张图片的切图，基本要使用什么效果，自己做两张图就可以了。                           

创建图片切换

图片的切换有很多种，gif、flash、js、甚至自己用c++做插件实现，这里提供一种很方面，但是又不失功能强大的方式。网页的形式！你很容易能自己写一段js实现多张图片的切换，这里唯一要解决的就是如何在Form上创建一个浏览器用于加载自己的本地html页面。

System::Call `*(i,i,i,i)i(1,34,518,200).R0`
    System::Call `user32::MapDialogRect(i$HWNDPARENT,iR0)`
    System::Call `*$R0(i.s,i.s,i.s,i.s)`
    System::Free $R0
    FindWindow $R0 "#32770" "" $HWNDPARENT
    System::Call `user32::CreateWindowEx(i,t"STATIC",in,i${DEFAULT_STYLES}|${SS_BLACKRECT},i1,i34,i518,i200,iR0,i1100,in,in)i.R0`
    StrCpy $WebImg $R0
    WebCtrl::ShowWebInCtrl $WebImg "$PLUGINSDIR/index.htm"
首先在界面上创建一个STATIC的对象(对话框)，通过MapDialogRect来定位该对话框的位置(1,34)，大小(518,200)，然后通过WebCtrl控件来把自己的网页index.htm加载进去，WebCtrl插件实现的就是调用本地浏览器。这里定位浏览器的位置以及大小是难点，还需多多熟悉才是。

网页里面的代码我就不讲解了，主要是图片js切换，你也可以更换js代码，网络中很多效果都有。

如果发现界面上的图片切换画面跟外边框有距离，就去查看网页里面的css，是否把margin跟padding都设成了0，这样就没有间隙了，看上去跟贴在form上面的一样。：）                                     

多线程安装

界面都画好了，现在说到安装文件的时候释放了。源码中我是通过sleep来模拟的，具体的释放跟这个差不多。首先创建的是一个只运行一次的定时器Timer：

GetFunctionAddress $0 NSD_TimerFun
    nsDialogs::CreateTimer $0 1
其中nsDialogs::CreateTimer后面的参数1代表的是1毫秒，正常情况下就是1毫秒执行一次NSD_TimerFun介个方法。

Function NSD_TimerFun
    GetFunctionAddress $0 NSD_TimerFun
    nsDialogs::KillTimer $0
    !if 1   ;是否在后台运行,1有效
        GetFunctionAddress $0 InstallationMainFun
        BgWorker::CallAndWait
    !else
        Call InstallationMainFun
    !endif
FunctionEnd
本身定时器就是一个异步的功能，如果定时器是同步的，那就没有意义了对不，那么既然是异步的，为什么在NSD_TimerFun里面做Sleep操作的时候，主界面会卡死不响应呢。这个就要看看nsDialogs插件如何实现这个定时器的了，我也没有详查，应该不是创建子线程Thread 的方式。

这里如果抛弃Timer，直接使用BgWorker的话，也可以，但是会有一个缺陷：当你需要同时运行两个方法的时候，只有通过创建两个Timer来实现，并且在Timer调用的方法里面采用BgWorker来实现子线程操作。

看看上面代码在Timer执行方法里面，第一步是停止Timer：因为我们只用了他的异步的特点，并没有想做定时器。然后使用BgWorker插件。

使用BgWorker插件非常简单，只需用$0接收方法地址，然后调用BgWorker::CallAndWait。顾明思意，BgWorker调用过程是同步的，而且会Wait到InstallationMainFun方法执行结束，如果BgWorker::CallAndWait调用下面还有代码的话，只有在执行完InstallationMainFun方法后才能执行。

下面看看InstallationMainFun的实现：

Function InstallationMainFun
    SendMessage $PB_ProgressBar ${PBM_SETRANGE32} 0 100
    SendMessage $PB_ProgressBar ${PBM_SETPOS} 10 0
    Sleep 1000
    SendMessage $PB_ProgressBar ${PBM_SETPOS} 20 0
    Sleep 1000
    SendMessage $PB_ProgressBar ${PBM_SETPOS} 30 0
    Sleep 1000
    SendMessage $PB_ProgressBar ${PBM_SETPOS} 40 0
    Sleep 1000
    SendMessage $PB_ProgressBar ${PBM_SETPOS} 50 0
    Sleep 1000
    SendMessage $PB_ProgressBar ${PBM_SETPOS} 60 0
    Sleep 1000
    SendMessage $PB_ProgressBar ${PBM_SETPOS} 70 0
    Sleep 1000
    SendMessage $PB_ProgressBar ${PBM_SETPOS} 80 0
    Sleep 1000
    SendMessage $PB_ProgressBar ${PBM_SETPOS} 90 0
    Sleep 1000
    SendMessage $PB_ProgressBar ${PBM_SETPOS} 100 0
                                   
    ShowWindow $Btn_Next ${SW_SHOW}
    ShowWindow $Btn_Install ${SW_HIDE}
    EnableWindow $Btn_Cancel 0
FunctionEnd
所做的工作主要是操作滚动条的位置。在实际的释放过程中，这个设置也是这样的，目前还没有自定义页面的释放插件，能回调得到精确的释放量，以及释放文件等等信息。所以只能模拟，尽可能的细化下来，估计着大概多少。这个是一个需要情商的工作，如果你有洁癖，偏执之类的病，我想你还是用系统自带的释放好了，也不需要自定义页面了 。

最后上几张图吧，好看点，也算是阶段的结束 。

（安装过程）



（安装过程结束）



（安装结束）



**结束语**

四节的自定义教程也就到此为止了，日后会补充一些技巧性的东西，也准备搞搞，修修插件，发布发布。That's all,Thanx!       

NSIS进阶教程(五)
来自：http://www.pylife.net/post/2013-05-29/40051693246
在线下载，下载后的安装，本地解压安装

前言

想做在线安装包教程已经很久了，一直找不到合适的时间，如今在一个慵懒的周末早晨终于开始了。今天晚上还要去虹口体育场看申花跟国安的比赛，还是要抓紧时间呀。 在线安装是以前教程的一个延续。

意义一：国内很多的IT厂商在做安装包的时候首先考虑的是用户下载的时间可接受度，而一个动辄100mb的包会把用户吓跑，而且下载的入口还控制在诸如迅雷，旋风，IE，等等，稍等，还有360安全浏览器。。。你如果没有给360打个招呼，它说不定会报你的程序有毒。。。太可拍了。入口最重要，掌握在任何人手里都会成为收门票的工具。

意义二：用户对着迅雷，旋风的界面，千遍一律，没有任何信息的展示，就是一个进度条，你丧失了在第一时间占领用户意识的契机。如果你在安装的时候有动画播放，并且让用户在安装的过程中获取产品的使用技巧，或者像广告植入一样的严肃的对待这个不起眼的时间，你会收获很多。

本篇主要讲讲以下几点：

在线下载

下载后的安装

下载后的解压

所用到的插件【新增】：

inetc

nsis7z

详细地址： http://hamletsoft.com/blog/2013/05/26/nsis-slash-learn-nsis-step-by-step-setuponline/

附加其他：Qt之打包发布
Qt发布方式  
Qt发布的时候，通常使用两种方式：
（1）静态编译
（2）动态编译
静态编译：把相关联的库一并引入可执行程序，虽然发布简单，但可执行程序较大。。。
动态编译：相关联的库，以dll的形式引用，不被包含进可执行程序，发布不方便，但可执行程序较小。。。
静态发布虽然不需要较多的dll，发布简单、方便，但是往往会牵扯到授权问题（详情请查看Qt LGPL授权），动态发布则可以避免。。。如果附带了Qt的dll，就相当于发布了Qt的dll，而这些库是属于Qt的，这足以保证使用者知道程序使用了LGPL版本的Qt。
查找依赖项

1、检测可执行程序依赖模块
下载工具：DependencyWalker
打开可执行程序，检测依赖项

 检测完成之后，将所需依赖库拷贝进去。。。再次进行检测，反复进行。
2、常用依赖库
（1）Qt模块库
    Qt5Cored.dll
    Qt5Guid.dll
    Qt5Widgetsd.dll
（2）ICU依赖库
    icudt51.dll
    icuin51.dll
    icuuc51.dll
（3）EGL依赖库
    libEGLd.dll
    libGLESv2d.dll
（4）插件库（Qt安装目录下即可找到D:\Software\Qt\Qt5.1.0\5.1.0\msvc2010\plugins\platforms）
    图片支持库：imageformats
   音频、视频支持库：mediaservice
    平台支持库：platforms
    等等。。。
    注意：查找对应的插件文件夹，粘贴到安装目录（一定要保持目录结构，例如“platforms/***.dll”），详细结构见打包发布准备的文件组织结构。   
（5）VS运行时库（在VS安装目录下即可找到D:\Software\Microsoft VisualStudio\VC\redist）
    msvcp100d.dll
    msvcr100d.dll
   注意：发布程序的时候注意版本（Debug/Release）
  如果是Debug版本的则是.前面带d的（Qt5Cored.dll、Qt5Guid.dll、Qt5Widgetsd.dll）
   如果是Release版本的则是.前面不带d的（Qt5Core.dll、Qt5Gui.dll、Qt5Widgets.dll）
关于NSIS
1、NSIS简介
（1）NSIS是什么？
   一款免费的Win32安装、卸载系统！
（2）NSIS有什么特点？
  脚本简洁高效、系统开销小，进行安装、卸载、设置、解压文件也不在话下，几乎可以做所有的事情。
2、工具
   NSIS Edit + NSIS
3、使用方式
   脚本向导 + 修改代码 = 个性化安装包
准备文件
以下是我即将打包的所有文件，安装包目录结构（包括：可执行程序、插件库、运行时库、授权文件、卸载程序图标等等！）如下图所示：

1、利用向导制作安装包：

2、填写应用程序基本信息

3、指定安装程序所用选项
   注意：这里选择语言为SimpChinese

4、设置应用程序安装目录与授权文件

5、指定应用程序文件

6、指定创建应用程序图标

7、选择安装程序完成后运行的动作

8、指定接触安装程序属性

9、进行脚本编译、保存

10、等待编译完成，即可运行打包后的安装包

   大功告成之后，即可进行安装！
（1）安装程序

（2）此处显示授权文件中的内容

（3）选择安装目录

（4）运行程序，并显示“自述文件”

（5）运行结果

  关于Qt的打包工具了解一些，个人感觉NSIS用起来比较顺手，脚本修改起来也方便，所以就推崇一下。。。若想将安装包变得更加美观，则需要手动修改脚本，更多信息请查找相关资料，此处不再多做介绍！
