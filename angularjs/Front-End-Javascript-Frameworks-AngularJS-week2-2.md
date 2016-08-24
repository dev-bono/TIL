## Task Runners, Angular Scope, Forms and Form Validation-2

### Angular Scope

스코프는 다음과 같이 정의된다.

> 스코프는 application model과 관련된 object다.
> This is at the core of Angular's two-way data binding
> view와 controller 사이를 연결해주는 역할을 한다.

우선 controller에 scope를 만들어 속성을 부여한다. 그리고 view는 controller에 정의된 속성들을 bind 한다. 이런 방식을 통해 view와 controller의 sync를 맞춰준다.

#### $rootScope

가장 상위의 scope다. app이 시작되면 만들어진다. 새로운 스코프는 ng-controller와 같은 directives 를 통해 만들수 있다. scope의 구조는 DOM의 구조와 비슷하다. 그렇기 때문에 child scope에서는 parent scope의 속성에 접근이 가능하지만 그 반대는 불가하다.


#### app.js

```
// 기존 코드
angular.module('confusionApp', [])
    .controller('MenuController',function(){
        this.tab = 1;
        this.filtText = '';
        
        ...

	});	

// scope 적용 코드
angular.module('confusionApp', [])
	// scope 정의
    .controller('MenuController', ['$scope',function($scope){
        $scope.tab = 1;
        $scope.filtText = '';

        ...

    )]};
```

기존의 코드를 보면 this접근자를 통해서 속성들을 부여하고 있다. 여기서 스코프를 사용하면, this가 아닌 scope에 속성들을 부여함으로써 view단에서 scope를 통해 각 속성에 접근하게 된다. 


#### menu.html

```
<!-- 기존 코드 -->
<div class="container">
    <div class="row row-content" ng-controller="MenuController as menuCtrl">
        <div class="col-xs-12">
            <ul class="nav nav-tabs" role="tablist">
                <li role="presentation" ng-class="{active:menuCtrl.isSelected(1)}">
                    <a ng-click="menuCtrl.select(1)" aria-controls="all menu"
                     role="tab">The Menu</a>
                     ...
                </li>
            </ul>
            <div class="tab-content">
                <ul class="media-list tab-pane fade in active">
                    <li class="media" ng-repeat="dish in menuCtrl.dishes | filter:menuCtrl.filtText">
                    ...
	                </li>
	            </ul>
	        </div>
	    </div>
	</div>
</div>


<!-- scope 적용 코드 -->
<div class="container">
    <div class="row row-content" ng-controller="MenuController">
        <div class="col-xs-12">
            <ul class="nav nav-tabs" role="tablist">
                <li role="presentation" ng-class="{active:isSelected(1)}">
                    <a ng-click="select(1)" aria-controls="all menu"
                     role="tab">The Menu</a>
                     ...
                </li>
            </ul>
            <div class="tab-content">
                <ul class="media-list tab-pane fade in active">
                    <li class="media" ng-repeat="dish in dishes | filter:filtText">
                    ...
	                </li>
	            </ul>
	        </div>
	    </div>
	</div>
</div>
```

기존코드에서는 menuController의 alias인 menuCtrl을 만들어 각 변수와 함수 앞에 붙여줬었다. 하지만 MenuController에 스코프가 적용되면서 해당 컨트롤러가 속한 DOM 내부에서는 컨트롤러 이름없이 변수/함수명으로 직접 접근이 가능해진다. 스코프를 적용함으로써 훨씬 코드가 간단해졌다. 


#### ng-show

ng-show directive는 주어진 조건(true/false)에 따라서 해당 DOM 객체를 보여줄것인지(show) 말것인지(not show)를 결정해준다. 길게 설명할 필요없이 예제를 보자

```
<div class="col-xs-12">
    <button ng-click="toggleDetails()" class="btn btn-xs btn-primary pull-right" type="button">
        {{showDetails ? 'Hide Details':'Show Details'}}
    </button>
    <ul class="nav nav-tabs" role="tablist">
	    ...
	</ul>
	...
        <p ng-show="showDetails">{{dish.description}}</p>
    ...
</div>
```

$scope.showDetails를 app.js에 설정해놓고 기본값으로 false를 지정해놓자. 그리고는 button을 하나 만들어 showDetails가 true이면 Hide Details 문구의 버튼을 노출하고 false면 Show Details 문구의 버튼을 노출한다. 리스트의 요소중에 description 부분에 ng-show directive를 선언하고 showDetails를 넣어주자. 그리고 버튼을 클릭하면(showDetails가 toggle 된다. app.js에 function을 만들어둔다.) showDetails가 true가 됐다가 false가 되면서 description이 보여졌다가 가려졌다가 할 것이다.



