## Full-Stack Web Development: The Big Picture

### Three Tier Architecture

보통 웹개발에서 말하는 3-티어 아키텍쳐는 아래의 세 영역으로 나뉜다.

* Presentaion Layer : HTML, CSS, JS
* Business Layer : Ruby, Python, Java, C++
* Data Access Layer : DBMS

반면에 Full-Stack 웹 개발은 자바스크립트를 이용하여 위의 세 layer의 개발을 가능하도록 하는 방법을 말하며, 조금씩 다른 프레임웍을 사용할 수 있겠지만, 이 강의에서는 presentaion layer에는 자바스크립트 프레임웍인 AngularJS, business layer에서는 NodeJS, 그리고 Data Access Layer 영역에서는 자바스크립트를 이용한 MongoDB를 사용하여 개발한다.

이 세 영역은 모두 자바스크리트 기반으로 만들어져 있기 때문에 서버, 브라우저, 모바일 디바이스 등 모든 영역에서 JSON 형태로 통신이 가능하다.

### Course Overview

이번 강의에서 다루는 주제는 다음과 같다.
 
* AngularJS : 자바스크립트 프레임워크 (버전: 1.4.2)
* Web Tools : Grunt, Gulp, Yo and Yeoman
		
 
## Introduction to AngularJS

### Front-End JavaScript Frameworks OverView

> 소프트웨어 라이브러리란 어떤 동작을 실행하는 잘 정의된 인터페이스(or functions or methods)의 집합이라 할 수 있다. 재사용성과 모듈화를 통해서 더 효율적인 개발이 가능하다. 대표적인 예로 jQuery가 있다.

> 소프트웨어 프레임워크란 라이브러리와는 다소 차이가 있다. 라이브러리는 이미 잘 만들어진 유용한 코드(functions)를 사용한다고 한다면, 프레임워크는 개발자가 작성한 코드가 프레임워크안에서 적절하게 실행할 수 있도록 환경을 제공하는 것이라 할 수 있다. 프레임워크는 일반적인(generic) 함수 셋을 제공하며 개발자가 구체적인 코드를 구현해야한다. 그리고 구현된 코드는 프레임워크가 필요한 경우, 어떤 임무를 완수하기 위해 불려진다. 즉, 라이브러리는 개발자에게 코드를 컨트롤 할 수 있는 권한이 주어지는 반면 프레임워크는 코드의 컨트롤 권한이 프레임워크에게 있는 것과 같다. 이번 강의에서 배울 AngularJS나 durandal, backbone 등이 대표적이다.

#### 10가지 Javascript Framework

* Angular : one of the three Major JS framework
* Ember : one of the three Major JS framework
* Backbone : one of the three Major JS framework
* React : 프레임웍이라기 보다 라이브러리에 가까움
* Aurelia
* Meteor : 요즘 각광받기 시작
* Polymer
* Knockout
* Vue
* Mercury

#### 3가지 메이저 JS Frameworks 비교 (3 > 2 > 1)

|                 | Ember     | Angular   | Backbone  |
| :-------------- | :-------: | :-------: | :-------: |
| Opinionated     | 1         | 2         | 3         |
| Ease of Use     | 3         | 2         | 1         |
| Learning Curve  | 1         | 2         | 3         |
| Popularity      | -         | 3         | -         |



### Introduction to AngularJS

HTML은 static한 문서이기 때문에 동적인 웹 어플리케이션을 지원하기에는 HTML만으로는 한계가 있다. 보통은 자바스크립트의 DOM 객체를 이용해서 HTML을 동적으로 만들 수 있지만, 자바스크립트 프레임웍인 Angular를 이용하면 다음과 같은 이점을 가질 수 있다.

> Solving the impedance mismatch

back end 데이터와 static content를 출력하는데 HTML만으로는 한계가 있기 때문에 이 문제(impedance mismatch)를 해결이 가능하도록 해준다. 

> Designed with CRUD applications (data-driven) in mind

데이터가 변경되면 변경된 데이터에 맞춰서 동적으로 HTML이 update 된다. Create, Read, Update, Delete 이 네 가지를 이르는 CRUD에 대해서는 다음 모듈에서 자세하게 다루기로 한다.

> Declarative approach

Angular는 선언적인 개발방법을 지원하는데, 이는 개발자가 원하는 것을 기술하면 Angluar가 그에 맞게 처리해내는 것을 뜻한다(무슨 말인지 잘...)


#### Angular Vocabulary

앞으로 배울 Angular의 문법은 아래와 같은 것들이 있다.

* Two-way Data Binding
* Scope
* Directives
* Templates
* Routing
* Testing
* Modules
* Controllers
* Filters
* Factory
* Service
* Provider

우선은 Two-way Data Binding과 Directives에 대해서 먼저 알아보자

#### Directives

Angular Directives는 ng-* or data-ng-* 로 시작하는 HTML 속성이다. 다음의 몇가지 예를 살펴보자

* ng-app : Angular 앱을 시작과 끝을 나타낸다. 어떠한 태그에도 붙일 수 있다. 즉, 하나의 HTML 파일에 하나 이상의 Angular 앱이 존재할 수도 있다는 말과 같다.

* ng-init : 자바스크립트 변수를 선언하는것과 같다. Angular expression을 Evaluation한다(좀더 매끄러운 한글 표현이 필요할 것 같다). object, array도 사용가능하다.  

```
<p ng-init="index=1"></p>
<div ng-init="dish={name:'example', ...}"></div>
```

* ng-model : input value를 변수로 바인드 한다. (Two-way data binding) ng-model 속성을 부여하면 언제든지 어떤 변수든지 변경이 가능하게 된다.

```
<p>Comment: {{dish.comment}}</p>
<p>Type your comment:
	<input type="text" ng-model="dish.comment" />
</p>
```

dish object의 comment는 위에서 이미 정의를 한 상태다. 이때 아래의 input tag에서 ng-model 속성을 부여하고 이미 정의된 dish.comment를 입력하면 Two-way data binding에 의해 처음 설정된 값이 input 태그의 변경되는 값으로 동적으로 변경된다.

양 방향에서 바인딩이 가능하기 때문에 아마도 Two-way data binding이라고 부르는 것 같다.

* ng-repeat : 반복적인 태그를 작성할 필요가 있을때 사용할 수 있다.

#### Angular Expressions

* Evaluated against an Angular scope object
Angular 스코프에 맞게 value가 평가된다.

* No conditionals, loops, or exceptions
위의 사항들은 expression으로 사용이 불가하다.

* Expressions enclosed in {{ expression }}
중괄호 두개를 겹쳐서 사용한다.

```
<p>6 + 5 = {{ 6 + 5 }}</p>
<h2>{{ dish.name }}</h2>
```
