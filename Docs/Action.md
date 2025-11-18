# 自定义操作
## 基础行动
```csharp
public class CustomAction : Pathfinder.Action.PathfinderAction
{
    [XMLStorage]
    public string Attribute1;

    [XMLStorage]
    public string Attribute2 = "Default value!";

    public override void Trigger (object os_obj)
    {
        OS os = (OS)os_obj;
        os.write("Custom action triggered!");
        os.write(Attribute1);
        os.write(Attribute2);
    }
}
```
### 可延迟的行动
DelayablePathfinderAction是PathfinderAction的子类型，实现了DelayHost和Delay属性。(无需添加    [XMLStorage]
    public string Attribute1;属性用于存储延迟，他已经默认包含了)
```csharp
public class CustomDelayableAction : Pathfinder.Action.DelayablePathfinderAction
{
    public override void Trigger (OS os)
    {
        if (DelayHost != null)
        {
            os.write("Custom action triggered after " + Delay + " seconds with host " + DelayHost + "!");
        }
    }
}
```
### 注册
操作可以手动注册，也可以使用操作属性注册。
```csharp
Pathfinder.Action.ActionManager.RegisterAction<CustomAction>("CustomActionXMLTag");
```
或者
```csharp

[Pathfinder.Meta.Load.Action("CustomActionXMLTag")]
public class CustomAction : PathfinderAction
```
### 将自定义操作添加到操作文件中
```xml
<CustomActionXMLTag Attribute1="Value" />
```

Action更类似于一个用于连接Mod功能与Extension作者推进剧情的助手，它的及时性和延迟效果更适合用来触发一些临时性的事件。同时你也可以将一些Action或者原版没有的操作进行打包，方便在Extension中使用。

下一章我们会讲解它的触发条件的自定义方法。

这里，我们以一个简单的Action例子为例。他的效果是延迟后执行一个指令。

```csharp
[Pathfinder.Meta.Load.Action("CommanderAction")]
public class CommanderAction : Pathfinder.Action.DelayablePathfinderAction
{
    public override void Trigger (OS os)
    {
        [XMLStorage]
        public string Command;


        if (DelayHost != null)
        {
            os.execute(Command);
        }
    }
}
```

