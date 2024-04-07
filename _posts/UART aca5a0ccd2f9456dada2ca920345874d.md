# UART

Created: 2022년 6월 19일 오후 3:34
Tags: debug, hardware

[Logic analyzer](https://www.notion.so/Logic-analyzer-a42bafaadaad4f94ade18abff2e84c00?pvs=21)

[Bootloader](https://www.notion.so/Bootloader-3040393037f7496e988de2f98f30a589?pvs=21)

## UART 란?

UART는 우리에게 시리얼 통신(Serial communication)으로 더 잘 알려져 있는데, UART의 통신 방법은 마치 사람이 대화 하는 것과 같은 원리를 가지고 있다. 

UART를 하기 위해서는 Rx(데이터 수신), Tx(데이터 송신), GND가 서로 연결이 되어야 하며, 비동기 통신이기 때문에 둘 간의 baud rate를 일치 시켜주어야 한다.

![UART%20aca5a0ccd2f9456dada2ca920345874d/Untitled.png](UART%20aca5a0ccd2f9456dada2ca920345874d/Untitled.png)

## UART 환경 구성

- 멀티테스터기
- USB to TTL
    
    병렬 UART 포트를 직렬 USB로 연결용 도구
    
    ![UART%20aca5a0ccd2f9456dada2ca920345874d/Untitled%201.png](UART%20aca5a0ccd2f9456dada2ca920345874d/Untitled%201.png)
    
    USB to TTL  색상 별 기능
    
    - 빨강 : VCC
    - 검정 : GND
    - 흰색 : RXD
    - 녹색 : TXD
- 테스트 용 IPCAM(GW-1122) → 테스트용 IoT 장비
- 점퍼 케이블
    
    ![UART%20aca5a0ccd2f9456dada2ca920345874d/Untitled%202.png](UART%20aca5a0ccd2f9456dada2ca920345874d/Untitled%202.png)
    

## UART 포트 구성

장치를 기준으로 본다.

- TX(Transmit) : 장치에서 다른 쪽으로 데이터 전송
- RX(Receive) : 다른 쪽에서 장치 쪽으로 데이터 수신
- GND  : 접지 기준 핀
- VCC : 일반적으로 3.3V 또는 5V의 전압 공급 핀

## UART 핀 구별 하는 방법

- 멀티 테스터기 이용

검은선은 중간에, 빨간선은 제일 오른쪽에 연결 후 테스트한다.

VCC, TX, RX 테스트 시에는 200mA 멀티테스터기의 옵션은 직류전류 200m으로 설정한다.

GND 테스트 시에는 부저로(소리) 설정후 테스트 한다.

![UART%20aca5a0ccd2f9456dada2ca920345874d/1-1_.jpg](UART%20aca5a0ccd2f9456dada2ca920345874d/1-1_.jpg)

① VCC : 3.5V 및 5.5V의 전류가 흐른다

<aside>
💡 IPCAM에는 별도로 VCC핀이 존재하지 않아 테스트하지 않음

</aside>

② TX : 데이터를 계속 전송하기 때문에 전류가 계속 흐르고 있다

![UART%20aca5a0ccd2f9456dada2ca920345874d/1-1_(TX).jpg](UART%20aca5a0ccd2f9456dada2ca920345874d/1-1_(TX).jpg)

③ RX : 수신 받는 데이터가 없기 때문에 전류가 흐르지 않는다.

![UART%20aca5a0ccd2f9456dada2ca920345874d/1-1_(RX).jpg](UART%20aca5a0ccd2f9456dada2ca920345874d/1-1_(RX).jpg)

④ GND : 부저를 이용해 전원 어댑터 판에 연결하고 그라운드라고 생각 되는 것에 연결 시 소리가 난다.

![UART%20aca5a0ccd2f9456dada2ca920345874d/1-1_(GND).jpg](UART%20aca5a0ccd2f9456dada2ca920345874d/1-1_(GND).jpg)

## UART 포트 찾기

<aside>
💡 1. 존재하는 모든 IC칩 datasheet 분석
2. 3핀 또는 4핀이 나란히 존재하는 포트 분석

</aside>

1. IPCAM에서 사용되는 IC칩은 확인

![UART%20aca5a0ccd2f9456dada2ca920345874d/Untitled%203.png](UART%20aca5a0ccd2f9456dada2ca920345874d/Untitled%203.png)

IC 칩 Datasheet 검색결과가 나오지 않음

2.  확인된 3개의 핀이 MCU와 연결되는 것을 확인 위에서 서술한 것과 같이  VCC를 제외한 RX, TX, GND를 확인할 수 있음

![UART%20aca5a0ccd2f9456dada2ca920345874d/1-1_IPCAM_IC_Chip().jpg](UART%20aca5a0ccd2f9456dada2ca920345874d/1-1_IPCAM_IC_Chip().jpg)

3. USB to TTL에 M/M 점퍼케이블을 이용해 연결하고 각각 맞는 핀을 연결한다.

![UART%20aca5a0ccd2f9456dada2ca920345874d/2-1_UART__().png.jpg](UART%20aca5a0ccd2f9456dada2ca920345874d/2-1_UART__().png.jpg)

4. 장치관리자를 통해 Serial 번호를 확인하고 터미널 소프트웨어 Putty를 이용해 Serial 접속을 open

Speed의 경우 장치마다 다른 속도를 가지고 통신을 하는데 보통 자주 사용하는 Speed를 모두 입력하고 시도한다. 해당 장비에서는 115200의 Boud Rate를 사용하고 있음

자주 사용하는 Boud Rate는 9600, 14400, 19200, 38400, 57600, 115200, 128000

![UART%20aca5a0ccd2f9456dada2ca920345874d/2-2_UART__(-putty).png](UART%20aca5a0ccd2f9456dada2ca920345874d/2-2_UART__(-putty).png)

5.  리눅스 기반 쉘 획득

![UART%20aca5a0ccd2f9456dada2ca920345874d/2-3_UART__(_).png](UART%20aca5a0ccd2f9456dada2ca920345874d/2-3_UART__(_).png)

6.  쉘 획득 시 각종 바이너리 획득 가능 

dd 명령어를 통해 파일시스템 바이너리 획득 

획득한 바이너리는 설치된 tftp를 이용해 로컬pc로 전송

```c
dd if=/dev/mtdblock4 of=file.bin bs=512
tftp -l file.bin -p 192.168.0.3
```

![UART%20aca5a0ccd2f9456dada2ca920345874d/2-4_dd___ftp.png](UART%20aca5a0ccd2f9456dada2ca920345874d/2-4_dd___ftp.png)

7. netstat 이용해  TCP/UDP 서비스 별로 매핑된 프로세스 확인

```c
netstat -anp
```

![UART%20aca5a0ccd2f9456dada2ca920345874d/2-5_netstat.png](UART%20aca5a0ccd2f9456dada2ca920345874d/2-5_netstat.png)

8. 쉘을 획득하지 못했을 시 nmap 통해서 포트 스캐닝

```c
/* Pn 옵션은 host가 살아있다고 가정하고 스캐닝 */
nmap -Pn 192.168.0.8

```

8-1 일반적인 옵션으로 nmap 스캐닝 시 host가 죽은 것 같다고 판단

![UART%20aca5a0ccd2f9456dada2ca920345874d/Untitled%204.png](UART%20aca5a0ccd2f9456dada2ca920345874d/Untitled%204.png)

8-2 -Pn 옵션 스캐닝 시 포트 확인 가능

![UART%20aca5a0ccd2f9456dada2ca920345874d/2-6_nmap.png](UART%20aca5a0ccd2f9456dada2ca920345874d/2-6_nmap.png)

9. 확인된 telnet으로 접속

![UART%20aca5a0ccd2f9456dada2ca920345874d/2-7_telnet.png](UART%20aca5a0ccd2f9456dada2ca920345874d/2-7_telnet.png)

9-1 패스워드가 설정되지 않아 root로 쉽게 로그인 가능

![UART%20aca5a0ccd2f9456dada2ca920345874d/2-8_telnet2.png](UART%20aca5a0ccd2f9456dada2ca920345874d/2-8_telnet2.png)

<aside>
💡 참조
uart with python([https://github.com/pyserial/pyserial](https://github.com/pyserial/pyserial))
- UART 연결을 Python으로 하고 읽은 내용을 파일로 쓰는 Python code

</aside>