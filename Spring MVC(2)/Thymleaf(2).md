# Thymleaf(2) : 스프링 통합과 폼

> 스프링 통합으로 추가되는 기능들

 + 스프링의 SpringEL 문법 통합
 + ${@myBean.doSomething()} 처럼 스프링 빈 호출 지원
 + 편리한 폼 관리를 위한 추가 속성
   + `th:object` (기능 강화, 폼 커맨드 객체 선택)
   + `th:field` , `th:errors` , `th:errorclass`
 + 폼 컴포넌트 기능
   + `checkbox`, `radio button`, `List` 등을 편리하게 사용할 수 있는 기능 지원
 + 스프링의 메시지, 국제화 기능의 편리한 통합
 + 스프링의 검증, 오류 처리 통합
 + 스프링의 변환 서비스 통합(ConversionService)

## 입력 폼 처리
 + `th : object` : 커맨드 객체를 지정
   + 등록 폼 같은 경우, 빈 객체(해당 오브젝트)를 생성해서 뷰에 넘겨주어야 함
 + `*{...}` : 선택 변수 식, th:object 에서 선택한 객체에 접근
   + ex) ${~~item.Name~~} => * {itemName}
 + :star2: `th:field` : HTML 태그의 **id , name , value 속성을 자동으로 처리**

> 렌더링 전
<input type="text" th:field="*{itemName}" /></br>

> 렌더링 후
<input type="text" id="itemName" name="itemName" th:value="*{itemName}" /></br>

```html
<form action="item.html" th:action th:object="${item}" method="post">
 <div>
     <label for="itemName">상품명</label>
     <input type="text" id="itemName" th:field="*{itemName}" class="form-control" placeholder="이름을 입력하세요">
 </div>
```
 + `th:object="${item}"` : <form> 에서 사용할 객체를 지정, 선택 변수 식( *{...} )을 적용 가능
 + `th:field="*{itemName}"` : id, name, value 모두 field에서 지정한 변수 이름과 같이 생성  ex)id="itemName"
  
  
## 체크 박스
  
### 체크 박스: 단일1(Html)
 + :pencil2: 단순 HTML의 체크 박스는, 체크 되었을때는 HTML Form에 `open=on`(스프링이 on->true 변환)으로 값이 넘어감
   + But, 체크되지 않으면 open 이라는 필드 자체가 서버로 전송되지 않음!(null)

#### 체크 해제를 인식하기 위한 히든 필드
 + <input type="hidden" name="_open" value="on"/>
 + 항상 전송되기 때문에, 체크 해제한 경우에도 전송이 온 `_open`을 통해 체크 해제를 판단
   + 체크 : `open=on&_open=on` , true값 
   + 해제 : `_open=on`, false값

### 체크 박스: 단일2(Tyhmleaf)

> 타임리프 - 체크 박스 코드 추가
  
```HTML
  <!-- single checkbox -->
<div>판매 여부</div>
<div>
 <div class="form-check">
     <input type="checkbox" id="open" th:field="*{open}" class="form-check-input">
     <label for="open" class="form-check-label">판매 오픈</label>
 </div>
</div>
```
  + ❓ 실행 결과
    + `<input type="hidden" name="_open" value="on"/>` : 타임리프가 자동으로 생성(히든 필드 관련 부분)
    + `value="true"` : 타임리프가 판별해서 <input> 태그에 자동으로 추가
    + `checked="checked"` : 타임리프의 th:field 를 사용하면, 값이 true인 경우 체크를 자동으로 처리
  
### 체크 박스: 멀티
  
> FormItemController - 추가
```java
@ModelAttribute("regions")
public Map<String, String> regions() {
     Map<String, String> regions = new LinkedHashMap<>();
     regions.put("SEOUL", "서울");
     regions.put("BUSAN", "부산");
     regions.put("JEJU", "제주");
     return regions;
}
```
 + `@ModelAttribute("regions")` : 해당 컨트롤러를 요청시, regions 에서 반환한 값이 자동으로 Model에 담김
   + 각 컨트롤러에서 `model.attribute()`를 사용해 데이터를 넣어주는 과정을 반복하지 않아도됌
 
> addForm.html - 추가
```HTML
  <!-- multi checkbox -->
<div>
 <div>등록 지역</div>
 <div th:each="region : ${regions}" class="form-check form-check-inline">
     <input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
     <label th:for="${#ids.prev('regions')}"
            th:text="${region.value}" class="form-check-label">서울</label>
 </div>
</div>
```
 + `th:for="${#ids.prev('regions')` : 여러 체크박스를 만들때, id 값을 동적으로 생성(1,2,3..)

> 타임리프 HTML 생성 결과
```HTML
  <!-- multi checkbox -->
<div>
 <div>등록 지역</div>
  
 <div class="form-check form-check-inline">
     <input type="checkbox" value="SEOUL" class="form-check-input" id="regions1" name="regions">
     <input type="hidden" name="_regions" value="on"/>
     <label for="regions1"
            class="form-check-label">서울</label>
 </div>
  
 <div class="form-check form-check-inline">
     <input type="checkbox" value="BUSAN" class="form-check-input" id="regions2" name="regions">
     <input type="hidden" name="_regions" value="on"/>
     <label for="regions2"
            class="form-check-label">부산</label>
 </div>
```
 + 서울, 부산 선택한 경우 : `regions=SEOUL&_regions=on&regions=BUSAN&_regions=on&_regions=on`
 + 지역 선택 X : `_regions=on&_regions=on&_regions=on`
  
## 라디오 버튼

## 셀렉트 박스


