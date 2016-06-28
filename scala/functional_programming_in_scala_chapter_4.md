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
