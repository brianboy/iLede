### 配置路径
```
./.config
./package/lean/default-settings/files/zzz-default-settings
./include/target.mk
./target/linux/x86/Makefile
```

### 导入镜像
```
rm -rf openwrt-x86-64-generic-rootfs.tar.gz && docker rmi openwrt-x86-generic-rootfs
unzip openwrt-x86-64-generic-rootfs.tar.gz.zip && rm -rf openwrt-x86-64-generic-rootfs.tar.gz.zip
docker import openwrt-x86-64-generic-rootfs.tar.gz openwrt-x86-generic-rootfs
```

### 调试网络
```
docker exec -it openwrt /bin/sh
vi /etc/config/network
/etc/init.d/network restart

# =====================================

config interface 'loopback'
        option device 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config globals 'globals'
        option packet_steering '1'

config interface 'lan'
        option proto 'static'
        option ipaddr '6.6.8.2'
        option netmask '255.255.255.0'
        option gateway '6.6.8.1'
        option ip6assign '60'
        option dns '6.6.8.2'
        option delegate '0'
        option device 'eth0'
```

### 容器文件 docker-compose.yml
```
version: "3.7"
services:
  openwrt:
    image: openwrt-x86-generic-rootfs:latest
    container_name: openwrt
    privileged: true
    restart: always
    networks:
      default:
        ipv4_address: 6.6.8.2
    logging:
      driver: json-file
      options:
        max-size: 4M
    volumes:
      - /etc/localtime:/etc/localtime
      - ./resolv.conf:/etc/resolv.conf
    command: /sbin/init
networks:
  default:
    name: docker
    ipam:
      config:
        - subnet: "6.6.8.0/24"
```

### 容器文件 resolv.conf
```
nameserver 127.0.0.1
options ndots:0
```

### 备份相关
```
rm -rf /passwall.config
uci export passwall > passwall.config

/etc/init.d/passwall stop
uci import passwall < passwall.config
uci commit passwall
lua /usr/share/passwall/subscribe.lua start
/etc/init.d/passwall start
lua /usr/share/passwall/rule_update.lua
```

### 等待记录
```
https://github.com/kenzok8/small-package
https://github.com/NueXini/NueXini_Packages
```
