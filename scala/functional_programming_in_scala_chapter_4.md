## 4.1 Objects Everywhere

> 퓨어 object-oriented 언어란 모든 value가 object라는 말인데, 그렇다면 스칼라가 퓨어 object-oriented language인가?

일단은 primitive types라는 예외가 있기 때문에 아니라 할 수 있다. 그래서 pure object-oriented 하도록 만들기 위해서 scala.Boolean 대신 Boolean 클래스를 정의한다(자바의 래핑클래스(Integer 등)처럼)

Boolean 클래스에서는 실제 스칼라 Boolean으로 사용할 수 있었던 연산을 모두 재정의해준다. 아래는 '<' 함수를 정의한 예제이다.
ifThenElse는 if(cond) f1 else f2과 같다(여기서 f1, f2는 ifThenElse의 파라미터)
```
claass Boolean {
	...
	def < (x: Boolean): Boolean = ifThenElse(false, x)
 }
```


## 4.2 Functions as Objects



## 4.3 Subtyping and Generics

스칼라 언어에서 다형성을 표현하는 두가지 방법은 subtyping과 generic 이다.

#### Type Bounds
> takes an IntSet
> returns the IntSet itself if all tis elements are positive
> throws an exception otherwise

위의 세가지 조건을 충족시킬 수 있는 함수를 생각해보자.

```
def assertAllPos(s: IntSet): IntSet
```
대부분의 경우는 위의 함수로 충분하지만 정확히 하자면 다음과 같이 쓸수 있다.
```
def assertAllPos[S <: IntSet](r: S): S = ...
```
"S <: IntSet"을 type parameter S의 upper bound라고 한다. 이것은 S가 반드시 IntSet의 subType(또는 자신)이어야 한다는 말과 같다. 
반대로 "S :> T"는 S가 T의 superType이거나 T가 S의 subType이라는 말이다. 이를 lower Bounds 라고 한다.
```
[S >: NonEmpty]
```
위에서 말했듯이 위의 의미는 S가 NonEmpty 클래스의 supertype인데, S는 NonEmpty의 모든 base 클래스(자신 포함)가 해당된다. 여기서 S는 NonEmpty, IntSet, AnyRef, Any가 될 수 있다.

마지막은 Mixed Bound
```
[S >: NonEmpty <: IntSet]
```
이것의 의미는 S가 NonEmtpy와 IntSet 타입 사이의 모든 타입이 될 수 있다는 말과 같다.

#### Covariance
서브클래스의 인스턴스 컬렉션을 상위클래스의 컬렉션으로 보내는 것을 Covariance라고 한다.
```
NonEmpty <: IntSet
// 위가 성립된다면 아래도 성립
List[NonEmpty] <: List[IntSet]
```

#### Arrays in Scala
```
T[] 		// Java
Array[T] 	// Scala

// Covariance에 의해 아래가 성립
NonEmpty[] <: IntSet[]		 		// Java
Array[NonEmpty] <: Array[IntSet]	// Scala
``` 
자바의 Array Typing에는 타입과 관련된 아래의 문제가 있다.
```
NonEmpty[] a = new NonEmpty[]{new NonEmpty(1, Empty, Empty)}
IntSet[] b = a
b[0] = Empty
NonEmpty s = a[0]
```
a는 NonEmpty 타입의 Array를 가리키는 포인터이다. 두번째 줄에서 IntSet Array b 에 a를 대입하였다. b가 실제로 카리키는 대상은 NonEmpty List지만, covariance 규칙에 의해 상위 타입의 컬렉션이 하위 타입의 컬렉션을 대신할 수 있다. 세번째 줄에서 b의 첫번째 item에 Empty 클래스를 대입하였다. 마지막으로 a의 첫번째 item을 NonEmpty 타입의 s에 대입하였다. b와 a 는 실제로 가리키는 대상이 같기 때문에 세번째 줄에서 b[0]에 들어간 Empty는 a[0]에서도 동일하게 작동한다. 그런데 마지막 줄에서 Empty 타입의 item 을 NonEmpty 타입에 할당하기 때문에 런타임 에러가 발생한다.

```
// in scala
val a: Array[NonEmpty] = Array(new NonEmpty(1, Empty, Empty))
val b: Array[IntSet] = a
b(0) = Empty
val s: NonEmpty = a(0)
```
스칼라의 경우에는 두번째 줄에서 컴파일 에러가 난다. 그 이유는 스칼라의 Array는 covariant하지 않기 때문이다.


## 4.4 Variance

C[T]에서 A <: B인 경우 다음이 성립한다.
B가 A의 수퍼타입이면서 C[B]가 C[A]의 수퍼타입인 경우에는 covariant, C[A]가 C[B]의 수퍼타입이면 contravariant

* C[A] <: C[B] 이면 C는 covariant (class C[+A])
* C[A] >: C[B] 이면 C는 contravariant (class C[-A])
* C[A]와 C[B] 둘다 다른것의 서브타입이 아니면 C는 nonvariant (class C[A])

다음의 두 타입중 어떤 타입이 수퍼타입이고, 어떤 타입이 서브타입인가?
결과를 얘기하자면, B가 A의 수퍼타입이다. 함수의 파라미터는 contravariant하고 함수의 리턴값은 covariant 하기 때문에 A <: B가 참이다.
```
type A = IntSet => NonEmpty
type B = NonEmpty => IntSet
```
위의 내용을 요약하면 아래와 같다.
```
If A2 <: A1 and B1 <: B2, then
  A1 => B1  <:  A2 => B2
```

