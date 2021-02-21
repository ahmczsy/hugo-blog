# WSL安装和使用LxRunOffline备份


# 安装

### 开启WSL组件

  ```powershell
  Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
  ```
### 下载镜像

  由于我的Win10是精简版所以没用应用商店所以只能手动下载镜像

  各个发行版 https://docs.microsoft.com/en-us/windows/wsl/install-manua

  <!--more-->

### 安装

  * 把Ubuntu.appx重命名为Ubuntu.zip

  * 解压

  * 对Ubuntu.exe右键以管理员运行，输入用户名和密码即安装完成

### 修改默认用户

在解压的根目录打开powershell

```powershell
ubuntu config --default-user <username>
```



# 备份WSL

### 安装LxRunOffline

https://github.com/DDoSolitary/LxRunOffline

安装好后最好重启下电脑，如果不重启可能有问题

### 常用LxRunOffline命令

```powershell
//已经安装的wsl
LxRunOffline.exe list 
//安装wsl
LxRunOffline.exe install
//备份wsl
LxRunOffline.exe export
//启动一个wsl
LxRunOffline.exe run
```

###  备份wsl

* 先查看当前系统中存在的wsl

  ```powershell
  > LxRunOffline.exe list
  
  Ubuntu-18.04
  ```

* 开始备份

  ```
  LxRunOffline.exe export -n Ubuntu-18.04 -f backup.tar.gz
  ```

  -n ：wsl的别名，就是之前用list查看的其中一个

  -f  ：备份的路径，我这直接备份到当前路径backup.tar.gz

* 还原wsl

  ```
  LxRunOffline.exe install -n new-linux -d .\ -f D:\temp\backup.tar.gz
  ```

  -n ：起个名字

  -d ：wsl安装目录

  -f ：备份文件目录

* 启动备份的wsl

  ```
   LxRunOffline.exe run -n new-linux
  ```

# 参考

https://github.com/DDoSolitary/LxRunOffline

https://docs.microsoft.com/en-us/windows/wsl/install-on-server

https://askubuntu.com/questions/816732/how-to-change-default-user-in-wsl-ubuntu-bash-on-windows-10


