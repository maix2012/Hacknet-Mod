# 有关 EXE 部分的高级用法与常见问题总结

## PS：本文档建议在完全理解 EXE 部分且有一定编程能力后阅读

### 一、目标计算机需求前置端口的破解设计

这种需要前置端口的 EXE 设计模式非常实用

在你的自定义 EXE 中，可以借鉴这种设计思路，通过检查端口、服务状态等条件来创建更有趣的破解工具。

---

### 二、SSL-443 破解器代码示例与分析

最典型的例子就是 **SSL-443** 破解器，它需要 **Web-80**、**Ftp-21**、**SSH-22**、**RTSP-554** 的其中一个开启后才能进行破解。

```csharp
using System;
using Hacknet.Effects;
using Hacknet.Gui;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;

namespace Hacknet;

internal class SSLPortExe : ExeModule
{
    // 定义SSL模块的四种工作模式枚举
    // 每种模式对应一个特定的服务端口
    private enum SSLMode
    {
        SSH,    // SSH服务模式，对应端口22
        FTP,    // FTP服务模式，对应端口21
        Web,    // Web服务模式，对应端口80
        RTSP    // RTSP服务模式，对应端口554
    }

    // 定义程序运行的时间常量
    private const float RUN_TIME = 12f;    // 程序完成破解所需的时间（秒）
    private const float IDLE_TIME = 15f;   // 程序完全退出前等待的时间（秒）
  
    // 程序运行状态变量
    private float elapsedTime = 0f;        // 记录程序已运行的时间
    private SSLMode Mode = SSLMode.SSH;    // 当前SSL模块的工作模式，默认为SSH
    private Vector2 LastCentralRenderOffset = Vector2.Zero;  // 用于界面渲染的偏移量
    private bool IsComplete = false;       // 标记程序是否已完成破解

    // 私有构造函数，初始化SSL破解器实例
    private SSLPortExe(Rectangle location, OS operatingSystem, SSLMode mode)
        : base(location, operatingSystem)
    {
        name = "SSLTrojan";                // 程序显示名称
        ramCost = 220;                     // 程序运行所需内存
        IdentifierName = "SSLTrojan";      // 程序标识名称
        Mode = mode;                       // 设置工作模式
        // 标记对目标计算机
        ((os.connectedComp == null) ? os.thisComputer : os.connectedComp).hostileActionTaken();
    }

    // 静态工厂方法：根据命令行参数生成SSL破解器实例
    // 如果参数无效则返回null并显示错误信息
    public static SSLPortExe GenerateInstanceOrNullFromArguments(string[] args, Rectangle location, object osObj, Computer target)
    {
        OS oS = (OS)osObj;  // 获取操作系统实例
      
        // 检查参数数量是否足够（需要4个参数：程序名、目标端口、模式、模式端口）
        if (args.Length < 4)
        {
            // 显示详细的用法说明和错误信息
            oS.write("--------------------------------------");
            oS.write("SSLTrojan " + LocaleTerms.Loc("ERROR: Not enough arguments!"));
            oS.write(LocaleTerms.Loc("Usage:") + " SSLTrojan [" + LocaleTerms.Loc("PortNum") + "] [-" + LocaleTerms.Loc("option") + "] [" + LocaleTerms.Loc("option port num") + "]");
            oS.write(LocaleTerms.Loc("Valid Options:") + " [-s (SSH)] [-f (FTP)] [-w (HTTP)] [-r (RTSP)]");
            oS.write("--------------------------------------");
            return null;  // 参数不足，返回null
        }
      
        try
        {
            // 解析命令行参数
            int num = Convert.ToInt32(args[1]);     // 目标端口号（443）
            int num2 = Convert.ToInt32(args[3]);    // 模式对应的端口号（如22）
            string text = args[2].ToLower();        // 模式选项（如"-s"）
          
            // 根据模式选项设置SSL工作模式
            SSLMode sSLMode = SSLMode.SSH;  // 默认模式为SSH
            switch (text)
            {
                case "-s":
                    sSLMode = SSLMode.SSH;  // SSH模式
                    break;
                case "-f":
                    sSLMode = SSLMode.FTP;  // FTP模式
                    break;
                case "-w":
                    sSLMode = SSLMode.Web;  // Web模式
                    break;
                case "-r":
                    sSLMode = SSLMode.RTSP; // RTSP模式
                    break;
                default:
                    text = null;  // 无效模式选项
                    break;
            }

            // 检查模式选项是否有效
            if (text == null)
            {
                oS.write("--------------------------------------");
                oS.write("SSLTrojan " + string.Format(LocaleTerms.Loc("Error: Mode {0} is invalid."), args[2]));
                oS.write(LocaleTerms.Loc("Valid Options:") + " [-s (SSH)] [-f (FTP)] [-w (HTTP)] [-r (RTSP)]");
                oS.write("--------------------------------------");
                return null;  // 模式无效，返回null
            }

            // 根据所选模式检查对应端口状态
            int num3 = -1;      // 存储期望的端口号
            bool flag = false;  // 标记端口是否开放
            switch (sSLMode)
            {
                case SSLMode.SSH:
                    flag = target.isPortOpen(22);                           // 检查SSH端口22是否开放
                    num3 = target.GetDisplayPortNumberFromCodePort(22);     // 获取SSH端口的显示编号
                    break;
                case SSLMode.FTP:
                    flag = target.isPortOpen(21);                           // 检查FTP端口21是否开放
                    num3 = target.GetDisplayPortNumberFromCodePort(21);     // 获取FTP端口的显示编号
                    break;
                case SSLMode.Web:
                    flag = target.isPortOpen(80);                           // 检查Web端口80是否开放
                    num3 = target.GetDisplayPortNumberFromCodePort(80);     // 获取Web端口的显示编号
                    break;
                case SSLMode.RTSP:
                    flag = target.isPortOpen(554);                          // 检查RTSP端口554是否开放
                    num3 = target.GetDisplayPortNumberFromCodePort(554);    // 获取RTSP端口的显示编号
                    break;
            }

            // 如果前置端口未开放，显示错误信息
            if (!flag)
            {
                oS.write("--------------------------------------");
                oS.write("SSLTrojan " + LocaleTerms.Loc("Error: Target bypass port is closed!"));
                return null;  // 端口关闭，返回null
            }

            // 验证用户输入的端口号是否与期望值匹配
            int num4 = -1;
            try
            {
                num4 = Convert.ToInt32(num2);  // 尝试转换用户输入的端口号
            }
            catch (FormatException)
            {
                // 端口号格式无效
                oS.write("--------------------------------------");
                oS.write("SSLTrojan " + string.Format(LocaleTerms.Loc("Error: Invalid tunnel port number : \"{0}\""), num2));
                return null;  // 端口号无效，返回null
            }

            // 检查用户输入的端口号是否与所选模式对应的标准端口匹配
            if (num4 != num3)
            {
                oS.write("--------------------------------------");
                oS.write("SSLTrojan " + string.Format(LocaleTerms.Loc("Error: Tunnel port number {0} does not match expected service \"{1}"), num4, text));
                return null;  // 端口号不匹配，返回null
            }

            // 所有验证通过，创建并返回SSL破解器实例
            return new SSLPortExe(location, oS, sSLMode);
        }
        catch (Exception ex2)
        {
            // 处理其他未知异常
            oS.write("SSLTrojan " + LocaleTerms.Loc("Error:"));
            oS.write(ex2.Message);
            return null;  // 发生异常，返回null
        }
    }

    // 每帧更新方法：处理程序运行逻辑和时间控制
    public override void Update(float t)
    {
        base.Update(t);
      
        // 检查是否达到破解完成时间（12秒）
        if (elapsedTime < 12f && elapsedTime + t >= 12f)
        {
            Completed();  // 调用破解完成方法
        }
        // 检查是否达到程序退出时间（15秒）
        else if (elapsedTime < 15f && elapsedTime + t >= 15f)
        {
            isExiting = true;  // 标记程序开始退出
        }
      
        elapsedTime += t;  // 累计运行时间
    }

    // 破解完成方法：打开目标计算机的443端口
    public override void Completed()
    {
        base.Completed();
      
        // 获取目标计算机实例
        Computer computer = Programs.getComputer(os, targetIP);
        if (computer != null)
        {
            computer.openPort(443, os.thisComputer.ip);  // 打开SSL端口443
            os.write(":: " + LocaleTerms.Loc("HTTPS SSL Reverse Tunnel Opened"));  // 显示成功信息
            IsComplete = true;  // 标记破解完成
        }
    }
}
```

