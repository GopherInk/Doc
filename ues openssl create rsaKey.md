公私玥可使用OPENSSL工具生成。

在Windows环境下，可自行下载OPENSSL工具（ http://www.openssl.org/related/binaries.html）。

在Linux环境下，可安装OPENSSL工具包（以ubuntu为例，执行sudo apt-get install openssl）。

在Windows环境下，打开OPENSSL安装目录bin文件下面的openssl.exe。在Linux环境下，直接在终端中运行openssl。

1）生成RSA私钥：
```
openssl genrsa -out rsa_private_key.pem 2048
```
该命令会生成2018位的私钥，生成成功的界面如下：

 

如何使用openssl生成RSA公钥和私钥对
此时我们就可以在当前路径下看到rsa_private_key.pem文件了。

 

2）把RSA私钥转换成PKCS8格式
输入命令:
```
pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM –nocrypt
```
得到生成功的结果，这个结果就是PKCS8格式的私钥，如下图：

如何使用openssl生成RSA公钥和私钥对


 

3) 生成RSA公钥

输入命令
```
rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
```
得到生成成功的结果，如下图：

如何使用openssl生成RSA公钥和私钥对


此时，我们可以看到一个文件名为rsa_public_key.pem的文件，打开它，可以看到
-----BEGIN PUBLIC KEY-----开头，
-----END PUBLIC KEY-----结尾的没有换行的字符串，这个就是公钥。
