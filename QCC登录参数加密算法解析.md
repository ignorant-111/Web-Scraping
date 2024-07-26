# QCC登录参数加密算法解析

## 1.基本信息

**目标网站**：5LyB5p+l5p+l（Base64）

**加密参数**：QCCSESSID参数，Key-Value都为加密部分的参数

**调试/抓包工具：**F12，Fiddler

## 2.参数解析

1.QCCSESSID参数

url：https://www.qcc.com/api/auth/qrcode-create

参数作用：QCCSESSID作为登录的有效Cookie，用来判断用户是否登录。

参数值获取方法：

（1）打开Fidder软件，搜索QCCSESSID关键字，找到参数最早出现的请求，请求地址为：https://www.qcc.com/

（2）在返回数据内查找Set-Cookie内容，其中就有QCCSESSID，该参数由服务器返回。



2.Key-Value都为加密部分的参数

url：https://www.qcc.com/api/auth/qrcode-create

参数作用：对请求信息的完整性进行校验

参数值获取方法：

（1）首先看一下请求的内容

![1](..\image\1.png)

​														图1

（2）从上图可知有一对参数5b0a480da148fa73fc4d: 4df8cabdecb9a0039b6b2e609aba57be16fff6b21bc40f299f571d36c8374d68f5d47aeca2de71f8109799e6d134f92c77e99bed9b5739cdce9560f1776f90db被加密，且直接去除会导致请求失败。

（3）对于这种情况无法直接搜索参数的Key，因此先对其他的参数进行搜索，会发现其他参数几乎搜索不到相关内容，搜索“api/auth/qrcode-create”或请求体的关键字可以定位到下图位置：

![image-20240726004709470](C:\Users\HZP\AppData\Roaming\Typora\typora-user-images\image-20240726004709470.png)

​														图2

（4）在上述.post()函数处打断点进行调试可获取参数加密的位置，我这里跳过这些步骤，直接定位到加密算法的位置：

![image-20240727010739054](C:\Users\HZP\AppData\Roaming\Typora\typora-user-images\image-20240727010739054.png)

​														图3

其中i和l分别代表Key和Value的值，具体的文件名可能会变动，不再列出，一般代码不会变动。

（5）由上图可以看出来，Key和Value是用同一个方式进行加密的，以Key为例，继续跟进函数：

![image-20240727011133213](C:\Users\HZP\AppData\Roaming\Typora\typora-user-images\image-20240727011133213.png)

​														图4

![image-20240727011232638](C:\Users\HZP\AppData\Roaming\Typora\typora-user-images\image-20240727011232638.png)

​														 图5

![image-20240727011314816](C:\Users\HZP\AppData\Roaming\Typora\typora-user-images\image-20240727011314816.png)

​														  图6

其中图5中的t的值就是加密的密钥，由图6可以知道是使用HMAC摘要算法进行加密，在这里可以不继续Trace，我们可以大胆假设使用的标准HMAC算法加密，直接在网上找一个HMAC在线加密网站进行加密，输入字符串e和密钥t，可以发现果然是使用的标准加密算法，具体是使用的HMACsha512算法。

（6）若只是想知道参数如何进行加密则后续可以直接使用标准HMAC库进行加密，若想搞懂加密代码则继续Trace。至此请求中的关键参数都大功告成！

## 3.登录请求流程

在解决了关键参数的加密算法后，我们对qcc登录流程中的所有请求进行梳理，主要包括以下几条，我只给出请求，具体请求头和请求体自行抓包。

（1）https://www.qcc.com/

​	该请求主要是获取请求返回的QCCSESSID值。

（2）https://www.qcc.com/api/auth/qrcode-create

​	该请求主要是创建登录的二维码，若使用短信验证码自行替换该请求。

（3）https://www.qcc.com/api/auth/qrcode-status

​	若是二维码登录，该请求的作用是检查是否扫码登录成功，通过一定间隔发送请求，登录后QCCSESSID的值就作为登录的有效Cookie。

（4）若登录和查询中出现验证码，换ip或其他方式解决验证码问题。
