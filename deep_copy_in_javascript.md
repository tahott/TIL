자바스크립트에서 깊은 복사를 통해 데이터를 다루는 것에 대하여 정리하자

데이터를 다루다보면 외부 API를 호출하여 데이터를 받아 오거나 DB에서 조회한 값을 사용할 때가 있고, 이럴 때 조회한 데이터를 그대로 사용하기 보다는 내보내는 포맷에 맞춰 변형/가공을 해야 한다. 이렇게 원본 데이터를 변형해야 할 때 얕은 복사와 깊은 복사와의 차이를 알고 적절하게 사용할 수 있었으면 한다.

javascript에서 객체를 복사한다는것은 주로 **얕은 복사**를 의미한다. 얕은 복사를 간단하게 말하면 원본과 메모리 주소를 공유하는 객체를 말하며 복사본의 값이 변경되면 원본에도 영향이 미친다.
```ts
const originObject = {
  title: '개발 뮤직 페스티벌',
  date: '2023-09-XX',
  address: {
    base: '서울특별시 송파구',
    detail: '석촌호수'
  }
}

const shallowCopy = originObject;
shallowCopy.date = '2023-11-XX'

console.log(originObject.date) // '2023-11-XX'
console.log(shallowCopy === originObject) // true
```
이처럼 얕은 복사 후에 데이터를 변경하면 원본에도 영향이 미치고 `===`을 통해 서로 같은 객체임을 알 수 있다. 같은 메모리 주소를 공유하는게 어때서? 라고 묻는다면 역할이 끝난 데이터는 메모리가 해제 되었으면 하는데 메모리 주소를 공유한다면 원본이 더 이상 사용 되고 있지 않아도 gc가 메모리 회수를 하지 않을 것이다. 이러한 데이터가 많아진다면 메모리 누수로 이어진다거나 문제가 발생할 수 있다.

그러면 새로운 객체를 생성해서 데이터를 다루면 될까? 일반적으로 새로운 객체를 생성하는 방법을 알아보자
```ts
const shallowCopy = { ...originObject }
console.l.og(shallowCopy === originObject) // false
```
위는 같은 복사 된 객체가 원본의 주소를 참조하지 않아 같은 객체냐고 하였을 때 거짓값이 나왔다. 이렇게 하면 해결이 된 것일까? 그렇지 않다. 객체 속 객체값인 address는 여전히 원본의 데이터를 향하고 있다.
```ts
const shallowCopy = { ...originObject }
console.log(shallowCopy === originObject) // false
console.log(sahllowCopy.address === originObject.address) // true
```

그럼 이제 깊은 복사를 통해 원본 데이터와 복사된 데이터를 분리해보자
단순히 새 객체에 담는데서 그치는게 아닌 중첩된 객체까지 분리해서 복사하는 방법엔 대표적인게 2가지 있다.
JSON.parse(JSON.stringify(obj))를 사용하거나 깊은 복사를 지원하는 유틸리티의 함수를 이용하는 것이다. 대표적으로는 lodash의 deepClone이 있다. 하지만 이 방법들도 완전(?)하지는 않다.
우선 JSON을 이용한 변환은 자바스크립트의 원시 타입만 지원을 한다. 객체 속에 Date나 Map, Set등이 있다면 값의 변형이 일어나고 또한 서로를 참조하는 객체가 있다면 순환 참조 오류가 일어난다.
lodash의 deepClone도 많이들 사용할 수 있으나 lodash를 전체적으로 많이 이용한다면 모를까 저 함수만을 위해 lodash를 설치해야할까?? 그럼 이도저도 완전하지 않으니 직접 만들어서 사용해야할까?

나와 같은 사람을 위해 자바스크립트에서 직접 제공하는 함수가 있다. `structuredClone`라는 함수이고 아래와 같은 설명으로 시작된다.
> 글로벌 structuredClone() 메서드는 구조화된 클론 알고리즘을 사용하여 주어진 값의 딥 클론을 생성합니다.

웹 브라우저는 모두 지원하고있으며 node에서는 17버전 이상, deno는 1.14이상부터 사용 가능하다
```ts
const deepCopy = structuredClone(originObject)
console.log(deepCopy === originObject) // false
console.log(deepCopy.address === originObject.address) // false
```
structuredClone을 사용하면 깊은복사가 되어 중첩된 객체도 다른 메모리 주소를 가지게 된다.

깊은 복사를 통한 데이터는 왜 사용할까?
당장 깊은 복사를 한다면 메모리를 더 사용 될 수가 있다. 하지만 사용하지 않는 객체가 있다면 메모리는 회수 될 것이고 복사되는 객체의 양이나 크기가 너무 많지 않다면 차이 또한 크지 않을 것이다. 그렇다면 그 때에는 깊은 복사를 통해 데이터를 안전하게 다루는게 맞을 수 있다.

결론은 개발자가 다루는 데이터가 어디까지 이용되는지 어떻게 변환되는지 잘 알고 있어야하고 상황에 맞게 사용 할 줄 알아야한다.


## link
[https://developer.mozilla.org/en-US/docs/Web/API/structuredClone](https://developer.mozilla.org/en-US/docs/Web/API/structuredClone)