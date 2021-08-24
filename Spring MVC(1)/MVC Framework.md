
# MVC 프레임워크 생성

## Front Controller
  + 프론트 컨트롤러(Servlet) 하나로 클라이언트의 요청을 받고, 프론트는 요청에 맞는 컨트롤러를 찾아서 호출
    + 프론트를 제외한 나머지 컨트롤러 => 서블릿 사용 필요성 X
 
<img src="https://user-images.githubusercontent.com/71436576/127743885-20e4f0e1-6bc2-4465-a92e-f5b2285f16bc.png" width=50% height=50%>

### :pushpin: 프론트 컨트롤러 도입-V1
**1)기존 구조를 최대한 유지하면서, 프론트 컨트롤러 도입**</br>
 
> Controller 인터페이스
```java
public interface ControllerV1 {
    void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

> 인터페이스 구현한 각각 컨트롤러 클래스
```java
public class MemberFormControllerV1 implements ControllerV1 {
    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);      //다른 서블릿,JSP 로 이동
    }
}
```

> Front Controller 클래스
```java
@WebServlet(name="frontControllerServletV1", urlPatterns = "/front-controller/v1/*" )
public class FrontControllerServletV1 extends HttpServlet {

    private Map<String, ControllerV1> controllerMap = new HashMap<>();