---

### 三、代码解析与关键点说明

#### 1. 枚举类型定义与模式选择

```csharp
private enum SSLMode
{
    SSH,    // SSH服务模式，对应端口22
    FTP,    // FTP服务模式，对应端口21
    Web,    // Web服务模式，对应端口80
    RTSP    // RTSP服务模式，对应端口554
}
```

**作用**：定义了SSL破解器支持的四种工作模式，每种模式对应一个特定的服务端口。这种设计使得程序能够通过不同的前置服务来破解SSL端口。

#### 2. 参数数量验证

```csharp
if (args.Length < 4)
{
    // 显示详细的用法说明和错误信息
    oS.write("--------------------------------------");
    oS.write("SSLTrojan " + LocaleTerms.Loc("ERROR: Not enough arguments!"));
    oS.write(LocaleTerms.Loc("Usage:") + " SSLTrojan [" + LocaleTerms.Loc("PortNum") + "] [-" + LocaleTerms.Loc("option") + "] [" + LocaleTerms.Loc("option port num") + "]");
    oS.write(LocaleTerms.Loc("Valid Options:") + " [-s (SSH)] [-f (FTP)] [-w (HTTP)] [-r (RTSP)]");
    oS.write("--------------------------------------");
    return null;  // 参数不足，返回null
}
```

