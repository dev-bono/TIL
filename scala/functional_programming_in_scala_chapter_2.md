## 2.1 Higher-Order Functions
함수형 언어에서는 function이 first-class value이다. 이 말은 특정 함수가 다른 함수의 파라미터나 결과로써 return 될 수 있다는 것이다(higher order functions).
```
def sum(f: Int => Int, a: Int, b: Int): Int = 
  if (a > b) 0
  else f(a) + sum(f, a+1, b) 

def sumInts(a: Int, b: Int)       = sum(id, a, b)
def sumCubes(a: Int, b: Int)      = sum(cube, a, b)
def sumFactorials(a: Int, b: Int) = sum(fact, a, b)

def id(x: Int): Int = x
def cube(x: Int): Int = x * x * x
def fact(x: Int): Int = if (x == 0) 1 else x * fact(x -1)
```
sum() 함수의 첫번째 파라미터로 f라는 임의의 함수가 들어가 있다.
이는 Int 파라미터를 받아서 Int 타입을 리턴하는 함수라면 이 파라미터 위치에 적합하다라는 뜻이다. 즉, 어떠한 함수가 됐든간에 저 유형만 유지하면 된다. 재사용 가능하다는 말이다. 실제로 sumInts(), sumCubes(), sumFactorials() 함수는 모두 sum()함수를 호출한다. sum()함수의 첫번째 파라미터는 각각 id(), cube(), fact()함수를 호출하게 되는데 세 함수 모두 파라미터와 리턴 타입이 Int이므로 sum()함수의 파라미터로 적합하다.

#### 익명함수
```
(x: Int, y:Int) => x + y
```
익명함수는 보통 함수를 선언하는 방식과는 달리 이름이 없다.
대신 파라미터 타입과 body만 존재한다. 
```
def sumCubes(a: Int, b: Int) = sum(x => x * x * x, a, b)
```
위에서 보았던 sumCubes의 익명함수 버전이다.
cube() 함수가 재사용되지 않는다고 가정한다면 매우 심플하기 때문에 따로 함수를 선언하기 보다는 sum() 파라미터에 익명함수로 직접 선언할수 있다.

```
// linear recursion, a와 b의 차이가 커지면 stackOverFlow 익셉션이 발생할 수 있다.
def sum(f: Int => Int, a: Int, b: Int): Int = 
  if (a > b) 0
  else f(a) + sum(f, a+1, b) 

// tail-recursive version
def sum(f: Int => Int, a: Int, b: Int): Int = {
  def loop(a: Int, acc: Int): Int = {
    if(a > b) acc
    else loop(a+1, f(a)+acc)
  }
  loop(a, 0)	
}
```

## 2.2 Currying
```
def sumInts(a: Int, b: Int)       = sum(x => x, a, b)
def sumCubes(a: Int, b: Int)      = sum(x => x*x*x, a, b)
def sumFactorials(a: Int, b: Int) = sum(fact, a, b)
```
위의 함수들에서 a, b 파라미터는 sumInts(), sum()에서 반복된다. 이를 제거하여 함수를 더 간소하게 만드는 방법은 무엇일까?
우선 아래의 함수를 살펴보자.
```
def sum(f: Int => Int): (Int, Int) => Int = {
  def sumF(a: Int, b: Int): Int = {
    if (a > b) 0
    else f(a) + sumF(a+1, b)
  }
  sumF
}
```
기존에 봤던 sum() 함수와 조금 다르게 생겼다. 우선 sum() 함수의 파라미터가 f() 하나로 줄었고, sum() 함수는 (Int, Int) => Int 와 같은 타입의 익명함수를 리턴하고 있다.
sumF를 호출하는 부분이 조금 이해하기 어려운데, 위의 내부함수 sumF()를 자세히 보면 익명함수를 설명했을 때의 형태와 같다는 것을 알 수 있다.
```
(x: Int, y: Int): Int => x * y
// 이 익명함수는 아래와 같이 표현가능하다.
def f(x: Int): Int = x * y; f
```

위에서 썼던 sumInts(), sumCubes(), sumFactorials() 함수들을 가져오자. 조금 다르다. sum() 함수의 파라미터가 f 하나로 줄면서 아래의 각 함수들도 sum() 호출부의 인자가 익명함수 하나만 남았다.
```
def sumInts(a: Int, b: Int) = sum(x => x)
def sumCubes(a: Int, b: Int) = sum(x => x*x*x)
def sumFactorials(a: Int, b: Int) = sum(fact)
```
이제 sum() 함수는 함수를 리턴하고 있으므로, sumCubes(1, 10)를 호출하면
```
sumCubes(1, 10)
// res0: (Int, Int) => Int = <function2>
```
이는 또한 이렇게 바꿔 쓸수 있다.
```
sum (cube) (1, 10) // multiple parameters
(sum (cube)) (1, 10)  
```
위의 multiple parameters를 적용하면 sum() 함수를 더 짧게 작성할 수 있다.
```
def sum(f: Int => Int)(a: Int, b: Int): Int =
  if (a > b) 0 else f(a) + sum(f)(a+1, b)
```
여러개의 파라미터 list를 가진 함수 f에서 파라미터list의 개수가 1보다 큰 경우 마지막 파라미터 list를 f 함수에서 제거하면 이 마지막 파라미터 list를 파라미터로 가지는 또다른 함수 g를 만들 수 있다. 물론 이 g 함수는 f 함수의 리턴함수가 된다. 식으로 나타내면 다음과 같다. 참고로 각각의 식은 모두 동일하다
```
def f(arg1)(arg2)(arg3)(arg4) = E
def f(arg1)(arg2)(arg3) = {def g(arg4) = E;g} // arg4를 이용해 g함수 생성
def f(arg1)(arg2)(arg3) = (arg4 => E) // g를 익명함수로 변경
def f = (arg1 => (arg2 => (arg3 => (arg4 => E)))) // 파라미터를 하나씩 우측으로 전달함
```
이와같은 형태의 함수정의를 curring 이라 부른다. 

###Excercise
1. Write a product function that calculates the product of the values of a function for the points on a given interval
```
def product(f: Int => Int)(a: Int, b: Int): Int = {
  if (a > b) 1 else f(a) * product (f)(a+1, b)
}
product(x => x*x)(3,4)
```
1. Write factorial in terms of product.
```
def fact(n: Int): Int = product(x => x)(1, n)
fact(4)
```
1. Can you write a more general funciton, which generalizes both sum and product?
```
def mapReduce(f: Int => Int, combine: (Int, Int) => Int, zero: Int)(a: Int, b: Int): Int = {
  if (a > b) zero
  else combine(f(a), mapReduce(f, combine, zero)(a+1, b))
}

def productForMapReduce(f: Int => Int)(a: Int, b: Int): Int = mapReduce(f, (x, y) => x*y, 1)(a, b)
productForMapReduce(x => x*x)(3, 4)
```



