
```
iptables -t nat -A POSTROUTING --out-interface tun0 -j MASQUERADE  
```
```
iptables -t nat -D POSTROUTING --out-interface tun0 -j MASQUERADE  
```
```
iptables -A FORWARD --in-interface eth0 -j ACCEPT
```
```
net.ipv4.ip_forward=1
```
```
echo "1" > /proc/sys/net/ipv4/ip_forward

```
```iptables -Lvn
```

