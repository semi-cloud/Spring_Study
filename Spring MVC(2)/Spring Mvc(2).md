# Spring MVC(2)

## Thymleaf

### Tyhmleaf 소개와 특징
+ **서버 사이드 HTML 렌더링(SSR)**: 백엔드 서버에서 HTML을 동적으로 렌더링
+ **네츄럴 템플릿** : 순수 HTML을 최대한 유지하면서, 뷰 템플릿도 사용가능한 특징
  + 서버 뿐만 아니라, 웹 브라우저에서 직접 열어도 내용 확인 가능
+ **스프링 통합 지원**
 
### :pushpin: Tyhmleaf 기능(1)-Text
+ HTML 문서는 `<`, `>`와 같은 특수 문자를 기반으로 정의됌
+ `HTML 엔티티` : `<`와 같이 테그의 시작을 알리는 기호를 **문자** 로 표현 가능한 방법 
  + ex) `<`  => `&lt`; 로 변경됌
#### Escape
+ HTML에서 사용하는 특수 문자를, HTML 엔티티로 변경하는 것
+ `th:text` , `[[...]]` 는 기본적으로 escape 제공

#### Unscape
+ 이스케이프 기능을 사용하지 않음
+ `th:utext` , `[(...)]`

```HTML
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>text vs utext</h1>
<ul>
    <li>th:text = <span th:text="${data}"></span></li>     //hello spring
    <li>th:utext = <span th:utext="${data}"></span></li>   //hello spring(진하게)
</ul>
<h1><span th:inline="none">[[...]] vs [(...)]</span></h1>
<ul>
    <li><span th:inline="none">[[...]] = </span>[[${data}]]</li>
    <li><span th:inline="none">[(...)] = </span>[(${data})]</li>
</ul>
</body>
</html>
```
  + `th:inline="none"` : 타임리프가 해석하지 말라는 옵션

### :pushpin: Tyhmleaf 기능(2)-변수

#### SpringEL 다양한 표현식 사용

**1)Object**</br>
+ `user.username` : user의 username을 프로퍼티 접근 user.getUsername()
+ `user['username']` : 위와 같음 user.getUsername()
+ `user.getUsername()` : user의 getUsername() 을 직접 호출

**2)List**</br>
+ `users[0].username` : List에서 첫 번째 회원을 찾고 username 프로퍼티 접근 -> list.get(0).getUsername()
+ `users[0]['username']` : 위와 같음
+ `users[0].getUsername()` : List에서 첫 번째 회원을 찾고 메서드 직접 호출

**3)Map**</br>
+ `userMap['userA'].username` : Map에서 userA를 찾고, username 프로퍼티 접근 -> map.get("userA").getUsername()
+ `userMap['userA']['username']` : 위와 같음
+ `userMap['userA'].getUsername()` : Map에서 userA를 찾고 메서드 직접 호출

#### 지역 변수의 사용
```html
<h1>지역 변수 - (th:with)</h1>
<div th:with="first=${users[0]}">
 <p>처음 사람의 이름은 <span th:text="${first.username}"></span></p>
</div>
```
 + `th:with` : 선언한 테그 안에서만 지역변수 사용 가능

### :pushpin: Tyhmleaf 기능(3)-객체

#### :heavy_check_mark: 기본 객체
+ `{#request}`
+ `${#response}`
+ `${#session}`
+ `${#servletContext}`
+ `${#locale}`

#### 기본 객체의 데이터 접근 편의기능
+ HTTP 요청 파라미터 접근: `param`
  + ex) ${param.paramData}
+ HTTP 세션 접근: `session`
  + ex) ${session.sessionData}
+ 스프링 빈 접근: `@`
  + ex) ${@helloBean.hello('Spring!')}

#### :heavy_check_mark: 유틸리티 객체와 날짜
+ 타임리프는 문자, 숫자, 날짜, URI등을 편리하게 다루는 다양한 유틸리티 객체들을 제공
+ `#message` : 메시지, 국제화 처리
+ `#uris` : URI 이스케이프 지원
+ `#dates` : java.util.Date 서식 지원
+ `#calendars` : java.util.Calendar 서식 지원
+ `#temporals` : 자바8 날짜 서식 지원
+ `#numbers` : 숫자 서식 지원
+ `#strings` : 문자 관련 편의 기능
+ `#objects` : 객체 관련 기능 제공
+ `#bools` : boolean 관련 기능 제공
+ `#arrays` : 배열 관련 기능 제공
+ `#lists , #sets , #maps` : 컬렉션 관련 기능 제공
+ `#ids` : 아이디 처리 관련 기능 제공, 뒤에서 설명

### :pushpin: Tyhmleaf 기능(4)-URL 생성

#### 단순 URL
 + `@{/hello}` => /hello
#### 쿼리 파라미터
 + `@{/hello(param1=${param1}, param2=${param2})}` => /hello?param1=data1&param2=data2
 + **( )** 내부의 부분은 쿼리 파라미터로 처리
