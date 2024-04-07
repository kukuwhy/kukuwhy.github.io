# BlueTooth Sniffing (OKLOK)

Created: 2022년 6월 19일 오후 3:34
Tags: bluetooth

## 환경 구성

- OKLOK 자물쇠
- Adfruit LE Sniffer (nrf51822)
- Wireshark
- Python

- Adfruit LE Sniffer 란
    
    ![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled.png)
    
    Adafruit BlueFruit LE Sniffer [[https://www.adafruit.com/product/2269](https://www.adafruit.com/product/2269)]
    
    IoT 기기 간 BLE 통신 시 패킷을 스니핑할 수 있는 도구로 Classic Bluetooth는 스니핑할 수 없으며, 오직 BLE 통신만 스니핑이 가능
    

## 환경 구축

1. Adfruit LE Sniffer 드라이버 설치

아래 링크에서 드라이버를 받아 설치 후 장치관리자에서 포트가 정상적으로 인식되는 것을 확인

```bash
https://ftdichip.com/drivers/
```

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%201.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%201.png)

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%202.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%202.png)

2. wireshark 설치

Wireshark는 3.4.3 버전을 설치했으며, Npcap이 필수로 설치되어야 함

Npcap 설치 시 아래 항목에 체크

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%203.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%203.png)

3. Python3 설치

nrf sniffer 도구는 Python을 기반으로 동작하여 Python 3버전을 설치

Add Python 3.6 to PATH를 체크하여 환경변수를 등록

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%204.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%204.png)

4. nrf sniffer 설치

4-1. 아래 링크에서 nrf sniffer V3.1.0 도구를 다운로드

```bash
https://www.nordicsemi.com/Software-and-tools/Development-Tools/nRF-Sniffer-for-Bluetooth-LE/Download
```

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%205.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%205.png)

4-2. Wireshark를 실행하여 Help > About Wireshak 선택

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%206.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%206.png)

4-3. Folder > Global Extcap path 경로를 클릭

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%207.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%207.png)

4-4. 다운로드 한 nrf sniffer v3.1.0을 압축해제 한 후 extcap폴더의 내용을 Global Extcap path 경로에 복사 및 붙여넣기

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%208.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%208.png)

4-5. nrf sniffer 요구사항 설치

extcap 폴더의 requirements.txt를 이용해 python 요구사항 설치

```bash
pip install -r requirements.txt
```

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%209.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%209.png)

4-6. nrf sniffer 동작 확인

extcap 폴더의 nrf_sniffer_ble.bat을 실행하여 동작 및 USB 인터페이스를 확인

```bash
nrf_sniffer_ble.bat --extcap-interfaces
```

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2010.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2010.png)

## 스니핑

Adfruit LE Sniffer를 PC에 재 연결 시킨 후 Wireshark를 실행

Wireshark를 실행한 후 View > Interface Toolbars > nRF Sniffer for Bluetooth LE 선택

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2011.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2011.png)

nrf sniffer toolbar에 인터페이스가 활성화 확인 후 캡처 시작

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2012.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2012.png)

- FIFO 에러
    
    아래와 같은 에러가 빈번하게 발생하며 스니핑이 될 때까지 Wireshark를 재 실행해야 함
    
    ![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2013.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2013.png)
    

## OKLOK 좌물쇠 패킷 분석

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2014.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2014.png)

스니핑을 시작한 후 Device(OKLOK 좌물쇠)를 대상 기기로 필터링

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2015.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2015.png)

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2016.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2016.png)

- 패킷 필터링 - 잘못된 패킷 제외

다음과 같이 Malformed Packet(잘못된 패킷)이 종종 잡힌다면 필터링을 통해 제외

또는 **!(_ws.malformed)** 라고 필터링 구문을 직접 작성

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2017.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2017.png)

필터링 적용 시 아래와 같이 정상적으로 통신된 패킷 확인 가능

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2018.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2018.png)

- 패킷 필터링 - 메인 서비스 패킷만 포함

OKLOK 좌물쇠(slave)와 휴대폰 앱(Master)가 연결되어 통신하게 되면 Slave & Master로 표기되면서 L2CAP 프로토콜을 통해 데이터 교환 시작 ⇒ 유의미한 데이터 값

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2019.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2019.png)

정상적인 패킷과 OKLOK 좌물쇠와 휴대폰 앱 간의 L2CAP 프로토콜을 모두 필터링 하기 위해  

**(!(_ws.malformed)) && (btl2cap.cid == 0x0004)** 로 필터링 문구 작성

 - L2CAP 프로토콜 CID 값 확인 및 필터링 구문 작성

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2020.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2020.png)

필터링 적용 화면

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2021.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2021.png)

BLE 통신 시 암호화 여부 확인

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2022.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2022.png)

자물쇠 열림 시 Service, Characteristic UUID와  Value 값 확인

![BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2023.png](BlueTooth%20Sniffing%20(OKLOK)%20908b396db9134ec4951ea69604f48c22/Untitled%2023.png)