**作用**：验证命令行参数的数量是否足够。典型的用法是：

```
@IP>>> SSLTrojan 443 -s 22
```

参数解析：
- **Args[0]** = "SSLTrojan" (程序名称)
- **Args[1]** = "443" (目标SSL端口)
- **Args[2]** = "-s" (模式选项，这里是SSH模式)
- **Args[3]** = "22" (模式对应的端口号)

#### 3. 模式选择与端口验证逻辑

```csharp
// 根据模式选项设置SSL工作模式
SSLMode sSLMode = SSLMode.SSH;  // 默认模式为SSH
switch (text)
{
    case "-s": sSLMode = SSLMode.SSH; break;   // SSH模式
    case "-f": sSLMode = SSLMode.FTP; break;   // FTP模式
    case "-w": sSLMode = SSLMode.Web; break;   // Web模式
    case "-r": sSLMode = SSLMode.RTSP; break;  // RTSP模式
    default: text = null; break;               // 无效模式选项
}

// 根据所选模式检查对应端口状态
int num3 = -1;      // 存储期望的端口号
bool flag = false;  // 标记端口是否开放
switch (sSLMode)
{
    case SSLMode.SSH:
        flag = target.isPortOpen(22);       // 检查SSH端口22是否开放
        num3 = target.GetDisplayPortNumberFromCodePort(22);  // 获取SSH端口的显示编号
        break;
    // ... 其他模式类似
}
```

**关键点**：
- 使用`target.isPortOpen()`方法检查目标计算机的对应端口是否开放
- 使用`target.GetDisplayPortNumberFromCodePort()`获取端口的显示编号，这考虑了端口重映射的情况
- 如果前置端口未开放，程序会显示错误信息并退出

#### 4. 端口号验证与匹配检查

```csharp
// 验证用户输入的端口号是否与期望值匹配
if (num4 != num3)
{
    oS.write("--------------------------------------");
    oS.write("SSLTrojan " + string.Format(LocaleTerms.Loc("Error: Tunnel port number {0} does not match expected service \"{1}"), num4, text));
    return null;  // 端口号不匹配，返回null
}
```

