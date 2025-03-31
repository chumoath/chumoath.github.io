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