    public FrontControllerServletV1() {
        controllerMap.put("/front-controller/v1/members/new-from", new MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
        controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        //front-controller/v1/members
        String requestURI = request.getRequestURI();

        //**다형성 => 부모(인터페이스)에 자식이 담길 수 있음!
        ControllerV1 controller = controllerMap.get(requestURI);
        if(controller == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        //**다형성 => override 된 process 호출됌
        controller.process(request,response);
    }
}
```
  + `/front-controller/v1/*` : /front-controller/v1 를 포함한 하위 모든 요청은 이 서블릿에서 받아들임
  + `ControllerMap`
    + `key` : 매핑 URL
    + `value` : 호출될 컨트롤러
  + `service()` : requestURI를 조회해서 실제 호출할 컨트롤러를 MAP에서 찾은 후, process()를 통해 컨트롤러 실행


### :pushpin: View 분리-V2
**1)단순 반복 되는 뷰 로직 분리**</br>
  + 모든 컨트롤러에서 뷰로 이동하는 부분에 중복이 존재 => **별도로 뷰 처리하는 객체 생성**

<img src="https://user-images.githubusercontent.com/71436576/127867213-357edbbe-57f7-44fc-9c68-b1e60f6380fe.png" width=50% height=50%>

> 뷰 처리 객체 MyView
```java
public class MyView {
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    //JSP 렌더링
    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request,response);
    }
}
```

> Controller 인터페이스
```java
public interface ControllerV2 {
    //MyView(뷰 객체)를 반환
    MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

> 인터페이스 구현 Controller 클래스
```java
public class MemberSaveControllerV2 implements ControllerV2 {
    MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        //비즈니스 로직
        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관
        request.setAttribute("member", member);

        //경로 넣어서 MyView 반환
        return new MyView("/WEB-INF/views/save-result.jsp");
    }
}
```
> Front Controller 중 일부
```java
@Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ...
        MyView view = controller.process(request, response);
        view.render(request,response);     //forward 로직 수행
    }
```
### :pushpin: Model 추가-V3

<img src="https://user-images.githubusercontent.com/71436576/127869644-d4b88c4d-3df0-4378-a7e2-9de4907f14be.png" width=50% height=50%>

#### :heavy_check_mark: 1) Servlet 종속성 제거: ModelView 추가
 + 컨트롤러에서 HttpServletRequest를 사용, request.setAttribute()를 통해 데이터 저장하고 뷰에 전달(종속적)
 + :star2: `ModelView` : 위 두 기능을 대체할 객체, 뷰이름과 뷰에 전달할 Model 데이터 포함

> ModelView
```java
public class ModelView {
  
    private String viewName;      //view 이름  

    private Map<String,Object> model = new HashMap<>();  //뷰 렌더링 시 필요한 model 객체

    public ModelView(String viewName) {
        this.viewName = viewName;
    }

    public String getViewName() {
        return viewName;
    }

    public void setViewName(String viewName) {
        this.viewName = viewName;
    }

    public Map<String, Object> getModel() {
        return model;
    }

    public void setModel(Map<String, Object> model) {
        this.model = model;
    }
}
```
  + `model` : 뷰를 렌더링 할때 필요한 데이터를 (key,value)형태로 저장
  
> Controller 인터페이스
```java
public interface ControllerV3 {
    //서블릿에 종속적 X
    ModelView process(Map<String, String> paramMap);
}
```
  + HttpServletRequest가 제공하는 파라미터 => Front Controller에서 paramMap에 담아서 호출
  
> 인터페이스 구현 controller
```java
public class MemberSaveControllerV3 implements ControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();
    @Override
    public ModelView process(Map<String, String> paramMap) { 
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);
        
        ModelView mv = new ModelView("save-result");
        mv.getModel().put("member", member);
        return mv;
    }
}
``` 
  + `paramMap` : 요청 파라미터의 정보가 담겨 있음
  + `save-result` : view의 논리적 이름으로 지정, 물리적 이름은 Front Controller에서 처리
  
> Front Controller 중 일부
```java
@Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        ...
        //paramMap을 넘겨주기(모든 파라미터 정보)
        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        String viewName = mv.getViewName();  //논리 이름 가져오기
        MyView view = viewResolver(viewName);

        view.render(mv.getModel(),request,response);  //모델(렌더링시 필요한 데이터 존재)도 함께 넘겨주기
    }

    //뷰의 논리이름으로 물리 경로를 생성해서 반환
    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    //모든 파라미터 정보 가져와서 Map에 담는 함수
    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                    .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
```
  + 맨 앞의 Front 컨트롤러에서만 servlet에 종속적이기 때문에, 여기서 모든 파라미터 정보를 Map에 담아 넘겨줘야함
  + 나머지 컨트롤러에서는 <뷰이름 + 렌더링 시 필요한 데이터>가 담긴 ModelView를 반환하는 역할만 함
 
#### :heavy_check_mark: 2) 뷰 이름 중복 제거: viewResolver 추가
  + 컨트롤러가 반한환 논리 뷰 이름을 변환 후, 실제 물리 뷰 경로가 있는 MyView 객체 반환
    + `MyView`: HTML 화면을 렌더링
  + `view.render(mv.getModel(),request,response)`: 이후 렌더링 시 필요한 데이터가 담긴 model도 함께 넘겨줘야함
   
> MyView 중 일부
```java
public class MyView {
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }    
    ...
    public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        modelToRequestAttribute(model, request);
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request,response);
    }

    private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
        model.forEach((key, value) -> request.setAttribute(key,value));
    }
}
```

### :pushpin: 단순,실용적 컨트롤러-V4
+ 구현 입장에서 ModelView를 직접 생성해서 반환하지 않도록 편리한 인터페이스 제공
+ **컨트롤러가 ModelView 를 반환하지 않고, ViewName 만 반환!**

> Controller Interface
```java
public interface ControllerV4 {
    String process(Map<String,String> paramMap, Map<String, Object> model);
}
```
  + model 객체가 외부에서 전달되기 때문에 그냥 사용하면 되고, 뷰 이름만 반환 

> Interface 구현 Controller
```java
public class MemberSaveControllerV4 implements ControllerV4 {

    private MemberRepository memberRepository = MemberRepository.getInstance();
    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);
        
        //변경 부분(모델 객체 생성 X)
        model.put("member", member);
        return "save-result";       //뷰 이름만 반환
    } 
```
> Front Controller 중 일부
```java
 @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ...
        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>();   //추가

        String viewName = controller.process(paramMap, model);   //파라미터와 함께 model(빈 객체)도 함께 전달

        MyView view = viewResolver(viewName);      //뷰 논리 이름을 직접 반환하기 때문에 바로 사용
        view.render(model,request,response);
    }
```

### :pushpin: 유연한 컨트롤러-V5
+ Adapter 도입 => 프레임워크를 유연하고 확장성 있게 설계
+ Front Controller가 한가지 방식이 아닌, 다양한 방식의 컨트롤러를 처리 가능하도록 변경!(ControllerV3,V4..)
<img src="https://user-images.githubusercontent.com/71436576/127876210-185a98f3-1227-4853-80fb-c4dcb3d16674.png" width=50% height=50%>

#### :heavy_check_mark: 1) Handler Adapter

> Handler Adapter 인터페이스
``` java
public interface MyHandlerAdapter {

    boolean supports(Object handler);  //handler == controller

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```
  + `supports()` : Adapter가 해당 컨트롤러를 처리 할 수 있는지 판단하는 메서드
  + `handle()` :  Front Controller가 아닌 **Adapter가 실제 컨트롤러를 호출**하고, 결과로 ModelView 반환

> ControllerV3HandlerAdapter.java
```java
public class ControllerV3HandlerAdapter implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {         //ControllerV3 인터페이스를 구현했는지 체크
        return (handler instanceof ControllerV3);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV3 controller = (ControllerV3) handler;     //캐스팅

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);         //Adapter에서 실제 컨트롤러 호출

        return mv;          //ModelView 반환
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }  
}
```

#### :heavy_check_mark: 2) Hadler
 + 컨트롤러를 직접 매핑해서 사용하지 않고, Adpater가 지원하는 한에서 어떤 것이라도 URL에 매핑해서 사용 가능(더 넓은 범위)

> Front Controller
```java
@WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {

    //모든 컨트롤러 지원(ControllerV4 -> Object)
    private final Map<String, Object> handlerMappingMap = new HashMap<>();      //핸들러 매핑(컨트롤러 매핑) 저장
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();   //핸들러 어댑터 리스트

    public FrontControllerServletV5(){
        initHandlerMappingMap();     //핸들러 매핑 초기화
        initHandlerAdapters();       //어댑터 초기화
    }

    private void initHandlerMappingMap() {
        handlerMappingMap.put("/front-controller/v5/v3/members/new-from", new MemberFormControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());

        handlerMappingMap.put("/front-controller/v5/v4/members/new-from", new MemberFormControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members/save", new MemberSaveControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members", new MemberListControllerV4());
    }
    
    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdapter());
        handlerAdapters.add(new ControllerV4HandlerAdapter());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Object handler = getHandler(request);              //1.핸들러 매핑

        if(handler == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyHandlerAdapter adapter = getHandlerAdapter(handler);   //2.핸들러 처리 가능한 어댑터 조회
        ModelView mv = adapter.handle(request, response, handler);   //3.실제 어댑터 호출(controller.process()과정)

        String viewName = mv.getViewName();       
        MyView view = viewResolver(viewName);      //4.논리 -> 물리 주소 변환

        view.render(mv.getModel(),request,response);  //5.뷰 렌더링(모델뷰 같이 전달)
    }

    private Object getHandler(HttpServletRequest request) {   
        String requestURI = request.getRequestURI();
        return handlerMappingMap.get(requestURI);
    }
    
    private MyHandlerAdapter getHandlerAdapter(Object handler) { 
        MyHandlerAdapter a;
        for (MyHandlerAdapter adapter : handlerAdapters) {
            if(adapter.supports(handler)){
                return adapter;
            }
        }
        throw new IllegalArgumentException("handler adapter 를 찾을 수 없습니다. handlers" + handler);
    }
    
    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}
```
  + `getHandler()` : URL에 매핑된 핸들러 객체 찾아서 반환
  + `getHandlerAdapter()` : 핸들러 처리 가능한 어댑터 찾기

> ControllerV4HandlerAdapter 중 일부
```java
    ModelView mv = new ModelView(viewName);

    mv.setModel(model);
    return mv;
```
 + :star2: Controllerv4는 모델뷰가 아닌 뷰의 '이름'을 반환해야함 => `Adpater` 덕분에 형식을 맞추어 반환 가능!
