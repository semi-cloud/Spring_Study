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

 + `RejectedValue` :  **오류 발생 시, 사용자 입력 값을 저장하는 필드**
 
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
 + `th:field`는 정상 상황에는 모델 객체의 값을 사용하지만, _**오류가 발생하면 FieldError 에서 보관한 값을 사용해서 값을 출력!**_

## 오류 코드와 메시지 처리
 + 오류 메시지 체계적으로 다루는 방법
 + 오류 메시지를 구분하기 쉽게, `errors.properties`에 담아서 관리
 + 스프링 부트가 해당 메시지 파일을 인식 가능하게, `application.properties`에 설정을 추가해야함

> errors.properties
```
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
```
> application.properties
```java
spring.messages.basename=messages,errors
```

> ValidationItemControllerV1
 + bindingResult에 `new String[]{"required.item.itemName"}` 형식으로 추가

> ValidationItemControllerV2: **rejectValue, reject**
 + `rejectValue()` , `reject()` : FieldError , ObjectError 를 직접 생성하지 않고 검증 오류 처리 가능

> rejectValue()
```
void rejectValue(@Nullable String field, String errorCode, @Nullable Object[] errorArgs, @Nullable String defaultMessage);
```
 + `field` : 오류 필드명
 + `errorCode` : 오류 코드(메시지에 등록된 코드가 아닌, messageResolver를 위한 오류 코드)
 + `errorArgs` : 오류 메시지에서 {0} 을 치환하기 위한 값
 + `defaultMessage` : 오류 메시지를 찾을 수 없을 때 사용하는 기본 메시지

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
 +  BindngResult는 본인이 검증해야 할 객체인 target을 이미 알고 있음 => 객체(item)에 대한 정보 필요 X
 +  `range.item.price` 대신 `range`만 입력해도 오류 메시지가 정상 출력됌
 +  `codes` : `required.item.itemName` 를 사용해서 메시지 코드를 지정
   +  메시지 코드는 하나가 아니라 배열로 여러 값을 전달할 수 있는데, 순서대로 매칭해서 처음 매칭되는 메시지가 사용됌
+ `arguments` : `Object[]{1000, 1000000}` 를 사용해서 코드의 {0} , {1} 로 치환할 값을 전달
 
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
 +  _객체명과 필드명을 조합한 세밀한 메시지 코드가 있으면, 우선순위로 사용!_
 
#### 오류 코드 전략
:star2: **핵심은 구체적인 것이 우선순위를 가지게 됌!** :star2: </br>
  + 크게 중요하지 않은 메시지: 범용성 있는 requried 같은 메시지로 끝냄
  + 정말 중요한 메시지 : 필요할 때 구체적으로 적어서 사용하는 방식이 더 효과적!
 
> errors.properties에 오류 코드 전략 도입
```
#==ObjectError==
#Level1
totalPriceMin.item=상품의 가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}

#Level2 - 생략
totalPriceMin=전체 가격은 {0}원 이상이어야 합니다. 현재 값 = {1}

#==FieldError==
#Level1
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.

#Level2 - 생략

#Level3
required.java.lang.String = 필수 문자입니다.
required.java.lang.Integer = 필수 숫자입니다.
min.java.lang.String = {0} 이상의 문자를 입력해주세요.
min.java.lang.Integer = {0} 이상의 숫자를 입력해주세요.
range.java.lang.String = {0} ~ {1} 까지의 문자를 입력해주세요.
range.java.lang.Integer = {0} ~ {1} 까지의 숫자를 입력해주세요.
max.java.lang.String = {0} 까지의 숫자를 허용합니다.
max.java.lang.Integer = {0} 까지의 숫자를 허용합니다.

#Level4
required = 필수 값 입니다.
min= {0} 이상이어야 합니다.
range= {0} ~ {1} 범위를 허용합니다.
max= {0} 까지 허용합니다.
```
 + 먼저 오류 메시지 전략에 따라 메시지가 생성되고, 이 메시지 코드를 기반으로 순서대로 `MessageSource`에서 메시지를 찾게됌(구체적인 것 => 덜 구체적인 것)

