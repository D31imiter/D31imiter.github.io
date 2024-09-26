### 直接部署

```
socat tcp-l:6666,fork exec:[path],reuseaddr
```

### ctf_xinetd

基于docker容器部署pwn

#### 配置文件

##### ctf.xinetd

```
service ctf
{
    disable = no
    socket_type = stream
    protocol    = tcp
    wait        = no
    user        = root
    type        = UNLISTED
    port        = 9999
    bind        = 0.0.0.0
    server      = /usr/sbin/chroot
    # replace helloworld to your program
    server_args = --userspec=1000:1000 /home/ctf ./0ctf_2017_babyheap
    banner_fail = /etc/banner_fail
    # safety options
    per_source  = 10 # the maximum instances of this service per source IP address
    rlimit_cpu  = 20 # the maximum number of CPU seconds that the service may use
    #rlimit_as  = 1024M # the Address Space resource limit for the service
    #access_times = 2:00-9:00 12:00-24:00
}
```

容器端口：`port        = 9999`
题目：`server_args = --userspec=1000:1000 /home/ctf ./0ctf_2017_babyheap`，此处的 `./0ctf_2017_babyheap` 为bin目录下的题目文件

##### Dockerfile

```
FROM ubuntu:16.04

RUN sed -i "s/http:\/\/archive.ubuntu.com/http:\/\/mirrors.tuna.tsinghua.edu.cn/g" /etc/apt/sources.list && \
    apt-get update && apt-get -y dist-upgrade && \
    apt-get install -y lib32z1 xinetd

RUN useradd -m ctf

WORKDIR /home/ctf

RUN cp -R /lib* /home/ctf && \
    cp -R /usr/lib* /home/ctf

RUN mkdir /home/ctf/dev && \
    mknod /home/ctf/dev/null c 1 3 && \
    mknod /home/ctf/dev/zero c 1 5 && \
    mknod /home/ctf/dev/random c 1 8 && \
    mknod /home/ctf/dev/urandom c 1 9 && \
    chmod 666 /home/ctf/dev/*

RUN mkdir /home/ctf/bin && \
    cp /bin/sh /home/ctf/bin && \
    cp /bin/ls /home/ctf/bin && \
    cp /bin/cat /home/ctf/bin

COPY ./ctf.xinetd /etc/xinetd.d/ctf
COPY ./start.sh /start.sh
RUN echo "Blocked by ctf_xinetd" > /etc/banner_fail

RUN chmod +x /start.sh

COPY ./bin/ /home/ctf/
RUN chown -R root:ctf /home/ctf && \
    chmod -R 750 /home/ctf && \
    chmod 740 /home/ctf/flag

CMD ["/start.sh"]

EXPOSE 9999
```

`EXPOSE 9999`是容器的端口

#### 整体流程

1. 进入bin目录，将默认的helloword文件更换为目标题目文件，flag即文本文件
2. 进入 `ctf_xinetd`目录下，修改 `Dockerfile `和 `ctf.xinetd`两个配置文件，修改容器端口以及目标题目文件
3. 创建镜像 `docker build -t [REPOSITORY:TAG] [dockerfile path]`，`REPOSITORY`名称，`TAG`标签
   `docker images`查看已有镜像
4. 创建容器 `docker run -d -p "[ip]:[host port]:[Container port]" -h [Hostname] --name [Container Name] [image REPOSITORY]`
   `docker ps`查看已有容器

发生冲突时，`docker rm [ID]`删除镜像，`docker stop [ID]`停止容器


`nmap -sT -sV -O -A -p22,80,111,2049,3128,38519,40003,43033,45293 192.168.1.118`
