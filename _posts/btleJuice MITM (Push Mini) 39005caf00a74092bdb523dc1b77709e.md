# btleJuice MITM (Push Mini)

Created: 2022년 6월 19일 오후 3:34
Tags: IoT, bluetooth

## 1. 제품 정보

제조사 : Push Mini

지원 통신 모듈 : Bluetooth

## 2. blteJuice MITM 공격

### 환경구축

- AttifyOS v3.0 on VM (Core용, Proxy용)
- Miaomi Push Mini (BlueTooth)
- BLE USB Dongle (CRS8510 V4.0) 2개
- VMware 설정
    
    Virtual Machine Settings → USB Controller
    
    Share Bluetooth devices with the virtual machine 체크 해제
    
    ![btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled.png](btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled.png)
    

### 사전 설치작업 (Core, Proxy)

apt-get을 사용해 BtleJuice에 필요한 종속성을 설치한다.

```bash
# sudo apt-get install bluetooth bluez libbluetooth-dev libudev-dev
```

blteJuice는 최신 버전의 Node.js로는 동작하지 않아 다운그레이드 작업이 필요

다운그레이드 후 버전에 맞는 btlejuice 재설치

```bash
# sudo npm install -g n
# sudo n 8.9.0
# sudo npm install bluetooth-hci-socket --unsafe-perm
# sudo npm remove -g btlejuice
# sudo npm install -g btlejuice --unsafe-perm
```

1. Proxy용 AttifyOS 셋팅

1-1 Proxy용 OS에 bluetooth 동글을 연결하고 bluetooth 서비스를 시작한다. 시작 후 잘 인식 되었는지 확인 후 bluetooth 인터페이스 명(hci1)을 확인한다.

```bash
# sudo service bluetooth start
# hciconfig
```

![btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%201.png](btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%201.png)

```bash
# sudo btlejuice-proxy -i hci1
```

![btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%202.png](btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%202.png)

2. Core용 AttifyOS 셋팅

2-1. Core용 AttifyOS에 USB Dongle 어댑터를 삽입한 후 인터페이스 명 확인(hci0)

```bash
# sudo hciconfig
```

![btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%203.png](btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%203.png)

2-2. btlejuice proxy로 btlejucie core를 연결

```bash
# sudo btlejuice -i [interface] -u [Proxy IP] -w

[ex]
# sudo btlejuice -i hci0 -u 192.168.200.33 -w
```

![btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%204.png](btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%204.png)

2-3. Core AttifyOS에서 웹 브라우저로 Web UI에 접속되는지 확인

```bash
http://localhost:8080
```

![btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%205.png](btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%205.png)

3. MITM Attack

3-1. 웹 브라우저에서 Select Target Device 선택

![btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%206.png](btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%206.png)

3-2. 디바이스 목록에서 공격 대상 기기 선택

![btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%207.png](btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%207.png)

3-3. 휴대폰 앱(Mini Big)에서 Bluetooth off → on 후 동작 실행

 - 아래와 같이 캡쳐 내역 확인 가능

![btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%208.png](btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%208.png)

3-4. Replay 수행

캡처된 write 패킷을 오른쪽 클릭하여 Replay write 실행 가능

![btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%209.png](btleJuice%20MITM%20(Push%20Mini)%2039005caf00a74092bdb523dc1b77709e/Untitled%209.png)