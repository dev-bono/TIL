## 6.1 Other Collections

이제껏 리스트에 대해서 알아보았다. 이번 챕터에서는 리스트 외에도 스칼라의 기본적인 컬렉션(Seq, Set, Map)에 대해 알아보도록 하자. 

#### Vector
리스트의 경우에는 첫번째 element에 접근하는게 마지막 element에 접근하는 것보다 훨씬 빠르다. 리스트의 구조가 트리 형태로 구성되어 있고 우측으로 갈수록 트리의 깊이가 깊어지기 때문에 그렇다. 이에 반해 Vector는 리스트 보다 access pattern이 훨씬 균형잡혀 있다.

벡터는 최초에 2^5(32개)의 원소를 가지는 array가 만들어 진다. 32개가 넘는 원소가 필요하게 되면, 그다음은 32개를 추가로 만드는 것이 아니라, 2^5 * 2^5개, 그러니깐 총 2^10(1024개)의 원소가 들어갈 수 있는 리스트가 추가로 만들어 지는 것이다. 왜 이런식으로 커질까? 원소가 32개보다 커지면, 처음 만들어진 32개짜리 벡터는 각각의 32개짜리 원소를 가지는 벡터의 포인터가 되고 그 아래로 각각 벡터가 만들어진다. 즉, 처음 32개짜리 원소에서 1024개를 담을 수 있는 벡터로 바뀌게 된다. 그렇기 때문에 어떤 값을 찾는데 log32(N) 시간만 소요된다. 리스트보다 훨 낫다. 

또 다른 장점으로는 map, for, filter, fold와 같은 연산(bulk operations)이 빠르다는 것이다. 왜냐하면 벡터는 리스트와 달리 원소가 32개가 하나의 묶음이다보니 a single cache line에 위치하기 때문에 접근이 훨씬 빠를것이기 때문이다. 즉 list는 vector에 비해서 locality가 나쁘다고 할 수 있다.

vector가 이렇게 좋다는데, list를 써야하는 이유는 무엇일까? recursive data structure의 head를 취하고 나머지를 다시 연산하는 모델 안에서는 list가 훨씬 쉽다. 왜냐하면 한 노드에 하나의 아이템만 존재하기 때문에 head를 취하기가 훨씬 쉽기 때문이다. 하지만 vector의 경우 depth가 1일때는 몰라도 depth가 1만 더 증가해도 훨씬 복잡해진다.

list의 concat 연산 (::)은 vector에서 다음과 같이 쓰인다.
```
x +: xs 		// xs 앞에 x를 포함하는 새로운 벡터를 붙인다.
xs :+ x 		// xs 뒤에 x를 포함하는 새로운 벡터를 붙인다.
```

벡터 또한 immutable하기 때문에 위의 연산을 처리하기 위해서는 기존의 벡터를 새롭게 만들어지는 벡터와 함께 새로운 포인터에 연결해야한다. root까지 새로운 포인터로 연결이 되고 나면 연산이 완료된다.


#### Collection Hierachy	

스칼라의 collection은 크게 세가지로 나뉘는데, 하나는 지금껏 살펴봤던 List와 Vector가 속하는 Seq, 나머지는 Set과 Map이다. 

