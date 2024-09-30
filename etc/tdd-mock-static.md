# Mocking, static 메서드와 관련하여

토이 프로젝트 진행 중 '임시 비밀번호 생성' 기능을 개발하면서 TDD를 적용해보기로 했다.
진행 중 입력된 이메일에 대해 '이메일 형식 검증' 기능 리팩토링을 진행했고 이와 동시에 테스트 코드를 작성했다.

이메일 형식 검증 클래스를 유틸리티 클래스로 사용하기 위해 static 메서드로 개발하였는데,
테스트 과정에서 static 메서드를 일반 메서드처럼 mocking 하고자 했더니 오류가 발생했다.

하여 관련된 내용을 검색해보고 뤼튼에 물어본 결과를 적어둔다.

---

# 왜 static은 mocking이 어려울까
Mockito 3.4.0 버전 이전에는 PowerMockito를 이용하지 않으면 static 메서드에 관해서는 모킹이 어려웠다.

일반 mocking 라이브러리는 객체의 특정 메서드를 가로채어 원하는 동작을 수행하도록 만드는데,
이런 방식은 인스턴스 메서드에는 잘 동작하지만 static 메서드는 작용이 어렵다.

static 메서드가 객체 인스턴스와 관계 없이 클래스 자체에 속해 있기 때문이다.

# PowerMockito
클래스를 로딩하는 과정에서 바이트 코드를 조작하여 static 메서드, private 메서드, final 클래스 등을 모킹할 수 있다. JVM이 어떤 클래스를 로딩하고 실행할 때 원래의 메서드 대신 모킹된 클래스가 호출되도록 만들 수 있다.

Java에서 JVM이 클래스를 처음 참조할 때 해당 클래스의 바이트 코드를 메모리에 로딩하는데,
PowerMockito는 custom class loader를 사용하여 원래의 바이트 코드 대신 수정된 바이트 코드를 로딩한다.

PowerMockito는 설정도 복잡하고 유지 관리가 까다로워서 비추라고 하더라...
그럼 뭘 사용해야 하느냐,

# MockedStatic
mockito 라이브러리 기능 중의 하나로, Mockito 3.4.0 버전부터 도입되었다.

1. mockito-inline 를 `dependency`로 추가한다. 
2. try-with-resources 구문 내에 mockStatic() 메서드를 이용해 MockedStatic 인스턴스를 생성한다.
3. when() -> thenReturn() 또는 thenAnswer() 등을 사용하여 모킹한다.

2번에서 **try-with-resources를 사용하는 이유**   
MockedStatic< T > 객체는 AutoCloseable 인터페이스를 구현한다.   
하여 close() 메서드를 호출하여 static mocking을 종료해야 한다.

이때 수동으로 호출해도 되지만,   
try-with-resources가 자동 리소스 관리를 해주고   
중간에 예기치 못 한 오류가 발생할 수도 있기에 해당 구문 안에 넣어서 관리하자.

@BeforeEach, @BeforeAll에 넣어서 관리하면 안 될까?   
테스트 클래스에 n개의 테스트 메서드가 있고, 각 테스트마다 MockedStatic이 필요한 상황일 때를 말한다.   
나도 이 생각을 했다... After 어노테이션으로 close 해주면 되니까.   
그러나 각각의 테스트는 독립적이어야 한다는 뤼튼의 말에 따로따로 적어주기로 하자.

---

> 참고 url
https://www.baeldung.com/mockito-mock-static-methods