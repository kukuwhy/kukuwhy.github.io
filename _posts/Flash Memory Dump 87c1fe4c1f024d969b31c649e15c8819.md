# Flash Memory Dump

Created: 2022년 6월 19일 오후 3:34
Tags: hardware

## 1. Minipro

IC에 대한 롬 라이터기이며, MCU 및 Flash Memory를 읽을 때 사용된다.

추가로 IC 소켓이 여러 종류 제공된다.

### 1-1 IC 소켓

IC를 직접적으로 장착해서 사용한다. 여러 종류의 IC를 읽으려면, 각각의 크기에 맞는 IC 소켓이 필요하다.

![Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled.png](Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled.png)

### 1-2 롬 라이터

IC를 소켓에 장착하고 소켓을 읽는 장치다.

![Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%201.png](Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%201.png)

## 2. Flash Memory Dump

### 2.1 Driver 및 Utility 설치

[http://autoelectric.cn/EN/download.html](http://autoelectric.cn/EN/download.html)

구매한 minipro에 맞는 Application Software를 설치한다.

![Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%202.png](Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%202.png)

### 2-2 Dump

(1) 분석하고자하는 IC를 디바이스에서 Chip off하고 맞는 IC 소켓에 끼워 롬라이터에서 읽어준다.

![Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%203.png](Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%203.png)

(2) 

① 롬 라이터기에 읽은 칩셋을 자동으로 칩셋의 종류를 찾아준다.

② 장착된 칩셋을 읽어준다.

③ 읽은 내용이 표시된다.

![Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%204.png](Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%204.png)

(3) auto를 클릭하고, 8 Pins를 선택 후 Detect를 클릭하면 자동으로 현재 연결된 IC를 읽어준다.

![Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%205.png](Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%205.png)

(4) Read 버튼을 누르면 IC를 읽는다.

![Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%206.png](Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%206.png)

(5) 저장 후 7zip을 이용해 압축을 풀어주면, Flash Memory에 저장된 파일 또는 내용을 확인할 수 있다. 해당 binary는 시스템에서 사용되는 resource Data만 저장되어 있었으며, 펌웨어가 저장되어 있었다면 추가적으로 Binwalk, Fmk 등을 이용해 분석할 수 있다.

![Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%207.png](Flash%20Memory%20Dump%2087c1fe4c1f024d969b31c649e15c8819/Untitled%207.png)