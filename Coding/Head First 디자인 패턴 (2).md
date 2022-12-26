# [Head First 디자인 패턴 (2)]

## 옵저버 패턴

**정의**

한 객체의 상태가 바뀌면, 이 객체에 의존하는 다른 객체들의 내용이 갱신되는 방식이다.

(일대다 의존성을 정의한다.)

**상황**

북클럽 스터디를 운영하는 리더(진행자 ㅎ)가 있다고 하자,

스터디 스케줄이 변경되어 이를 다른 스터디 멤버들에게 알려야 한다.

**기존 코드**

```java
package designpattern.observer.bef;

public class StudyLeader {

    //스케쥴 시간이 담겨있다.
    private String scheduleMessage = "2022-12-08 21시";
    
		//각 멤버가 굉장히 다른 형태라고 상상해보자!
    private StudyMember1 studyMember1 = new StudyMember1();
    private StudyMember2 studyMember2 = new StudyMember2();
    private StudyMember3 studyMember3 = new StudyMember3();

    public String getScheduleMessage() {
        return scheduleMessage;
    }

    public void scheduleChanged() {
        String schedule = getScheduleMessage();

        //스케줄이 변경된다면, 스터디원들에게 알려줘야한다.
        studyMember1.update(schedule);
        studyMember2.update(schedule);
        studyMember3.update(schedule);

    }
}
```

**기존 코드의 문제점**

1. 구체적인 구현에 의존한다.
    
    (DIP 원칙 위배)
    
2. 멤버 추가/삭제에 일일이 대응해야 한다.
3. 실행 중(런타임)에 멤버 추가/삭제에 대응할 수 없다.

**변경된 코드**

**StudyLeader.java**

```java
public class StudyLeader implements Subject{

    private String scheduleMessage = "2022-12-08 21시";
    private List<Observer> observers;

    public StudyLeader() {
        this.observers = new ArrayList<>();
    }

		//가입
    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }
		//제거
    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }
		//변경사항 알림
    @Override
    public void notifyObservers() {
        observers.forEach(observer -> observer.update(scheduleMessage));
    }

    //스케줄을 변경하고, 옵저버들에게 알린다.
    public void updateSchedule(String updateSchedule) {
        this.scheduleMessage = updateSchedule;
        notifyObservers();
    }
}
```

- observers에 현재 옵저버들을 관리하고 있다.
- 스케줄 변경 시 observers를 이용하여, 변경사실을 알린다.

**Main.java**

```java
//스터디 멤버목록이다.
List<StudyMember> studyMembers = List.of(studyMemberA, studyMemberB, studyMemberC);

//A,B,C 모두 가입한다.
registerAll(studyLeader, studyMembers);

//구독자들(A,B,C) 모두 알리고 기억하고 있는 스케줄을 모두 출력한다.
studyLeader.notifyObservers();
printRememberSchedule(studyMembers);

//C를 구독자에거 제거한다.
studyLeader.removeObserver(studyMemberC);

//새로운 스케줄로 변경한다.
studyLeader.updateSchedule("2022-12-15 21시");
System.out.println("\n일정 변경 발생\n");

//A,B,C 모두 알리고 기억하고 있는 스케줄을 모두 출력한다.
printRememberSchedule(studyMembers);
```

- 옵저버 A,B,C 중 C가 구독을 탈퇴하였다.
- C만 변경된 스케줄 알림을 받지 못한다.

```java
A, schedule = 2022-12-08 21시
B, schedule = 2022-12-08 21시
C, schedule = 2022-12-08 21시

일정 변경 발생

A, schedule = 2022-12-15 21시
B, schedule = 2022-12-15 21시
**C, schedule = 2022-12-08 21시**
```

**옵저버 패턴의 구성**

![Untitled](5%E1%84%8C%E1%85%AE%E1%84%8E%E1%85%A1%20%5BHead%20First%20%E1%84%83%E1%85%B5%E1%84%8C%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%AB%20%E1%84%91%E1%85%A2%E1%84%90%E1%85%A5%E1%86%AB%20(2)%5D%20682679a8837c4fd79557d0e5a80711d8/Untitled.png)

