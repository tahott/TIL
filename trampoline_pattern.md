# Trampoline pattern
꼬리재귀최적화를 지원하지 않는 언어에서 꼬리재귀를 사용할 수 있게 해주는 트램폴린 패턴에 대해 정리한다

### 꼬리 재귀(tail recursion)
함수의 마지막에 `자기 자신을 호출`하는 형태를 말하며 이렇게 하면 스택이 쌓이지 않아 재귀에 대한 호출이 많아져도 스택오버플로우가 발생하지 않게 된다.
```ts
function fatorialTailRecursion(n: number, total: number) {
  return n < 2
    ? total
    : fatorialTailRecursion(n - 1, total * n);
}
```
```
|-----------------------------|
|fn(5)|fn(4)|fn(3)|fn(2)|fn(1)|
```
스택이 쌓이지 않기에 많은 호출이 가능한 구조이다.

## 언어에서 꼬리 재귀를 지원하지 않거나 꼬리재귀가 아닌 형태의 재귀라면?
```
|----------------------------------------------------|
|                       |fn(1)                       |
|                 |fn(2)|fn(2)|fn(2)                 |
|           |fn(3)|fn(3)|fn(3)|fn(3)|fn(3)           |
|     |fn(4)|fn(4)|fn(4)|fn(4)|fn(4)|fn(4)|fn(4)     |
|fn(5)|fn(5)|fn(5)|fn(5)|fn(5)|fn(5)|fn(5)|fn(5)|fn(5)
```
호출이 많아질수록 스택이 쌓이게 되고 일정 스택 이상이 쌓이면 스택 오버플로우가 발생한다.

### 꼬리재귀최적화를 지원하지 않는 환경에서 트램폴린 패턴을 사용하자
```ts
type Thunk<T> = () => T | Thunk<T>;
function trampoline<T>(fn: Thunk<T>): T {
  let result = fn();
  while (result instanceof Function) {
    result = result();
  }

  return result;
}

trampoline(fatorialTailRecursion(10000))
```
트램폴린 함수는 반환값이 T 또는 자기 자신을 호출하는 함수를 인자값으로 사용하여 반환 된 타입이 함수면 반복적으로 호출하여 최종 결과를 얻는 역할을 한다.

```
# trampoline(factorialTailRecursion(3))
|-----------------------------------------|
|     |fn(3)|     |fn(2)|     |fn(1)|     |
|t(fn)|t(fn)|t(fn)|t(fn)|t(fn)|t(fn)|t(fn)|
```
꼬리재귀가 동작할 때 보다 추가 동작들이 필요하지만 스택이 쌓이는 구조는 아니어서 스택 오버플로우가 발생할 일은 없게된다.

꼬리재귀최적화를 지원한다면 사용할 일은 없겠지만 그런 환경이 아니고 많은 스택이 쌓일 수 있는 프로세스가 동작하는 상황이라면 트램폴린 패턴을 적용하여 해결을 고려해볼 수 있을 것 같다.