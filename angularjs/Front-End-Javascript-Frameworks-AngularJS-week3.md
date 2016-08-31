## Single Page Applications

### Angular Factory, Service and Dependency Injection

#### Dependency Injection

Dependency Injection(DI)이란 의존성을 해결하기 위한 Inversino of Control(IoC)을 구현한 소프트웨어 디자인 패턴이다. IoC(제어의 역전)와 DI(의존성 주입)는 여전히 영어로든, 한글로 번역해서든 이해하기 힘든건 마찬가지다. 우선, Ioc라는건 내가 짜놓은 것을 무언가 대신 해주는 개념으로 받아들이면 되겠다. 스프링 같은 프레임웍에서는 컨테이너라는 녀석이 바로 IoC에 해당한다. IoC 개념을 이해하고 다시 DI를 살펴보자. DI는 구성 요소간의 의존관계를 무언가 대신 처리해주는(IoC, 설정 or 객체 or 무엇이든) 디자인 패턴이라 말할 수 있다. 여전히 조금 모호하다. 

마틴 파울러가 말한 DI를 구현하는 세가지 방법은 다음과 같다.

* Create dependency using new operator
* Look up dependency using a global variable
* Have dependency passed to it where needed 

위의 두가지에 비해서 세번째는 필요할때만 의존성을 주입하면 되므로 가장 유연하다.

#### Angular and DI

AngularJS에서는 DI를 구현하기 위해 비지니스 로직을 별도로 구현해 놓는다. 그리고 필요할 때마다 주입시켜서 사용한다.

services, directives, filters and animations 등의 컴포넌트에서 DI가 사용된다. 주입 가능한 factory method나 생성자를 정의한 다음 Controller와 같은 곳에 주입시켜 호출한다.

또한 config와 run method 역시 어떤 컴포넌트를 주입하여 사용 가능한다.
몇가지 예를 들어보자.

```
// 첫번째 방법은 array에 명시
module.controller('MenuController', ['$scope', 'menuFactory', function($scope, menuFactory) {
	
}]);

// 두번째 방법은 property 추가
var MenuController = funciton($scope, menuFactory) {
	
};
MenuController.$Inject = ['$scope', 'menuFactory'];
module.controller('MenuController', MenuController);

// 세번째 방법은 Implicit annotation
module.controller('MenuController', function($scope, menuFactory) {
	
});
```


#### Angular Factory and Service

factory와 service는 controller에 데이터를 제공하기 위한 객체(?)이다. 그리고 아마 백엔드에서 데이터를 주고 받을때 factory와 service를 사용하지 않을까 싶다. 해당 내용은 추후 나오니 일단은 factory와 service에 집중하자.

#### Angular Services

service는 여러개의 app에 공통으로 사용되는 데이터 or 메서드를 담고 있다. 그렇기 때문에 DI를 이용하여 필요한 곳에 주입되어야 하고 Lazily instantiated(필요할 때만 인스턴스화 되어야 함) 되어야 하며 singlton 객체여야한다. 

$http, $scope, $rootScope, $location, $parse, $templateCache, $animate $injector와 같은 built-in 서비스들이 존재한다. 이외에 개발자가 직접 서비스를 등록할 수도 있는데, service(), factory(), provider(), constant(), value() 등의 메서드를 이용하면 가능하다.

#### Angular Factory

factory는 보통 컨트롤러에 데이터를 제공하는 역할을 한다. factory에서 미리 데이터와 메서드 셋을 만들어 둔 다음 필요한 컨트롤러에 의존적으로 주입해서 사용한다. 이렇게 보면 service와 factory의 차이가 별로 안 느껴지는데, 결정적으로 service는 초기화 과정이 있기 때문에 prototype 상속이 가능한다. 그래서 보통 상속이 필요한 데이터 or 모델링은 서비스를 이용하고 정적인 메서드나 데이터 같은 경우에는 factory를 많이 이용한다.

우선 factory 사용 예를 보자

```
// factory code
angular.module('confusionApp')
    .factory('menuFactory', function() {
    	var menufac = {};
    	var dishes = [ ... ];

    	menufac.getDishes = function() {
    		return dishes;
    	};

    	menufac.getDish = function (index) {
    		return dishes[index];
    	};
    	// 객체를 리턴한다.
    	return menufac;
    });

// usage
angular.module('confusionApp')
    .controller('MenuController', ['$scope', 'menuFactory', function($scope, menuFactory) {
    	$scope.dishes = menuFactory.getDishes();
    }]);

```

service 사용과 비교해보자

```
// service code
angular.module('confusionApp')
    .service('menuFactory', function() {
    	// 객체선언, 리턴이 필요없다. 
    	var dishes = [ ... ];

    	this.getDishes = function() {
    		return dishes;
    	};

    	this.getDish = function (index) {
    		return dishes[index];
    	};
    });

// usage
angular.module('confusionApp')
    .controller('MenuController', ['$scope', 'menuFactory', function($scope, menuFactory) {
    	$scope.dishes = menuFactory.getDishes();
    }]);

```

service와 factory의 사용법은 완전히 같다. 하지만 차이가 있는데, factory는 객체를 생성해서 리턴을 한다. 하지만 service의 경우에는 service() 함수가 호출되면서 기본 생성자가 만들어진다. 그래서 service 코드 내에서 this로 바로 접근이 가능하기 때문에 별도로 객체를 만들어 리턴해줄 필요가 없다.

