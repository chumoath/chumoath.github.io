# wireguard_hand

## wireguard手动配置

```shell
# 添加网络接口
ip link add wg0 type wireguard

WG_CONFIG=""
WG_CONFIG+="[Interface]"$'\n'
WG_CONFIG+="PrivateKey = 4O6gBgrMBz+nGR0lqEiMdzwq4IzwRiom6T1RYxQbI0M="$'\n'
WG_CONFIG+="ListenPort = 51820"$'\n'
WG_CONFIG+="[Peer]"$'\n'
WG_CONFIG+="PublicKey = 5dYw8RR4dEmP172lV0povGO/hUemFRUfegZU7TvrLG0="$'\n'
WG_CONFIG+="AllowedIPs = 192.168.3.2/32"$'\n'
WG_CONFIG+="[Peer]"$'\n'
WG_CONFIG+="PublicKey = FTt7tPJG8Vi049qKDTnCSl+4ggJmdGJTPbjz2Mjx+XA="$'\n'
WG_CONFIG+="AllowedIPs = 192.168.3.3/32, 192.168.33.0/24"$'\n'

# 配置接口
wg setconf wg0 <(echo "$WG_CONFIG")

# 配置 IP
ip -4 address add 192.168.3.1/24 dev wg0

# 激活网口
ip link set mtu 1420 up dev wg0

# 配置掩码路由
ip -4 route add 192.168.33.0/24 dev wg0

# 必须有，否则就是 ping 不通 docker0 的 container 的 IP
iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o br-6a7541fe74a1 -j MASQUERADE

# 查看配置
wg showconf wg0

# 运行时配置 AllowIPs 等
wg set ...
```

## 验证 net.ipv4.ip_forward的有效性

```shell
echo 1 > /proc/sys/net/ipv4/ip_forward
echo 0 > /proc/sys/net/ipv4/ip_forward
echo 1 > /proc/sys/net/ipv4/ip_forward

wsl:
docker run --privileged=true -d --rm ubuntu_system

windows vpn:
# 肯定能ping通，因为本身就是wsl的IP，不需要wsl NAT
ping 172.17.0.1

# 必须 NAT
ping 172.17.0.2


# net/ipv4/Kconfig
IP_ADVANCED_ROUTER
bool "IP: advanced router"
# NAT 是为了能收到报文响应
# - 桥接：在同一网段的一定能收到响应
# - NAT：不在同一网段，请求报文可以发过去，但由于源地址对于目标不可达，所以网关必须进行NAT，以修改源IP
```

