# Message

## 메시지 개요
+ 상품명, 가격, 수량 과 같은 `label`에 있는 단어는 하드코딩 되어 있는 상태
   + 단어명 변경시, 수십 개의 파일을 모두 고쳐야 하는 문제점 발생!
+ `Message` : 다양한 메시지를 한 곳에서 관리하도록 하는 기능

> messages.properties
```
  item=상품
  item.id=상품 ID
  item.itemName=상품명
  item.price=가격
  item.quantity=수량
```
> HTML에서 객체에 접근 할때
```
<label for="itemName" th:text="#{item.itemName}"></label>
```
 + 해당 데이터를 key값으로 불러서 사용

## 메시지 Source 설정

### Spring
 + `MessageSource` : 스프링이 제공하는 메시지 관리 기능 (Interface)

```java
@Bean
public MessageSource messageSource() {
     ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();  //인터페이스 구현 객체 사용
     messageSource.setBasenames("messages", "errors");  //여러 파일 지정 가능
     messageSource.setDefaultEncoding("utf-8");
     return messageSource;
}
```
 + `basenames` : 설정 파일의 이름을 지정
   + messages 로 지정=> messages.properties 파일을 읽어서 사용
   + 국제화 기능 지정 => 파일명 마지막에 언어 정보 붙이기 ex)messages_ko.properties
 + 파일의 위치 : `/resources/messages.properties` 
 + `defaultEncoding` : 인코딩 정보를 지정(utf-8 을 사용)

### Spring boot
 + 스프링 부트가 자동으로 MessageSource를 스프링 빈으로 등록!
 + 메시지 소스 기본 값 : `spring.messages.basename=messages`
   + `locale` 정보가 없으면, basename에서 설정한 기본 이름 메시지 파일로 조회  ex)messages.properties
   
## Spring 메시지 소스 사용

```java
@SpringBootTest
public class MessageSourceTest {

    @Autowired
    MessageSource ms;  
    
    @Test
    void helloMessage(){
        String result = ms.getMessage("hello", null, null);   //code, args, locale 파라미터
        assertThat(result).isEqualTo("안녕");
    }

    @Test
    void notFoundMessageCode(){
        assertThatThrownBy(() -> ms.getMessage("no_code", null, null))
            .isInstanceOf(NoSuchMessageException.class);   //메시지 없으면 NoSuchMessageException발생
    }
```
> 매개변수 사용
```java
    @Test
    void argumentMessage(){
        String message = ms.getMessage("hello.name", new Object[]{"Spring"}, null);  //인자는 Object 배열로 전달
        assertThat(message).isEqualTo("안녕 Spring");
    }
```
 + 메시지 파일의 {0}부분 : 매개변수를 전달해서 치환 가능
   + ex) `hello.name=안녕 {0}` => `안녕 Spring`
 
> 국제화 파일 선택
```java
    @Test
    void defaultLang(){
        assertThat(ms.getMessage("hello", null, null)).isEqualTo("안녕");  //messages 사용
        assertThat(ms.getMessage("hello", null, Locale.KOREA)).isEqualTo("안녕");  //messages 사용

    }

    @Test
   void enLang(){
        assertThat(ms.getMessage("hello", null, Locale.ENGLISH)).isEqualTo("hello"); //messages_en 사용
    }
}
```
 + Locale 정보 기반으로 국제화 파일 선택 
   + 구체적인 것을 우선으로 파일을 선택!
   + ex) `message_en_US => messages_en => messages`
  
## 웹 애플리케이션에 메시지 적용

### 메시지 추가 등록

> messages.properties
```
  label.item=상품
  label.item.id=상품 ID
  label.item.itemName=상품명
  label.item.price=가격
  label.item.quantity=수량

  page.items=상품 목록
  page.item=상품 상세
  page.addItem=상품 등록
  page.updateItem=상품 수정

  button.save=저장
  button.cancel=취소
```

### 타임리프 메시지 적용 : #{ }
 + `#{...}` : 타임리프에서 제공하는 기능으로, 스프링 메시지를 편리하게 조회 가능
   + ex) 상품 조회 => #{lable.item}

# 국제화

## LocaleResolver
 + 웹 브라우저에서 언어 설정 값을 변경하면, `Accept-Language`의 값이 변경됌
   + :pencil2 : `Accept-Language` : HTTP 요청 헤더, 서버에 기대하는 언어 정보가 담겨있음
 + Spring은 이 Locale 선택 방식을 변경하도록 `LocaleResolver` Interface 제공
   + Spring boot : 기본값으로 `Accept-Language` 활용 `AcceptHeaderLocaleResolver`사용

> LocaleResolver 인터페이스
```java
public interface LocaleResolver {
    Locale resolveLocale(HttpServletRequest request);
    void setLocale(HttpServletRequest request, @Nullable HttpServletResponse response, @Nullable Locale locale);
}
```
 + 헤더 이외에도, 구현체를 변경하여 쿠키나 세션 기반의 Locale 선택 기능 사용 가능!

 
