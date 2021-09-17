# Mac 下 6.S081 的环境搭建




## 介绍

虽然我个人是计算机的科班毕业生，由于在学习操作系统的课程中并没有很好的代码实践，以至于到现在已经完全忘记了操作系统课程到内容。再加上在背各种“八股文”的过程中涉及到很多的底层的操作系统的知识，所以打算系统的学习下 mit 6.S081 课程。该课程的重点就是课后的实验（lab）。在做实验之前需要准备编程的环境，所以这篇文章记录的就是在 mac 下搭建 6.S081 课程的前置环境。



## 环境搭建

MacOS 下需要事先准备好 xcode 和 homebrew，我的系统版本是11.5.2。

homebrew 是 mac 的必备软件，所以它的安装过程就省略了

xcode 安装：`xcode-select --install`

实验环境主要包括三个部分：

- **RISC-V工具链：** 包括一系列交叉编译的工具，用于把源码编译成机器码，如gcc，binutils，glibc等
- **QEMU模拟器：** 用于在我们机器上(X86)模拟RISC-V架构的CPU
- **xv6源码：** xv6操作系统源码



### RISC-V toolchain

```shell
$ brew tap riscv/riscv
$ brew install riscv-tools
```

上面的安装好后把 riscv 加入到 mac 的环境变量中

```
PATH=$PATH:/usr/local/opt/riscv-gnu-toolchain/bin
```



### QEMU 模拟器

如果你直接按照官方的方式使用 homebrew 安装 qemu，安装好的 qemu 版本肯定大于 5.1，而 xv6 在高版本的 qemu 下编译有 [bug](https://github.com/mit-pdos/xv6-riscv/issues/76) 。所以我们只能手动的下载源码，编译 5.1 版本的 qemu



```shell
# 卸载高版本 qemu
brew uninstall qemu
# 删除 qemu 的残留
rm -rf /usr/local/bin/vdeqemu
# 下载源码
wget https://download.qemu.org/qemu-5.1.0-rc3.tar.xz
tar xvf qemu-5.1.0-rc3.tar.xz
cd qemu-5.1.0-rc3.tar.xz
./configure 
# 编译
make && make install  
```



### vx6

下载 xv6 源码

```bash
git clone git://g.csail.mit.edu/xv6-labs-2020
```

编译

```shell
 make qemu
```

退出 qemu 模拟器的方法：先按住 `ctrl` 和 `a` 松开后再按下 `x`





### 参考

* https://pdos.csail.mit.edu/6.828/2020/tools.html
* https://zhayujie.com/mit6828-env.html