- Observer, Subject로 나뉜다.
- Observer이 Subject를 구독한다.
- Subject의 변경사항을 Observer에게 알린다.

**옵저버 패턴의 효과**

- 느슨한 결합을 만든다.
    - 느슨한 결합: 상호작용을 하는 객체가, 서로에 대해 잘 모른다. (리더와 멤버)

**느슨한 결합이 주는 장점**

1. 주제(발행자, 리더)는 **옵저버에 대해 아는 것이 거의 없다.** Observer 인터페이스를 구현했음 만을 안다. → **옵저버 형식이 변하더라도, 대응하기 쉽다.**
2. 옵저버(멤버) 목록의 변경이 일어나도, **주제(리더)의 코드변경은 일어나지 않는다.**
3. 주제와 옵저버 모두 재사용이 가능하다.
4. 주제, 옵저버가 변하더라도 서로에게 영향을 주지 않는다.

### **JDK에서의 옵저버 패턴**

Java.utils에서 제공하는 Observer 인터페이스, Observable 클래스를 이용하여, 옵저버 패턴을 활용할 수 있다.

**단점**

- `Observable`이라는 클래스를 항상 extends 상속해야 한다 → 재사용성이 떨어진다.
- `Observable`의 `setChanged()`는 `protected` 로 되어 있다. → 외부에서 호출이 불가능하다.
    - `setChanged()`: Subject의 변화가 생길 때만 알린다.

**정리**

- 단점들이 부각되지 않는다면 사용해도 나쁘지 않다.

## 데코레이션 패턴

**정의**

객체에 추가적인 첨가물을 동적으로 추가하는 방법이다.

**상황**

다양한 종류의 커피가 있고, 첨가물(우유, 모카 휘핑 크림 등)에 따라 가격이 달라져야 한다.

**기존 코드**

**Beverage.java**

```java
public class Beverage {
		//설명이 저장될 문자열
    String description;

    private boolean milk;
    private boolean soy;
    private boolean mocha;
    private boolean whip;
		
		// 가격을 계산한다.
    public int cost() {
        int ret = 0;

        if(milk) {
            ret += 300;
        }
        if(soy) {
            ret += 500;
        }
        if(mocha) {
            ret += 100;
        }
        if(whip) {
            ret += 500;
        }

        return ret;
    }
```

**Main.java**

```java
Beverage espresso = new Espresso();

espresso.setMilk(true);
espresso.setSoy(true);
espresso.setMocha(true);

System.out.println("espresso.getDescription() = " +
	 espresso.getDescription());
System.out.println("espresso.cost() = " + espresso.cost());
```

**문제점**

변화에 취약하다.

1. 첨가물 가격이 변동
2. 새로운 첨가물 추가
3. 첨가물을 2번 추가한다면..?! 

**변경된 코드**

**CondimentDecorator.java**

```java
public abstract class CondimentDecorator extends Beverage {
}
```

**Milk.java**

```java
public class Milk extends CondimentDecorator {
    Beverage beverage;

    public Milk(Beverage beverage) {
        this.beverage = beverage;
    }

    public String getDescription() {
				//기존 설명에 우유 첨가물 설명을 덧붙인다.
        return beverage.getDescription() + ", 우유";
    }

    public int cost() {
				//기존 가격에 우유 첨가물 가격을 덧붙인다.
        return beverage.cost() + 300;
    }
}
```

**Mocha.java**

```java
public class Mocha extends CondimentDecorator {
    Beverage beverage;

    public Mocha(Beverage beverage) {
        this.beverage = beverage;
    }

    public String getDescription() {
        return beverage.getDescription() + ", 모카";
    }

    public int cost() {
        return beverage.cost() + 100;
    }
}
```

**Main.java**

```java
Beverage espresso = new Milk(new Soy(new Mocha(new Espresso())));

System.out.println("espresso.getDescription() = " + espresso.getDescription());
System.out.println("espresso.cost() = " + espresso.cost());
```

**출력**

```
espresso.getDescription() = 에스프레소, 모카, 두유, 우유
espresso.cost() = 2900
```

