# wowAddons开发指南

Lua 是一个小巧的脚本语言，其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。Lua由标准C编写而成，几乎在所有操作系统和平台上都可以编译，运行。 Lua并没有提供强大的库，这是由它的定位决定的。所以Lua不适合作为开发独立应用程序的语言。Lua 有一个同时进行的JIT项目，提供在特定平台上的即时编译功能。
**魔兽世界、仙剑奇侠传五**等的插件用的也是**lua**。

在wow的跟目录中
```
Cache 缓存
Data 魔兽主要数据
Errors 错误信息
Fonts 字体
Interface 接口(第三方插件模型等)
Logs 记录(聊天记录和战斗记录等)
Screenshots 截图（在wow中按prt可以截图存放）
Utils 工具(魔兽内置浏览器应该没有用)
WTF 配置
```
在Interface中建一个AddOns文件夹用来存放插件

## 插件的结构

WoW的插件由3个文件组成，分别是`XXX.toc` / `XXX.xml` / `XXX.lua`
XXX.toc中标注了关于插件信息的定义，以及需要加载的.xml的位置。
XXX.xml是插件的核心，包括界面、功能等元素都在其内。
XXX.lua是一种嵌入式脚本语言的源文件，在WoW中实现.xml调用的函数。
换句话说，.toc告诉WoW插件的名字和.xml的位置，.xml告诉WoW插件都有什么，怎样工作，具体事宜则交由.lua来处理。

### XXX.toc

XXX.toc中标注了关于插件信息的定义，以及需要加载的.xml的位置。
TOC全称为Table of Contents就是目录的意思。
XXX.toc的位置在World of Warcraft\Interface\AddOns\XXX\XXX.toc。其中XXX为插件名称，必须保证完全一致，包括大小写。Windows系统不区分大小写，但WoW客户端区分(case-sensitive)，在开发过程中一定要注意。

*WoW只能读取1024字节/行，富裕出的部分将被忽略并且不报错。*

```
## Interface: 适用的魔兽版本号 
## Title: 显示的标题（默认语言） 
## Notes: 显示的说明（默认语言） 
## Title-zhCN: 特定语言的标题（简体中文） 
## Notes-zhCN: 特定语言的说明（简体中文） 
## Author: 作者（不显示） 
## Version: 版本 
## eMail: 如题 
## UIType: 插件类型 
## Dependencies: 依赖的插件 
## RequiredDeps: 必须依赖的其他插件 
## OptionalDeps: 可选倚赖 
## SavedVariables: 统一存放的变量 
## SavedVariablesPerCharacter: 按角色存放的变量 
## LoadOnDemand: 1 （调用时加载） 
## LoadWith: 当指定插件加载时才加载，前提是调用时加载 
## DefaultState: disabled 默认状态 
## Secure: 安全（功能未知） 
# 注释1 dklasjfkasdj 
Script.lua -- 脚本文件 
% 注释2 dskajfklasdjfklsdaj 
Layout.xml -- 布局文件
```
客户端识别标签`## `开头

- Interface
    ```
    ## Interface: 70100 
    ```
    wow客户端版本（如果插件的Interface号和当前游戏客户端版本号不一致，那么游戏默认不会加载插件）
- Title
    ```
    ## Title: |cffC495DDXXX|rXXX`
    ## Title-zhCN: |cffC495DD插件|r插件 
    ## Title-zhTW: |cffC495DD插件|r插件
    ```
    Title表示插件的名称，也就是插件管理面板里显示的那个。支持多语言定义，默认为英语。其他语言要在Title后面加上后缀，就像上面那样。zhCN表示简体中文，zhTW就是台湾的繁体中文。前面2个小写是语言，后面2个大写是国家和地区。
    
    还有个好玩的东西就是颜色，WoW支持字串形式的颜色定义。插件面板默认字体颜色是黄，你可以根据自己的喜好自行定义文本颜色，这里就用到颜色字串。颜色字串以"|c"开头"|r"结束。中间是16进制αRGB颜色代码和要显示的文本。比如我要将"我的插件"几个字显示为蓝色。那么就应该是:`##Title: |cff0000ff我的插件|r   `
    
    其中|c表示接下来的8位字符是颜色代码，以α(alpha透明度)、R(红)、G(绿)、B(蓝)顺序排列。前面2位ff表示被着色的文本完全不透明，但需要指出的是并非所有的地方都支持透明。之后就是着色的文本了，这里就是"我的插件"，|r表示颜色结束，如果没有结束标记，那么WoW将颜色代码以后到这一行结束的全部文本都进行着色。
    
    值得一提的是，这个颜色字串应用相当广泛，包括人物ID也可以，这也是为什么暴雪禁止使用符号和数字以及英文汉字混合的原因了，当然也是考虑到搜索的方便。
