# Validation

## 검증 개요
 + 상품 등록 폼에서, 고객이 입력된 상품명이 없거나, 가격이 검증 범위를 넘어서는 경우가 존재
 + Model에 검증 오류 결과를 포함해서 다시 폼을 고객에게 보여주어야함
 
<img src = "https://user-images.githubusercontent.com/71436576/129212946-bdf17456-dafe-4c6d-a079-01a105283afa.png" width=50% height=50%>
 
> ValidationItemControllerV1 - Map
```java
     @PostMapping("/add")
    public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes, Model model) {

        //검증 오류 결과를 보관
        Map<String, String> errors = new HashMap<>();

        //검증 로직
        if(!StringUtils.hasText(item.getItemName())){   //글자가 없으면
            errors.put("itemName", "상품 이름은 필수입니다.");
        }

        if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000){
            errors.put("price", "가격은 1000 ~ 100,0000 까지 허용합니다.");
        }

        if(item.getQuantity() == null || item.getQuantity() >= 9999){
            errors.put("quantity", "수량은 최대 9,999 까지 허용합니다. ");
        }

        //특정 필드가 아닌 복합 룰 검증
        if(item.getPrice() != null && item.getQuantity() != null){
            int resultPrice = item.getPrice() * item.getQuantity();
            if(resultPrice < 10000 ){
                errors.put("globalError", "가격 * 수량의 합은 10,000 이상이어야 합니다.");
            }
        }

        //검증에 실패하면 다시 입력 폼으로
        if(!errors.isEmpty()){ 
            model.addAttribute("errors", errors);    //오류 메시지 출력 위해 뷰로 전송
            return "validation/v1/addForm";
        }

        //성공 로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v1/items/{itemId}";
    }
```
 + `Map<String, String> errors = new HashMap<>()` : 검증 오류가 발생하면, 오류 발생 검증 정보를 담아둠
 + `globalError` : 특정 필드를 넘어서는 오류 처리 경우

> css 추가
```css
.field-error {
     border-color: #dc3545;
     color: #dc3545;
}
```
> addForm.html 변경
```html
<form action="item.html" th:action th:object="${item}" method="post">
        <div th:if="${errors?.containsKey('globalError')}">
            <p class="field-error" th:text="${errors['globalError']}"> 전체 오류 메시지</p>
        </div>

        <div>
            <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
            <input type="text" id="itemName" th:field="*{itemName}"
                   th:class="${errors?.containsKey('itemName')} ? 'form-control fied-error' : 'form-control'"
                   class="form-control" placeholder="이름을 입력하세요">
            <div class="field-error" th:if="${errors?.containsKey('itemName')}" th:text="${errors['itemName']}">
                상품명 오류
            </div>
        </div>
```
 + errors에 내용이 있을 경우에만, 오류 메시지 출력
 + `errors?.` : errors 가 null 일 경우, NullPointerException 이 발생하는 대신, null 을 반환(실패)

### 😕 초기 검증 코드의 문제점
+ 뷰 템플릿에서의 중복 처리
+ 타입 오류 처리 불가능
  + Controller에 진입하기도 전에, **400 Exception이 발생해버려서 컨트롤러 자체가 호출 X**
+ 바인딩 불가능
  + Item의 price 필드에 문자를 입력하는 타입 오류가 발생해도, 고객이 입력한 **문자 정보를 보관해야함**
  + But, price는 Integer 타입으므로 문자 보관 불가능

## BindingResult
 + `BindingResult` : Spring이 제공하는 검증 오류 처리 방법, Errors 인터페이스 상속
   + 구현체 : `BeanPropertyBindingResult`
 + :star2: **`BindingResult` 파라미터 위치는, 무조건 `@ModelAttribute` 다음에 와야함**
 + BindingResult는 자동으로 Model에 포함돼 뷰에 전달되므로, ~~model.attribute()~~ 필요 X
 + `@ModelAttribute`에 데이터 바인딩시, **오류가 발생해도 컨트롤러 호출 가능**
   + 오류 정보( FieldError )를 BindingResult 에 담아서 컨트롤러를 정상 호출


### ✔️ FieldError

> ValidationItemControllerV2 
```java
 public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

        //필드 오류
        //검증 로직
        if(!StringUtils.hasText(item.getItemName())){   
            bindingResult.addError(new FieldError("item","itemName","상품 이름은 필수입니다." ));
        }

        if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000){
            bindingResult.addError(new FieldError("item","price","가격은 1000 ~ 100,0000 까지 허용합니다." ));
        }

        if(item.getQuantity() == null || item.getQuantity() >= 9999){
            bindingResult.addError(new FieldError("item","quantity","수량은 최대 9,999 까지 허용합니다. " ));
        }
        ...
   
}
```

> FieldError 생성자 요약
```java
public FieldError(String objectName, String field, String defaultMessage) {}
```
 + 필드에 오류가 있으면, FieldError 객체를 생성해서 bindingResult 에 담으면 됌
 + `objectName` : @ModelAttribute 이름
 + `field` : 오류가 발생한 필드 이름
 + `defaultMessage` : 오류 기본 메시지