**作用**：确保用户输入的端口号与所选模式对应的标准端口匹配，防止用户输入错误的端口号。

#### 5. 时间控制与破解完成逻辑

```csharp
public override void Update(float t)
{
    base.Update(t);
  
    // 检查是否达到破解完成时间（12秒）
    if (elapsedTime < 12f && elapsedTime + t >= 12f)
    {
        Completed();  // 调用破解完成方法
    }
    // 检查是否达到程序退出时间（15秒）
    else if (elapsedTime < 15f && elapsedTime + t >= 15f)
    {
        isExiting = true;  // 标记程序开始退出
    }
  
    elapsedTime += t;  // 累计运行时间
}
```

**时间流程**：
- **0-12秒**：程序运行破解过程
- **12秒**：破解完成，打开目标SSL端口
- **12-15秒**：显示成功信息，程序继续运行但不进行破解
- **15秒**：程序开始退出

---

### 三、为什么我们不更近一步呢？


  如果你想要设计这样的效果该怎么办呢？

  运行EXE后检测Proxy是否开启，并要求代理过载后和需要SSH端口开放后才能运行，完成后会自动破解防火墙，但是会关闭SSH端口。

  想要实现，你可以这么做。
  ```csharp
    // 检查Proxy是否开启
    Computer computer = Programs.getComputer(os, os.thisComputer.ip);
    if (computer.proxyActive)//检查代理是否启动
    {
        os.write("Error: Proxy is not enabled!");
        needsremoved = true;
        return;
    }
    if (!computer.isPortOpen(computer.GetDisplayPortNumberFromCodePort(22))){
        os.write("Error: SSH port is not open!");
        needsremoved = true;
        return;
    }//检查SSH端口是否开放
    
  ```
  将这些放在 **LoadContent()** 中，用来达成检测。
  同时，我们在 **Update()** 中，我们在破解后开放你的端口的代码下方添加如下代码
  ```csharp
    // 开放端口
    if (IsComplete)
    {
        //---------------------------------------------
        //你的其他代码和打开这个破解器对应端口的代码
        //---------------------------------------------
        computer.closePort(computer.GetDisplayPortNumberFromCodePort(22),computer.ip);//关闭SSH端口
        computer.firewall.solved = true;//解决防火墙
    }
  ```
  或者，再劲爆点，直接给玩家丢个forkbomb？
  ```csharp
  os.execute("forkbomb");
  ```
### 四、一些AI导致的错误和解决方案

#### 1.AI画EXE动画时导致EXE的上方信息条被覆盖

这是一个很常见的问题，由于AI会错误的计算上下边框的距离导致。

解决方案：加入提示词：
```csharp

drawTarget("app:");
Rectangle drawArea = Hacknet.Utils.InsetRectangle(new Rectangle(this.bounds.X, this.bounds.Y + Module.PANEL_HEIGHT, this.bounds.Width, this.bounds.Height - Module.PANEL_HEIGHT), 2);
```
确保绘制区域在上下边框之间，而不会覆盖原本顶部的一小条。
#### 2.AI画EXE动画时，使用了错误的重载方法/冲突的变量名
解决方案：

1.在按住 **CTRL+左键** 情况下，点击那个有问题的函数，然后在新弹出的反编译文件中复制整个函数体连带着报错一起发给AI

2.如果是变量名冲突（这个比较少见，我就见过一回。是AI使用了sb和原版的函数定义的sb冲突了），那么就把原版的函数定义中的sb改成别的名字，或者直接告诉AI让他帮你换个名字

#### 3.字体问题
首先，我们要记住的是，在没有安装中文Mod的情况下，都是不可以在无论是 **draw()** 还是其他的输出函数中输出中文的。

这会导致一个控制台报错。

解决方案：换成英文的就行，或者安装中文Mod换字体。

#### 5.其他的
剩下的要注意的点就是，尽量在每个if判断后面使用else进行补充判断，防止出现漏判的情况。导致出现bug。

或者，再保守一些，外面再套一层try-catch，这样即使出现bug，也能保证程序的稳定运行。
