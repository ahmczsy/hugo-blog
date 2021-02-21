# 使用idea自带的plugin反编译jar


### 问题
之前想反编译jar包,在网上了找了一些反编译的工具感觉都不太好用.就来我发现idea自带的反编译插件挺好用,但是没有GUI只能使用命令.

### 使用
这个插件在<idea_home>/plugins/java-decompiler/lib/java-decompiler.jar.这个jar不能直接用java -jar运行,这个jar没有只能main函数,直接java -jar会报错,下面是使用方法
```shell
java -cp java-decompiler.jar   org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompiler -dgs=true siddhi-core.jar  mysrc
```
### 参数说明
java-decompiler.jar是插件

org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompiler是主函数入口

-dgs=true 是这个主函数的参数,具体的含义和其他参数具体参考 [该插件github链接](https://github.com/fesh0r/fernflower)

siddhi-core.jar指需要反编译的jar

mysrc指定反编译后的输出路径 

命令执行完成之后输出的是.jar文件,解压之后就是源码了


