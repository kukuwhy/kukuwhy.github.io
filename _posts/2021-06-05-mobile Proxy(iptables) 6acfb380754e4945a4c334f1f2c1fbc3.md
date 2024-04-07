---
layout: post
title:  "mobile Proxy(iptables)"
tags: mobile android
---

### 현재 설정 내역 확인

```bash
iptables -t nat -L
```

### iptables 룰 설정

```bash
iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to-destination 192.168.200.103:8080
iptables -t nat -A OUTPUT -p tcp -j DNAT --to-destination 192.168.200.103:8080
```

```docker
iptables -t nat -A OUTPUT -p tcp -j DNAT --to-destination 192.168.200.157:8080
```

### iptables 룰 삭제

```bash
target = OUTPUT 삭제
iptables -t nat -D OUTPUT 1

전체 삭제
iptables -t nat -F
```

> iptables 설정 시 burp 옵션
> 

invisible proxy support allows non-proxy-aware clients to connect directly to the listener