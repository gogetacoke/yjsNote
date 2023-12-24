# netstat -anp|grep 80 -w

> 过滤准确进程

```shell
[root@web1 ~]# netstat -anp|grep 80 
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      31287/nginx: master 
tcp6       0      0 :::80                   :::*                    LISTEN      31287/nginx: master 
unix  3      [ ]         STREAM     CONNECTED     85480    31445/NetworkManage  
unix  3      [ ]         STREAM     CONNECTED     19480    493/systemd-logind   

[root@web1 ~]# netstat -anp|grep 80 -w
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      31287/nginx: master 
tcp6       0      0 :::80                   :::*                    LISTEN      31287/nginx: master 
```

# lsmod

> 查看系统驱动

# ethtool -i eth0

> 查看网卡驱动

```shell
[root@lvs1 ~]#  ethtool -i eth1
driver: virtio_net
version: 1.0.0
firmware-version: 
expansion-rom-version: 
bus-info: 0000:00:03.0
supports-statistics: yes
supports-test: no
supports-eeprom-access: no
supports-register-dump: no
supports-priv-flags: no
```

# 抓包

tcp -i 网卡 host 抓去来自那个IP的包

```shell
[root@web1 ~]# tcpdump -i eth0 host 192.168.88.10
```

