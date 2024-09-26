记录：

1. 信息收集得到yash用户ssh登录密码 `ya5hay`
2. 获取立足点后发现以root权限运行了/yash/perm.sh脚本，而yash用户有权限对perm脚本修改，将反弹shell写入perm.sh

```
root@MiWiFi-RB03-srv:~# cat root_flag.txt
cat root_flag.txt
                _                                                          _           _                                             _                 _        
               (_)                                                        (_)         (_)                                         _ (_)             _ (_)       
       _  _  _ (_)  _  _  _  _      _  _  _  _  _  _  _    _  _  _  _   _ (_) _  _  _  _       _  _  _     _  _  _  _            (_)(_)            (_)(_)       
     _(_)(_)(_)(_) (_)(_)(_)(_)_  _(_)(_)(_)(_)(_)(_)(_)_ (_)(_)(_)(_)_(_)(_)(_)(_)(_)(_)   _ (_)(_)(_) _ (_)(_)(_)(_)_             (_)               (_)       
    (_)        (_)(_) _  _  _ (_)(_)       (_) _  _  _ (_)(_)        (_)  (_)         (_)  (_)         (_)(_)        (_)            (_)               (_)       
    (_)        (_)(_)(_)(_)(_)(_)(_)       (_)(_)(_)(_)(_)(_)        (_)  (_)     _   (_)  (_)         (_)(_)        (_)            (_)     _  _      (_)       
    (_)_  _  _ (_)(_)_  _  _  _  (_)_  _  _(_)_  _  _  _  (_) _  _  _(_)  (_)_  _(_)_ (_) _(_) _  _  _ (_)(_)        (_)          _ (_) _  (_)(_)   _ (_) _     
      (_)(_)(_)(_)  (_)(_)(_)(_)   (_)(_)(_) (_)(_)(_)(_) (_)(_)(_)(_)      (_)(_) (_)(_)(_)  (_)(_)(_)   (_)        (_)         (_)(_)(_) (_)(_)  (_)(_)(_)    
                                                          (_)                                                                                                   
                                                          (_)                                                                                                  




----------------------------------
Visit our website : https://www.haclabs.org
Submit walkthrough at : yash@haclabs.org
root@MiWiFi-RB03-srv:~# ip a
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:eb:ef:74 brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.199/24 brd 192.168.31.255 scope global dynamic enp0s3
       valid_lft 36205sec preferred_lft 36205sec
    inet6 fe80::a00:27ff:feeb:ef74/64 scope link 
       valid_lft forever preferred_lft forever
root@MiWiFi-RB03-srv:~# whoami
whoami
root
```
