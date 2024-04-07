# RF - Jam and Replay Attacks

Created: 2022년 6월 19일 오후 3:34
Tags: RF, hardware

### 1. 수행 환경

<aside>
💡 - 도어락 (RF 통신)
- HackRF One : 무선 신호 송·수신 하드웨어 장비(SDR)
- USRP : 무선 신호 송수신 하드웨어 장비
- Universal Radio Hacker

</aside>

### 2. 시나리오

<aside>
💡 1. 공격자는 피해자가 리모컨을 누르기를 기다리며, 재밍을 시도한다.
2. 피해자는 도어락을 열려고 리모컨을 통해 열림 버튼을 누른다.
3. 피해자의 리모컨 신호는 공격자의 재밍에 의해 실패하고 공격자는 1번째 리모컨 신호를 탈취한다.
4. 피해자는 리모컨이 동작하지 않음으로 다시 한번 누르게 되고 공격자는 2번째 리모컨 신호를 탈취함과 동시에 1번째 탈취한 신호를 송신해 도어락을 연다.
5. 피해자가 외출을 할 때 까지 지켜본다.
6. 공격자는 탈취한 2번째 신호를 통해 김규완의 도어락을 무력화하고 침입한다.

</aside>

 

- Jamming Data 생성
    
    1. 임의의 주파수 캡처 또는 오픈
    
    URH 툴에서 임의의 신호를 캡처하거나 불러온다.
    
    ![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled.png)
    
    2. Generator 탭에서 불러온 신호를 Generated Data에 Drag&Drop 한다.
    
    ![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%201.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%201.png)
    
    3. 데이터가 등록된 것을 확인한 후 Edit... 버튼을 누른다.
    
    ![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%202.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%202.png)
    
    4. Data (raw bits)를 1로 채운 후 Original Signal을 Drag&Drop하고 창을 닫는다.
    
    ![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%203.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%203.png)
    
    5. Pauses 탭에서 임의의 값에 오른쪽 클릭 후 Edit all을 누른다.
    
    ![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%204.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%204.png)
    
    6. 값을 0으로 수정 후 OK하면 모든 Pause 값이 0으로 바뀐다.
    
    ![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%205.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%205.png)
    
    7. Send data... 를 눌러 Jamming Data를 송신할 수 있다.
    
    ![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%206.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%206.png)
    

### 3. Jam and Replay Attacks

1. 공격자는 피해자가 리모컨을 누르기를 기다리며, 재밍을 시도한다.

![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%206.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%206.png)

2. 피해자는 도어락을 열려고 리모컨을 통해 열림 버튼을 누른다.

![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%207.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%207.png)

3. 피해자의 리모컨 신호는 공격자의 재밍에 의해 실패하고 공격자는 1번째 리모컨 신호를 탈취한다.

![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%208.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%208.png)

4. 피해자는 리모컨이 동작하지 않음으로 다시 한번 누르게 되고 공격자는 2번째 리모컨 신호를 탈취함과 동시에 1번째 탈취한 신호를 송신해 도어락을 연다.

![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%209.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%209.png)

![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%2010.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%2010.png)

5. 피해자가 외출을 할 때 까지 지켜본다. (생략)

6. 공격자는 탈취한 2번째 신호를 통해 피해자의 도어락을 무력화하고 침입한다.

![RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%2011.png](RF%20-%20Jam%20and%20Replay%20Attacks%208f070299f00c4254bc458f85d4b65563/Untitled%2011.png)