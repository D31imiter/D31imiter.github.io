1. 下载解压
2. make menuconfig
3. make install
4. 进入生成的_install目录

   * 打包文件系统：
     `find . | cpio -o --format=newc > ../rootfs.img`
   * 解包
     `cpio -idmv < rootfs.img`
