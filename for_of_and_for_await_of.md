Node.js로 개발을 하다보면 map, filter, reduce외에 for...of 혹은 for await...of를 사용하기도 하는데 사실 별 생각없이 for await...of로 사용할 때가 많았다. 그래서 두 구문의 차이를 알아보고자 한다.

# for...of
for...of를 실행하면 Symbol.Iterator를 호출하므로 사용하려면 해당 데이터가 `[Symbol.Iterator]()` 속성을 가지고 있어야 한다.
JavaScript에서 해당 속성을 기본적으로 포함하는 데이터 타입은 Array, Map, Set, String이 있다

# for await...of
for await...of를 실행하면 Symbol.AsyncIterator가 1회 호출되고 이후엔 next()의 반환값을 따른다. Promise 객체가 있다면 값을 꺼내서 사용한다. Symbol.AsyncIterator가 없다면 Symbol.Iterator를 사용하는듯하다. JavaScript Array, Map, Set, String은 Symbol.AsyncIterator가 없지만 for await...of를 사용할 수 있다.

Symbol.AsyncIterator가 있지 않다면 Promise 객체 외엔 차이가 없는 반복문법이다.