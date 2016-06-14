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
위의 함수들에서 a, b 파라미터는 sumInts(), sum()에서 반복된다. 이를 제거하는 방법
