---
layout: post
title: "IIS8中使用OpenSSL来创建CA并且签发SSL证书"
date: 2013-06-12 00:49
comments: true
categories: OpenSSL
keywords: IIS8, OpenSSL, CA, SSL, certificate, sign, 证书, 签名
description: IIS8中使用OpenSSL来创建CA并且签发SSL证书
---

##前言

最近在为新的iOS app考虑安全机制，第一个进入脑海里的就是HTTPS和SSL。所以研究了一下Windows服务器下IIS部署HTTPS和证书的方法，以及如何让app与server进行安全的信息交互。

由于网上千篇一律的只是教大家怎么怎么操作，并没有告诉大家为什么这么操作。而作为一个喜欢打破砂锅问到底的强迫症患者，自己又花了一些时间研究了各个步骤及参数的原理，在这里把这些小小的理解及经验记录下来，即给有同样需求的同行们做个参考，也给未来的自己留作备份。

##申明

本人并非互联网安全专家，也不是OpenSSL老手。如果这篇文章对你有用，本人非常高兴。如果无法解决问题，你可以google其他更加专业的文章，我相信只要花点时间肯定能够找到答案。本文中所有步骤都经过本人多次测试，但不能保证一定正确，在此仅供参考。如有不对之处，欢迎留言探讨及指正。

##准备