### ✔️ ObjectError
```java
        //특정 필드가 아닌 복합 룰 검증
        if(item.getPrice() != null && item.getQuantity() != null){
            int resultPrice = item.getPrice() * item.getQuantity();
            if(resultPrice < 10000 ){
                bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000 이상이어야 합니다."));
            }
        }
```
> Object 생성자 요약
```java
public ObjectError(String objectName, String defaultMessage) {}
```
 + 특정 필드를 넘어서는 오류가 있으면, ObjectError 객체를 생성해서 bindingResult 에 담으면 됌
 + `objectName` : @ModelAttribute 의 이름
 + `defaultMessage` : 오류 기본 메시지

###  ✔️ 타임리프 스프링 검증 오류 통합 기능

> GlobalError
```html
       <div th:if="${#fields.hasGlobalErrors()}">
                  <p class="field-error" th:each="err : ${#fields.globalErrors()}"
                     th:text="${err}"> 글로벌 오류 메시지</p>
       </div>
```
 + `#fields` : #fields 로 BindingResult 가 제공하는 검증 오류에 접근 가능
 
> Field-error
```html
       <div>
            <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
            <input type="text" id="itemName" th:field="*{itemName}"
                   th:errorclass="field-error" class="form-control" placeholder="이름을 입력하세요">
            <div class="field-error" th:errors="*{itemName}">
                상품명 오류
            </div>
       </div>
```
 + `th:errors` : 해당 필드에 오류가 있는 경우에 태그를 출력,  th:if 의 편의 버전
 + `th:errorclass` : th:field 에서 지정한 필드에 오류가 있으면 class 정보를 추가

### 😕 오류 발생시 사용자 입력 값을 유지? : RejectedValue

 + `RejectedValue` :  오류 발생 시, 사용자 입력 값을 저장하는 필드
 
> FieldError/ ObjectError의 또 다른 생성자

```java
public FieldError(String objectName, String field, @Nullable Object rejectedValue, 
  boolean bindingFailure, @Nullable String[] codes, @Nullable Object[] arguments, @Nullable String defaultMessage)
```
 + `objectName` : 오류가 발생한 객체 이름
 + `field` : 오류 필드
 + :star2: `rejectedValue` : 사용자가 입력한 값(거절된 값)
 + `bindingFailure`  : 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값
 + `codes` : 메시지 코드
 + `arguments` : 메시지에서 사용하는 인자
 + `defaultMessage` : 기본 오류 메시지
 
> ValidationItemControllerV2
```java
 if(!StringUtils.hasText(item.getItemName())){   //글자가 없으면
            bindingResult.addError(new FieldError("item","itemName", item.getItemName(),false,null,null, "상품 이름은 필수입니다." ));
 }
```

> 타임리프 사용자 입력 값 유지
 + `th:field="*{price}"`
 + `th:field`는 정상 상황에는 모델 객체의 값을 사용하지만, 오류가 발생하면 FieldError 에서 보관한 값을 사용해서 값을 출력

## 오류 코드와 메시지 처리
 + 오류 메시지를 구분하기 쉽게, `errors.properties`에 담아서 관리

> errors.properties
```
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
```

> ValidationItemControllerV1
 + bindingResult에 `new String[]{"required.item.itemName"}` 형식으로 추가

> ValidationItemControllerV2
 + `rejectValue()` , `reject()` : FieldError , ObjectError 를 직접 생성하지 않고 검증 오류 처리 가능

```java
   // 필드 오류
  if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000){
              bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
   }

  //글로벌 오류
  if(item.getPrice() != null && item.getQuantity() != null){
              int resultPrice = item.getPrice() * item.getQuantity();
              if(resultPrice < 10000 ){
                  bindingResult.reject("totalPrice", new Object[]{1000, resultPrice}, null);
              }
    }

```
 +  BindngResult는 본인이 검증해야 할 객체인 target을 이미 알고 있음 => item에 대한 정보 필요 X
 +  `range.item.price` 대신 `range`만 입력해도 오류 메시지가 정상 출력됌

### ✔️ 오류 메시지 생성: MessageCodesResolver
 + `MessageCodesResolver` : 검증 오류 코드로 , 메시지 코드들을 생성
   + Interface이며 기본 구현체는 DefaultMessageCodesResolver
 
#### DefaultMessageCodesResolver 메시지 생성 규칙

> 객체 오류
```
 1.: code + "." + object name
 2.: code
 
 ex)reject("totalPriceMin")
 totalPriceMin.item       //우선순위 순서
 totalPriceMin 
 
 ```

> 필드 오류
```
 1.: code + "." + object name + "." + field
 2.: code + "." + field
 3.: code + "." + field type
 4.: code

 ex)rejectValue("itemName", "required")
 required.item.itemName        //우선순위 순서
 required.itemName
 required.java.lang.String
 required
```
 + `rejectValue()`는 내부에서 MessageCodesResolver를 사용해 메시지 코드들 자동으로 생성
 
#### 오류 코드 전략
:star2: **핵심은 구체적인 것이 우선순위를 가지게 됌!** :star2: </br>
  + 크게 중요하지 않은 메시지: 범용성 있는 requried 같은 메시지로 끝냄
  + 정말 중요한 메시지 : 필요할 때 구체적으로 적어서 사용하는 방식이 더 효과적!
 
> errors.properties에 오류 코드 전략 도입


> ValidationItemControllerV3

> ValidationItemControllerV4

> ValidationItemControllerV5



## Validator 분리

# Bean Validation

## 빈 검증 개요

## 빈 검증-Spring 적용

## Form 전송 객체 분리

## 빈 검증- HTTP 메시지 컨버터
