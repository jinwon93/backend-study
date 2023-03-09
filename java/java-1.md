#Java - 제네릭



##1. 제네릭
   1. 클래스나 메서드에서 사용할 내부 데이터 타입을 외부에서 지정하는 기법
   2. 타입을 파라미터화 해서 컴파일 시 구체적인 타입이 결정되도록 하는 것
   3. 타입 캐스팅으로 인한 자료형에 대한 검증은 컴파일 시 이뤄지지 않아 런타임에 오류가
      발생할 수 있지만 제네릭 사용시 컴파일 타임에 강한 타입 체크 가능
      불필요한 타입 변환 제거
        
    List list = new ArrayList();
    list.add("hello"); // object 타입으로 저장
    String str = (String) list.get(0);

    List<String> list = new ArrayList(); // 스트링 타입으로 지정
    list.add("hello"); // 스트링 타입으로 저장
    String str = list.get(0);
   4. add 시점에 String 타입으로 저장되는게 아니라 Object 타입으로 저장된다. 따라서 반환할 때도 Object 타입으로 리턴하기 때문에 String 타입으로 강제 타입 변환이 필요하다. 제네릭을 사용하면 강제 타입 과정을 줄일 수 있다.


##2. 제네릭 타입

    public class 클래스명<T>{...}
    public interface 인터페이스명<T>{...} 


- 타입을 파라미터로 가지는 클래스와 인터페이스 
- 일반적으로 대문자 알파벳 한문자로 표현

  + E : Element를 의미하며 컬렉션에서 요소임을 나타냄
  + T : Type을 의미
  + V : Value를 의미
  + K : Key를 의미
  + N  : Number를 의미
  + R : Result를 의미