* Windows 8 + IIS 8
* 直接下载编译好的[OpenSSL](http://slproweb.com/products/Win32OpenSSL.html)，由于我的系统是64位的，所以我下的是最新版的**Win64 OpenSSL v1.0.1e Light**
* 安装OpenSSL之前要先装**Visual C++ 2008 SP1 Redistributables**，根据系统选择[32位](http://www.microsoft.com/zh-cn/download/details.aspx?id=5582)的和[64位](http://www.microsoft.com/zh-cn/download/details.aspx?id=2092)下载并安装

##开始

###第一步：安装OpenSSL

1. 尽管我们已经安装了**Visual C++ 2008 SP1 Redistributables**，安装刚开始还是会提示未安装**Visual C++ 2008 Redistributables**，不管它直接点击确定
2. 一路下一步就可以了，安装文件夹我选择**C:\OpenSSL**
3. 在**Copy OpenSSL DLLs to:**的地方我选择**The OpenSSL binaries (/bin) directory**，我不喜欢把什么DLL都往Windows目录丢，这样放在应用程序目录下比较干净
4. 完成安装

###第二步：配置OpenSSL

1. 将路径**C:\OpenSSL\bin\\**添加到系统路径中（控制面板 > 系统与安全 > 系统 > 高级系统设置 > 环境变量 > 系统变量 > Path），这样在任何路径中都能运行OpenSSL命令
2. 打开openssl.cfg，修改一下配置：
```
dir		= . # 存放CA文件的文件夹，里面还需要手动建立子文件夹及文件，后面会提
default_days		= 10950 # 证书有效期，设为30年比较省心
policy		= policy_anything # CA资料和证书申请资料的匹配策略，改为这个比较方便
countryName_default		= CN # 默认国家
stateOrProvinceName_default		= Jiangxi # 默认省份
localityName_default		= Nanchang # 默认城市，在localityName = Locality Name (eg, city)下增加这一条
0.organizationName_default		= Kashuo # 默认组织
```
3. 接着准备文件夹及文件：
	* 新建文件夹**C:\OpenSSL\bin\KashuoCA**
	* 新建文件夹**C:\OpenSSL\bin\KashuoCA\newcerts**
	* 新建文件**C:\OpenSSL\bin\KashuoCA\serial**（无后缀名），里面写入`01`，用来存放签发证书流水号
	* 新建空文件**C:\OpenSSL\bin\KashuoCA\index.txt**，用来存放签发证书记录
4. 为了省去每次运行命令都要指定openssl.cfg的麻烦，打开命令提示符（cmd.exe），将cfg文件设为系统变量：
```
set OPENSSL_CONF=C:\OpenSSL\bin\openssl.cfg
```
5. 重新打开命令提示符，进入KashuoCA文件夹：`cd C:\OpenSSL\bin\KashuoCA`，准备好以后开始下面的步骤

###第三步：建立CA

命令：`openssl req -x509 -newkey rsa:1024 -keyout ca.key -out ca.cer`
参数：
**req** # 用来生成PKCS#10格式的证书申请文件，也可以生成自签名的CA根证书
**-x509** # 有这个参数就是生成CA根证书，没有就是生成证书申请文件
**-newkey rsa:1024** # 同时生成1024位RSA算法的私钥
**-keyout** # CA私钥
**-out** # 证书申请文件或CA根证书
由于我们之前已经设置了证书有效期和cfg文件的地址，所以在命令里就不需要重复设置了

运行命令以后会要求输入私钥密码，并且再输入一次确认密码。在输入国家省份等资料的时候直接回车使用之前我们设置的默认值就可以了，但是在**Organizational Unit Name**、**Common Name**和**Email Address**三个地方没有设置默认值，因为这三个资料在CA证书和服务器证书里是不一样的。

```
C:\OpenSSL\bin\KashuoCA>openssl req -x509 -newkey rsa:1024 -keyout ca.key -out ca.cerLoading 'screen' into random state - doneGenerating a 1024 bit RSA private key...........++++++.........................++++++writing new private key to 'ca.key'Enter PEM pass phrase:Verifying - Enter PEM pass phrase:-----You are about to be asked to enter information that will be incorporatedinto your certificate request.What you are about to enter is what is called a Distinguished Name or a DN.There are quite a few fields but you can leave some blankFor some fields there will be a default value,If you enter '.', the field will be left blank.-----Country Name (2 letter code) [CN]:State or Province Name (full name) [Jiangxi]:Locality Name (eg, city) [Nanchang]:Organization Name (eg, company) [Kashuo]:Organizational Unit Name (eg, section) []:KashuoCACommon Name (e.g. server FQDN or YOUR name) []:KashuoCAEmail Address []:ca@kashuo.com
```

这时候我们就有了CA根证书和私钥了！

###第四步：通过IIS生成证书申请文件

由于本例中证书文件是部署在IIS中，所以通过IIS直接生成证书文件会比较方便。当然第二步中提到了通过OpenSSL的req方法也可以生成证书申请文件。

1. 打开**IIS**
2. 在左侧**连接**中选择服务器
3. 在中间**主页**里的**IIS**中选择**服务器证书**
4. 在右侧**操作**中选择**创建证书申请**
5. 在打开的窗口中填入以下信息：
```
通用名称：www.kashuo.com
组织：Kashuo
组织单位：KashuoServer
城市/地点：Nanchang
省/市/自治区：Jiangxi
国家/地区：CN
```
通用名称里填写的域名要与该证书所绑定的网站域名一致，否则用户在浏览网站的时候会提示证书与域名不一致。
6. 加密服务选择RSA和1024位
7. 最后保存证书申请的文件为**C:\OpenSSL\bin\KashuoCA\certreq.txt**

###第五步：签发服务器证书

现在，CA证书文件ca.cer、CA私钥ca.key、服务器证书申请certreq.txt三个文件都在KashuoCA文件夹下

命令：`openssl ca -in certreq.txt -cert ca.cer -keyfile ca.key -out iis.cer`
参数：
**ca** # 主要用来签发证书申请
**-in** # 证书申请文件
**-cert** # CA证书
**-keyfile** # CA私钥
**-out** # 签发的证书

```
C:\OpenSSL\bin\KashuoCA>openssl ca -in certreq.txt -cert ca.cer -keyfile ca.key -out iis.cerUsing configuration from C:\OpenSSL\bin\openssl.cfgLoading 'screen' into random state - doneEnter pass phrase for ca.key:Check that the request matches the signatureSignature okCertificate Details:        Serial Number: 1 (0x1)        Validity            Not Before: Jun 13 10:32:25 2013 GMT            Not After : Jun  6 10:32:25 2043 GMT        Subject:            countryName               = CN            stateOrProvinceName       = Jiangxi            localityName              = Nanchang            organizationName          = Kashuo            organizationalUnitName    = KashuoServer            commonName                = www.kashuo.com        X509v3 extensions:            X509v3 Basic Constraints:                CA:FALSE            Netscape Comment:                OpenSSL Generated Certificate            X509v3 Subject Key Identifier:                E0:8A:69:4A:D1:0A:98:26:EA:AE:AF:5E:6D:A7:A7:C4:DE:07:13:DF            X509v3 Authority Key Identifier:                keyid:37:48:69:62:0E:FD:FB:1E:83:EB:DE:2D:0D:F6:55:C1:E1:76:EF:BACertificate is to be certified until Jun  6 10:32:25 2043 GMT (10950 days)Sign the certificate? [y/n]:y1 out of 1 certificate requests certified, commit? [y/n]yWrite out database with 1 new entriesData Base Updated
```

最后我们得到了服务器证书**iis.cer**

###第六步：在服务器上导入CA根证书

由于第三步生成的CA根证书是自签名的，并非由系统可以识别的第三方信任机构签发，所以需要将CA根证书导入到服务器中。

1. 双击**ca.cer**打开证书详情
2. 点击**安装证书**打开**证书导入向导**
3. 存储位置选择**本地计算机**然后下一步
4. 选择**将所有的证书都放入下列存储**
5. 点击浏览，选择**受信任的根证据颁发机构**
6. 导入完成

###第七步：完成IIS证书申请

现在可以将第五步生成的**iis.cer**导入到IIS中了：

1. 点击第四步**创建证书申请**下方的**完成证书申请**
2. 选择证书文件：**C:\OpenSSL\bin\KashuoCA\iis.cer**
3. 输入一个好记名称，如**www.kashuo.com**
4. 证书存储默认为**个人**不变
5. 点击确定完成证书导入

这个时候在服务器证书列表里就可以看到这个证书了。

###第八步：打开网站的SSL设置

安装好IIS证书以后就可以打开网站的SSL设置了：

1. 在IIS左侧**连接**中选择**网站**
2. 在右侧**操作**中点击**绑定**
3. 在打开的**网站绑定**中添加一条记录：
	* **类型**选择**https**
	* **主机名**填写绑定的域名，如**www.kashuo.com**
	* **证书**就选择我们刚刚添加的证书
	* 确定完成
4. 然后在中间**主页**里的**IIS**中选择**SSL设置**，这里根据自己的需求进行设置
	* **要求SSL**：勾选以后只能通过https访问，否则http和https都可以访问
	* **客户证书**：
		* 忽略：不论客户端有没有证书都不检验
		* 接受：如果客户端没有证书就不检验，否则弹出提示框让用户选择证书并检验证书合法性
		* 必需：客户端必需提供合法证书才能进行访问
5. 设置完成以后点击右边**应用**就可以了

对于单向认证SSL连接，在**客户证书**里选择**忽略**就可以，教程到这里也就结束了。客户端在访问https地址的时候会收到一个提示，即服务器证书并非由信任的第三方证书颁发机构颁发，客户端选择继续或者保存为信任的证书就可以继续访问了。

如果对于安全性要求很高的网站，需要使用双向认证SSL连接，在**客户证书**里就要选择**必需**，即服务端和客户端互相检验证书合法性。那就还需要为客户端生成一个服务端认可的证书。