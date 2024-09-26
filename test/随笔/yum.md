#### 换源

1. 下载yum配置文件

   ```
   wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
   ```
2. 清理yum缓存，并生成新的缓存

   ```
   yum clean all
   yum makecache
   ```
3. 更新yum源检查是否生效

   ```
   yum update
   ```