#### Variance Checks


## 4.5 Decomposition

다음과 같은 class 구조가 있다고 하자
```
trait Expr {
  // classification
  def isNumber: Boolean
  def isSum: Boolean
  // accessor
  def numValue: Int
  def leftOp: Expr
  def rightOp: Expr
}

class Number(n: Int) extends Expr {
  def isNumber: Boolean = true
  def isSum: Boolean = false
  def numValue: Int = n
  def leftOp: Expr = throw new Error("Number.leftOp")
  def rightOp: Expr = throw new Error("Number.rightOp")
}

class Sum(e1: Expr, e2: Expr) extends Expr {
  def isNumber: Boolean = false
  def isSum: Boolean = true
  def numValue: Int = throw new Error("Sum.numValue")
  def rightOp: Expr = e1
  def leftOp: Expr = e2
}
```
무척 쓸모 없어 보이는 메서드들이 여럿 보인다. 일단은 더 나은 코드를 설명하기 위한 단계이므로 참고 살펴보자.
그리고 위의 클래스 구조를 evaluation하는 간단한 인터프리터 함수인 eval이 다음과 같다
```
def eval(e: Expr): Int = {
  if (e.isNumber) e.numValue
  else if (e.isSum) eval(e.leftOp) + eval(e.rightOp)
  else throw new Error("Unknown expression " + e)
}
```
이때 다음과 같은 코드가 있다면, 우선 eval 함수가 실행되면서 e가 어떤 타입인지 찾기 위해 classification method인 isSum으로 Sum 타입인지 찾을 것이다. 그리고 그 안의 두 인자가 각각 Number 이므로 또다시 eval 함수 내에서 isNumber에 의해 Number 타입인지 찾을 수 있을 것이다. 뭔가 비효율적으로 보인다.
```
eval(Sum(Number(1), Number(2))) = 3
```

여기서 만약에 아래와 같은 두개의 클래스가 추가 된다면 어떨까?
```
class Prod(e1: Expr, e2: Expr) extends Expr   // e1 * e2
class Var(x: String) extends Expr 			  // Variable 'x'
```
위의 두 클래스는 Number나 Sum과 마찬가지로 Expr을 상속받으므로 trait Expr의 메서드를 모두 구현해야한다. 그리고 isNum, isSum 과 같은 클래스 타입을 찾기 위한 메서드를 2개(isVar, isProd)더 추가해야 할 것이다. 또 var 값을 가져오기 위한 name 메서드도 추가되서 총 3개가 추가된다. 위의 구조에서만 15개의 메서드가 있는데, 단 2개의 클래스만 추가하더라도 더 필요한 메서드가 25개(Expr에 3개, Number에 3개, Sum에 3개, 그리고 새로운 클래스에 각각 8개)나 된다. 이건좀 아닌거 같다.

메서드를 좀 줄여보자
자바에서 사용하는 type test, type cast 메서드를 이용한다.
```
Scala  				  Java
x.isInstanceOf[T]  	  x instanceof T    // type test
x.asInstanceOf[T]     (T) x				// type cast
```

평가함수인 eval을 조금 고쳐보자.
```
def eval(e: Expr): Int = {
  if (e.isInstanceOf[Number])
    e.asInstanceOf[Number].numValue
  else if (e.isInstanceOf[Sum])
    eval(e.asInstanceOf[Sum].leftOp) + eval(e.asInstanceOf[Sum].rightOp)
  else throw new Error("Unknown expression " + e)
}
```
자바에서 사용하는 타입 test 함수인 instanceof와 타입 캐스팅 하는 방법을 적용하였다. 스칼라에서는 각각의 방법을 함수로 만들어 두었다. 이 방법을 사용하면 위에서 보았던 classification 메서드(isNum, inSum)를 사용할 필요가 없다. 대신에 타입 체크 및 캐스팅 함수가 low-level 함수이기 때문에 불안정한다는 단점이 있다. 

Object-Oriented Decomposition을 이용한 또다른 해법을 살펴보자
```
trait Expr {
  def eval: Int
}

class Number(n: Int) extends Expr {
  def eval: Int = n
}

class Sum(e1: Expr, e2: Expr) extends Expr {
  def eval: Int = e1.eval + e2.eval
}
```
각각의 클래스에 eval 함수를 구현하였다. 각 클래스에 맞게 구현되기 때문에 accessor 함수들도 불필요하다. 이제 많이 깔끔해졌다. 하지만 문제는 여전히 있다. rait에 하나의 메서드가 추가된다면, 나머지 클래스에 모두 구현해야한다는 점이다. 또다른 문제가 있다.
```
a * b + a * c = a * (b + 3)
```
위와 같이 축약하기 어렵다. 왜냐하면 이것은 non-local simplification이기 때문이다. 이것은 single object의 메서드로 캡슐화 할 수 없다. sub-tree를 모두 테스트하고 접근해야하는 문제가 있다. 


## 4.6 Pattern Matching


## 4.7 Lists

가장 기본적인 리스트 형태는 아래와 같이 정의할 수 있다.
```
// List(X1, ..., Xn)
val fruit: List[String] = List("Apples", "oranges", "pears")
val nums: List[Int] = List(1, 2, 3, 4)
val diag3: List[List[Int]] = List(List(1, 0, 0), List(0, 1, 0), List(0, 0, 1))
val empty: List[Nothing] = List()
```

스칼라에서 List와 Array는 중요한 두가지 차이가 있다.
* List are immutable - the elements of a list cannot be changed
* Lists are recursive, while arrays are flat



