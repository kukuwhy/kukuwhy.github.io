---
layout: post
title:  "RF 통신 hacking"
tags: RF hardware
---

# RF 통신 hacking
## URH(Universal Radio Hacker)를 이용한 RF 통신 조작

### 1. 수행 환경

<aside>
💡 - wall pad: RF 통신 장비
- HackRF One : 무선 신호 송·수신 하드웨어 장비(SDR)
- Universal Radio Hacker : 무선환경 분석을 위해 사용하는 소프트웨어 도구
- 분석 PC(Windows 10 Pro)

</aside>

### 2. Replay Attack 수행

1) 주파수 분석

- URH를 실행한 후 상단의 [File] - [Spectrum Analyzer...]를 클릭한다.
    
    ![/assets/images/3/__1.png](/assets/images/3/__1.png)
    

- Device Settings에서 사전 설정을 진행한 후 Start 버튼을 클릭하여 주파수 분석을 시작한다.
- RF 주파수(Frequency)
    - 315, 433, 868, 915MHz
    - 13.56Mhz : RFID ...
    - 2.4Ghz : WiFi, Bluetooth, Zigbee ...
    
    ISM band
    ISM 대역(ISM band)은 산업·과학·의료(Industry-Science-Medical) 등에 쓰이는 주파수 대역
    
    ![/assets/images/3/Untitled.png](/assets/images/3/Untitled.png)
    

<aside>
💡 - Device : 주파수 확인 시 사용할 장비를 선택하는 항목으로, HackRF를 선택한다.
- Frequency(Hz) : 확인할 주파수를 설정하는 항목으로, 433.920M로 설정한다.
- Sample rate(Sps) **:** 1초에 샘플링을 몇 번 수행할지 설정하는 항목으로, 크게 설정하면 RF 신호 캡처 시 하드디스크 용량을 많이 차지하게 된다.
- Bandwidth(Hz) : 위에서 설정한 주파수를 중심으로 앞뒤로 캡처할 범위를 정한다.
- Gain : 
- IF Gain : 기본값(16)으로 설정한다.
- Baseband gain : 기본값(14)으로 설정한다.

</aside>

![/assets/images/3/__2.png](/assets/images/3/__2.png)

- 주파수 분석 시 다음과 같이 주파수 대역 확인이 가능하다.
    
    ![/assets/images/3/__3.png](/assets/images/3/__3.png)
    

- 리모콘을 이용하여 RF신호를 월패드로 전송하면 그래프가 변동되는 것을 확인할 수 있으며, 이를 확인하여 월패드가 사용하는 RF 신호의 주파수 대역을 알 수 있다.
    
    ![/assets/images/3/__4.png](/assets/images/3/__4.png)
    

2) RF 신호 저장

- URH 메인화면의 상단의 [File] - [Recode signal...]을 클릭한다.
    
    ![/assets/images/3/RF___1.png](/assets/images/3/RF___1.png)
    

- RF 신호 저장을 위해 Devices Settings에서 사전 설정을 진행한다. 이 때 설정은 주파수 분석과 동일하게 설정하며, 설정을 마친 후 Start 버튼을 클릭하여 신호를 캡처한다.
    
    ![/assets/images/3/RF___2.png](/assets/images/3/RF___2.png)
    

- 리모콘을 이용하여 RF 신호를 월패드로 전송 시 다음과 같이 송·수신되는 RF 신호 확인이 가능하다. RF 신호 캡처가 끝난 후에는 Stop버튼을 클릭하여 캡처를 중지시킬 수 있으며 Save 버튼을 클릭하여 캡처한 RF 신호의 저장이 가능하다.
    
    ![/assets/images/3/RF___3.png](/assets/images/3/RF___3.png)
    

3) Replay Attack 수행

- 캡처한 신호를 저장하면 URH 메인화면에 자동으로 캡처한 RF 신호 확인이 가능하며, 왼쪽의 플레이 버튼을 클릭한다.
    
    ![/assets/images/3/Reply_Attack_1.png](/assets/images/3/Reply_Attack_1.png)
    
- 플레이 버튼 클릭 시 나타나는 Send Signal 창에서 Start 버튼을 클릭하여 Replay Attack 공격을 시도한다.
    
    ![/assets/images/3/Reply_Attack_2.png](/assets/images/3/Reply_Attack_2.png)
    

