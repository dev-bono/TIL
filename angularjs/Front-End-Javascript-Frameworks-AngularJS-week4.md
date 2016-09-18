## Client-Server Communication and Angular Testing

### Networking Essentials

이번 챕터에서는 클라이언트 서버 구조에서 AngularJS가 어떻게 동작하는지 살펴볼것이다.
요즘 가장 많이 사용되는 Client-Server 관계는 HTTP프로토콜과 REST API를 이용한 백엔드(서버)와 브라우저(클라이언트)간의 통신으로 이뤄진다. 이전 강의까지 보았던 방식은 브라우저에 출력되는 데이터가 services.js 파일의 자바스크립트 오브젝트로 존재했었다. 하지만, Client-Server 구조에서는 거의 모든 데이터를 서버사이드에서 클라이언트 사이드로 제공해준다. HTTP와 REST API에 대한 내용은 자세히 다루지는 않겠다(강의 보세요).

HTTP Response는 클라이언트로 보낼 HTML 페이지나 특정 포맷으로 data를 가지는데, 주로 XML 또는 JSON을 많이 사용한다. 이 중에서 JSON(Javascript Object Notation)은 가장 많이 사용되는 데이터 포맷이다. 이름만 보면 자바스크립트에서만 사용될것 같지만, 모바일이나 웹서비스 등 일반적인 HTTP 통신에서 대부분 사용된다. XML이 잘 사용되지 않는 이유는 JSON에 비해서 데이터를 parsing 하는 과정이 복잡하기 때문이다.

JSON 데이터 구조를 간단히 살펴보자
자바스크립트 object와 구조가 비슷하다. name : value 구조로 이뤄지며, value 내에는 array, 스트링, 숫자, object 등의 데이터 타입 등이 들어갈 수 있다.

```
{"promotion":
	[
		{
			"id": 0,
			"name": "weekend buffet",
			"image": "images/buffet.png",
			"label": "New",
			"price": "19.99",
			"description": "asdjfkljaskdlfjas..."
		}
	]
}
```

### Client-Server Communication using $http

#### Angular $http

$http는 브라우저에서 서버와의 통신을 위한 가장 핵심적인 서비스로 HTTP protocol을 사용한다. 아마 내부적으로는 AJAX 통신을 할것이기 때문에 비동기로 처리될 것이다.

#### Promise

자바스크립트에서 Promise란 비동기 통신이 완료된 후에 상태에 따라 특정 콜백을 리턴해 줄 것이라는 일종의 약속 같은 것을 말한다. $http 서비스 역시 프로미스를 리턴한다. 패턴은 아래와 같다.

```
$http({method: 'GET', url:'/dishes'})
	.then(function() { ... }, function() { ... });
```

then 뒤에는 두개의 function이 파라미터로 들어가 있다. 첫번째는 request가 성공했을 때 발생하는 함수이고, 두번째는 요청이 실패했을때 발생하는 함수이다. 보통 AJAX 통신을 사용할 때, 'success', 'error' 속성을 사용하는 것과 비슷하다.

#### HTTP Response

요청의 결과로 response라는 object가 서버로부터 넘어온다. 이때 reponse 객체에는 다음과 같은 속성들이 포함되어 있다.

* response.data : 메세지 바디를 포함하느 string / object
* response.status : 상태 코드 (200, 400 등)
* response.headers : 헤더정보
* response.config : configuration object
* response.statusText : response 상태 텍스트 값 

#### ng-if directive

html 코드를 작성할 때 해당 DOM을 보여줄지 말지를 결정하기 위해 ngIf directive를 사용할 수 있다. 사용법은 간단하다.

```
<div class="col-xs-12" ng-if="!showMenu">
	<h3>{{message}}</h3>
</div>
```

showMenu가 true면 ng-if는 false가 되므로 해당 message는 안보일것이고, showMenu가 false면 반대로 message가 나타날 것이다.


### Brief Representational State Transfer (REST)

본격적으로 서버와의 통신을 해보기 전에 가장 많이 사용하는 방식인 REST에 대해서 알아보자. 웹 서비스를 만들때 네트워크와 연결하는 방법에 대한 시스템은 보통 아래 두개 접근법이 가장 일반적이다.

* SOAP (Simple Object Access Protocol) : Uses WSDL(Web Service Description Language), 프레젠테이션 영역에 XML을 사용 

* REST (Representational State Transfer) : 가장 많이 사용됨, Use Web standards, Exchange of data using either XML or JSON, SOAP나 WSDL보다 훨씬 간단한다.

REST는 WWW(World Wide Web)과 같이 분산 hypermedia systems을 위한 아키텍쳐 스타일 중 하나이며, 웹 페이지의 리소스에 어떻게 접근하고 어떻게 정의되고 어떻게 이동하는지에 대한 법칙을 모아놓은 시스템이다. 

#### REST의 기본 법칙

* HTTP 메소드를 사용하며 (Use HTTP methods explicitly)
* 상태를 저장하지 않는다 (Be stateless)
* 리소스는 URI와 같이 디렉토리 스트럭쳐가 그대로 노출된다 (Expose directory structure-like URIs)
* 통신의 매개체로 XML, JSON 등이 사용된다.

REST는 세가지 컨셉으로 표현할 수 있다. 특정 URI로 표현되는 _Nouns(Resources)_, GET or POST 등의 HTTP 메소드인 _Verbs_, XML or JSON 등으로 표현하는 _Representaions_

#### Resources

REST의 리소스는 다음과 같이 표기한다. 리소스에는 문서나 이미지, 리소스 컬렉션 등이다.

```
http://www.conFusion.food/dishes/			# Retrieve all dishses
http://www.conFusion.food/dishes/123		# Retrieve information about the specific dish (id:123)
http://www.conFusion.food/promotions/
http://www.conFusion.food/leadership/
http://www.conFusion.food/leadership/456
```

#### Verbs

Verb에 해당하는 HTTP 메소드는 각각 다음과 같은 의미를 가진다.

* GET -> READ
* POST -> CREATE
* PUT -> UPDATE 
* DELETE -> DELETE

GET 메소드는 클라이언트가 서버에게 어떤 리소스를 요청하는 메소드이다. 어떤 GET 메소드가 서버에 요청(Request)이 들어오면, 서버는 XML 또는 JSON 형태로 클라이언트(브라우저)에게 응답(Response)을 한다.

#### Representaions

표현을 위해 클라이언트에게 어떤 데이터를 보낼 것이냐 하는 것인데, 가장 많이 쓰이는 두가지 방식이 바로 JSON과 XML이다. 최근에는 JSON 타입이 정보 교환을 위한 standards로 여겨지고 있다. 

#### Stateless

REST는 상태를 저장하지 않다. 매 요청바다 항상 새로운 요청이 발생하게 된다. 그렇기 때문에 클라이언트측에서 요청에 따라 상태가 변화하는 것을 트래킹하기 위해서는 클라이언트 자신이 요청 전의 상태를 기억하고 있어야 한다.





