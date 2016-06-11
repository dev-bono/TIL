## 1.1 Programming Paradigms

## 1.2 Elements of Programming
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

## 1.3 Evaluation Strategies and Termination
```
def first(x: Int, y: Int) = x
first(1, loop)
```
first 함수를 호출하게 되면 CBN같은 경우는 인자를 해석하지 않고 바로 1을 출력하겠지만, CBV인 경우에는 loop 인자를 해석하기 위해서 무한루프에 빠지게 된다.

* 스칼라는 기본적으로 CBV를 사용
* 함수 파라미터가 =>로 시작하면 CBN 사용

## 1.4 Conditionals and Value Definitions
```
def loop: Boolean = loop
def x = loop 
val x = loop // infinite loop
```
def는 우측의 loop가 해석되지 않는다. 반면에 val(value)는 우측의 코드를 해석하기 때문에 위와 같은 코드의 경우 무한루프에 빠지게 된다.

```
and(x,y) == x && y
def and(x: Boolean, y: Boolean)
  if (x) y else false
// and(x, loop)와 같은 문제가 발생할 수 있으므로, 아래와 같이 변경
def and(x: Boolean, y: => Boolean)
  if (x) y else false
```  
* 그런데 왜 y만 CBN으로 변경해 줬을까? and(loop, b)하면 어떻게될까?

## 1.5 Example: square roots with Newton's method
뉴튼 메소드를 이용해서 제곱근을 구하는 예제를 작성해본다.
* 한가지 주의할점은 스칼라에서 recursive(재귀) 함수인 경우에는 반드시 return 타입을 정해주어야 한다.
```
def abs(x: Double) = if (x < 0) -x else x

def sqrtIter(guess: Double, x: Double): Double = 
  if (isGoodEnough(guess, x)) guess
  else sqrtIter(improve(guess, x), x)

def isGoodEnough(guess: Double, x: Double) = 
  (guess + x / guess) / 2

def sqrt(x: Double) = sqrtIter(1.0, x)

sqrt(2) // res1: Double = 1.4142156862745097
```