- 캡처한 RF 신호에 맞춰 월패드의 경비모드가 활성화 되며, 이를 통해 Replay Attack 공격이 정상적으로 수행됨을 확인할 수 있다.
    
    ![/assets/images/3/Replay_Attack_3.png](/assets/images/3/Replay_Attack_3.png)
    

### 3. Fuzzing을 이용한 RF 통신 신호 조작

- URH 상단의 [File] - [Open...]에서 경비모드 활성화 신호, 비활성화 신호 두 가지를 불러온다.
    
    ![/assets/images/3/1.png](/assets/images/3/1.png)
    

- 신호를 하나씩 지워가며 보내면서 경비모드 활성화 관련 신호가 어느 구간인지 확인한다.
    
    ![/assets/images/3/2.png](/assets/images/3/2.png)
    

- 마우스 휠을 이용해 신호를 확대하여 해당 신호가 ASK 방식을 이용하고 있음을 확인한다.
    
    ![/assets/images/3/3.png](/assets/images/3/3.png)
    
- Modulation을 ASK로 선택한 후 설정창에서 Pause Threshold의 값을 0으로 설정한다.
    
    ![/assets/images/3/4.png](/assets/images/3/4.png)
    

- Show Signal as항목에서 Hex를 선택하고 Autodetect parameters를 클릭한 후 뒤에 불필요한 공백 부분을 제거한다.
    
    ![/assets/images/3/5.png](/assets/images/3/5.png)
    

- [Analysis] - [Protocols] 탭에서 Mark diffs in protocol을 체크하여 경비모드 활성화/비활성화 신호를 비교한다.
    
    ![/assets/images/3/6.png](/assets/images/3/6.png)
    

- 두 신호에서 서로 다른 구간을 확인한 후 마우스 우클릭 - [create label...]을 클릭하여 해당 구간에 라벨을 생성한다.
    
    ![/assets/images/3/7.png](/assets/images/3/7.png)
    

- [Generator] - [Protocols] 탭에서 경비모드 비활성화 신호를 선택하여 라벨이 생성된 구간을 확인한다.
    
    ![/assets/images/3/8.png](/assets/images/3/8.png)
    

- 해당 구간 우클릭 후 [Edit label...]을 클릭한다.
    
    ![/assets/images/3/9.png](/assets/images/3/9.png)
    

- Fuzzing 창에서 Add to Fuzzed Values를 클릭한다.
    
    ![/assets/images/3/10.png](/assets/images/3/10.png)
    

- 창을 종료한 후 [Generator] - [Fuzzing] 탭에서 Fuzz 버튼을 클릭하여 신호들을 생성한다.
    
    ![/assets/images/3/11.png](/assets/images/3/11.png)
    

- Send data 창에서 생성된 신호들을 재생하면 특정 구간에서 경비모드가 활성화된다
    
    ![/assets/images/3/12.png](/assets/images/3/12.png)
    
    ![/assets/images/3/14.png](/assets/images/3/14.png)
    

## GNURadio를 이용한 Replay Attack 공격

### 1. 수행환경

<aside>
💡 - wall pad: RF 통신 장비
- HackRF One : 무선 신호 송·수신 하드웨어 장비(SDR)
- GNURadio : 무선환경 분석을 위해 사용하는 소프트웨어 도구
- 분석 PC(Kali Linux 2020.4)

</aside>

### 2. Replay Attack 수행

1) 주파수 분석

- 툴 설치

```bash
  gqrx 설치 및 실행
# apt-get install gqrx-sdr
# gqrx -r

  hackrf 설치 및 정보 확인
# apt-get install hackrf
# hackrf_info

  GNU Radio 설치 및 실행
# apt-get install gnuradio
# gnuradio-companion

```

- 주파수 분석을 위한 장비가(HackRF) 잘 연결되었는지 확인한다.

  

![/assets/images/3/Untitled%201.png](/assets/images/3/Untitled%201.png)

- 터미널 창에서 gqrx -r 명령어를 입력하여 주파수 분석 도구인 gqrx를 실행한다.

<aside>
💡 - root 권한으로 실행 시 실행 불가능
- r 옵션: gqrx 최초 실행 시에만 Configure I/O device를 선택하도록 하는 옵션

</aside>

![/assets/images/3/__0.png](/assets/images/3/__0.png)

- gqrx 실행 시 Configure I/O device에서 장비를 선택한다.
    
    ![/assets/images/3/__1%201.png](/assets/images/3/__1%201.png)
    

