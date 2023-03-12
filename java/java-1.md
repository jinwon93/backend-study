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
- 코드에서 타입 파라미터 자리에 구체적 타입을 지정 

    
    // 제네릭 T를 사용할 것임을 명시
    public class Box<T> {
        pricate T t;
        public T get() {return t;}
        public void set(T t) {this.t = t;}
    }
    
    // T에 구체적 타입 지정
    Box<String> box = new Box<>();
    
    
    // 제네릭 클래스가 아닌 메서드만 제네릭으로 사용하기 - 제네릭 메서드
    class Name{
    public <T> void printClassName(T t){...}
    }
    
    // 클래스에서 제네릭 T를 사용할 것을 명시했으나, 제네릭 메서드에서는 다른 T를 사용하고 싶을 경우
    public Name<T>{
    // 메서드 앞에 다른 T를 사용하겠다고 명시 -> 이를 제네릭 메서드라고 함
    public <T> void printClassName(T t){..}
    }
    
    Name<String> name = new Name<>(); // 클래스의 제네릭 타입은 String 이지만
    name.printClassName(3.14); // 메서드에 타입을 Double로 넣었으므로 반환값은 double
    name.printClassName(1); // 메서드에 타입을 Integer 로 넣었으므로 반환값은 Integer

- 제네릭 클래스가 아닌데 제네릭 메서드를 사용할 경우에는 반환타입 앞에 <T>와 같이 명시하면 된다.
- 제네릭 클래스와 제네릭 메서드에 각각 제네릭 타입 매개변수가 정해져있다면 메서드에서는 메서드 타입으로 들어온 것을 따라간다
    
## 3. 멀티 타입 파라미터

    class <K,V,...> {...}
    interface <K,V,...> {...}
    
    // 예시
    public class Product<T,M>{
    private T kind;
    private M model;
    }
    
    product<Tv,String> product = new Product<>();

- 두 개 이상의 타입 파라미터를 사용해서 선언할 수 있다.

## 4. 제네릭 메서드

    public <타입파라미터,...> 리턴타입 메서드명(매개변수,...) {...}
    
    // 예시
    // 타입 파라미터에 T를 사용할 것이라고 앞에 기술해준 것이라 보면 된다.
    public<T> box<T> boxing(T t) {...}
    
    // 함수 호출
    // 타입파라미터로 T를 추정한다.
    Box<Integer> box = boxing(100); // T를 Integer로 추정 

#####  매개변수 타입과 리턴 타입으로 타입 파라미터를 갖는 메서드 

- 제네릭 메서드 선언 방식
  + 리턴 타입 앞에 < > 기호를 추가하고 타입 파라미터 기술
  + 기술한 타입 파라미터는 리턴타입과 매개 변수에 사용 가능

### 정적 제네릭 메서드 주의사항 

    public static void print(T param) {
    System.out.println(param.hello(0));
    }

위 코드는 두 가지 오류가 있다.

 1. T타입이 무엇인지 알 수 없기 때문에 hello 메서드를 호출할 수 없다.
 2. 타입 파라미터에 대한 정의가 없다.
 
정적(static) 메서드이므로 인스턴스에 관계없이 클래스에 붙어있는 메서드인데, 위 코드에서 사용하고 있는 T는 클래스에 표시하는 <T>로 인스턴스마다 달라지는 타입 파라미터이다.
즉, 정적 메서드에 인스턴스의 변수로 여겨지는 T를 타입 파라미터로 사용하고 있으므로 컴파일 에러가 난다.
따라서 정적 제네릭 메서드는 아래와 같이 사용해야 한다.


    public static <T extends Course> void print(T param) {
    System.out.println(param.hello(0));
    }



## 5. 제한된 타입 파라미터

    public <T extends 상위타입> 리턴타입 메서드(매개변수,...) {...}
- 상속 및 구현 관계를 이용해서 타입을 제한하는 방식 
- 상위 타입 부분에는 상위 클래스와 인터페이스를 지정 할 수 있다
    + 상위 타입 자리에 클래스가 오게 되면 T에는 상위 타입 클래스나 하위 타입 클래스가 올 수 있다 
    + 상위 타입 자리에 인터페이스가 온다면 T에는 인터페이스의 구현체가 올 수 있다.
- 실행 블록 안에서는 상위 타입의 필드와 메서드만 사용이 가능하다 


## 6. 와일드카드 타입

+ 제한된 타입 파라미터와 마찬가지로 파라미터나 리턴타입에 제한을 걸 때 사용한다.
+ 3가지 형태
    - 제네릭타입< ? > : 제한 없음
        - 타입 파라미터를 대치하는 것으로 모든 클래스나 인터페이스 타입이 올 수 있다.
    - 제네릭타입< ? extends 상위타입>
        - 상위 타입을 포함하여 하위 타입만 올 수 있다.
        - get한 원소는 상위타입이다. 이는 어떤 하위 타입인지 모르기 때문에 잘못된 타입 캐스팅을 막기 위함이다.
    - 제네렉타입< ? super 하위타입>
        - 하위타입을 포함하여 상위 타입만 올 수 있다.
        - get한 원소는 object로 나오게 된다.


public class Calcu {
    public void printList(List<?> list) {
        for (Object obj : list) {
            System.out.println(obj + " ");
        }
        //list.add("hello") -> 불가능 -> ?이므로 어떤 타입일지 모르기 때문에 null을 제외한 어떤 것도 넣을 수 없음 
    }

    public int sum(List<? extends Number> list) {
        int sum = 0;
        for (Number i : list) {
            sum += i.doubleValue();
        }
        // list.add(a); ?가 무엇인지 알 수 없기 때문에 null을 제외한 아무것도 넣을 수 없음
        return sum;
    }

    public List<? super Integer> addList(List<? super Integer> list) {
        for (int i = 1; i < list.size(); i++) {
            list.add(i); // ?는 무조건 Integer를 포함하는 타입이기 때문에 Integer만 삽입 가능
        }
        return list;
    }
}

//주의사항
public class Main {
    public static void printList(List<Object> list) {
        list.forEach(System.out::println);
    }

    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3);
        printList(numbers); // 컴파일 에러 
    }
}
```
위 코드는 컴파일 에러가 발생한다.  
Object는 모든 클래스의 상위 타입이므로 Integer를 받을 수 있지 않느냐고 생각할 수 있지만, 그건 Object와 Interger의 관계이다.  
즉, List\<Object>와 List\<String>과는 전혀 관계 없는 이야기다.  
__따라서 List\<Object>는 List\<Integer>, List\<String>등 모든 타입 List의 상위 타입이 아니라는 점을 기억해야 한다.__

### 와일드 카드를 사용할 때와 사용하지 않을 때의 차이
---

public <E extends People> void printList(List<E> list1, List<E> list2) {
    ...
}

와일드 카드를 사용하지 않으면 위와 같이 파라미터에 직접 extends로 제한을 줄 수 없다.  
또한, 두 개의 파라미터 List에는 같은 타입이 와야만 한다.

public void printList(List<? extends Number> list, List<? extends String> list2) {
    ...
}

반면에 와일드 카드를 사용하면 파라미터에 직접 extends를 사용할 수 있고, 파라미터 마다 다른 타입을 받을 수 있다.