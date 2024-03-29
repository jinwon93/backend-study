# JPA 기본 - 연관관계 매핑 기초

## 1. 단방향 연관관계
---
### 객체를 테이블에 맞추어 모델링
![그림1](https://github.com/backtony/blog-code/blob/master/jpa/img/2/4-1.PNG?raw=true)

위와 같이 객체를 테이블의 연관관계에 맞춰서 모델링할 경우 참조 대신에 외래 키를 그대로 사용하게 되면서 객체지향적이지 않은 코딩을 해야 한다. 예를 들어 member을 만들고 team에 넣고 persist했다고 가정하자. 멤버를 찾아서 멤버가 있는 팀을 확인하려면 먼저 멤버를 find로 찾아온 뒤에 가지고 있는 teamid(식별자)를 이용해서 다시 find로 team을 찾아야 한다. __즉, 객체를 테이블에 맞추어 모델링하면 협력 관계를 만들 수 없다.__ 테이블은 외래 키로 조인을 사용해서 연관된 테이블을 찾는 반면에 객체는 참조를 사용해서 연관된 객체를 찾기 때문에 둘 사이에는 큰 간격이 있다.
<br>

### 객체 지향 모델링
![그림2](https://github.com/backtony/blog-code/blob/master/jpa/img/2/4-2.PNG?raw=true)

객체와 테이블 사이에 간격을 위와 같이 해결할 수 있다. 그냥 객체는 객체대로 참조를 하고 테이블은 테이블대로 조인으로 연관관계를 형성하게 하고 객체에서 애노테이션을 이용해 조인시 사용할 FK를 매핑해주면 된다.
```java
// Member와 Team 클래스가 Entity 애노테이션을 붙였다고 가정

@Entity
public class Member {    
    // 다른 필드 생략

    /* 
    @Column(name="TEAM_ID")
    private Long teamId;
    위는 객체를 테이블에 맞추어 모델링 한 것인데
    협력 관계를 만들 수 없다.
    */
 
    @ManyToOne // 항상 자기 자신을 기준으로
    @JoinColumn(name="TEAM_ID")
    private Team team;
}
```
먼저 ManyToOne 애노테이션이 없다면 Team에 밑줄이 그어진다. jpa가 아니라면 밑줄이 그어지지 않았을 것이다. 그저 다른 클래스를 가져다 쓰는 것이니 말이다. 밑줄이 그어진 이유는 Team 클래스와 Member 클래스 둘 다 Entity 애노테이션으로 jpa, db에서 사용할 것이라고 명시해줬는데 Member가 Team을 가지고 있으니 서로 무슨 관계냐고 묻는 것이다. 즉, 객체 연관관계는 문제가 없으나 db 테이블에서의 관계가 정해지지 않았으니 정해달라는 것이다. 따라서 @ManyToOne 애노테이션으로 테이블의 연관관계를 지정해준 것이다. 이러한 연관관계는 @ManyToOne @OneToMany @ManyToMany가 있는데 항상 자기 자신을 기준으로 생각하면 된다.  
@joincolumn 생략해도 알아서 일쪽의 PK가 다쪽의 FK로 매핑이 된다. 양방향 관계에서는 mappedby를 사용해 연관관계의 주인을 지정해주기 때문에 굳이 @joincolumn를 사용할 필요가 없지만 name 속성을 사용하면 fk 이름을 지정할 수 있다.  
기본적으로 name 속성을 주지 않으면 일쪽의 테이블명_id 이렇게 들어간다. 만약 @Column의 name 속성으로 일쪽의 id 칼럼명을 바꿔줬다면 테이블명+바꾼이름 으로 들어간다.
단방향 관계에서는 @joincolumn를 명시한 쪽이 연관관계의 주인이 된다. 다대일 단방향 관계에서는 @joincolumn를 명시하지 않아도 중간테이블이 생성되지 않기 때문에 굳이 @joincolumn를 명시하지 않아도 된다. 하지만 일대다 단방향 관계에서는 @joincolumn를 명시하지 않으면 중간테이블이 생성된다. 중간테이블 관리가 불편할 경우 @joincolumn를 명시해줘야 한다.  
하지만 이런 모든 과정을 생각해서 코딩하는 것을 실수를 유발하기 딱 좋다. @joincolumn을 이용해 name속성을 전부 명시해줘도 상관 없으므로 웬만하면 전부 명시해서 사용하는 것이 좋다.
```java
alter table Member
  add constraint 외래키 명
  foreign key (Team_id) // fk
  references Team (id) // pk
```
참고로 Table을 만들 때 아래 두 가지 경우가 있다.
+ joinColumns = @JoinColumn : 현재 엔티티를 참조하는 외래키 -> 현재 엔티티의 pk를 fk로 참조한다고 보면 된다.
+ inverseJoinColumns = @JoinColumn : 반대방향 엔티티를 참조하는 외래키 -> 반대방향 엔티티의 pk를 fk로 참조한다고 보면 된다.

<br>

이런 설계로 인해 이제 멤버가 소속된 팀을 찾으려 할 때 find로 멤버를 찾아온 뒤에 식별자로 다시 찾아오는게 아니라 바로 getTeam으로 꺼내서 소속 팀을 확인할 수 있다.


<br>

## 2. 양방향 연관관계와 연관관계의 주인
---
### 양방향 연관관계
![그림3](https://github.com/backtony/blog-code/blob/master/jpa/img/2/4-3.PNG?raw=true)

양방향 연관관계는 위와 같이 양쪽으로 이동할 수 있는 관계를 말한다.
```java
@Entity
public class Team {
    // 나머지 필드 생략

    @id
    @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "team")
     //관례상 arraylist로 초기화
     // add할때 nullpoint 안뜨게 하기 위해
    private List<Member> members = new ArrayList<>();
}
```
앞선 단방향 관계에서 설명했듯이 Team도 jpa에서 사용하도록 Entity 애노테이션으로 명시해뒀는데 Member라는 Entity를 가지고 있으니 서로 어떤 관계인지 명시해줘야 한다. 따라서 Team 자기 자신을 기준으로 보면 Team과 Member는 일대다 관계이니 @OneToMany 애노테이션을 사용한다.  
<br>



### 연관관계의 주인
두 객체 연관관계 중 하나를 정해서 테이블의 외래키를 관리해야 하는데 외래키를 관리하는 쪽을 연관관계의 주인이라고 한다. 양방향 연관관계에서는 mappedBy가 연관관계의 주인을 지정하는 애노테이션이다. 양방향 연관관계에서는 객체와 테이블간에 간격이 있다. 객체를 기준에서 보면 객체끼리 양방향 연관관계 즉, 서로를 참조하기 위해서는 서로 다른 내용을 가지고 있어야 한다. 예를 들면, Member는 Team을 참조하기 위해서는 Team을 가지고 있어야하지만, Team은 Member을 참조하기 위해서는 Member을 가지고 있어야 한다. 하지만 테이블의 경우는 단 하나의 FK만 있으면 양쪽을 서로 오갈 수 있다.  
__정리하자면, 객체 기준에서는 양방향 연관관계를 가지기 위해서 단방향 연관관계를 2개 만들어야 하지만(서로가 서로를 갖고 있어야 하지만) DB 상 테이블의 양방향 연관관계는 외래 키 하나로 해결할 수 있다는 것이다.__  
이제 여기서 문제가 생긴다. 만약 객체기준에서 Member가 팀을 바꾸고 싶다면 Member클래스에서 team을 변경해줘야 할까, Team에서 members를 변경해줘야 할까? 객체 기준에서는 둘 다 처리해주는게 맞다. 하지만 테이블 기준에서는 둘 다 고려할 필요가 없다. 일단 Member 테이블에 FK가 존재하는데, 객체상에서는 Member 객체에서 team을 바꿔도 FK가 변경되어야 맞고, Team에서 members를 바꿔도 해당 member의 FK가 변경되어야 한다. 결과적으로는 이 둘중 어느 한쪽에서 DB 테이블의 FK를 변화시켜줄 건지 한곳에서만 관리하라고 정하는 것이 연관관계 주인이다.  
다시말해, DB의 Member 테이블에 FK를 객체기준에서 FK 연관관계 매핑을 해줘야 하는데 이 외래키를 객체상의 Member 클래스에서 관리할지 Team에서 관리할지 정해줘야 된다는 것이다.  
간단히 정리하자면, 양방향 연관관계를 위해서 객체 연관관계는 2개의 참조가 필요한데 테이블 연관관계는 1개만 필요하므로 객체 2개의 참조 중 하나를 선택하는 것이 연관관계 주인을 정하는 과정이다.  
양방향 매핑 규칙 = 연관관계 주인 특징은 다음과 같다.
+ 객체의 두 관계중 하나를 연관관계 주인으로 지정
+ 연관관계의 주인만이 외래 키를 관리(등록,수정)
+ __주인이 아닌쪽은 읽기만 가능__
    - 위의 예시에서 보면 Team에 있는 members는 조회만 가능하고 db에 값을 변경할 때는 주인인 Member의 team을 사용해야 한다.
    - __주인이 아닌 쪽에서는 읽기만 가능하고 쓰기작업을 진행해도 DB상에 적용되지 않는다.__
+ 주인이 아닌 쪽에 mappeedBy으로 자신이 무엇에 의해 매핑이 되었음을 명시 -> 자신의 주인을 지정하는 것

### 주인 지정법
__외래 키가 있는 곳을 주인으로 지정해라__  
위에서 보면 FK 매핑할 때 Member 엔티티랑 Member테이블을 매핑했다 그럼 사실 FK도 Member에 있는 것이다. 만약 Team을 연관관계 주인으로 지정하면 나중에 Team의 members의 값을 바꿨는데 db관점에서는 Team이 아니라 Member 테이블로 쿼리가 나가게 되기에 헷갈리는 부분도 많고 성능이슈도 있기 때문에 그냥 외래 키가 있는 곳을 주인으로 지정하는게 좋다.  
좀 더 구체적으로 공식화해보면, db입장에서 외래 키가 있는 곳이 무조건 N이다.(N:1 관계) 즉, db의 N쪽이 무조건 연관관계 주인이 되는 것이다.


__간단히 정리하면, 양방향 연관관계 N:1에서 주인을 N으로 만들면 된다. -> N:1에서 1에다가 mappedby 사용__

<br>

### 주의사항
#### 수정 관련
앞서 설명했듯이 mappedBy로 매핑된 순간 해당 코드에 수정, 삽입을 하는 것은 DB에 아무런 영향을 주지 못한다. 예를 들면, Team의 members에 mappedBy 애노테이션을 준 경우, team.getMembers().add(member) 코드를 작성하면 객체 상에서는 추가되지만, mappedBy 애노테이션이 붙어있기에 db상에서는 적용이 안된다. 따라서 db상에 반영시켜주기 위해서는 연관관계 주인인 Member의 team에서 작업을 해야 한다. member.setTeam(team)을 해주면 알아서 db상의 Team테이블에 members에 적용이 된다. 비유하자면 연관관계 주인이 아닌 것은 연관관계 주인의 거울이 된 것이라 보면 된다.  
<br>

#### 값의 세팅 관련
그럼 객체에서는 연관관계 주인에만 값을 세팅해주면 될까? 사실 db관점에서는 그렇다고 볼 수 있다. 하지만 여러가지 문제로 객체상 양쪽에 모두 값을 세팅해줘야 한다. 예를 들어, team을 만들고 memebers는 세팅하지 않은 상태에서 persist하고 member을 만들어 team을 세팅하고 persist 했다고 한다면 사용자는 member가 연관관계 주인이니 team을 세팅해서 보냈으므로 db에 team 테이블에 members가 반영되었다고 생각할 수 있다. 하지만 실제는 아직 flush가 나가지 않았으므로 db로 쿼리가 나가지 않고 영속성 컨텍스트의 1차 캐시에 있을 것이다. 이때 team을 find하게 되면 db가 아니라 영속성 컨텍스트의 1차 캐시에서 가져오게 된다. 즉, 이때는 db team의 members값에는 아직 업데이트가 되지 않은 상태이므로 그냥 빈 team을 가져오게 되는 것이다. 따라서 문제가 생기게 된다. find로 team을 가져오면 영속성 컨텍스트에서 가져오게 되고 아직 쿼리들이 db로 날라가지 않았으니 find로 가져온 team은 members가 세팅이 안된 team을 가져오게 된 것이니 team.getMembers 하면 아무것도 안찍힌다는 것이다. 또, 테스트 작성에도 문제가 생긴다. 테스트 케이스는 jpa없이도 가능해야 하는데 jpa가 없으면 당연히 한쪽에만 값을 넣은 것이니 반대쪽은 값이 null일 것이다. 이러한 문제들 때문에 양쪽에 값을 세팅해주는게 옳다.
```java
member.setTeam(team);
team.getMembers().add(member);
```
세팅은 위와 같이 할 수 있는데 사실 로직이 저렇게 되어 있으면 사람인지라 한쪽만 세팅하고 실수할 수도 있다. 따라서 한 번의 처리로 저 과정을 해주는 메서드를 따로 만들어 주는게 좋다. 예를 들면 다음과 같다.
```java
// 연관관계 편의 메서드
public void changeTeam(Team team){
  this.team = team;
  team.getMembers().add(this);
}
```
이 메서드를 주인쪽에 만드냐, 아닌쪽에 만드냐는 정해진 것이 아니라 상황에 따라 어느쪽에 만들지 선택하면 된다. 위 코드는 Member에 만든 것이다. set이 아니라 change를 이름으로 사용한 이유는 단순히 세팅이 아니라 어느정도 로직이 들어있기 때문이다. 위와 같이 메서드를 만들어주면 메서드 호출로 한 번에 양쪽 값을 세팅할 수 있기 때문에 더 좋은 방법이다.  
<br>

프로젝트를 진행하면서 느낀건데 만약 작업이 끝나고 트랜잭션이 닫히게 된다면 굳이 세팅할 필요는 없는 것 같다.(참고로 각 트랜잭션은 각각의 영속성 컨텍스트를 가지고 있어 다른 곳에서 참조할 때의 문제는 걱정안해도 된다.)  
위 작업은 생성할 때의 작업이었는데 만약 수정하기 위해서 가져온다고 했을 때 딱 수정을 하기 위해서 불필요하게 패치조인으로 끌고와야 하는 문제가 있다.  
수정작업만 딱 하고 트랜잭션을 닫는다면 굳이 양쪽을 수정할 필요는 없어보인다.

<br>

#### 무한 루프
__양방향 관계에서 toString, lombok, JSON 생성 라이브러리에서 무한 루프를 주의해야 한다.__  
예를 들어, toString으로 Member을 찍으면 team의 toString을 호출할텐데, 그럼 team에서는 members에서 tostring을 호출하게 되니 무한 루프가 된다.  
또한, 컨트롤러에서 response로 Entity를 직접 보낼때 양방향 연관관계가 걸려있으면 Entity가 JSON으로 바뀌면서 Member는 team이 있고 team에 가니 members가 있고 toString처럼 무한 루프가 된다. 그리고 Entity를 반환해버리면 Entity가 수정되는 순간 API스펙이 변한다.
해결책으로 lombok은 toString은 웬만하면 사용하지 않는것, JSON의 경우는 컨트롤러에는 Entity를 절때 반환하지 않는 것이다. eneity가 변경될 수 있는데 api에 반환해버리면 entity를 변경하는 순간 api 스펙이 변하기 때문에 entity는 dto로 변환해서 반환하는 것이 좋다.
<br>

## 3. 정리하기
---
+ 단방향 매핑만으로도 이미 연관관계 매핑은 완료된 상태라고 볼 수 있다.
    - JPA 모델링 할 때, 단방향 매핑으로 처음에 설계를 끝내야 한다.
    - 양방향 매핑은 반대 방향으로 조회 기능만 추가된 것일 뿐
    - 양방향 연관관계를 만드는 이유는 개발상 편의와 조회를 위한 것
    - 결론은 최대한 단방향, 실무에서 JPQL 작성 편의, 조회의 편의상 양방향 관계를 추가하게 된다.
+ 단방향 매핑을 잘 하고 양방향은 필요할 때 추가해도 된다.
    - 단방향과 양방향의 테이블은 차이가 없다. 단지 객체만 차이가 있을 뿐
+ 외래 키의 위치가 연관관계 주인이 된다. -> 외래 키 위치가 아닌 반대쪽에 mappedBy 사용