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
Base 클래스의 foo 메서드는 구현체가 있고 bar 메서드는 구현체가 없다. 구현체가 없는 bar 메서드 같은 경우에는 Sub 클래스에서 바로 구현해주면 되지만, 구현체가 없는 foo 메서드는 반드시 메서드 앞에 override 키워드를 붙여서 재정의 해야한다.