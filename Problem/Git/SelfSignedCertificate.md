# 「SSL certificate problem: self signed certificate」的对应方法

当SSL的证明书中有问题时，会出现错误信息：SSL certificate problem: self signed certificate。
这个时候在DOS下执行下面的命令：  
``git config --global http.sslVerify false``
这样在.git/config文件中就会设置为不检查证明书。

> http://code-plus.jp/git-ssl-certificate-problem/