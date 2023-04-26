## Item3) 코드 생성과 타입이 관계없음을 이해하기

타입스크립트 컴파일러의 역할 두 가지

- 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일한다.
- 코드의 타입 오류를 체크한다.

<br/>

컴파일은 타입 체크와 독립적으로 동작하기 때문에, **타입 오류가 있는 코드도 컴파일이 가능**하다. 타입스크립트 오류는 C나 JAVA같은 언어들의 경고(warning)와 비슷하다. 문제가 될 만한 부분을 알려주지만, 그렇다고 빌드를 멈추지는 않는다.

만약 오류가 있을 때 컴파일하지 않으려면 `tsconfig.json`에 `noEmitOnError`를 설정하거나 빌드 도구에 동일하게 적용하면 된다.

<br/>

**런타임에는 타입 체크가 불가능**하다.

```tsx
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
                    // ~~~~~~~~~ 'Rectangle'은(는) 형식만 참조하지만,
                    //           여기서는 값으로 사용되고 있습니다.
    return shape.width * shape.height;
                    //         ~~~~~~ 'Shape' 형식에 'height' 속성이
                    //                없습니다.
  } else {
    return shape.width * shape.width;
  }
}
```

`instanceof` 체크는 런타임에서 일어나지만 Rectangle은 타입이기 때문에 런타임 시점에 아무런 역할을 할 수 없다. 실제로 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 그냥 제거되어 버린다.

<br/>

런타임에 타입 정보를 유지하는 방법으로 다음 세 가지가 있다.

1. 속성 체크
    
    height 속성이 존재하는지 체크해본다.
    
    ```tsx
    interface Square {
      width: number;
    }
    interface Rectangle extends Square {
      height: number;
    }
    type Shape = Square | Rectangle;
    function calculateArea(shape: Shape) {
      if (**'height' in shape**) {
        // shape;  // Type is Rectangle
        return shape.width * shape.height;
      } else {
        // shape;  // Type is Square
        return shape.width * shape.width;
      }
    }
    ```

<br/>

2. 태그 기법
    
    런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 태그 기법이다.
    
    ```tsx
    interface Square {
      **kind: 'square';**
      width: number;
    }
    interface Rectangle {
      **kind: 'rectangle';**
      height: number;
      width: number;
    }
    type Shape = Square | Rectangle;
    
    function calculateArea(shape: Shape) {
      if (shape.kind === 'rectangle') {
        // shape;  // Type is Rectangle
        return shape.width * shape.height;
      } else {
        // shape;  // Type is Square
        return shape.width * shape.width;
      }
    }
    ```

<br/>

3. 클래스로 만들기
    
    타입(런타임 접근 불가)과 값(런타임 접근 가능)을 둘 다 사용하는 기법도 있다.
    
    ```tsx
    class Square {
      constructor(public width: number) {}
    }
    class Rectangle extends Square {
      constructor(public width: number, public height: number) {
        super(width);
      }
    }
    type Shape = Square | Rectangle;
    
    function calculateArea(shape: Shape) {
      if (shape instanceof Rectangle) {
        // shape;  // Type is Rectangle
        return shape.width * shape.height;
      } else {
        // shape;  // Type is Square
        return shape.width * shape.width;  // OK
      }
    }
    ```

일반적으로는 태그된 유니온과 속성 체크 방법을 사용한다.

<br/>

**타입 연산은 런타임에 영향을 주지 않는다.**

```tsx
function asNumber(val: number | string): number {
  return val as number;
}

// 변환된 js
function asNumber(val) {
	return val
}
```

`as number`는 타입 연산이고 런타임 동작에는 아무런 영향을 미치지 않는다. 값을 정제하기 위해서는 런타임의 타입을 체크해야하고 js 연산을 통해 변환을 수행해야 한다.

```tsx
function asNumber(val: number | string): number {
  return typeof(val) === 'string' ? Number(val) : val;
}
```

<br/>

**런타임 타입은 선언된 타입과 다를 수 있다.**

```tsx
function turnLightOn() { }
function turnLightOff() { }
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      turnLightOn();
      break;
    case false:
      turnLightOff();
      break;
    default:
      console.log(`실행되지 않을까 걱정이다.`);
  }
}
```

console.log를 실행할 수 있는가? JS에서는 setLightSwitch를 “ON”으로 호출할 수도 있었을 것이다. 순수 타입스크립트에서도 마지막 코드를 실행할 수 있는 방법이 있다. 예를들어, 네트워크 호출로부터 받아온 값으로 함수를 실행하는 경우가 있다. API를 잘못 파악해서 받아온 값의 타입이 다를 수 있다. 이때 런타임에는 함수까지 전달된다.

<br/>

타입스크립트에서는 런타임 타입과 선언된 타입이 맞지 않을 수 있다. 선언된 타입이 언제든지 달라질 수 있다는 것을 명심해야 한다.

<br/>

**타입스크립트 타입으로는 함수를 오버로드할 수 없다.**

```tsx
function add(a: number, b: number) { return a + b }
      // ~~~ 중복된 함수 구현입니다.
function add(a: string, b: string) { return a + b }
      // ~~~ 중복된 함수 구현입니다.

// JS
function add(a, b) { return a + b }
```

구현체는 오직 하나뿐이다.

<br/>

**타입스크립트 타입은 런타임 성능에 영향을 주지 않는다.**

타입과 타입 연산자는 JS 변환 시점에 제거되기 때문에 런타임의 성능에 아무런 영향을 주지 않는다.

런타임 오버헤드가 없는 대신 타입스크립트 컴파일러는 빌드타임 오버헤드가 있다. 오버헤드가 커지면 빌드 도구에서 ‘트랜스파일만(transpile only)’을 설정하여 타입체크를 건너뛸 수 있다.