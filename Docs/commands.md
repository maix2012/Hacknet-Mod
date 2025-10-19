# 你的第一个Command
## 一、如何使用模版
打开由Arkhist在Pathfinder内部提供的[Command模版](https://arkhist.github.io/Hacknet-Pathfinder/mod/commands/)
```c#
public static void TestCommand(OS os/*声明系统*/, string[] args /*声明参数*/)
{
    os.write("Arguments passed in: " + string.Join(" ", args));
}
```
并通过
```c#
Pathfinder.Command.CommandManager.RegisterCommand("CommandName", TestCommand);
```
注册你的命令

（PS：另外一个版本由中括号括起来的我不是很推荐，因为一旦你的mod代码没有做好分类，大概率就会出现EXE注册到Action里面的情况出现，迫使你再找一遍bug……）

下面，我们通过一个小例子完成Command的基本使用指南
```c#
public static void ExampleCommand(OS os, string[] args)
{
    
    try // 尝试输出
    {
        os.write(args[1]);
    }
    catch // 捕获错误
    {
        return;
    }
}
```