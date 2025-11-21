# 开始
### 欢迎来到 Hacknet-mod 文档库！

### 本文档库将帮助你快速上手 Hacknet-mod，并提供详细的教程和使用说明。
### 下面是一些准备工作，如果你颇有自信，你可以选择跳过他们翻到文档底部。
## 一、我需要准备什么？
-------------------
- ### 1.一台安装了Microsoft Visual Studio 2022的电脑和.net环境
- ### 2.拥有DLC+已经安装的Pathfinder
- ### 3.一定的编程基础知识（最好是C#的，python需要过渡，c++和java问题不大）
- ### 4.耐心
-------------------
## 二、开始配置环境
- ### 1.先前往官网下载符合电脑规格的Visual Studio 2022和.NET 4.5
[VS 2022](https://visualstudio.microsoft.com/zh-hans/thank-you-downloading-visual-studio/?sku=Community&channel=Release&version=VS2022&source=VSLandingPage&cid=2030&passive=false) \
[.NET 4.5](https://dotnet.microsoft.com/zh-cn/download/dotnet-framework/thank-you/net45-web-installer)
- ### 2.下载这个模版完成准备工作
[模版](https://github.com/Windows10CE/HacknetPluginTemplate)
- ### 3.首先用记事本打开模版的.sln解决方案文件你会看到如下内容
```
﻿
Microsoft Visual Studio Solution File, Format Version 12.00
Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "HacknetPluginTemplate", "HacknetPluginTemplate.csproj", "{8F66B2F8-F985-4B27-ABAD-42E9CD52B911}"
EndProject
Global
	GlobalSection(SolutionConfigurationPlatforms) = preSolution
		Debug|Any CPU = Debug|Any CPU
		Release|Any CPU = Release|Any CPU
	EndGlobalSection
	GlobalSection(ProjectConfigurationPlatforms) = postSolution
		{8F66B2F8-F985-4B27-ABAD-42E9CD52B911}.Debug|Any CPU.ActiveCfg = Debug|Any CPU
		{8F66B2F8-F985-4B27-ABAD-42E9CD52B911}.Debug|Any CPU.Build.0 = Debug|Any CPU
		{8F66B2F8-F985-4B27-ABAD-42E9CD52B911}.Release|Any CPU.ActiveCfg = Release|Any CPU
		{8F66B2F8-F985-4B27-ABAD-42E9CD52B911}.Release|Any CPU.Build.0 = Release|Any CPU
	EndGlobalSection
EndGlobal

```
#### 修改其中的 **HacknetPluginTemplate**和 **HacknetPluginTemplate.csproj**分别为你的模组名和你改名后的.csproj文件名并保存。
#### 现在，重命名.csproj文件为你修改后的文件名然后打开解决方案文件（.sln）。
# 到这里，恭喜你，你已经完成了最麻烦的部分，接下来就是愉快的mod开发过程了。
#### 点击这里开始你的第一个破解器开发吧！
[你的第一个破解器](./你的第一个破解器.md)
