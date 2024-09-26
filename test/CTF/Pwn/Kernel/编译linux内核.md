

1. 内核源码下载并解压
2. 相关依赖的安装，部分如下
   ```
   sudo apt install ncurses-dev
   sudo apt-get install libssl-dev
   sudo apt-get install libelf-dev
   sudo apt install dwarves
   sudo apt install zstd
   ```
3. 编译前的配置
   1. `make defconfig`，进行 `make menuconfig` 之前必须要做这一步，否则编译得到的bzImage无法启动
   2. 编译内核缺少证书：[编译内核报错 No rule to make target ‘debian/canonical-certs.pem‘ 或 ‘canonical-revoked-certs.pem‘ 的解决方法_make[1]: *** no rule to make target &#39;debian/canoni-CSDN博客](https://blog.csdn.net/m0_47696151/article/details/121574718)，需要关闭证书需求
4. 编译内核驱动
   1. 编译时不能使用sudo make，必须以用户身份编译
   2. 创建的源文件以ko_test.c明名
