# Issues
1. MAC系统无法自动打包补丁，原因可能是路径分隔符问题
2. 使用谷歌multidex分包后无法注入代码(开启multidex之后，jar包保存路径改变了)
3. 暂不支持productFlavors

以上问题有空再改，最近正在忙其他事情，公司项目也准备重构。
而且此项目主要是科普和学习热补丁技术，有兴趣的可以自行解决上述问题。

# 一、HotFix简介
一个基于dex分包的热补丁框架，目前只支持gradle 1.5以上版本
具有以下特性：

1. 支持混淆
2. 自动生成补丁包
3. 加载补丁包时进行签名校验

项目演示

![image](https://github.com/AItsuki/HotFix/raw/master/image/Demo.gif)

# 二、使用方式
首先在build.gradle中有两个dsl需要进行配置。

![image](https://github.com/AItsuki/HotFix/raw/master/image/dsl.png)

### 2.1 fixMode
- fixMode是在debug模式下运行项目的配置，可以控制是否使用javassist注入代码，是否自动生成补丁，日常开发的话两个设置成false即可
- 因为注入代码后，自定义控件在preview预览的时候会报空指针（找不到Antilazy.class），所以需要将debugOn关掉才能方便预览。

### 2.2 fixSignConfig
这个是配置补丁包的签名文件，需要和Release签名打包时使用的一致，否则加载补丁的时候会校验失败，这也是为了安全性考虑，防止恶意注入代码。

storeFile，storePassword，keyAlias，keyPassword对应如下

![image](https://github.com/AItsuki/HotFix/raw/master/image/sign.png)

build.gradle配置完毕后，只需要运行一次Release签名打包，然后修改代码，再次运行debug打包即可自动生成补丁了。

# 三、说明
1. 在Release签名打包的时候会重新生成hash.txt，如果开启混淆的话还会生成mapping.txt，自动生成的补丁包是基于这个版本校验而来的。
2. 在debug模式下直接运行或者打包会校验hash.txt和mapping.txt，自动生成补丁包并且为补丁包签名。
3. 将生成的补丁包复制到sdcard根目录，重启应用即可实现热修复。

需要注意的是，如果在Release打包中开启了混淆，那么自动生成补丁的时候也需要将debug开启混淆，否则会将整个项目的所有类都打包成补丁包。

debug开启混淆方式如下

![image](https://github.com/AItsuki/HotFix/raw/master/image/debug-minify.png)

### 3.1 关于签名校验
只有使用Release产出的apk，加载补丁的时候才会进行签名校验。

如果你手机上安装的是debug包，那么不会进行签名校验。

### 3.2 关于android6.0以上
补丁包不能从sdcard中加载，因为android6.0后有运行时权限处理。从sdcard中加载只是为了方便测试和演示，一般情况下是建议放在私有目录中。
