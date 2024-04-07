---
layout: post
title:  "Objection"
tags: IOS Mobile hacking
---



# [Mobile] Objection
Mobile 진단 시 쓰기 괜찮(편한)은 Python & Frida 기반 툴\
Android / iOS 모두 사용 가능

# 시작

### 요구사항

- Python 3.x
- Frida

### 설치

```bash
pip install objection
```

### Usage

```bash
objection -g <PID or Identifier> explore

PID = Attach
Identifier = Spawn
```

![2-1](/assets/images/2/Untitled.png)

### Application using directories

해당 앱이 사용하는 디렉터리 주소를 알 수 있음 (설치경로, 내부 저장경로 등)

```bash
env
```

![2-1](/assets/images/2/Untitled%201.png)

### cd/ls/pwd on device

마치 쉘에 접속한 것처럼 명령어를 사용해 디바이스를 돌아다닐 수 있음(?)

```bash
cd <path>
ls
pwd
```

![2-1](/assets/images/2/Untitled%202.png)

### File Download

기기 내부에 저장된 파일을 로컬에 다운로드 할 수 있음

db 또는 xml 파일에서 중요 정보 확인할 때 사용

```bash
file download <filename>
```

![2-1](/assets/images/2/Untitled%203.png)

![2-1](/assets/images/2/Untitled%204.png)

## for iOS

### iOS Jailbreak bypass

objection이 제공하는 탈옥 감지 우회기능이지만, 아직 우회 앱을 확인 못함 (검증 필요)

```bash
objection -g <Identifier> explore --startup-command "ios jailbreak simulate"
```

![2-1](/assets/images/2/Untitled%205.png)

### Key chain dump

해당 앱이 사용하는 키체인을 덤프할 수 있음

```bash
ios keychain dump
```

![2-1](/assets/images/2/Untitled%206.png)

### List bundles

앱의 번들과 프레임워크 리스팅

```bash
ios bundles list_bundles
ios bundles list_frameworks
```

![2-1](/assets/images/2/Untitled%207.png)

### Cookies

앱의 쿠키 값 출력

```bash
ios cookies get --json
```

![2-1](/assets/images/2/Untitled%208.png)

### Info binary

앱의 바이너리 파일 목록

```bash
ios info binary
```

![2-1](/assets/images/2/Untitled%209.png)

### nsurlcredentialstorage

앱의 자격증명 저장소 확인 (추가정보 필요)

```bash
ios nsurlcredentialstorage dump
```

![2-1](/assets/images/2/Untitled%2010.png)

### nsuserdefaults

앱이 공통으로(?) 사용되는 default property 확인

```bash
ios nsuserdefaults dump
```

![2-1](/assets/images/2/Untitled%2011.png)

### Read plist file

plist 형식의 파일을 읽어줌

```bash
ios plist cat <plist file>
```

![2-1](/assets/images/2/Untitled%2012.png)

### SSL Pinning disable

SSL pinning을 우회해줌

```bash
ios sslpinning disable
```

![2-1](/assets/images/2/Untitled%2013.png)

### UI Alert

화면에 Alert창을 띄워줌.....(?)

```bash
ios ui alert 'securityhub'
```

![2-1](/assets/images/2/Untitled%2014.png)

### Biometrics Bypass

앱의 생체인식을 우회해줌. iOS 13 기준 우회가 잘됨. 진단 시 애매..한 기능

```bash
ios ui biometrics_bypass
```

### UI dump

현재 화면의 UI View에 대한 정보를 출력

```bash
ios ui dump
```

![2-1](/assets/images/2/Untitled%2015.png)

### UI Screenshot

앱 화면을 찍어 로컬에 저장해줌

```bash
ios ui screenshot <save file name>
```

![2-1](/assets/images/2/Untitled%2016.png)

![2-1](/assets/images/2/Untitled%2017.png)