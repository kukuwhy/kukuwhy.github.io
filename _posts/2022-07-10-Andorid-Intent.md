---
layout: post
title:  "Android Intent"
tags: Andorid Intent
---

# Intent?

안드로이드의 애플리케이션 구성은 4대 컴포터넌트로 이루어져 있습니다.

- Activity
- Service
- Broadcast Receiver
- Content Provider

이 같은 컴포넌트 간의 통신을 맡고 있는 것이 `Intent` 입니다.

intent의 통신 방법은 2가지 방법이 있습니다.

- 명시적 intent
- 암시적 intent

## 명시적 intent

가장 많이 볼 수 있는 방법으로 앱의 화면전환을 하는 방법입니다.

하나의 acitivity에서 다른 activity로 화면 전환 시 사용하는 방법입니다.

```java
Intent intent = new Intent(A_Acitivity this, B_Activity.class);
startAcitivity(intent);
```

B_Activity.Class 생성 시에는 AndroidManifest.xml에도 Activity를 사용하겠다고 정의가 필요합니다.

```java
<activity android:name=".B_Activity">
```

이렇게 intent를 이용해 Activity간의 데이터를 전달할 수 있고. 리턴 값을 받을 수도 있습니다.

## 암시적 intent

Intent의 Action에 따라 해당하는 적합한 Application의 class를 호출할 수 있습니다.

이 때 여러개가 호출 될 수도 있습니다.

암시적 intent는 웹 브라우저 호출, 이메일 전송, 전화앱으로의 통화 등이 해당합니다.

- web browser call

```java
Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://m.naver.com"));
startActivity(intent);
```

- email send

```java
Intent intent = new Intent(Intent.ACTION_SEND);
intent.setType("text/plain");
intent.putExtra(Intent.EXTRA_EMAIL, "sampleXXX@sampleXXX.com");
intent.putExtra(Intent.EXTRA_SUBJECT, "전달할 이메일 제목");
intent.putExtra(Intent.EXTRA_TEXT, "전달할 내용");
startActivity(Intent.createChooser(intent, "Choose Email"));
```

- 전화걸기 호출

```java
Intent intent = new Intent(Intent.ACTION_VIEW,Uri.parse("tel:010-0000-0000"));
startActivity(intent);
```

## Activity Manager(adb)

Activity Manager(am)은 adb 쉘 내에서 Activity Manager 도구로 명령어를 실행하여 액티비티 시작, 프로세스 강제 종료, 인텐트 브로드캐스트, 기기 화면 속성 수정 등 다양한 식스템 작업을 실행할 수 있습니다.

```java
// <intent>에서 지정한 activity를 실행한다.
am start-activity [option] <intent>
```

exported option이 false로 설정되어 있다면 권한이 없는 Activity에 접근될 수 있음

별도의 권한 확인 로직이 필요합니다.