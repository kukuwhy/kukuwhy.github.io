# iOS Security Solution bypass

Created: 2022년 6월 15일 오후 1:26
Tags: frida, iOS

## 서론

iOS 진단 시 보안 솔루션이 적용되어 있으면 실행 불가부터 시작해서 진단에 많은 제약사항이 생긴다. 원할한 진단을 위해 솔루션이 적용되지 않은 `*.ipa`를 제공받거나, 직접 솔루션을 우회한 후 진단을 시작해야 한다.

보안 솔루션을 우회하기 위해 상황에 따른 바이너리 분석 방법 및 Frida를 통한 우회 스크립트를 정리하는 가이드를 작성하고자 한다.

## 환경

- iOS 13.1.3 (iPhone 7)
- Ghidra 10.1.1 & IDA
- Frida 15.1.14

---

**목차**

---

## 탈옥 감지

`ipa`를 추출해서 바이너리 분석을 시작하면 “jail” 또는 “jailbreak”와 같은 문자열 식별이 가능할 경우와 난독화 등으로 인해 불가능할 경우가 있다. 문자열 식별이 가능하면 우회 포인트를 찾아가는게 어렵지 않지만, 식별이 불가능할 경우 분석과 우회 난이도가 매우 올라갈 가능성이 있다.

### 1. 함수에 특정 문자열이 포함된 경우

가장 쉬운 난이도의 탈옥감지 로직은 함수 명에 “jail” 등의 문자열이 포함되어 있으며, 해당 함수가 모든 검증을 수행하여 참 또는 거짓 결과 값에 따른 탈옥 여부를 체크하는 로직이다.

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled.png)

“jail”로 문자열을 검색하여 `+[AppJailBrokenChecker isAppJailbroken]`를 확인할 수 있다. 해당 함수는 `unsigned __int8 __cdecl`형식의 값을 반환하지만, 이와 비슷한 유형의 탈옥 감지 함수들은 주로 `bool`형식으로 `true` 또는 `false`를 반환한다.

```jsx
if(ObjC.available) {
    var className = "AppJailBrokenChecker";
    var methodName = "+ isAppJailbroken";
    var hookMethod = eval("ObjC.classes." + className + '["'  + methodName + '"]');

    Interceptor.attach(hookMethod.implementation, {
        onEnter: function(args) {
            
        }
        ,onLeave: function(retval) {
            var newretval = ptr("0x0");
            retval.replace(newretval);
            console.log("[*] Jailbreak Detection Bypass !");
        }
    });
}
```

클래스와 메서드명이 있기 때문에 해당 함수를 후킹하여 `0x0`을 반환하도록 하면 된다. `bool`형식의 반환이라도 `0x0`이 `false`이므로 같은 방법으로 후킹이 가능하다.

<aside>
💡 위와 같은 상황이면 frida-trace툴을 이용해도 쉽게 우회가 가능하다.

```bash
frida-trace -U -f com.test.app -m "+[* *jail*]" -m "+[* *Jail*]" -m "-[* *jail*]" -m "-[* *Jail*]"
```

frida-trace는 frida설치 시 frida-ps와 같이 기본적으로 내장된 툴이며, `-m` 옵션은 `include ObjC Method` 를 의미한다. 위 명령어로 앱을 실행하는 순간부터 “jail”이 포함된 모든 클래스  명과 함수 명의 호출을 추적할 수 있다.

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%201.png)

실행 결과를 보면 `+[AppJailBrokenChecker isAppJailbroken]`가 호출된 것을 확인할 수 있으며, 해당 함수가 존재할 시 스크립트를 생성해준다.

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%202.png)

`Auto-generated handler at ~` 경로에 가면 함수명으로 된 스크립트가 생성된 것을 확인할 수 있다. frida-trace 사용 시 해당 경로에 스크립트가 없을 시 자동으로 생성하며 이를 사용하여 추적한다.

```jsx
onLeave(log, retval, state) {
      var newretval = ptr("0x0");
      log("[*] +[AppJailBrokenChecker isAppJailbroken] >>>> Bypass !");
      retval.replace(newretval);
  }
```

생성된 스크립트 내에 `onEnter`와 `onLeave`가 자동으로 작성되어 있으며, 각각 함수 진입과 반환 시 동작할 코드를 작성해주면 된다. 이 경우엔 함수 반환 시 `0x0` 값을 주도록 `onLeave`내에 코드를 작성하였다.

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%203.png)

다시 처음에 사용한 frida-trace 명령어를 그대로 실행하면 수정한 스크립트가 동작하여 탈옥 감지가 우회되는 것을 확인할 수 있다.

</aside>

### 2. 문자열은 존재하지만 정의된 함수명이 아닐 경우

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%204.png)

위 로직은 탈옥 관련 앱인 “cydia”와 관련된 문자열을 참조하는 함수이며, 함수 명이 정의되어있지 않아 IDA가 임의로 `sub_address`형식으로 지정하였다. 위 함수는 `sub_1002A80BC`이다.

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%205.png)

`sub_1002A80BC`함수를 호출하는 함수(xref)를 추적하면 `sub_1002A79DC`함수에서 온갖 탈옥 감지 기법을 검증하는 것을 알 수 있다.

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%206.png)

계속 타고 올라가면서 추적하다 보면 `sub_1002A6028`의 리턴 값에 따라 `_exit`함수를 호출해 앱을 종료시키는 것을 알 수 있다.

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%207.png)

