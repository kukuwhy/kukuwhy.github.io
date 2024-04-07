# iOS Life Cycle

Created: 2022년 6월 19일 오후 3:34
Tags: iOS

## 앱 실행과정

1. main 함수 실행
2. main 함수는 UIApplicationMain 함수 호출
3. UIApplicationMain 함수는 앱의 본체에 해당하는 UIApplication 객체 생성
4. nib, Info.plist 파일 등을 정보를 참조하여 필요한 데이터 로드
5. AppDelegate 객체를 만들어 App 객체와 연결하고 런루프 등 실행에 필요한 준비
6. 실행 완료 전 App 객체가 AppDelegate 에게 application:didFinishiLaunchingWithOptions: 메시지 전송

## Main Function

`Objective-C`는 C언어 기반 언어로써 시작점은 `main` 함수이다. 하지만 `Swift`는 C 기반 언어가 아니라 `main` 함수가 존재하지 않으며 시작 포인트 또한 존재하지 않는다. 다만 시작 진입점은 존재해야하기 때문에 어노테이션 표기로 대체한다.

```objectivec
// Objective-C Main

#import <UIKit/UIKit.h>
#import "AppDelegate.h"
 
int main(int argc, char * argv[])
{
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

`Objective-C`는 `main` 함수가 기본적으로 생성된다.

```swift
// Swift

import UIKit

@UIApplicationMain // 어노테이션으로 표기

class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {

    }
}
```

`Swift`에서는 `UIKit framework`가 `main`함수를 관리하여 직접 작성하지 않는다.

![Untitled](iOS%20Life%20Cycle%208e4bbfa1873c4bffaf6e82f41ed66c3e/Untitled.png)

![Untitled](iOS%20Life%20Cycle%208e4bbfa1873c4bffaf6e82f41ed66c3e/Untitled%201.png)

`UIApplicationMain`함수를 호출하는 함수를 확인하면 `main` 또는 `start`인 진입점을 확인할 수 있다.

## UIApplicationMain

`UIApplicationMain` 함수는 코코아 터치 프레임워크에서 앱의 라이프 사이클을 시작하는 함수이다. `UIApplication` 객체의 인스턴스를 만들고 앱으로서의 기능을 위한 기반을 마련하는 앱 로딩 프로세스가 진행된다.

```swift
public func UIApplicationMain(_ argc: Int32,
_ argv: UnsafeMutablePointer<UnsafeMutablePointer<Int8>>!,
_ principalClassName: String?,
_ delegateClassName: String?) -> Int32
```

`UIApplicationMain`은 총 4가지 인자를 받는다. 이 중 네 번째 인자가 `AppDelegate` 클래스 이름을 전달한다.

![Untitled](iOS%20Life%20Cycle%208e4bbfa1873c4bffaf6e82f41ed66c3e/Untitled%202.png)

진입점에서 `UIApplicationMain` 함수를 호출할 때 네 번째 인자 값으로 `AppDelegate` 클래스 이름을 확인할 수 있다.

## AppDelegate

iOS 상태는 총 5가지가 있다.

`App State:`

- Not Running : 실행되지 않았거나, 시스템에 의해 종료된 상태
- Inactive : 실행 중 이지만 이벤트를 받고있지 않는 상태
- Active : 앱이 실질적으로 활동하고 있는 상태
- Background : 백그라운드에서 실질적인 동작을 하고있는 상태
- Suspended : 백그라운드에서 활동을 멈춘 상태

그리고, 앱 대부분의 상태 전환은 `AppDelegate` 객체의 메소드 호출을 거친다.

`AppDelegate:method`

- application:willFinishLaunchingWithOptions : 앱이 최초 실행될 때 호출
- application:didFinishLaunchingWithOptions : 앱이 실행된 직후 사용자의 화면에 보여지기 직전에 호출
- applicationDidBecomeActive : 앱이 Active 상태로 전환된 직후 호출
- applicationWillResignActive : 앱이 Inactive 상태로 전환되기 직전에 호출
- applicationDidEnterBackground : 앱이 백그라운드 상태로 전환된 직후 호출
- applicationWillEnterForeground : 앱이 Active 상태가 되기 직전 화면에 보여지기 직전에 호출
- applicationWillTerminate : 앱이 종료되기 직전에 호출

만약 보안 솔루션이 OS 변조를 탐지할 경우엔 보통 앱 실행 시 감지하기 때문에

`application:willFinishLaunchingWithOptions`, `application:didFinishLaunchingWithOptions`, `applicationDidBecomeActive` 메소드 내에 감지 로직이 존재한다고 예상할 수 있다.

또는 함수 명, 문자열 등이 검색되지 않을 때 진입점부터 분석할 수 있다.

![Untitled](iOS%20Life%20Cycle%208e4bbfa1873c4bffaf6e82f41ed66c3e/Untitled%203.png)

실제로 `applicationDidBecomeActive` 메소드를 확인한 결과 탈옥을 감지한 메시지를 띄우며 종료하는 로직을 볼 수 있으며, 메시지를 띄우는 조건을 분석하면 탐지 로직을 알 수 있다.