# Passionfruit tool

Created: 2022년 6월 19일 오후 3:34
Tags: frida, mobile

Nodejs로 제작된 frida 기반의 iOS app analyzer 도구

## 설치

```bash
npm install -g passionfruit
```

## 사용법

```bash
passionfruit
```

실행시킨 후 listening 중인 포트에 브라우저를 이용해 접속해 앱 선택

![Passionfruit%20tool%200b355e4d80f24c6d81388f3461e5da22/Untitled.png](Passionfruit%20tool%200b355e4d80f24c6d81388f3461e5da22/Untitled.png)

앱 정보와 File, Modules, Classes, Console, Storage 등 확인 가능

![Passionfruit%20tool%200b355e4d80f24c6d81388f3461e5da22/Untitled%201.png](Passionfruit%20tool%200b355e4d80f24c6d81388f3461e5da22/Untitled%201.png)

아직 많이 써보진 않았지만, Objection이 더 다양한 기능을 제공하는 것 같음

Windows 환경에서 NSLog를 확인할 때 괜찮은듯