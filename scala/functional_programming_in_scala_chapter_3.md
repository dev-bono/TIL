## 3.1 Class Hierachies
#### abstract class (추상 클래스)
추상 클래스는 abstract 키워드를 class 앞에 붙임으로써 정의할 수 있다. 추상 클래스는 구현체가 없는 추상함수를 1개 이상 멤버로 가진다. 스칼라의 추상클래스는 자바의 그것과는 달리 new로 선언할 수가 없다(자바의 추상클래스는 new 할당 가능, 대신 인터페이스는 new 할당이 불가함).
```
abstract class IntSet {
  def incl(x: Int): IntSet
  def contains(x: Int): Boolean
}

class Empty extends IntSet {
  def incl(x: Int): IntSet = new NonEmpty(x, new Empty, new Empty)
  def contains(x: Int): Boolean = false
  override def toString = "."
}

class NonEmpty(elem: Int, left: IntSet, right: IntSet) extends IntSet {
  def incl(x: Int): IntSet =
    if (x < elem) new NonEmpty(elem, left incl x, right)
    else if (x > elem) new NonEmpty(elem, left, right incl x)
    else this

  def contains(x: Int): Boolean =
    if (x < elem) left contains x
    else if (x > elem) right contains x
    else true

  override def toString = "{" + left + elem + right + "}"
}

val t1 = new NonEmpty(3, new Empty, new Empty)
// t1: NonEmpty = {.3.}
val t2 = t1 incl 4
//t1: NonEmpty = {.3.}
```
위에서 IntSet은 Empty와 NonEmpty클래스의 base class 이다. 구현체가 없는 incl 메소드와 contains 메소드는 각각 Empty와 NonEmpty 클래스에서 구현하게 된다.
```
abstract class Base {
	def foo = 1
	def bar: Int
}

class Sub extends Base {
	override def foo = 2
	def bar = 3
}
```
Base 클래스의 foo 메서드는 구현체가 있고 bar 메서드는 구현체가 없다. 구현체가 없는 bar 메서드 같은 경우에는 Sub 클래스에서 바로 구현해주면 되지만, 구현체가 없는 foo 메서드는 반드시 메서드 앞에 override 키워드를 붙여서 재정의 해야한다.(참고로 bar 메서드 앞에 override 키워드를 붙이는 것은 optional)
자바같은 경우 메소드 오버라이드 할때는 별도의 modifier를 지정해주지 않는다. scala에서 이렇게 하는 이유는 override 키워드가 없는 경우에는 실제로 어떤 메서드가 오버라이드되는지 모르기 때문에 잘못된 오버라이드로 인해 Base 클래스의 메서드가 영향을 받을 수도 있기 때문이다.

#### object
여러개의 instance를 만들 필요 없는 클래스 같은 경우에는 간단히 object를 이용한다. object 정의는 class 정의와 동일하지만 new 키워드로 여러개의 instance를 만들 수 없고 단지 하나의 object로 작동한다(singleton object).
오직 Empty 클래스에만 object를 쓸 수 있는데, 그 이유는 NonEmpty 클래스는 몇개의 파라미터를 가지고 있다. object는 value라 했으므로, object가 정의됨과 동시에 모두 평가되어야하므로 파라미터로 인한 값의 변화를 취할수 없다.
evaluation 측면에서 보자면, 위에서도 얘기 했듯이  object 자체는 이미 value기 때문에 아래와 같이 Empty로 바뀐 부분이 별도의 evalutaion 과정이 불필요하다.  

#### exercise - union 함수 구현
```
abstract class IntSet {
  def incl(x: Int): IntSet
  def contains(x: Int): Boolean
  def union(other: IntSet): IntSet
}

object Empty extends IntSet {
  ...
  def union(other: IntSet) = other
}

class NonEmpty(elem: Int, left: IntSet, right: IntSet) extends IntSet {
  ...
  def union(other: IntSet) =
    ((left union right) union other) incl elem
}
```
으아... 어렵다
대략 짐작해보자면, (left union right) 부분이 실행되면 left가 현재 node가 되고 right가 other가 된다. 계속해서 깊게 내려가다 보면, 언젠가 leaf node가 나오게 되는데 leaf node의 left와 right는 모두 Empty이므로 결국 incl elem에 의해 자기자신이 리턴된다. 그리고 한칸씩 올라오면서 left의 elem을 떼다가 right 트리에 include를 시도하는 과정을 거친다. 그 과정이 계속되면, 결국은 하나의 binary tree가 만들어진다.

#### 동적 바인딩
메소드를 포함하는 object 타입은 런타임에 메소드가 실행된다.
 