#### 경로 변수
 + `@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}` => /hello/data1/data2
 + **URL 경로에 변수가 있으면**, **()** 내부의 부분은 경로 변수로 처리
#### 경로 변수 + 쿼리 파라미터
 + `@{/hello/{param1}(param1=${param1}, param2=${param2})}` => /hello/data1?param2=data2
 
### :pushpin: Tyhmleaf 기능(5)-리터럴

> ❓ **Literals**</br>
+ 소스 코드 상의 고정된 값 ex)String a = "Hello"  
+ 타임리프에서 **문자 리터럴** => `' '`로 감싸야함  ex)<span th:text="'hello'">
+ But, 공백 없이 쭉 이어진다면 ' ' 생략 가능  ex)<span th:text"hello">
  + Hello world 과 같이 공백이 포함되면 주의!

#### 리터럴 대체(Literal Substitutions)
+ `| hello ${data} |` 
 
### :pushpin: Tyhmleaf 기능(5)-연산

+ 비교연산: HTML 엔티티를 사용해야 하는 부분을 주의
  + > (gt), < (lt), >= (ge), <= (le), ! (not), == (eq), != (neq, ne)
+ 조건식: 자바의 조건식과 유사
+ Elvis 연산자: 조건식의 편의 버전
+ No-Operation: _ 인 경우 마치 타임리프가 실행되지 않는 것 처럼 동작=> 잘 사용하면 HTML
의 내용 그대로 활용 가능!

### :pushpin: Tyhmleaf 기능(6)-속성 값 설정 
+ `th:*` : 기존 속성을 `th:*`로 지정한 속성으로 대체

#### 속성 추가
+ `th:attrappend` : 속성 값의 뒤에 값을 추가
+ `th:attrprepend` : 속성 값의 앞에 값을 추가
+ `th:classappend` : class 속성에 자연스럽게 추가
 
#### checked 처리
+ `th:checked` : 값이 false인 경우, checked 속성 자체를 제거하기 때문에, boolean값을 효율적으로 이용 가능
 
### :pushpin: Tyhmleaf 기능(7)- 반복
 
+ `th:each="user : ${users}"` : 오른쪽 컬렉션(users)에서 값을 꺼내서 왼쪽 변수(user)에 담아, 태그를 반복 실행

#### 반복 상태 유지
+ `<tr th:each="user, userStat : ${users}">`:  두번째 파라미터를 설정해서, 반복의 상태 확인 가능
  + '지정 변수명' + 'Stat' 으로 상태확인 변수명을 지으면, 파라미터 생략가능
  
### :pushpin: Tyhmleaf 기능(8)-조건부 평가
 
+ `th:if, th:unless` : 타임리프는 해당 조건이 맞지 않으면, 아예 태그 자체를 렌더링 X
  + ex) th:if="${user.age lt 20}"이 false 인 경우 <span>..<span>부분 자체가 사라짐
+ `th: switch ` : * 는 만족하는 조건이 없을때 사용하는 Default 값
  
```HTML
  <span th:text="'미성년자'" th:if="${user.age lt 20}"></span>
  
  <td th:switch="${user.age}">
     <span th:case="10">10살</span>
     <span th:case="20">20살</span>
     <span th:case="*">기타</span>
 </td>
```

### :pushpin: Tyhmleaf 기능(9)-주석
+ HTML 주석:  **<!--    -->**
+ Thymleaf 주석 : **<!--/*      */--> or **<!--/*-->  <!--*/-->
+ Thymleaf 프로토타입 주석 : <!--/*/    /*/-->
  + 타임리프 렌더링(서버)한 경우에만 보이고, HTML파일 그대로 열어보면 렌더링 하지 않아 보이지 않음 
  
### :pushpin: Tyhmleaf 기능(10)-블록
+ `th:block` : 타임리프 유일한 자체 태그로, `th:each`를 사용하기 어려울때 사용(반복) 
  
```HTML
<th:block th:each="user : ${users}">
  <div>
    사용자 이름1 <span th:text="${user.username}"></span>
    사용자 나이1 <span th:text="${user.age}"></span>
  </div>
  <div>
    요약 <span th:text="${user.username} + ' / ' + ${user.age}"></span>
  </div>
</th:block>
```
### :pushpin: Tyhmleaf 기능(10)-자바 스크립트 인라인
 
+ `th:inline="javascript"` 
+ 텍스트 렌더링 : 문자 타입인 경우 `"` 자동 포함, 이스케이프 처리(\)
+ 내추럴 템플릿 : HTML 파일을 직접 열어도 동작
  + ex)var username2 = /*[[${user.username}]]*/ "test username";
  + 인라인 사용 후 => var username2 = "userA"; (인라인 사용 전 => /*userA*/)
+ 객체 : 객체를 자동으로 JSON으로 변환 (인라인 사용 전 => 객체.toString() 호출값)
  
  
  
  
