# Windows开发环境

## chocolatey(包管理):package:

官方链接：[chocolatey](https://www.chocolatey.org/)

支持的SoftWare：[chocolatey packages](https://www.chocolatey.org/packages)

### -Everything(搜索)  :arrow_down_small:

### -HexChat(IRC客户端) :arrow_down_small:

### -Docker(虚拟容器引擎) :arrow_down_small:

官方链接：[Docker](https://www.docker.com/)

官方公共仓库：[docker hub](https://hub.docker.com/)

#### 一、安装

##### -windows 家庭版安装

[windows10 家庭版安装](https://blog.csdn.net/xiaozhou_zi/article/details/86137917)

1、开启Hyper-V

```powershell
pushd "%~dp0"
 
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt
 
for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
 
del hyper-v.txt
 
Dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /LimitAccess /ALL
```

2、修改注册表，伪装成专业版绕过Docker的安装检测

```powershell
REG ADD "HKEY_LOCAL_MACHINE\software\Microsoft\Windows NT\CurrentVersion" /v EditionId /T REG_EXPAND_SZ /d Professional /F
```

3、下载并安装Docker-Desktop

```powershell
choco install docker-desktop
```

##### -windows 非家庭版

这个安装十分简单，在**控制面板-程序-启用或关闭Windows功能**中开启Hyper-V即可

#### 二、使用

#### 三、进阶

CI\DI：



##### 

### 