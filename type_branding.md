# 명목적 타입 구현
- 개발을 하다보면 typescript의 원시 타입(number, string, boolean, null, undefined, symbol, object)로는 아쉬운 순간이 있다.
- 어떤 입력 폼에서 기본 주소와 상세 주소를 입력 받고 최종적으로 주소가 합쳐지는 코드가 있다고 하자. 두 개의 주소를 합치는것은 간단한 일이지만 사람인 이상 실수하고 있고 두 값 모두 string 타입의 합이기 때문에 자세히 보지 않으면 코드리뷰에서도 놓칠 수 있는 부분이기도 하다.
- 이럴 때는 우리가 <strong>타입</strong>스크립트를 쓰고 있으니 컴파일 단계에서 타입으로 구분되어지면 좋겠다는 생각이 들 것이다.

```ts
const addr1 = '서울특별시';
const addr2 = '송파구 롯데타워';

const combineAddr = (addr1: string, addr2: string) => `${addr1} ${addr2}`

const address = combineAddr(addr1, addr1) // '서울특별시 서울특별시'
```
- combineAddr 함수는 두 인자 모두 string으로 받으면 되니 위와 같은 이상한 주소가 나와도 오류라고 판별 할 수 없고 만약 합쳐진 address가 데이터베이스에 입력이 되고나서야 잘못 되었음을 알 수 있을 것이다.
- 아래와 같이 구분되는 주소의 타입을 생성하여 안전하게 주소를 합성해보자.
```ts
type Brand<K, V> = V & { __brand: K };
type BaseAddr = Brand<'BaseAddress', string>;
type DetailAddr = Brand<'DetailAddress', string>;

const convertAddr = <T extends BaseAddr | DetailAddr>(addr: string): T => {
  return addr as T;
}

const addr1 = convertAddr<BaseAddr>('서울특별시');
const addr2 = convertAddr<DetailAddr>('송파구 롯데타워');

const combineAddr = (addr1: BaseAddr, addr2: DetailAddr) => `${addr1} ${addr2}`

const wrongAddress = combineAddr(addr1, addr1) // <-- compile 이전 잘못 된 타입으로 오류를 보여줄 것이다.

const address = combineAddr(addr1, addr2); // '서울특별시 송파구 롯데타워'
```

> 간단하지만 사람이 실수하여 발생 할 수 있는 오류를 줄여가며 개발하자