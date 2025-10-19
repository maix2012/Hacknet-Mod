# 有关 EXE 部分的花里胡哨的高级用法 & 一些 AI 导致的常见问题总结

## PS：本文档建议在完全理解 EXE 部分且有一定的能力后食用

- ### 1.对于目标计算机需求其他端口进行破解

  这个最典型的例子就是**SSL-443**破解器。他需要

  **Web-80** ，**Ftp-21** ，**SSH-22** ，**RTSP-554** 的其中一个开启后进行破解。

  我们查阅源代码后可以看到这些东西。(动画部分已删除)

  ```csharp
  using System;
  using Hacknet.Effects;
  using Hacknet.Gui;
  using Microsoft.Xna.Framework;
  using Microsoft.Xna.Framework.Graphics;

  namespace Hacknet;

  internal class SSLPortExe : ExeModule
  {
    private enum SSLMode
    {
      SSH,
      FTP,
      Web,
      RTSP
    }

	private const float RUN_TIME = 12f;

	private const float IDLE_TIME = 15f;

	private float elapsedTime = 0f;

	private SSLMode Mode = SSLMode.SSH;

	private Vector2 LastCentralRenderOffset = Vector2.Zero;

	private bool IsComplete = false;

	private SSLPortExe(Rectangle location, OS operatingSystem, SSLMode mode)
		: base(location, operatingSystem)
	{
		name = "SSLTrojan";
		ramCost = 220;
		IdentifierName = "SSLTrojan";
		Mode = mode;
		((os.connectedComp == null) ? os.thisComputer : os.connectedComp).hostileActionTaken();
	}

	public static SSLPortExe GenerateInstanceOrNullFromArguments(string[] args, Rectangle location, object osObj, Computer target)
	{
		OS oS = (OS)osObj;
		if (args.Length < 4)
		{
			oS.write("--------------------------------------");
			oS.write("SSLTrojan " + LocaleTerms.Loc("ERROR: Not enough arguments!"));
			oS.write(LocaleTerms.Loc("Usage:") + " SSLTrojan [" + LocaleTerms.Loc("PortNum") + "] [-" + LocaleTerms.Loc("option") + "] [" + LocaleTerms.Loc("option port num") + "]");
			oS.write(LocaleTerms.Loc("Valid Options:") + " [-s (SSH)] [-f (FTP)] [-w (HTTP)] [-r (RTSP)]");
			oS.write("--------------------------------------");
			return null;
		}
		try
		{
			int num = Convert.ToInt32(args[1]);
			int num2 = Convert.ToInt32(args[3]);
			string text = args[2].ToLower();
			SSLMode sSLMode = SSLMode.SSH;
			switch (text)
			{
			case "-s":
				sSLMode = SSLMode.SSH;
				break;
			case "-f":
				sSLMode = SSLMode.FTP;
				break;
			case "-w":
				sSLMode = SSLMode.Web;
				break;
			case "-r":
				sSLMode = SSLMode.RTSP;
				break;
			default:
				text = null;
				break;
			}
			if (text == null)
			{
				oS.write("--------------------------------------");
				oS.write("SSLTrojan " + string.Format(LocaleTerms.Loc("Error: Mode {0} is invalid."), args[2]));
				oS.write(LocaleTerms.Loc("Valid Options:") + " [-s (SSH)] [-f (FTP)] [-w (HTTP)] [-r (RTSP)]");
				oS.write("--------------------------------------");
				return null;
			}
			int num3 = -1;
			bool flag = false;
			switch (sSLMode)
			{
			case SSLMode.SSH:
				flag = target.isPortOpen(22);
				num3 = target.GetDisplayPortNumberFromCodePort(22);
				break;
			case SSLMode.FTP:
				flag = target.isPortOpen(21);
				num3 = target.GetDisplayPortNumberFromCodePort(21);
				break;
			case SSLMode.Web:
				flag = target.isPortOpen(80);
				num3 = target.GetDisplayPortNumberFromCodePort(80);
				break;
			case SSLMode.RTSP:
				flag = target.isPortOpen(554);
				num3 = target.GetDisplayPortNumberFromCodePort(554);
				break;
			}
			if (!flag)
			{
				oS.write("--------------------------------------");
				oS.write("SSLTrojan " + LocaleTerms.Loc("Error: Target bypass port is closed!"));
				return null;
			}
			int num4 = -1;
			try
			{
				num4 = Convert.ToInt32(num2);
			}
			catch (FormatException)
			{
				oS.write("--------------------------------------");
				oS.write("SSLTrojan " + string.Format(LocaleTerms.Loc("Error: Invalid tunnel port number : \"{0}\""), num2));
				return null;
			}
			if (num4 != num3)
			{
				oS.write("--------------------------------------");
				oS.write("SSLTrojan " + string.Format(LocaleTerms.Loc("Error: Tunnel port number {0} does not match expected service \"{1}"), num4, text));
				return null;
			}
			return new SSLPortExe(location, oS, sSLMode);
		}
		catch (Exception ex2)
		{
			oS.write("SSLTrojan " + LocaleTerms.Loc("Error:"));
			oS.write(ex2.Message);
			return null;
		}
	}

	public override void Update(float t)
	{
		base.Update(t);
		if (elapsedTime < 12f && elapsedTime + t >= 12f)
		{
			Completed();
		}
		else if (elapsedTime < 15f && elapsedTime + t >= 15f)
		{
			isExiting = true;
		}
		elapsedTime += t;
	}

	public override void Completed()
	{
		base.Completed();
		Computer computer = Programs.getComputer(os, targetIP);
		if (computer != null)
		{
			computer.openPort(443, os.thisComputer.ip);
			os.write(":: " + LocaleTerms.Loc("HTTPS SSL Reverse Tunnel Opened"));
			IsComplete = true;
		}
	}
  }

  ```
