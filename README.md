wireguard

#
# 提前操作
#

## 查看内核版本
``` bash
uname -a
```

## 开启内核转发
``` bash
vim /etc/sysctl.conf
```
打开配置项
> net.ipv4.ip_forward=1
``` bash
sysctl -p
```

## 安装wireguard
``` bash
sudo apt-get update
sudo apt-get install wireguard
```

## 创建公钥私钥
``` bash
> wg genkey | tee privatekey | wg pubkey > publickey
```

#
# 手动配置
#

## 新建一个网络设备
``` bash
> ip link add wg0 type wireguard
```

## 设置本机ip
``` bash
> ip addr add 10.0.0.51/24 dev wg0
```

## 设置私钥
``` bash
> wg set wg0 private-key /etc/wireguard/privatekey
```

## 添加peer
``` bash
> wg set wg0 peer ${peer public key} allowed-ips 10.0.0.0/24 endpoint ${peer ip}:${peer port} persistent-keepalive 25
```

## up / down
``` bash
> ip link set wg0 up
> ip link set wg0 down
```

#
# 使用wg-quick进行配置
#
``` bash
vim /etc/wireguard/wg0.conf
```
``` txt
[Interface]
  # 本地ip
  Address = 10.0.0.50/24

  # 本地私钥
  PrivateKey = ${local private key}

  # 服务启动前的指令
  PostUp   = wg set wg0 private-key /etc/wireguard/privatekey;iptables -A FORWARD -i %i -o %i -j ACCEPT;
  
  # 服务终止后的指令
  PostDown = iptables -D FORWARD -i %i -o %i -j ACCEPT;

  # 监听端口
  ListenPort = ${local port}

  # DNS
  DNS = 8.8.8.8

  # MTU
  MTU = 1420

  # 如果有客户端链接,保存客户端相关信息到本文件
  SaveConfig = true

[Peer]
  PublicKey = ${peer public key}
  AllowedIPs = 10.0.0.0/24
  Endpoint = ${peer ip}:${peer port}
  PersistentKeepalive = 25
```

## wg-quick 服务启停相关
``` bash
sudo systemctl enable wg-quick@wg0.service
sudo systemctl start wg-quick@wg0.service
sudo systemctl stop wg-quick@wg0.service
sudo systemctl restart wg-quick@wg0.service
sudo systemctl status wg-quick@wg0.service
```
其中,wg0与上述配置文件的文件名相关
