## Protocols
- Defined in */etc/protocols*
- Well known ports are defined in */etc/services*

`cat /etc/protocols`
### TCP and UDP Protocols

See [tcp_ip_transport_and_applications](tcp_ip_transport_and_applications.md) and [tcp_ip_basic](tcp_ip_basic.md)

### ICMP

Send two pings to server 20
```bash
ping -c2 192.168.0.120
```

Ping the server's loopback interface:
```bash
ping 127.0.0.1
```

Send a traceroute to server 20 
```bash
traceroute 192.168.0.120
```

Or:
```bash
tracepath 192.168.0.120
```

### ICMPv6

- IPv6 version of ICMP
- enabled by default

Ping and ipv6 address:
```
ping6 
```

Trace a route to an IPv6 address:
```
tracepath6
```

```
traceroute6
```

Show IPv6 addresses:
```
ip addr | grep inet6
```