**데코레이터 패턴의 구성**

![Untitled](5%E1%84%8C%E1%85%AE%E1%84%8E%E1%85%A1%20%5BHead%20First%20%E1%84%83%E1%85%B5%E1%84%8C%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%AB%20%E1%84%91%E1%85%A2%E1%84%90%E1%85%A5%E1%86%AB%20(2)%5D%20682679a8837c4fd79557d0e5a80711d8/Untitled%201.png)

- 데코레이터 객체는 기존의 객체를 감싼다.
- 데코레이터 객체는 기존의 객체와 형식(타입)이 같다.

**데코레이터 패턴의 효과**

- 원본 객체를 멤버변수의 형태로 이용한다. → 유연성을 잃지 않는다.
- 새로운 데코레이터를 만들어, 추가기능을 생성하기 쉽다.

**데코레이터 패턴의 단점**

![Untitled](5%E1%84%8C%E1%85%AE%E1%84%8E%E1%85%A1%20%5BHead%20First%20%E1%84%83%E1%85%B5%E1%84%8C%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%AB%20%E1%84%91%E1%85%A2%E1%84%90%E1%85%A5%E1%86%AB%20(2)%5D%20682679a8837c4fd79557d0e5a80711d8/Untitled%202.png)

- 클래스 구성이 복잡해진다.
- 클라이언트는 데코레이터 패턴 사용 유무를 알 수 없다.
    - 의존성 있는 코드에 데코레이터가 적용되면 문제가 발생한다.
- 호출하는 코드가 복잡해진다.
    - `Beverage espresso = new Milk(new Soy(new Mocha(new Espresso())));`

## 빌더 패턴

**정의**

복합 객체의 생성 과정과 표현 방법을 분리하여 동일한 생성 절차에서 서로 다른 표현 결과를 만들 수 있게 하는 패턴

= 사용하고자 하는 객체를 생성하는 또 다른 객체를 도입하는 패턴이다.

**기존 코드**

**User.java**

```java
public class User {
    private String firstName;
    private String secondName;
    private String nickname;

    public User(String firstName, String secondName, String nickname) {
        this.firstName = firstName;
        this.secondName = secondName;
        this.nickname = nickname;
    }

    @Override
    public String toString() {
        return "User{" +
                "firstName='" + firstName + '\'' +
                ", secondName='" + secondName + '\'' +
                ", nickname='" + nickname + '\'' +
                '}';
    }
}
```

**Main.java**

```java
User user = new User("Hansu", "Park", "Invidam");
```

: First name, Second name, Nickname… 순서가 헷갈린다.

**변경된 코드**

**UserBuilder.java**

```java
public class UserBuilder {

    String firstName;
    String secondName;
    String nickname;

    public UserBuilder firstName(String firstName) {
        this.firstName = firstName;

        return this;
    }
    public UserBuilder secondName(String secondName) {
        this.secondName = secondName;

        return this;
    }
    public UserBuilder nickname(String nickname) {
        this.nickname = nickname;

        return this;
    }
    public User build() {
			 //빌더를 통해 User을 만드는 생성자를 User.java에 추가해뒀다.
        return new User(this);
    }
}
```

**Main.java**

```java
User user = new UserBuilder()
                .firstName("Hansu")
                .secondName("Park")
                .nickname("Invidam")
                .build();
```

: First name, Second name, Nickname… 순서가 명확하다!

**빌더 패턴의 구성**

객체 생성자의 인자 각각을 설정(set)하고, 빌더 자신를 반환하는 빌더를 만든다.

1. `XXX.setA().setB()` 를 반복하여 빌더 필드에 데이터를 넣는다.
2. `XXX.build()` 를 통해, 필드에 있는 데이터들을 이용하여 사용할 객체를 만든다.

**빌더 패턴의 효과**

1. 가독성의 향상 (vs 생성자 방식)
2. 세터의 사용을 억제할 수 있다. (vs 세터 방식)
3. 유연하고 관리하기 쉽다.
    - 생성과정을 빌더에 캡슐화 → 생성과정 변경이 생겨도 대응하기 쉽다.