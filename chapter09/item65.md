# 아이템 65. 리플렉션보다는 인터페이스를 사용하라

## 리플렉션의 사용 예 - Annotation을 이용한 DI
`DispatcherServlet`는 클라이언트의 요청의 URL에 따라 적절한 컨트롤러에게 작업을 위임하는 역할을 한다. 

다음은 Annotation을 이용한 컨트롤러 매핑 작업의 예이다. 

- `DispatcherServlet` 초기화
  ``` java
  @WebServlet(name = "dispatcher", urlPatterns = "/", loadOnStartup = 1)
  public class DispatcherServlet extends HttpServlet {
      // ...(생략)
      private List<HandlerMapping> mappings = Lists.newArrayList();

      @Override
      public void init() throws ServletException {
        AnnotationHandlerMapping ahm = new AnnotationHandlerMapping("next.controller");
        ahm.initialize();

        mappings.add(ahm);
        // ...(생략)
      }
      // ...(생략)
  }
  ```
  `DispatcherServlet`은 컨트롤러에 대한 리스트를 가지고 있고, `init` 메서드를 통해 초기화한다. 이 때, `AnnotationHandlerMapping`는 해당 패키지 내의 `@Controller` 어노테이션이 달린 클래스를 찾아 인스턴스화 하여 저장한다. 