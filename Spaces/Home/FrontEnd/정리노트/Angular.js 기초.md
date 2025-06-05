[W3schools 튜토리얼](http://w3schools.com/angular/angular_intro.asp)


## AngularJS의 핵심 개념
### 모듈(Module)
AngularJS에서 앱의 기본 단위는 모듈이다. 모듈은 컨트롤러, 서비스, 디렉티브, 필터 등 여러 기능을 묶는 컨테이너 역할을 한다. 예를 들어, 교사 관리 앱을 만든다면 아래처럼 모듈을
이렇게 정의된 TeacherApp 모듈 안에서 컨트롤러와 서비스 등을 등록해 사용할 수 있다.

### 컨트롤러(Controller)
컨트롤러는 뷰(HTML)와 데이터를 연결해주는 자바스크립트 객체이다. $scope 객체를 통해 데이터를 주입하고, 뷰에서 사용할 수 있는 함수들도 정의한다.

```js
app.controller('TeacherController', function($scope) {
    $scope.teacher = { id: 1, name: '이현' };
    $scope.greet = function() {
        alert('안녕하세요, ' + $scope.teacher.name + '님!');
    };
});
```

### $scope
$scope는 컨트롤러와 뷰 사이에서 데이터를 주고받는 전달자이다. 컨트롤러에서 $scope에 올린 변수나 함수는 HTML에서 {{}} 문법이나 ng- 지시어로 접근할 수 있다.

### 데이터 바인딩
AngularJS의 큰 특징 중 하나는 양방향 데이터 바인딩이다. 예를 들어, {{teacher.id}}처럼 HTML에서 JS 데이터를 바로 출력할 수 있고, JS 값이 바뀌면 화면도 자동으로 갱신된다.

### ng-app, ng-controller
ng-app은 AngularJS 애플리케이션의 시작점을 정의한다. ng-controller는 특정 HTML 영역에 컨트롤러를 연결한다.

```html
<body ng-app="TeacherApp">
    <div ng-controller="TeacherController">
        <p>{{teacher.name}}</p>
    </div>
</body>
```

### ng-repeat
ng-repeat는 배열 데이터를 반복 출력할 때 사용된다.

```html
<tr ng-repeat="teacher in teachers">
    <td>{{teacher.id}}</td>
    <td>{{teacher.name}}</td>
</tr>
```

### ng-click
ng-click은 클릭 이벤트를 처리할 때 사용한다.

```html
<button ng-click="addTeacher()">추가</button>
```

### $http
$http는 AngularJS의 내장 서비스로, AJAX 통신에 사용된다. 서버에서 JSON 등의 데이터를 받아와서 $scope에 저장할 수 있다.

```js
$http.get('/api/teachers').then(function(response) {
    $scope.teachers = response.data;
});
```

### 조건부 렌더링(ng-if)
ng-if는 특정 조건이 true일 때만 HTML 요소를 렌더링한다.

```html
<div ng-if="loading">로딩 중...</div>
```


## 동작 흐름 예시
다음은 AngularJS 앱이 동작하는 흐름을 간단히 정리한 내용이다.

|단계|설명|
|---|---|
|1|index.html에서 ng-app으로 AngularJS 애플리케이션을 초기화한다.|
|2|ng-controller로 특정 영역에 컨트롤러를 연결한다.|
|3|컨트롤러에서 $http로 서버에서 JSON 데이터를 받아와 $scope.teachers에 저장한다.|
|4|HTML에서 ng-repeat을 사용해 teachers 배열을 반복 출력한다.|
|5|ng-click 등의 이벤트를 통해 추가, 수정, 삭제 기능을 구현한다.|

```html
<body ng-app="TeacherApp">
    <div ng-controller="TeacherController">
        <table>
            <tr ng-repeat="teacher in teachers">
                <td>{{teacher.id}}</td>
                <td>{{teacher.name}}</td>
            </tr>
        </table>
        <button ng-click="addTeacher()">추가</button>
    </div>
</body>
```

```js
app.controller('TeacherController', function($scope, $http) {
    $http.get('/api/teachers').then(function(response) {
        $scope.teachers = response.data;
    });

    $scope.addTeacher = function() {
        var newId = $scope.teachers.length + 1;
        $scope.teachers.push({ id: newId, name: '새 선생님' });
    };
});
```
