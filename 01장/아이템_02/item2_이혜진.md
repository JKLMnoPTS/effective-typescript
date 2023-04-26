## Item2) 타입스크립트 설정 이해하기

```tsx
function add(a, b) {
  return a + b
}
add(10, null)
```

위 코드는 오류 없이 타입 체커를 통과할 수 있는가?

설정이 어떻게 되어있는지 모르면 대답할 수 없다. 가급적 설정 파일을 사용하는 것이 좋다. 그래야 타입스크립트를 어떻게 사용할 계획인지 알 수 있다.

<br/>


설정파일 생성하기

```bash
tsc --init
```

<br/>

**noImplicitAny**

변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어한다. 다음 코드는 해제되어 있을 때에 유효하다.

```tsx
function add(a, b) {
	return a + b
}
```

a, b, 반환값은 any로 추론되어 있다. any타입을 매개변수에 사용하면 타입 체커는 무력해진다. 만약 noImplicitAny가 설정되어 있으면 오류가 된다.

타입스크립트는 타입 정보를 가질 때 가장 효과적이기 때문에 되도록이면 noImplicitAny를 설정해야 한다. 자바스크립트로 되어 있는 기존 프로젝트를 타입스크립트로 전환할 때는 설정을 해제한다.

<br/>

**strictNullChecks**

null과 undefined가 모든 타입에서 허용되는지 확인하는 설정이다. 다음 코드는 ㄴ해제되어 있을 때에 유효하다.

```tsx
const x: number = null
```

하지만 strictNullChecks를 설정하면 오류가 난다. 만약 null을 허용하려고 한다면 의도를 명시적으로 드러냄으로써 오류를 고칠 수 있다.

```tsx
const x: number | null = null
```

<br/>

만약 null을 허용하지 않으려면 null을 체크하는 코드나 단언문을 추가해야 한다.

```tsx
const el = document.getElementById('status');
el.textContent = 'Ready';

if (el) {
	el.textContent = 'Ready';  // 정상, null은 제외된다.
}
el!.textContent = 'Ready';  // 정상, el이 null이 아님을 단언한다.
```

`!.`

확정 할당 어선셜, 값이 무조건 할당되어 있다고 컴파일러에 전달하여 값이 없어도 변수나 객체 사용이 가능하다.

strictNullChecks 설정 없이 개발하기로 했다면 `undefined는 객체가 아닙니다` 라는 끔찍(정말로 끔찍)한 런타임 오류를 주의해야 한다.