AArch64 ABI에 따르면 `x0`레지스터는 함수의 반환 값이 포함된다고 한다. 따라서 `sub_1002A6028`함수가 실행되고 난 다음인 `15E8C0`주소에서 `x0`레지스터의 값은 `sub_1002A6028`함수의 반환 값이 들어있다고 볼 수 있다.

<aside>
💡 ARM64 ABI 규칙

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%208.png)

[참고 링크](https://docs.microsoft.com/ko-kr/cpp/build/arm64-windows-abi-conventions?view=msvc-170)

주소 값에 대한 참조 속성

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%209.png)

[Frida Javascript API](https://frida.re/docs/javascript-api/)

`context` 속성은 `ia32/x64/arm` 아키텍처에 대한 레지스터를 참조할 수 있다.

</aside>

```jsx
var targetModule = 'binary_file_name';
var addr = ptr(0x15e8c0);
var moduleBase = Module.getBaseAddress(targetModule);
var targetAddress = moduleBase.add(addr);

Interceptor.attach(targetAddress, {
    onEnter: function(args) {
        if(this.context.x0 == 0x1) {
            this.context.x0 = 0x0
            console.log("[*] Jailbreak Detection Bypass !");
        }
    }
})
```

해당 주소에 접근해 `x0`레지스터 변조를 시도하면 탈옥이 우회되는 것을 알 수 있다.

### 3. 난독화 등의 이유로 문자열 식별이 어려울 경우

난독화 등으로 탈옥 감지 로직을 찾기 어려울 경우에는 많은 분석 시간이 소요된다. 하지만 모든 탈옥 감지는 비슷한 iOS 내장 함수들을 사용하기 때문에 감지 기법에 자주 사용되는 함수 몇 가지를 알면 분석 시간을 단축시킬 수 있다.

이와 같은 앱을 찾지 못하였기에 분석 시 확인할만한 함수를 정리한다.

- `canOpenUrl(_:)`
    - ‘카카오톡으로 로그인’처럼 다른 앱을 실행시키기 위해 해당 앱이 실행 가능한지 여부를 판단해준다. 이를 이용해 탈옥 관련 앱의 존재 유무를 검증할 수 있다.
- `fileExists(atPath:isDirectory:)` | `fileExistsAtPath:`
    - 파일의 경로가 존재하는지 확인한다. 이를 이용해 bash, sshd, sh 등 탈옥 시 활성화되는 바이너리의 존재 유무를 검증할 수 있다.
- `isReadableFile(atPath:)`
    - 파일의 읽기 권한을 확인한다. 이를 이용해 탈옥 관련 파일이 읽히는지 검증한다.
- `write(toFile:)`, `removeItem(atPath:)`
    - 파일 쓰기와 삭제를 실행한다. 이를 이용해 루트 권한이 필요한 디렉터리의 쓰기 유무를 검증한다.
- `destinationOfSymbolicLink(atPath:)`
    - 심볼릭 링크로 가리키는 문자열 경로를 확인한다. 이를 이용해 탈옥 관련 심볼릭 링크 존재 유무를 검증한다.
- `exit()`
    - 앱을 종료한다. 이를 이용해 보안 솔루션이 탈옥 및 디버깅 감지 시 앱을 강제로 종료한다. 대다수의 솔루션이 검증 시 종료를 유도하므로 앱의 반응에 따라 `exit()` 근처부터 분석하는 것을 추천한다.

---

## 안티 디버깅

iOS 의 안티 디버깅 기법은 여러가지가 존재하며, 아래 작성된 내용 이외에 더 많은 기법이 존재할 수 있다.

### 1. ptrace를 이용한 안티 디버깅

안티 디버깅 기법 중 `ptrace`를 이용한 기법이 적용될 시 Frida 나 lldb 등의 디버깅 도구가 attach하지 못한다.

`ptrace`는 생성된 프로세스의 움직임과 데이터의 읽고 쓰는 방법 또는 어떤 에러를 내는지 추적하기 위해 마련된 System call 중 하나이다. 주로 디버그를 위해 사용되며 디버거는 일종의 `ptrace` 명령어 집합 툴이라고 볼 수 있다.

이 기법이 적용되면 디버거가 attach를 시도할 시 `catch-able segmentation fault`가 발생하여 디버깅이되지 않는다.

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%2010.png)

애플 오픈소스 문서의 [ptrace.h](https://opensource.apple.com/source/xnu/xnu-201/bsd/sys/proc.h.auto.html) 내용을 확인하면 `#define PT_DENY_ATTACH 31`, 즉 플래그 값 31이 attach를 deny 시키는 것을 확인할 수 있다.

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%2011.png)

위 처럼 `dlsym`으로 `ptrace`함수의 주소값을 가져온 뒤 `1F(31)`로 `PT_DENY_ATTACH`플래그 값을 설정해 해당 프로세스에 attach하지 못하도록 한다.

```c
{
  void *v0;
  void *v1;

  v0 = dlopen(0LL, 10);
  v1 = dlsym(v0, "ptrace");
  ((void (__fastcall *)(__int64, _QWORD, _QWORD, _QWORD))v1)(31LL, 0LL, 0LL, 0LL);
  return dlclose(v0);
}
```

의사코드로 디컴파일하면 위와 같다.

<aside>
💡 `dlsym`은 동적 라이브러리를 로드할 때 사용하는 함수이며 보통 `dlopen`,  `dlsym`,  `dlclose` 가 묶여 실행된다.

