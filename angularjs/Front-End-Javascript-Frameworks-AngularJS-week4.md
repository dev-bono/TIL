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