#### :heavy_check_mark: Spring이 직접 만든 오류 메시지 처리
 + 개발자가 직접 설정한 오류 코드는 `rejectValue()`를 호출하면 됌
   + 하지만, _타입 정보가 맞지 않을때 처럼 스프링이 직접 검증 오류에 추가한 경우는 어떻게 해야할까?_
 + 스프링은 타입 오류가 발생하면 `typeMismatch` 오류 코드 사용
 
```
typeMismatch.item.price
typeMismatch.price
typeMismatch.java.lang.Integer
typeMismatch
```
 + 아직 `error.properties`에 메시지 코드를 추가 하지 않았기 때문에, 스프링이 생성한 기본 메시지가 출력됌

> error.properties
```
#추가
typeMismatch.java.lang.Integer=숫자를 입력해주세요.
typeMismatch=타입 오류입니다.
```
## Validator 분리
 + 컨트롤러에서 검증 로직이 차지하는 부분이 매우 큼
   + :bulb: **별도의 클래스로 역할을 분리**하게 되면, 분리한 검증 로직 재사용 가능!
 
> Validator 인터페이스
```java
public interface Validator {
  boolean supports(Class<?> clazz);
  void validate(Object target, Errors errors);
}
```
 + `supports() {}` : 해당 검증기를 지원하는 여부 확인
 + `validate(Object target, Errors errors)` : 검증 대상 객체와 BindingResult

> ItemValidator
```java
@Component
public class ItemValidator implements Validator {
   @Override
   public boolean supports(Class<?> clazz) {
     return Item.class.isAssignableFrom(clazz);
   }
   
   @Override
   public void validate(Object target, Errors errors) {
     Item item = (Item) target;
     ValidationUtils.rejectIfEmptyOrWhitespace(errors, "itemName","required");
     
     if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
       errors.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
     }

     if (item.getQuantity() == null || item.getQuantity() > 10000) {
       errors.rejectValue("quantity", "max", new Object[]{9999}, null);
     }
     
     //특정 필드 예외가 아닌 전체 예외
     if (item.getPrice() != null && item.getQuantity() != null) {
       int resultPrice = item.getPrice() * item.getQuantity();
       
       if (resultPrice < 10000) {
         errors.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
       }
     }
   }
}
```
### :pushpin: ItemValidator 호출 방법

> 방법(1): 직접 추가
```
 private final ItemValidator itemValidator;

 @PostMapping("/add")
 public String addItemV5(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
 
    itemValidator.validate(item, bindingResult);  //추가

    if (bindingResult.hasErrors()) {
      log.info("errors={}", bindingResult);
      return "validation/v2/addForm";
  }
```

> 방법(2): **WebDataBinder** 통해 사용
+ ValidationItemControllerV2에 다음 코드를 추가!

```java
 @InitBinder     //해당 컨트롤러에만 영향을 주므로, 글로벌 설정은 별도!
 public void init(WebDataBinder dataBinder) {
    log.info("init binder {}", dataBinder);
    dataBinder.addValidators(itemValidator);
 }
```
 + `WebDataBinder` : 스프링의 파라미터 바인딩의 역할을 해주고 검증 기능도 내부에 포함
 + 해당 컨트롤러에서 검증기를 자동으로 적용 가능
 
```java
@PostMapping("/add")
public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

   if (bindingResult.hasErrors()) {
     log.info("errors={}", bindingResult);
     return "validation/v2/addForm";
   }
   
   //성공 로직
   Item savedItem = itemRepository.save(item);
   redirectAttributes.addAttribute("itemId", savedItem.getId());
   redirectAttributes.addAttribute("status", true);
   return "redirect:/validation/v2/items/{itemId}";
}
```
 + validator를 직접 호출하는 부분이 사라지고, 대신에 _**검증 대상 앞에 @Validated 가 붙음**_
 + `@Validated` : 검증기를 실행하라는 애노테이션
   + `WebDataBinder`에 등록한 검증기를 찾아서 실행하는데, 이때 어떤 검증기가 실행되어야 할지 구분 하는 과정에서 `supports()`가 사용이 됌
