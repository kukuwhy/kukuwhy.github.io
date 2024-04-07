# CSP BYPASS

Created: 2022년 5월 3일 오전 10:38
Tags: csp, web, xss

# CSP?

CSP(Content Security Policy)는 XSS나 데이터를 삽입하는 공격 시 방어하고 모니터링할 수 있는 시스템입니다. 간혹 점검하시다가 XSS 공격 하다가 “어..? 잘 렌더링 됐는데 왜 실행이 안되지” 해본 경험이 있는 분이 계실텐데요 대부분 십중팔구 CSP에 의해서 방어되었다고 봅니다. 

console에 찍히는 에러로그를 확인하시면 아래와 같이 바로 확인할 수 있습니다.

![Untitled](CSP%20BYPASS%2053879db17b5a40e8a0d30d6dabcd8183/Untitled.png)

CSP는 기본적으로 **인라인 코드**를 차단하고 있습니다. (safe-inline)

<aside>
💡 인라인 코드(inline code)
스크립트(+css) 사용 시 src를 지정하지 않고 태그, 이벤트, 링크에 바로 스크립트를 삽입하는 코드 ex) 
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<iframe src=”javascript:alert(1)”></iframe>

</aside>

또! CSP는 기본적으로 문자열 텍스트를 실행하는 함수도 차단합니다. (safe-eval)

<aside>
💡 ex)
setTimeout(”alert(1)”, ..)
eval(”alert(1)”)

</aside>

## Policy Directive

기본적으로 정책은 `<directive>` `<value>` 형태로 구성됩니다

정책은 세미클론(;)을 기준으로 여러개의 정책을 설정할 수 있으며, 하나의 정책에 공백으로 여러개의 값을 할당할수 도 있습니다👍

그럼 어떤 정책이 있는지 확인해보겠습니다.

| default-src | -src 로 끝나는 모든 리소스의 기본 동작을 제어
CSP 정책 내에 지정되지 않은 리소스 지시어가 없으면 default 정책 |
| --- | --- |
| img-src | 이미지를 로드할 수 있는 출처 제어 |
| script-src | 스크립트 태그 관련 권한 및 출처 제어 |
| style-src | 스타일시트 관련 권한 및 출처 제어 |
| child-src | 페이지 내에 삽입된 프레임 컨텐츠에 대한 출처 제어 |
| base-uri | 페이지의 <base> 태그에 나타낼 수 있는 URL 제어 |

value에는 어떤 값이 올 수 있는지도 확인해보겠습니다.

| *://example.com | scheme는 와일드카드(*)를 이용해 표현 가능 합니다. |
| --- | --- |
| http://*.example.com | host subdomain는 와일드카드(*)를 이용해 표현 가능 합니다. |
| https://example.com:* | port는 와일드카드(*)를 이용해 표현 가능 합니다. |
| none | 모든 출처를 차단합니다. |
| self | 페이지의 현재 출처(Origin) 내에서 로드하는 리소스만 허용합니다. |
| unsafe-inline | 인라인 코드를 차단하지 않습니다. |
| unsafe-eval | 문자열 함수 실행을 차단하지 않습니다. |
| nonce-<base64-value> | nonce 속성을 설정해 예외적으로 인라인 코드를 사용합니다. nonce값을 매 요청마다 난수 값으로 설정하지 않으면 의미가 없을 수 있습니다. 해당 설정이 적용되면 
unsafe-inline은 무시됩니다. |
| <hash-algorithm>-<base64-value> | script 혹은 style 태그 내 코드의 해시를 표현합니다. 해당 설정이 적용되면
unsafe-inline 은 무시됩니다. |

---

# Bypass

### 1. JSONP API

---

## CSP Testsite Online

CSP Evaluator("[https://csp-evaluator.withgoogle.com/](https://csp-evaluator.withgoogle.com/)")에서 어떤 CSP 항목이 부족한지 테스트해줌

점검 시 CSP가 걸려있다면 우회할 부분이 있는지 여기서 체크하면 좋을듯