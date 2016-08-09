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
* AngularJS : 자바스크립트 프레임워크
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