![Collectino Hierachy](http://docs.scala-lang.org/resources/images/collections.immutable.png)

Array와 String는 점선으로 되어있는데, 이들도 Seq로써 아래와 같이 똑같이 동작한다. 다만 자바에서 가져온 타입이기 때문에 앞으로 어떻게 될지(Scala.sequence.String 같은걸 누군가 만들지도 모르므로..) 몰라 점선으로 연결해놓은거 같다.

```
// Array
val xs = Array(1, 2, 3, 44)
xs map (x => x * 2)
// res0 : Array[Int] = Array(2, 4, 6, 88)

// String
val s = "Hello World"
s filter (c => c.isUpper)
// res1: String = HW
```

#### Range

Range는 심플한 seq 타입이다. 사용법 또한 매우 간단하다.
```
val r: Range = 1 until 5		// 1, 2, 3, 4
val s: Range = 1 to 5			// 1, 2, 3, 4, 5
1 to 10 by 3					// 1, 4, 7, 10
6 to 1 by -2					// 6, 4, 2
```

#### Some more Sequence Operations
* xs exists p 		: p 함수의 조건을 만족하는 원소가 있는지
* xs forall p 		: 모든 원소가 p 함수의 조건을 만족하는지
* xs zip ys			: 두 collection의 원소를 pair로 가지는 새로운 collection 생성, 타입은 xs
* xs.unzip 			: zip 형태의 collection을 분해해서 List의 tuples을 만듦, Array는 안되는군각 요소(xy: pair)의 첫번째와 두번째 요소를 곱한다음 모든값을 sum 해준다.

* xs.flatMap f  	: 모든 element에 collection-valued functino f를 적용  
* xs.sum       		: numeric collection의 요소들의 합
* xs.product 		: numeric collection의 요소들의 곱
* xs.max 			: 최고값
* xs.min 			: 최소값

 
 #### Scalar Product

 ```
 val a: Vector[Double] = Vector(1.0, 2.0, 3.0)
 val b: Vector[Double] = Vector(3.0, 4.0, 5.0)
  
 def scalarProduct(xs: Vector[Double], ys: Vector[Double]): Double =
   (xs zip ys).map(xy => xy._1 * xy._2).sum

 scalarProduct(a, b)

 def scalarProduct2(xs: Vector[Double], ys: Vector[Double]): Double =
   (xs zip ys).map{ case (x, y) => x * y }.sum

 scalarProduct2(a, b)
 ```

두 벡터의 각 요소끼리 곱한다음 모든 값을 합하는 함수이다. 우선 xs와 ys를 zip으로 묶은 다음 각 요소(xy: pair)의 첫번째와 두번째 요소를 곱한다음 모든값을 sum 해준다. map 안의 함수는 case문으로 대체할 수 있다.
```
{ case p1 => e1 ... case pn => en}

// 위와 동일
x => x match { case p1 => e1 ... case pn => en }
```

#### isPrime
```
def isPrime(n: Int): Boolean = (2 until n) forall (x => (n % x) != 0)
```
2부터 n 전까지의 수가 모두 n으로 나누어 떨어지지 않으면 n을 prime number(소수)라 할 수 있다.



## 6.2 Combinatorial Search and For-Expressions

양수 n이 있고, 또다른 양수 i와 j가 1 <= j < i < n을 만족하고 i + j가 소수라는 조건이 있다. 
코드로 나타내면 다음과 같다.
```
val n = 7
(1 until n) map (i => (1 until i) map (j => (i, j)))

// result
res0: scala.collection.immutable.IndexedSeq[scala.collection.immutable.IndexedSeq[(Int, Int)]] = Vector(Vector(), Vector((2,1)), Vector((3,1), (3,2)), Vector((4,1), (4,2), (4,3)), Vector((5,1), (5,2), (5,3), (5,4)), Vector((6,1), (6,2), (6,3), (6,4), (6,5)))
```

음.. 분명 Range 타입을 사용하였는데 결과는 Vector of Vectores가 나왔다.
Range는 Seq의 하위 타입이지만 사실 그 중간에 IndexedSeq라는 타입이 존재한다. 결과값으로 pair가 나왔으므로, Range 타입에는 결과값을 담을 수 없다. 그래서 일단 바로 위의 상위 타입인 IndexedSeq 타입 중에서 pair를 담을 가장 적합한 타입인 Vector가 선택되어 나타난것이다.

원래 우리가 찾으려고 했던 결과는 Vector의 Vector가 아니라 pair를 하나의 single list에 담아져야 한다. 그래서 모든 sub-sequences를 foldRight와 ++를 이용해서 결합할 필요가 있다. (xss: seq of seq)

```
(xss foldRight Seq[Int]())(_ ++ _)

// equvalently
xss.flatten

// so, 원래식 포함해서 아래와 같이 나타낼 수 있다.
((1 until n) map (i => (1 until i) map (j => (i, j)))).flatten

// 아래의 법칙을 이용하면
xs flatMap f = (xs map f).flatten

// 더 간단해질 수 있다.
(1 until n) flatMap (i => (1 until i) map (j => (i, j)))

// result 
res0: scala.collection.immutable.IndexedSeq[(Int, Int)] = Vector((2,1), (3,1), (3,2), (4,1), (4,2), (4,3), (5,1), (5,2), (5,3), (5,4), (6,1), (6,2), (6,3), (6,4), (6,5))

// 두 합이 prime인것만 추려야함, 이전 챕터에서 만든 isPrime 이용
(1 until n) flatMap (i => (1 until i) map (j => (i, j))) filter (pair => isPrime(pair._1 + pair._2))

// result 
res0: scala.collection.immutable.IndexedSeq[(Int, Int)] = Vector((2,1), (3,2), (4,1), (4,3), (5,2), (6,1), (6,5))
```

#### For-Expression

for loop에 대해서 알아보자

```
case class Person(name: String, age: Int)

// 20살 이상인 사람만 가져오고 싶을때
for (p <- persons if p.age > 20) yield p.name

// 아래와 같음
persons filter (p => p.age > 20) map (p = > p.name)
```

기본적인 for loop 동작은 비슷하지만 결정적인 차이가 하나 있다. 보통 절차적 언어의 for loop는 어떤 요소가 변할 수 있는 side effect가 존재하지만 스칼라에서는 yield 키워드를 이용해서 iterable 객체를 생성한다.

```
for (s) yield e
```
for-expression을 위와같이 단순화 시킬 수 있다.
> s is a sequence of generators and filters
> e is an expression whose value is returned by an iteration

* generator의 p <- e에서 p는 하나의 패턴이고, e는 컬렉션의 값이다.
* filter의 f는 boolean expression이다.
* 몇개의 generator가 있을때는 마지막이 처음보다 빠르다. why?
* ( s )는 { s }처럼 사용할 수도 있다.

#### example

이전에 보았던 문제를 for loop를 이용해 다시 만들어 보자.
첫번째는 바로 전에 했던 두 수의 합이 prime 넘버인 것의 pair를 구하는 함수
```
for {
  i <- 1 until n
  j <- 1 until i
  if isPrime(i+j)
} yield (i, j)
```
매우 심플해졌다.

두번째는 scalarProduct
```
def scalarProduct(xs: Vector[Double], ys: Vector[Double]): Double =
  (for ((x, y) <- xs zip ys) yield x * y).sum
```
따로 필터링할 조건이 없으므로 모든 요소에 대해서 적용한다.


## 6.3 Combinatorial Search Example



## 6.4 Maps



## 6.5 Putting the Pieces Together

전화번호를 문자로 바꾸는 예제를 살펴보자.
참고로 해당 예제는 파이썬과 같은 스크립트 언어에서는 100라인 정도, 그외에 일반적 목적의 프로그래밍 언어에서는 200~300라인정도의 코드가 나왔다고 한다.

```
val mnemonics = Map(
       '2' -> "ABC", '3' -> "DEF", '4' => "GHI", '5' -> "JKL", 
       '6' -> "MNO", '7' -> "PQRS", '8' -> "TUV", '9' -> "WXYZ")
```

위와 같이 각 번호가 몇개의 문자열로 매핑되어 있다. 해당 숫자가 나왔을때 매핑된 문자열 중에 하나의 문자를 선택해서 출력해준다는 얘기다.

예를 들어 "7225247386"를 convert해보면 그 중 하나가 "SCALAISFUN"(Scala is fun)이 된다.

전체코드는 다음과 같다.

```
val in = Source.fromURL("http://lamp.epfl.ch/files/content/sites/lamp/files/teaching/progfun/linuxwords.txt")
val words = in.getLines.toList filter (word => word forall (chr => chr.isLetter))

val mnem = Map(
	'2' -> "ABC", '3' -> "DEF", '4' -> "GHI", '5' -> "JKL",
	'6' -> "MNO", '7' -> "PQRS", '8' -> "TUV", '9' -> "WXYZ")

val charCode: Map[Char, Char] =
  for ((digit, str) <- mnem; ltr <- str) yield ltr -> digit

/**
 * 파라미터로 들어온 문자열을 charCode로 변경함
 * @param word
 * @return
 */
def wordCode(word: String): String =
  word.toUpperCase map charCode

  wordCode("Java")  // res0: String = 5282

/**
 * A map form digit strings to the words that represent them,
 * e,g. "5282" -> List("Java", "Kata", "Lava", ...)
 * Note: A missing number should map to the empty set, e,g. "11111" -> List()
 */
val wordsForNum: Map[String, Seq[String]] =
  words groupBy wordCode withDefaultValue Seq()

/**
 * Return all ways to encode a number as a list of words
 */
def encode(number: String): Set[List[String]] =
  if (number.isEmpty) Set(List())
  // 1 to number.length는 IndexedReq 타입이므로 Set 타입으로 변경해준다
  else {
    for {
      split <- 1 to number.length
      word <- wordsForNum(number take split)
      rest <- encode(number drop split)
    } yield word :: rest
  }.toSet

encode("7225247386")

def translate(number: String): Set[String] =
  encode(number) map(_ mkString " ")

translate("7225247386")
```  
 
지금까지 살펴본 스칼라 collection은 아래와 같이 정리 할 수 있다.
* easy to use: few steps to do the job.
* concise: one word replaces a whole loop.
* safe: type checker is really good at catching errors.
* fast: collection ops art tuned, can be parallelized.
* universal: one vocabulary to work on all kinds of collections.



List(('a', 2), ('b', 2))

(c, count) = ('a', 2)
i = 1
일단 a가 추출됐으니