- 우측의 Input controls에서 RF, IF, BB Gain의 값을 설정한다.
    
    ![/assets/images/3/__2%201.png](/assets/images/3/__2%201.png)
    

- 우측의 Receiver options의 Frequency 항목에서 확인할 주파수를 설정한다.
    
    ![/assets/images/3/__3%201.png](/assets/images/3/__3%201.png)
    

- 우측의 FFT Settings의 Peak에서 Detect와 Hold를 체크한다.

<aside>
💡 - Detect: 높이가 올라가는 값에 대해 체크를 해준다.
- Hold: 측정 값 중 최대값을 기록한다.

</aside>

![/assets/images/3/__4%201.png](/assets/images/3/__4%201.png)

- 리모콘을 이용하여 RF신호를 월패드로 전송하면 그래프가 변동되는 것을 확인할 수 있으며, 이를 확인하여 월패드가 사용하는 RF 신호의 주파수 대역을 알 수 있다.
    
    ![/assets/images/3/__5.png](/assets/images/3/__5.png)
    

2) 수신모듈 작성

- 터미널 창에서 gnuradio-companion 명령어를 입력하여 gnuradio를 실행한다.
    
    ![/assets/images/3/__0%201.png](/assets/images/3/__0%201.png)
    

- Options에서 Id값을 입력한다.
    
    ![/assets/images/3/__1%202.png](/assets/images/3/__1%202.png)
    
- 우측의 기능 중 수신할 주파수 대역 설정을 위한 osmocom Source 기능을 드래그 또는 더블클릭하여 가져온다.
    
    ![/assets/images/3/__2%202.png](/assets/images/3/__2%202.png)
    

- osmocom Source를 더블클릭한 후, gqrx에서 주파수를 분석할 때 사용한 설정값을 입력한다.
    
    ![/assets/images/3/__3%202.png](/assets/images/3/__3%202.png)
    

- 신호 송·수신 시 주파수를 확인하기 위해 QT GUI Freqeuncy Sink를 추가한다.
    
    ![/assets/images/3/__4%202.png](/assets/images/3/__4%202.png)
    

- 송·수신한 신호를 파일에 저장하기 위해 File Sink를 추가한다.
    
    ![/assets/images/3/__5%201.png](/assets/images/3/__5%201.png)
    

- File Sink를 더블클릭한 후 파일명을 작성한 후 저장한다.
    
    ![/assets/images/3/__6.png](/assets/images/3/__6.png)
    

- 상단의 플레이 버튼을 클릭한 후 리모콘을 이용하여 RF 신호를 월패드로 전송 시 다음과 같이 송·수신되는 RF 신호 확인이 가능하다.
    
    ![/assets/images/3/__7.png](/assets/images/3/__7.png)
    

3) 송신모듈 작성

- 새 파일을 연 후, Options에서 Id 값을 입력한다.
    
    ![/assets/images/3/__1%203.png](/assets/images/3/__1%203.png)
    

- File Source 더블클릭 후, 수신모듈의 File Sink에 저장한 파일을 가져온다.
    
    ![/assets/images/3/__2%203.png](/assets/images/3/__2%203.png)
    

- osmocom Sink를 더블클릭한 후 주파수 및 RF,IF,BB Gain을 입력한다.

<aside>
💡 - 주파수 분석 시 사용한 설정값을 입력했는데 실패한 경우, IF Gain의 값을 높여서 민감도를 낮춘 후 시도한다.

</aside>

![/assets/images/3/__3%203.png](/assets/images/3/__3%203.png)

![/assets/images/3/__4%203.png](/assets/images/3/__4%203.png)

- 신호 송·수신 시 주파수를 확인하기 위해 QT GUI Freqeuncy Sink를 추가한다.
    
    ![/assets/images/3/__5%202.png](/assets/images/3/__5%202.png)
    

4) Replay Attack 수행

- 송신모듈을 모두 작성한 후 상단의 플레이 버튼을 클릭하여 Replay Attack 공격을 시도한다.
    
    ![/assets/images/3/replay_attack__1.png](/assets/images/3/replay_attack__1.png)
    

- 수신모듈에서 저장한 RF 신호에 맞춰 월패드의 경비모드가 활성화 되며, 이를 통해 Replay Attack 공격이 정상적으로 수행됨을 확인할 수 있다.
    
    ![/assets/images/3/__6%201.png](/assets/images/3/__6%201.png)
    

![/assets/images/3/Untitled%202.png](/assets/images/3/Untitled%202.png)