- Notes
    
    只有Title和Notes标签支持其他语言，除此之外都要用英语书写。Notes包含插件的说明内容，就是对插件功能的简单描述，将出现在插件管理面板中鼠标经过的地方。
    ```
    ## Notes: XXX......
    ```
- Dependencies
    ```
    ## Dependencies: someAddOn, someOtherAddOn
    ## RequiredDeps: someAddOn, someOtherAddOn
    ## Dependancies[sic]: someAddOn, someOtherAddOn
    ```
    Depend是依赖的意思。为了让插件更好地工作，某些插件开发者会使用第三方库或其他现成的插件作为基础和辅助，这么做的有点就是节约了开发成本，也使开发过程简便了许多，不足的地方就是，要让插件正常工作，系统必须确保所依赖的文件都要存在，否则，如果任何依赖的库或插件缺失，当前插件加载都会失败。依赖的名称即依赖插件的名称，也就是目录文件夹的名称。如果需要依赖多个库或插件，彼此用逗号","隔开。注意大小写一致。
    *注: Dependencies、RequiredDeps、Dependancies[sic]结果是一样的。*
- OptionalDeps   
    ```
    ## OptionalDeps: someAddOn, someOtherAddOn
    ```
    可选依赖是当前插件为了实现某些附加功能而依赖的外部库或插件，但如果依赖的东西不存在，那么当前插件也可以正常工作，但是使用可选依赖的插件必须写明当依赖不存在的时候也可以工作。
- LoadOnDemand
    ```
    ## LoadOnDemand: 1
    ## LoadOnDemand: 0
    ```
    从1.7开始，插件可以用命令来加载，而不用非得在用户第一次登录的时候加载。如果启用这一功能，此插件则必须在未来某个时候被另一个插件加载。这是为了避免加载一些特殊的不常用的插件而导致内存资源占用，非常有效。副魔助手(Enchantrix)就用到了这一特性。
- LoadWith   
    ```
    ## LoadWith: someAddOn, someOtherAddOn
    ```
    1.9新加的。和LoadOnDemand一起用，这使你的插件跟随某个插件一起被加载(通常是暴雪的UI模块，像Blizzard_AuctionUI)。   
- SavedVariables
    ```
    ## SavedVariables: someVariable, someOtherVariable
    ```
    从Interface版本2150开始有的，保存的变量是当前流行的存储不同人物角色的方式。这些变量在客户端启动或UI重载(reload)时被载入。SavedVariables标签现以取代RegisterForSave函数，后者已经不再受客户端支持。注意在OnLoad事件过程中，SavedVariables并未完全加载所以必须假设包含nil(空)值直到以插件文件名为参数的ADDON_LOADED事件被触发。   
    
    这比以前那个为保存注册变量的脚本强的多，因为即便你的插件被禁用或因为错误、版本不匹配等问题没有加载，SavedVariables依然会被保存。
- SavedVariablesPerCharacter   
    ```
    ## SavedVariablesPerCharacter: somePercharVariable
    ```
    这个标签和SavedVariables工作方式一样，只不过是给予每个角色创建的。以前这个标签只能用角色名来区分不同的角色，现在可以根据服务器和角色名一起来区分了。
- DefaultState
    ```   
    ## DefaultState: enabled
    ## DefaultState: disabled
    ```
    这里的enabled/disabled状态被写在WTF\Account\{youraccount}\AddOns.txt里。并且这个文件会覆盖的DisabledAddOns.txt，后者为了保持兼容仍然会被老版本的加载。   
