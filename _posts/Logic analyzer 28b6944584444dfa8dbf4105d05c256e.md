# Logic analyzer

Created: 2022년 6월 19일 오후 3:34
Tags: hardware

1. GND → 항상 Low 신호를 유지하다, TX, RX에 신호가 있을 경우 high
2. RX → 항상 low 부동
3. TX → low와 high를 번갈아가며 신호 됨
4. VCC → 항상 high

1. Saleae Logic 설치

[https://www.saleae.com/downloads/](https://www.saleae.com/downloads/)

설치 후 Logic Analyzer 연결하면 자동으로 드라이버 설치 됨

![Logic%20analyzer%2028b6944584444dfa8dbf4105d05c256e/Untitled.png](Logic%20analyzer%2028b6944584444dfa8dbf4105d05c256e/Untitled.png)

2. 분석할 핀을 Logic Analyzer에 연결

![Logic%20analyzer%2028b6944584444dfa8dbf4105d05c256e/Untitled%201.png](Logic%20analyzer%2028b6944584444dfa8dbf4105d05c256e/Untitled%201.png)

3. Saleae에서 Start 버튼을 누르고 전기 신호 분석 시작 

![Logic%20analyzer%2028b6944584444dfa8dbf4105d05c256e/3-3_logic_.png](Logic%20analyzer%2028b6944584444dfa8dbf4105d05c256e/3-3_logic_.png)

Channel1은 Low와 High를 번갈하가며 전기 신호가 탐색

Channel2은 Low를 유지하다 Channel1의 신호가 발생할 시 같이 신호를 받는다. Low를 유지하다 바뀌면 GND High를 유지하다 바뀌면 TX다.

Channel3은 계속 Low를 유지한다. RX다

4. Ctrl+휠을 이용해 신호를 확대할 수 있는데,  하나의 심볼이 전송되는 속도를 측정 후 1초를 측정된 속도로 나누면 1초에 전송되는 baud 수가 근접하게 확인된다.

baud rate  :  1초에 전송되는 초당 심볼 수 ( 1,000,000uF / 8.25uF = 121,212 ) 

장비에서 사용되는 baud rate는 정형화 되어 있기 때문에 쉽게 115200 baud rate를 사용 중인 것이 확인 가능하다.

![Logic%20analyzer%2028b6944584444dfa8dbf4105d05c256e/Untitled%202.png](Logic%20analyzer%2028b6944584444dfa8dbf4105d05c256e/Untitled%202.png)

5.  Analyzers 탭에 Async Serial을 채널과 Bit Rate를 맞게 설정하면 해당 채널에서 발생 된 비트를

Ascii, bin, hex 등의 값으로 확인할 수 있다.

![Logic%20analyzer%2028b6944584444dfa8dbf4105d05c256e/3-5_logic_.png](Logic%20analyzer%2028b6944584444dfa8dbf4105d05c256e/3-5_logic_.png)