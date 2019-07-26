# ipa相关

## [iOS App 签名的原理](https://wereadteam.github.io/2017/03/13/Signature/)

## 查看ipa的签名信息

1. 改后缀为zip并解压, 得到Payload文件. 
2. 终端进入Payload/xxx.app/目录.
	* 签名过的ipa会有`embedded.mobileprovision`文件.
	* 脱壳的ipa没有? (unCover的ipa没看到这个文件)
3. 输入`security cms -D -i embedded.mobileprovision`即可查看签名相关信息.

## ipa脱壳

### 原理

	> 对软件进行逆向操作，对已加密的软件进行解密，从而获取真实软件源码。App Store下载的包全都是经过苹果加密过的包。苹果不允许开发者自己加密ipa包，加密后的ipa包，我们是无法对其进行反编译的。也无法class-dump，需要对其进行解密才能反编译。(企业除外)

#### 静态脱壳

	使用已知的解密方法对软件进行解密叫静态砸壳，静态砸壳难度大，需要知道其软件的加密算法才能对其解密。

#### 动态脱壳
	从进程的内存空间中获取软件镜像(image)进行转存处理叫动态砸壳，动态砸壳无需关心软件的加密技术，只需要从内存中获取即可，这种方法相对简单。

	![](./ipa脱壳.png)

### 工具
	
	> ios11.4.1的系统三种都试过没法用, 好像非完美越狱的都不一定能脱壳成功.

	* [Clutch](https://github.com/KJCracks/Clutch)
	* [dumpdecrypted](https://github.com/stefanesser/dumpdecrypted)
	* frida-ios-dump [frida cydia源](https://build.frida.re/frida/)

## 实战

	[移动App入侵与逆向破解技术－iOS篇](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653577384&idx=1&sn=b44a9c9651bf09c5bea7e0337031c53c&scene=0%23wechat_redirect)