- Secure   
    ```
    ## Secure: 1
    ```
    这个标签被添加到Blizzard_UI(1.11中作为默认UI一部分)。它的确切目的无从知晓，但一个可能就是它告诉客户端是否要为插件寻找一个签名。
- 非标准标签   
    用##标示还可以添加更多额外的信息，某些第三方插件甚至使用自行提供的信息。下面是一些常用的标签:
    ```
    ## Author: MyName                          --作者的名字，也可以是Email地址。
    ## Version: 1.0                            --插件的版本号。可以是任何字符串，因为自动更新的工具会解析数字，所以最好是以数字版本开头。
    ## X-email: Author@Domain.com              --任何以X-开头的标签。
    ## X-Date: 11-24-2016                      --插件的发布日期。
    ```
    *这些标签和Title、Notes放在一起，并可用GetAddOnMetadata("addon", "field")来调用。*

- Ace2注视标签

    和Ace不一样，Ace2直接从TOC文件提取插件元数据，特定的域(field)进行特定的处理，即Version和X-Date，那么你就可以用CVS和Subversion关键字例如$Rev$作为它们的值。
    除了正经域以外，它还可以寻找下列自定义的域。
    ```
    ## X-eMail: frankcupid@hotmail.com         --Email地址
    ## X-Website: http://wowace.com            --插件的网站。
    ## X-Category: Raid                        --插件的Ace2目录。*
    ``` 
    *注*： 此目录在AceAddon.lua中声明。
    ```
    local CATEGORIES = {
        ["Action Bars"] = "Action Bars",
        ["Auction"] = "Auction",
        ["Audio"] = "Audio",
        ["Battlegrounds/PvP"] = "Battlegrounds/PvP",
        ["Buffs"] = "Buffs",
        ["Chat/Communication"] = "Chat/Communication",
        ["Druid"] = "Druid",
        ["Hunter"] = "Hunter",
        ["Mage"] = "Mage",
        ["Paladin"] = "Paladin",
        ["Priest"] = "Priest",
        ["Rogue"] = "Rogue",
        ["Shaman"] = "Shaman",
        ["Warlock"] = "Warlock",
        ["Warrior"] = "Warrior",
        ["Healer"] = "Healer",
        ["Tank"] = "Tank",
        ["Caster"] = "Caster",
        ["Combat"] = "Combat",
        ["Compilations"] = "Compilations",
        ["Data Export"] = "Data Export",
        ["Development Tools "] = "Development Tools ",
        ["Guild"] = "Guild",
        ["Frame Modification"] = "Frame Modification",
        ["Interface Enhancements"] = "Interface Enhancements",
        ["Inventory"] = "Inventory",
        ["Library"] = "Library",
        ["Map"] = "Map",
        ["Mail"] = "Mail",
        ["Miscellaneous"] = "Miscellaneous",
        ["Quest"] = "Quest",
        ["Raid"] = "Raid",
        ["Tradeskill"] = "Tradeskill",
        ["UnitFrame"] = "UnitFrame",
    }
    ```
    
XXX.toc文件样例:

```
## Interface: 20300
## Title : My AddOn
## Notes: This AddOn does nothing but display a frame with a button
## Author: My Name
## eMail: Author@Domain.com
## URL: [http://www.wowwiki.com/](http://www.wowwiki.com/)
## Version: 1.0
## Dependencies: Sea
## OptionalDeps: Chronos
## DefaultState: enabled
## SavedVariables: settingName, otherSettingName
myAddOn.lua
myAddOn.xml
```

## XXX.xml