```c
// Function Prototype

void *dlopen(const char *filename, int flag)
void *dlsym(void *handle, const char *symbol)
int dlclose(void *handle)
```

</aside>

```jsx
Interceptor.attach(Module.findExportByName(null, 'ptrace'), {
    onEnter: function(args) {
        console.log("[*] ptrace called !");
        if(args[0].toInt32() == 31) {
            console.log("ptrace Anti Debugging Bypass !");
            args[0] = ptr(0x0);
        }
    }
})
```

ptrace 를 후킹하여 첫 번째 인자 값으로 `0x1F`대신 `0x0`을 전달해 `ptrace`를 이용한 안티 디버깅을 우회할 수 있다.

### 2. SVC(Supervisor Call)을 이용해 ptrace를 호출하는 안티 디버깅

일반적인 응용프로그램은 `User Mode`에서 실행되는데, 이는 실행, 종료, I/O 작업 등의 사용자가 함부로 사용하면 영향이 커질 `System call` 명령에 제한을 두기 위함이다. 제한된 명령에 접근하는 것은 `Supervisor`만 실행할 수 있어야 하며 이것이 `Kernel Mode`가 된다. 즉, 제한된 명령을 실행할 때 응용프로그램은 `SuperVisor Call`을 통해 허락을 맡은 후 `Supervisor Mode(Kernel Mode)`로 변경되어 `System Call`이 실행되고 다시 `User Mode`로 변경되는 과정을 거친다. 따라서 운영체제가 제공하는 제한된 명령어가 `System Call`이고, `System Call`을 실행시키기 위한 CPU 명령어가 SVC이다.

즉, CPU 명령을 통해 ptrace를 호출한다고 볼 수 있다.

```
MOV          X0, #0x1F
MOV          X1, #0
MOV          X2, #0
MOV          X3, #0
MOV          X16, #0x1A
SVC          0x80
```

위 어셈블리어는 `syscall(ptrace(31), 0, 0, 0)`과 같으며, `MOV X0, #0x1F`로 디버깅 플래그 값인 `0x1F(31)`을 `MOV X16, #0x1A`로 `ptrace`를 인자값으로 지정하는 것을 알 수 있다. 이것은 첫 번째 안티 디버깅인 `ptrace`의 검증방법과 같지만 `dlsym`이 아닌 `SVC`를 통해 호출한다는 것이 다르다.

<aside>
💡 List of system calls from iOS 6.0 GM

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%2012.png)

