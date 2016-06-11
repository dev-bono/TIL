## 1.1

## 1.2
* call by value : 인자가 먼저 평가되는 방식
* call by name : 인자가 나중에 평가 되는 방식 
```
sumOfSquares(3, 2+2)

sumOfSquares(3, 4) // call by value
sumOfSquares(3, 2+2) // call by name
```
위와 같은 함수가 있을때, 2+2가 먼저 계산되어 인자가 4로 evaluation 된 후 reduce 되면 call by value, 2+2가 이름(name) 그대로 reduce 되면 call by name이라 할 수 있다. call by value의 장점은 모든 함수의 인자가 한번만 해석된다는 것이다. 반면에 call by name has the advantage that a function argument is not evaluated if the corresponding parameter is unused in the evaluation of the function body.

다음의 예를 보면 이해가 간다.

```
call by name 
test(3+4,2*4)
// (3+4) * (3+4)
// 7 * (3+4)
// 7 * 7
// 49

call by value
test(3+4,2*4)
// test(7,2*4)
// test(7,8)
// 7 * 7
// 49
```

## 1.3
```
def first(x: Int, y: Int) = x
first(1, loop)
```
first 함수를 호출하게 되면 CBN같은 경우는 인자를 해석하지 않고 바로 1을 출력하겠지만, CBV인 경우에는 loop 인자를 해석하기 위해서 무한루프에 빠지게 된다.