UI xml文件基本格式
```xml
<Ui xmlns="[http://www.blizzard.com/wow/ui/"] 
xmlns:xsi="[http://www.w3.org/2001/XMLSchema-instance"] 
xsi:schemaLocation="[http://www.blizzard.com/wow/ui/] ..\FrameXML\UI.xsd"> 
<Frame name="MyAddon_Frame"> 
</Frame> 
</Ui>
```
其中ui元素是整个Ui的根元素，下面的Frame元素是我们要添加的界面框体。我们可以这样理解，在魔兽世界中，所有的视觉界面元素都是一个Frame，比如窗口，按钮，文本框，他们都是具备特殊性质的Frame。一个Frame里面又可以包含多个Frame，例如一个窗口里面可以有几行文字，几个按钮。这样就可以构成任意复杂的界面了。 
Frame可以有一些属性，来指定他的名字，父容器，大小，位置之类的信息。我们的Frame应该具有如下的属性：
Frame代码 
```xml
<Frame name="HelloWorldTestFrame" parent="UIParent" hidden="false"> 
<Size x="300" y="150" /> 
<Anchors> 
<Anchor point="CENTER" /> 
</Anchors> 
</Frame>
```
其中name属性是指定这个Frame的名字，以便我们后面使用；parent属性指定了父容器，也就是说我们的Frame是被放在UIParent这个Frame里面的，UIParent是所有UI元素的父容器，你可以把它看作WoW的整个窗口；hidden属性决定了我们的窗口是否隐藏，为false即是不隐藏，直接显示出来；Size元素指定了窗口的大小；Anchors元素指定了窗口的位置，CENTER表示窗口应该被放在父容器的中心。 
这样子的Frame只是一个透明的容器，没有边框，没有背景，于是我们应该加上一个窗口的背景。
加上背景的Frame代码 
```xml
<Frame name="HelloWorldTestFrame" parent="UIParent" hidden="false"> 
<Size x="300" y="150" /> 
<Anchors> 
<Anchor point="CENTER" /> 
</Anchors> 
<Backdrop bgFile="Interface\DialogFrame\UI-DialogBox-Background" edgeFile="Interface\DialogFrame\UI-DialogBox-Border"> 
<BackgroundInsets> 
<AbsInset left="11" right="12" top="12" bottom="11" /> 
</BackgroundInsets> 
</Backdrop> 
</Frame>
```
Backdrop元素就是背景图专用的，bgFile指定了背景图片名，edgeFile指定了边框图片名，我们现在都用游戏内置的。BackgroundInsets元素指定了背景图的边距，这样我们可以把背景放在边框里面，而不会溢出。 现在我们的窗口有了背景，但是还没有Hello World！的文字。为了加入文字，我们还需要加入一些代码： 
```xml
<Ui xmlns="http://www.blizzard.com/wow/ui/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"> 
<Frame name="HelloWorldTestFrame" parent="UIParent" hidden="false"> 
   <Size x="300" y="150" /> 
   <Anchors> 
      <Anchor point="CENTER" /> 
   </Anchors> 
   <Backdrop bgFile="Interface\DialogFrame\UI-DialogBox-Background" edgeFile="Interface\DialogFrame\UI-DialogBox-Border"> 
      <BackgroundInsets> 
         <AbsInset left="11" right="12" top="12" bottom="11" /> 
      </BackgroundInsets> 
   </Backdrop> 
   <Layers> 
      <Layer level="ARTWORK"> 
         <FontString inherits="GameFontNormal" text="Hello World!"> 
            <Anchors> 
               <Anchor point="CENTER" relativeTo="HelloWorldTestFrame" /> 
            </Anchors> 
         </FontString> 
      </Layer> 
   </Layers> 
</Frame> 
</Ui>
```
我们的可视化元素，是被放进若干个Layer来渲染的，ARTWORK是其中一个Layer，位于背景Layer的上方。也就是说，被放进ARTWORK层的所有元素，都会被渲染在背景层的上方，盖住背景层的一切东西。层的渲染顺序是BACKGROUND，ARTWORK，OVERLAY。如果你想渲染一些东西在最上层，就放进OVERLAY层里。我们现在选择ARTWORK层当作我们Hello World文字的渲染层。 FontString就是可以画一些文字的地方，inherits是指所有未明确标注的属性都由GameFontNormal类继承，比如文字的字体、大小、颜色什么的，我们就不一一设置了，直接偷懒从GameFontNormal拿来。text属性就是我们要渲染出来的文字内容。Anchors元素继续用来指定位置。把上面的文件内容存入HelloWorld.xml中，就算搞定啦！ 
[http://bbs.ngacn.cc/read.php?tid=2013286&forder_by=postdatedesc](http://bbs.ngacn.cc/read.php?tid=2013286&forder_by=postdatedesc)