위 그림은 [Kernel syscalls문서](https://www.theiphonewiki.com/wiki/Kernel_Syscalls) 중 iOS 부분의 일부이다.
`0x80`은 system call의 호출번호이며, `ptrace`는 `26(0x1A)`인 것을 알 수 있다.

</aside>

```
E0 03 80 D2 (MOV X0, #0x1F)

01 00 80 D2 (MOV X1, #0)

02 00 80 D2 (MOV X2, #0)

03 00 80 D2 (MOV X3, #0)

50 03 80 D2 (MOV X16, #0x1A)

01 10 00 D4 (SVC 0x80)
```

`SVC`를 통해 `ptrace`를 호출하면 Frida로 후킹되지 않는다. 따라서 메모리 상의 hex 값을 스캔하여 매핑한 뒤 플래그 값을 덮어씌우는 방식으로 후킹을 해야한다. 어셈블리어를 hex로 표현하면 위 코드와 같다.

```jsx
var m = Process.findModuleByName('ModuleName');  // Modify target module name
var pattern = '50 03 80 D2 01 10 00 D4';
Memory.scan(m.base, m.size, pattern, {
	onMatch: function(address, size) {
		console.log('\x1b[34m[!] Anti Debugging Bypass ! (SVC ptrace)\x1b[0m');
		console.log(Memory.readByteArray(address, size));
		Memory.protect(address, size, 'rwx');
		Memory.writeByteArray(address.add(4), [0x1F, 0x20, 0x03, 0xD5]);
		console.warn(Memory.readByteArray(address, size));
	},
	onComplete: function () {
		console.log("[*] SVC call ptrace Anti Debugging Bypass !");
	}
})
```

hex 값 중 `SVC`로 `ptrace`를 호출하는 부분을 스캔한 뒤 `SVC 0x80` 부분을 `NOP` 처리하면 우회가 가능하다.

### 3. ptrace 변조 감지를 이용한 안티 디버깅

첫 번쨰로 설명한 `ptrace` 안티 디버깅 적용 시 디버거가 attach를 시도하면 `catch-able segmentation fault`를 발생시킨다고 했다. 앱은 이 특성을 이용해 `ptrace`가 변조되었는지 확인하여 안티 디버깅을 검증할 수 있다.

```objectivec
int deny_attach_successful = 0;

void sigsegv_handler(int sig) {
	printf("sigsegv_handler: %i\n", sig);
	deny_attach_successful = 1;
}

int main(int argc, char *argv[]) {
	pid_t pid = getpid();
	ptrace(PT_DENY_ATTACH, 0, 0, 0);
	signal(SIGSEGV, sigsegv_handler);
	ptrace(PT_ATTACH, pid, 0, 0);

	if (!deny_attach_successful) {
		printf("FAILURE\n");
		return 1;
	}

	printf("SUCCESS\n");
	return 0;
}
```

`deny_attach_successful` 전역변수를 설정해 0으로 초기화한 후 `catch-able segmentation fault`가 발생하면 전역변수를 1로 설정하는 커스텀 시그널 핸들러인  `sigsegv_handler()`를 만든다.
처음 `ptrace` 호출 시 `PT_DENY_ATTACH` 플래그를 등록한다. 이후 두 번째 `ptrace` 호출 시 `PT_ATTACH` 플래그를 이용해 자신에게 attach를 시도한다.
자신에게 attach 시도 시 첫 번째 `ptrace`로 인해 `catch-able segmentation fault` 가 발생한다면 커스텀 핸들러가 동작하여 `ptrace` 변조가 이루어지지 않았다고 판단한다.
만약 `ptrace`가 변조되었다면 `catch-able segmentation fault` 가 발생하지 않아 커스텀 핸들러가 동작하지 않으며 변조 및 디버깅이 감지된다고 판단한다.

우회 과정은 아래와 같다.

1. 첫 번째 `ptrace` 우회
2. 커스텀 시그널 핸들러 주소 확인
3. 두 번째 `ptrace` 우회
4. 커스텀 시그널 핸들러 강제 호출

또는 첫 번째 `ptrace`를 우회한 후 if 분기문을 우회하는 방법이다.

아직 해당 기법이 적용된 앱을 확인하지 못해 Frida 스크립트를 작성하지 않았으나 lldb 디버거를 이용한 우회 과정을 [참고 페이지](https://alexomara.com/blog/defeating-anti-debug-techniques-macos-ptrace-variants/)에서 확인할 수 있다.

### 4. sysctl을 이용한 안티 디버깅

`sysctl`은 시스템 정보를 설정하거나 시스템 정보를 확인할 수 있다. 이를 바탕으로 프로세스의 디버깅 유무를 판단할 수 있는데, 때문에 단순히 `sysctl`만 사용해서는 디버깅 감지만 할 뿐 완전한 안티 디버깅이 될 수 없다. 따라서 `sysctl`로 얻은 정보로 앱을 종료시키는 등의 추가 로직이 있어야 안티 디버깅이 구현된다.

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%2013.png)

애플 오픈소스 문서의 [proc.h](https://opensource.apple.com/source/xnu/xnu-201/bsd/sys/proc.h.auto.html) 내용을 확인하면 `#define P_TRACED 0x00800`이 프로세스가 디버그되고 있다는 값이라는 것을 확인할 수 있다.

<aside>
💡 `sysctl`가 호출되면 성공 시 `0`, 실패 시 `-1`을 반환한다. 이 때 세 번째 인자 값으로 시스템 정보가 담긴 구조체 형식의 변수(info)가 들어가는데 여기에는 `P_TRACED` 플래그 값이 같이 담겨있다.
`sysctl` 호출 성공 여부와 `P_TRACED` 값이 담긴 주소를 참조해 `0x800` 과 AND 연산을 통해 디버깅 중인지 판단한다.

```objectivec
int isDebugged()
{
    int name[4];
    struct kinfo_proc info;
    size_t info_size = sizeof(info);
    
    info.kp_proc.p_flag = 0;
    
    name[0] = CTL_KERN;
    name[1] = KERN_PROC;
    name[2] = KERN_PROC_PID;
    name[3] = getpid();
    
    if (sysctl(name, 4, &info, &info_size, NULL, 0) == -1) {
        //Program is being debugged
        return 1;
    }

		// P_TRACED == 0x800
		// p_flag & 0x800
    return ((info.kp_proc.p_flag & P_TRACED) != 0);
}
```

</aside>

![Untitled](iOS%20Security%20Solution%20bypass%201bcf407247bd4555872ca15d0c7722d9/Untitled%2014.png)

`sysctl`을 호출하며 `0x800`과의 AND 연산으로 디버깅 유무를 판단하는 구조를 볼 수 있다.

```jsx
Interceptor.attach(Module.findExportByName(null, 'sysctl'), {
	onEnter: function(args) {
		this.kp_proc = args[2];
		this.count = args[1];
	},
	onLeave: function(retval) {
		if(retval == 0x0) {
			if(this.count.toInt32() == 4) {
				var p_flag = Memory.readInt(this.kp_proc.add(32));
					if((p_flag & 0x800) !== 0) {
						console.log('[*] sysctl Anti Debugging Bypass !');
						Memory.writeByteArray(this.kp_proc.add(32), [0x00, 0x00]);
					}
			}
		}
	}
});
```

`sysctl`을 후킹하여 세 번째 인자 값의 `P_TRACED` 플래그 주소를 참조하여 해당 플래그의 값을 `0x00`으로 변조하면 우회가 가능하다.

### 5. getppid를 이용한 안티 디버깅

일반적으로 앱은 `launchd process`에 의해 실행된다. 이는 `User Mode`로 인해 처음 올라가는 프로세스이며 PID는 1이다. 따라서 일반적인 앱의 부모 프로세스 번호는 1이 되어야 한다. 하지만 디버거에 의해 실행(spawn)되면 해당 앱은 디버거가 부모 프로세스가 되어 PID가 디버거의 번호로 설정된다.

이를 이용해 부모 프로세스는 구하는 함수인 `getppid()`를 통해 결과가 1이 아니라면 디버거가 생성했다고 판단할 수 있다.

```objectivec
func AmIBeingDebugged() -> Bool {
	return getppid() != 1
}
```

위 코드는 `getppid`를 이용해 디버깅을 검증하는 코드이며, 결과 값을 이용해 앱을 강제로 종료시키면 안티 디버깅이 적용된다.

```jsx
Interceptor.attach(Module.findExportByName(null, 'getppid'), {
	onEnter: function(args) {
	},
	onLeave: function(retval) {
		console.log("[*] getppid Anti Debugging Bypass !");
		retval.replace(ptr("0x1"));
	}
});
```

간단하게 `getppid`의 반환을 1로 주면 우회가 된다.

---

## 위변조

iOS는 기본적으로 코드 서명을 통한 위변조 감지를 하고있다. `ipa`를 추출하여 변조한 후 별도의 서명 작업 없이 설치 및 실행할 경우 앱이 그냥 종료되어 버린다.

하지만 3utools 도구로 애플 계정을 이용해 서명하면 정상적으로 설치 및 실행이 가능하다. 때문에 서명 검증 이외에 앱 자체적으로 변조를 검증해야 한다.

다만 웬만한 진단은 ‘통신 패킷 변조’ 와 ‘후킹’으로 가능하기 때문에 원할한 진단을 하기 위한 위변조 검증을 우회작업이 불필요하다. 따라서 추후 시나리오 모의해킹에서 필요할 경우 정리한 검증 방법을 토대로 분석 및 스크립트를 작성할 수 있도록 검증 방법만 정리하였다.

```objectivec
private static func checkBundleID(_ expectedBundleID: String) -> Bool {
    if expectedBundleID != Bundle.main.bundleIdentifier {
        return true
    }
    return false
}
```

현재 `BundleID`와 특정 `BundleID`를 비교하여 검증한다. 미리 비교 할 원래 `BundleID`를 코드 내 정의 또는 서버를 이용해 비교 검증해야 한다.

```objectivec
private static func checkMobileProvision(_ expectedSha256Value: String) -> Bool {
        guard let path = Bundle.main.path(forResource: "embedded", ofType: "mobileprovision") else { return false }
        let url = URL(fileURLWithPath: path)
        if FileManager.default.fileExists(atPath: url.path) {
            if let data = FileManager.default.contents(atPath: url.path) {
                // Hash: SHA256
                var hash = [UInt8](repeating: 0, count: Int(CC_SHA256_DIGEST_LENGTH))
                data.withUnsafeBytes {
                    _ = CC_SHA256($0.baseAddress, CC_LONG(data.count), &hash)
                }
                if Data(hash).hexEncodedString() != expectedSha256Value {
                    return true
                }
            }
        }
        return false
    }
```

바이너리 파일의 위치를 참조해 hash 값을 비교하여 검증한다. 미리 비교 할 원래 바이너리 hash 값을 코드 내 정의 또는 서버를 이용해 비교 검증해야 한다.

```objectivec
// Get hash value of Mach-O "__TEXT.__text" data with a specified image target
func getTextSectionDataSHA256Value() -> String? {
        guard let sectionInfo = findSection(SEG_TEXT, secname: SECT_TEXT) else {
            return nil
        }
        
        guard let startAddr = UnsafeMutablePointer<Any>(bitPattern: Int(sectionInfo.addr)) else {
            return nil
        }
        
        let size = sectionInfo.section.pointee.size
        
        // Hash: SHA256
        var hash = [UInt8](repeating: 0, count: Int(CC_SHA256_DIGEST_LENGTH))
        _ = CC_SHA256(startAddr, CC_LONG(size), &hash)
        
        return Data(hash).hexEncodedString()
    }
}

// Find loaded dylib with a specified image target
func findLoadedDylibs() -> [String]? {
        guard let header = base else {
            return nil
        }
        guard var curCmd = UnsafeMutablePointer<segment_command_64>(bitPattern: UInt(bitPattern: header) + UInt(MemoryLayout<mach_header_64>.size)) else {
            return nil
        }
        var array: [String] = Array()
        var segCmd: UnsafeMutablePointer<segment_command_64>!
        for _ in 0..<header.pointee.ncmds {
            segCmd = curCmd
            if segCmd.pointee.cmd == LC_LOAD_DYLIB || segCmd.pointee.cmd == LC_LOAD_WEAK_DYLIB {
                if let dylib = UnsafeMutableRawPointer(segCmd)?.assumingMemoryBound(to: dylib_command.self),
                    let cName = UnsafeMutableRawPointer(dylib)?.advanced(by: Int(dylib.pointee.dylib.name.offset)).assumingMemoryBound(to: CChar.self) {
                    let dylibName = String(cString: cName)
                    array.append(dylibName)
                }
            }
            curCmd = UnsafeMutableRawPointer(curCmd).advanced(by: Int(curCmd.pointee.cmdsize)).assumingMemoryBound(to: segment_command_64.self)
        }
        return array
    }
```

메모리의 base 주소에 접근하여 `__TEXT**.**__text` 섹션이나 `LoadCommand` 헤더를 검증한다.

```objectivec
private static func checkMobileProvision(_ expectedSha256Value: String) -> Bool {
        guard let path = Bundle.main.path(forResource: "embedded", ofType: "mobileprovision") else { return false }
        let url = URL(fileURLWithPath: path)
        if FileManager.default.fileExists(atPath: url.path) {
            if let data = FileManager.default.contents(atPath: url.path) {
                // Hash: SHA256
                var hash = [UInt8](repeating: 0, count: Int(CC_SHA256_DIGEST_LENGTH))
                data.withUnsafeBytes {
                    _ = CC_SHA256($0.baseAddress, CC_LONG(data.count), &hash)
                }
                if Data(hash).hexEncodedString() != expectedSha256Value {
                    return true
                }
            }
        }
        return false
    }
```

프로비저닝 프로파일을 검증한다. 프로비저닝 내에는 인증서와 실행 가능 기기 목록, 유효기간 등이 명시되어 있다.

---

## SSL Pinning

HTTPS의 문제점은 MITM이다. 클라이언트와 서버 사이의 중간에 있는 어떤 요소가 마치 서버와 클라이언트처럼 동작하여 인증서를 주고받으며 통신함으로서 중간에서 패킷을 해석하는 것을 말한다.

이렇게 되면 클라이언트와 서버는 암호화를 하지만 중간에서 인증서를 교환하기 때문에 패킷을 모니터링하고 해석할 수 있어 문제가 된다.

이러한 문제를 해결하기위해서 SSL Pinning을 도입했다..

<aside>
💡 SSL Pinning
서버의 SSL 보안인증서 정보를 클라이언트가 가지고 있고 클라이언트에 서버를 고정시킨다.

</aside>

### 주요 함수

[boringssl](https://commondatastorage.googleapis.com/chromium-boringssl-docs/ssl.h.html)은 openssl을 기반으로 시작된 오픈소스이며 IOS는 11 버전부터 boringssl을 도입했다.

IOS 13 : `SSL_set_custom_verify`

IOS 12 : `SSL_CTX_set_custom_verify`

```cpp
//borringssl ssl_lib.cc
// iOS 13
void SSL_set_custom_verify(SSL *ssl, int mode,
enum ssl_verify_result_t (*callback)(SSL *ssl, uint8_t *out_alert)) {
  if (!ssl->config) {
    return;
  }
  ssl->config->verify_mode = mode;
  ssl->config->custom_verify_callback = callback;
}

// iOS 12
void SSL_CTX_set_custom_verify(
    SSL_CTX *ctx, int mode,
    enum ssl_verify_result_t (*callback)(SSL *ssl, uint8_t *out_alert)) 
{
  ctx->verify_mode = mode;
  ctx->custom_verify_callback = callback;
}
```

`SSL_CTX_set_custom_verify` 는 인증서 확인을 구성한다.  인자 중 mode는 인증서 검증 모드를 설정하고 callback은 인증서의 유효성 결과를 나타낸다. 

인증서의 검증 모드는 아래에 정의된 값을 통해서 설정한다. 

`SSL_VERIFY_NONE` 으로 설정된다면 서버의 인증서를 확인하지 않는다. → bypass point

```cpp
// SSL_VERIFY_NONE, on a client, verifies the server certificate but does not
// make errors fatal. The result may be checked with |SSL_get_verify_result|. On
// a server it does not request a client certificate. This is the default.
#define SSL_VERIFY_NONE 0x00

// SSL_VERIFY_PEER, on a client, makes server certificate errors fatal. On a
// server it requests a client certificate and makes errors fatal. However,
// anonymous clients are still allowed. See
// |SSL_VERIFY_FAIL_IF_NO_PEER_CERT|.
#define SSL_VERIFY_PEER 0x01

// SSL_VERIFY_FAIL_IF_NO_PEER_CERT configures a server to reject connections if
// the client declines to send a certificate. This flag must be used together
// with |SSL_VERIFY_PEER|, otherwise it won't work.
#define SSL_VERIFY_FAIL_IF_NO_PEER_CERT 0x02

// SSL_VERIFY_PEER_IF_NO_OBC configures a server to request a client certificate
// if and only if Channel ID is not negotiated.
#define SSL_VERIFY_PEER_IF_NO_OBC 0x04
```

callback은 인증서의 유효성에 따라 아래 enum으로 정의된 값으로 설정된다.

인증서가 유효한 경우 `ssl_verify_ok` 으로 설정된다 → bypass point

```cpp
//boringssl ssl.h
enum ssl_verify_result_t BORINGSSL_ENUM_INT {
  ssl_verify_ok, // 인증서가 유효한 경우
  ssl_verify_invalid, // 인증서가 유효하지 않은 경우
  ssl_verify_retry, // 비동기식으로 인증서를 확인할 경우
};
```

`SSL_set_custom_verify` 은 `SSL_CTX_set_custom_verify` 처럼 작동하지만 개별 SSL을 구성한다. 사실 크게 다른점은 없고 IOS의 버전에 따라 다른 메소드를 후킹 포인트로 잡으면 된다.

 

`SSL_get_psk_identity`

[frida codeshare에 공유된 스크립트](https://codeshare.frida.re/@federicodotta/ios13-pinning-bypass/)나 [objection](https://github.com/sensepost/objection/blob/master/agent/src/ios/pinning.ts), [ssl kill switch2](https://github.com/nabla-c0d3/ssl-kill-switch2/blob/release/SSLKillSwitch/SSLKillSwitch.m)에서 SSL pinning bypass를 시도할 때 빠지지 않고 아래 메소드를 후킹해서 임의 문자열로 리턴하고 있다. 사실 테스트를 진행할 떄 아래 함수를 후킹하지 않고 위에서 기술한 검증 부분을 후킹하면 문제없이 SSL pinning bypass가 진행되었지만 다양한 솔루션 및 탐지코드에서 MITM을 탐지할 수 있는 방법 중에 하나인 것 같다.

```cpp
const char *SSL_get_psk_identity(const SSL *ssl) {
  if (ssl == NULL) {
    return NULL;
  }
  SSL_SESSION *session = SSL_get_session(ssl);
  if (session == NULL) {
    return NULL;
  }
  return session->psk_identity.get();
}
```

먼저 PSK는 Pre-Shared Key의 줄임말로 보안이 필요한 통신을 하는 양자 간에 미리 공유된 대칭 키를 말한다. 서버와 올바른 TLS 연결을 맺었을 때 적합한 psk identity가 존재하는지를 체크해서 MITM를 탐지하는 기법으로 사용하는듯하다. 여러가지 테스트가 필요할 듯하지만 해당 메소드를 NULL이 아닌 임의 문자열로 반환해서 탐지 코드 회피가 가능한 것으로 보인다. → bypass point

참고용으로 [RFC 4279](https://datatracker.ietf.org/doc/html/rfc4279) 에 정의된 PSK 교환 방식은 아래 알고리즘과 같다. 

```cpp
2.  PSK Key Exchange Algorithm

   This section defines the PSK key exchange algorithm and associated
   ciphersuites.  These ciphersuites use only symmetric key algorithms.

   It is assumed that the reader is familiar with the ordinary TLS
   handshake, shown below.  The elements in parenthesis are not included
   when the PSK key exchange algorithm is used, and "*" indicates a
   situation-dependent message that is not always sent.

      Client                                               Server
      ------                                               ------

      ClientHello                  -------->
                                                      ServerHello
                                                    (Certificate)
                                               ServerKeyExchange*
                                             (CertificateRequest)
                                   <--------      ServerHelloDone
      (Certificate)
      ClientKeyExchange
      (CertificateVerify)
      ChangeCipherSpec
      Finished                     -------->
                                                 ChangeCipherSpec
                                   <--------             Finished
      Application Data             <------->     Application Data

   The client indicates its willingness to use pre-shared key
   authentication by including one or more PSK ciphersuites in the
   ClientHello message.  If the TLS server also wants to use pre-shared
   keys, it selects one of the PSK ciphersuites, places the selected
   ciphersuite in the ServerHello message, and includes an appropriate
   ServerKeyExchange message (see below).  The Certificate and
   CertificateRequest payloads are omitted from the response.
```

---

## Frida Script

Frida 스크립트를 정리한다. 모든 기법을 우회한 것이 아니기 때문에 모든 대상 앱에서 동작하지 않을 수 있다.

### Jailbreak bypass code

- 대상 앱이 추가적인 경로 및 파일을 검증할 경우 paths 배열에 추가하며 실행한다.
- 주석 처리된 우회 코드는 대상 앱의 맞게 수정하며 실행한다.

```jsx
// Jailbreak bypass script

// Detect String reference
var paths = [
    "/etc/apt",
    "/Library/MobileSubstrate/MobileSubstrate.dylib",
    "/Applications/Cydia.app",
    "/Applications/blackra1n.app",
    "/Applications/FakeCarrier.app",
    "/Applications/Icy.app",
    "/Applications/IntelliScreen.app",
    "/Applications/MxTube.app",
    "/Applications/RockApp.app",
    "/Applications/SBSetttings.app",
    "/private/var/lib/apt/",
    "/Applications/WinterBoard.app",
    "/usr/sbin/sshd",
    "/private/var/tmp/cydia.log",
    "/usr/binsshd",
    "/usr/libexec/sftp-server",
    "/Systetem/Library/LaunchDaemons/com.ikey.bbot.plist",
    "/System/Library/LaunchDaemons/com.saurik.Cy@dia.Startup.plist",
    "/var/log/syslog",
    "/bin/bash",
    "/bin/sh",
    "/etc/ssh/sshd_config",
    "/usr/libexec/ssh-keysign",
    "/Library/MobileSubstrate/DynamicLibraries/Veency.plist",
    "/System/Library/LaunchDaemons/com.ikey.bbot.plist",
    "/private/var/stash",
    "/usr/bin/cycript",
    "/usr/bin/ssh",
    "/usr/bin/sshd",
    "/var/cache/apt",
    "/var/lib/cydia",
    "/var/tmp/cydia.log",
    "/Applications/SBSettings.app",
    "/Library/MobileSubstrate/DynamicLibraries/LiveClock.plist",
    "/System/Library/LaunchDaemons/com.saurik.Cydia.Startup.plist",
    "/private/var/lib/apt",
    "/private/var/lib/cydia",
    "/private/var/mobile/Library/SBSettings/Themes",
    "/var/lib/apt",
    "/private/jailbreak.txt",
    "/bin/su",
    "/pguntether",
    "/usr/sbin/frida-server",
    "/private/Jailbreaktest.txt",
    "/var/mobile/Media/.evasi0n7_installed"
];

var resolver = new ApiResolver('objc');

// >> bypass *jail* objc function << //////////
// resolver.enumerateMatches('*[* *jail**]', {
//     onMatch: function(match) {
//         var ptr = match["address"];
//         Interceptor.attach(ptr, {
//             onEnter: function() {},
//             onLeave: function(retval) {
//                 retval.replace(0x0);
//             }
//         });
//     },
//     onComplete: function() {}
// });

// Hook fileExistsAtPath
resolver.enumerateMatches('*[* fileExistsAtPath*]', {
    onMatch: function(match) {
        var ptr = match["address"];
        Interceptor.attach(ptr, {
            onEnter: function(args) {
                var path = ObjC.Object(args[2]).toString();
                this.jailbreakCall = false;
                for (var i = 0; i < paths.length; i++) {
                    if (paths[i] == path) {
                        this.jailbreakCall = true;
                    }
                }
            },
            onLeave: function(retval) {
                if (this.jailbreakCall) {
                    retval.replace(0x0);
                }
            }
        });
    },
    onComplete: function() {}
});

// Hook canOpenURL
resolver.enumerateMatches('*[* canOpenURL*]', {
    onMatch: function(match) {
        var ptr = match["address"];
        Interceptor.attach(ptr, {
            onEnter: function(args) {
                var url = ObjC.Object(args[2]).toString();
                this.jailbreakCall = false;
                if (url.indexOf("cydia") >= 0) {
                    this.jailbreakCall = true;
                }
            },
            onLeave: function(retval) {
                if (this.jailbreakCall) {
                    retval.replace(0x0);
                }
            }
        });
    },
    onComplete: function() {}
});
```

### Anti Debugging bypass

- 주석 처리된 SVC call 코드는 ModuleName을 대상 앱으로 수정 후 실행한다.

```jsx
// Anti Debugging bypass script

// ptrace bypass
Interceptor.attach(Module.findExportByName(null, 'ptrace'), {
    onEnter: function(args) {
        console.log("[*] ptrace called !");
        if(args[0].toInt32() == 31) {
            console.log("ptrace Anti Debugging Bypass !");
            args[0] = ptr(0x0);
        }
    }
})

// >> SVC call ptrace bypass <<
// var m = Process.findModuleByName('ModuleName');
// var pattern = '50 03 80 D2 01 10 00 D4';
// Memory.scan(m.base, m.size, pattern, {
// 	onMatch: function(address, size) {
// 		console.log('\x1b[34m[!] Anti Debugging Bypass ! (SVC ptrace)\x1b[0m');
// 		console.log(Memory.readByteArray(address, size));
// 		Memory.protect(address, size, 'rwx');
// 		Memory.writeByteArray(address.add(4), [0x1F, 0x20, 0x03, 0xD5]);
// 		console.warn(Memory.readByteArray(address, size));
// 	},
// 	onComplete: function () {
// 		console.log("[*] SVC call ptrace Anti Debugging Bypass !");
// 	}
// })

// sysctl bypass
Interceptor.attach(Module.findExportByName(null, 'sysctl'), {
	onEnter: function(args) {
		this.kp_proc = args[2];
		this.count = args[1];
	},
	onLeave: function(retval) {
		if(retval == 0x0) {
			if(this.count.toInt32() == 4) {
				var p_flag = Memory.readInt(this.kp_proc.add(32));
					if((p_flag & 0x800) !== 0) {
						console.log('[*] sysctl Anti Debugging Bypass !');
						Memory.writeByteArray(this.kp_proc.add(32), [0x00, 0x00]);
					}
			}
		}
	}
});

// getppid bypass
Interceptor.attach(Module.findExportByName(null, 'getppid'), {
	onEnter: function(args) {
	},
	onLeave: function(retval) {
		console.log("[*] getppid Anti Debugging Bypass !");
		retval.replace(ptr("0x1"));
	}
});
```

### SSL Pinning bypass

- iOS 13을 기준으로 동작한다.

```jsx
// SSL Pinning bypass script

try {
	Module.ensureInitialized("libboringssl.dylib");
} catch(err) {
	console.log("libboringssl.dylib module not loaded. Trying to manually load it.")
	Module.load("libboringssl.dylib");	
}

var SSL_VERIFY_NONE = 0;
var ssl_set_custom_verify;
var ssl_get_psk_identity;	

ssl_set_custom_verify = new NativeFunction(
	Module.findExportByName("libboringssl.dylib", "SSL_set_custom_verify"),
	'void', ['pointer', 'int', 'pointer']
);

/* Create SSL_get_psk_identity NativeFunction 
* Function signature https://commondatastorage.googleapis.com/chromium-boringssl-docs/ssl.h.html#SSL_get_psk_identity
*/
ssl_get_psk_identity = new NativeFunction(
	Module.findExportByName("libboringssl.dylib", "SSL_get_psk_identity"),
	'pointer', ['pointer']
);

/** Custom callback passed to SSL_CTX_set_custom_verify */
function custom_verify_callback_that_does_not_validate(ssl, out_alert){
	return SSL_VERIFY_NONE;
}

/** Wrap callback in NativeCallback for frida */
var ssl_verify_result_t = new NativeCallback(function (ssl, out_alert){
	custom_verify_callback_that_does_not_validate(ssl, out_alert);
},'int',['pointer','pointer']);

Interceptor.replace(ssl_set_custom_verify, new NativeCallback(function(ssl, mode, callback) {
	//  |callback| performs the certificate verification. Replace this with our custom callback
	ssl_set_custom_verify(ssl, mode, ssl_verify_result_t);
}, 'void', ['pointer', 'int', 'pointer']));

Interceptor.replace(ssl_get_psk_identity, new NativeCallback(function(ssl) {
	return "notarealPSKidentity";
}, 'pointer', ['pointer']));
	
console.log("[+] Bypass successfully loaded ");
```