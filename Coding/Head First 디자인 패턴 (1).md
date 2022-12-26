# [Head First 디자인 패턴]

## 책 소개

**Head First 시리즈만의 특징은?**

두뇌 활등을 증가시키기 위하여 다양한 장치들(e.g. 문자, 대화체 등)을 이용한다.

**대상**

우테코 프리코스를 진행하며 객체 지향에 대한 이해가 생긴 분들

**좋았던 점**

1. 신선한 방식으로 내용을 소개한다.

   지루한 나열이 아닌, 온갖 장치들을 이용하여 흥미롭게 내용을 이끌어간다.

   A. **인터뷰 방식**: 헷갈려서 자주 비교되는 두 대상을 이해시키기 위하여 두 대상(?)에게 인터뷰를 진행한다.

   B. **바보같은 질문은 없습니다** 코너: 헷갈리고 궁금할만한 내용들을 설명해준다.

2. 변화에 대응하는 경험을 준다.

   객체 지향의 변화에 쉽게 대응할 수 있다는 장점은 혼자서 공부하는 경우 느끼기기 쉽지 않다. 이 책에서는 다양한 요구사항과 변경사항을 제시하며, 변화에 대응하는 경험을 준다.

**아쉬웠던 점**

내용이 산만하여, 별도의 정리가 필요하다고 느껴졌다.

생각보다 내용이 어려웠다.

## 책 내용 소개

### 디자인 패턴이란?

> 프로그램을 개발하는 과정에서 **빈번하게 발생하는** 디자인 상의 **문제를** 정리해서, 상황에 따라 **간편**하게 적용해서 쓸 수 있는 **패턴 형태로 만든 것.**

### 어떠한 이점이 있을까?

- 간단한 용어를 통하여 복잡한 내용을 축약할 수 있다.
  사용자 인터페이스로부터 비즈니스 로직을 분리하여 애플리케이션의 시각적 요소나 그 이면에서 실행되는 비즈니스 로직을 서로 영향 없이 쉽게 고칠 수 있는 애플리케이션을 만드는 패턴
  = **MVC 패턴**
- 복잡한 객체지향 시스템을 편리하게 사용할 수 있다.
  - 남이 빠졌던 함정에 빠지지 않을 수 있다. (?)

### 스트래티지(전략) 패턴

**변경**할 가능성이 **높은 기능**들을 **캡술화**하여 **교환**이 가능하도록(갈아끼울 수 있도록) 만드는 패턴이다.

**예시**

Duck class가 있다.

날기 기능과 소리 내기 기능이 있다. (이 둘은 객체 마다 다르다.)

일반적으로, Duck에 fly()와 quack()를 구현하고 여러 오리 객체들은 이를 상속받는 방식으로 구현할 것이다.

하지만, 이런 경우 고무 오리객체 or 나무 오리객체 등은 fly()나 quack()가 비어있어야 하는 경우가 생길 것이다. **[문제 1]: 기능이 없는 객체들이 생기고, 이들을 일일이 수정해주어야 한다.**

이번에는 이를 해결하기 위해 Flyable과 Quackable이라는 인터페이스를 만들어 날 수 있어야 하고 소리낼 수 있어야 하는 오리만 이를 상속받도록 하자. **[문제 1 해결] 일일이 수정해줄 필요가 없어졌다.**

그치만 이번에는, fly(), quack()을 모든 객체에 구현해주어야 하는 문제가 생겼다. **[문제 2]: 코드 재사용 불가**

→ **변경되는 부분(fly(), quack())을 캡슐화하자. 이는 추후 변경에 용이하다.**

변경이 잦은 두 메서드를 FlyBehavior, QuackBehavior 인터페이스로 만들자.

다양한 활용(나는 것 & 날지 않는 것), (꽥꽥 소리를 내는 것, 삑삑 소리를 내는 것{고무 오리}, 아무 소리를 내지 않는 것 )은 이 인터페이스들을 상속받아 구현하고 재사용하자.)

```java
abstract class Duck { // 오리 인터페이스
    FlyBehavir flyBehavir;
    QuackBehavior quackBehavior;

    public void performFly() {
        flyBehavir.fly();
    }

    public void performQuack() {
        flyBehavir.quack();
    }
}

//짖지 않고 날지 못하는 고무 오리
class RubberDuck extends Duck {
    public RubberDuck() {
        flyBehavior = new FlyNoWay();
        quackBehavir = new QuackNoWay();
    }
}

//카리스마 있게 날고, 오리가 아니라고 소리내는 카리스마 큰 오리(?)
class BigKarismaticDuck extends Duck {
    public BigKarismaticDuck() {
        flyBehavior = new FlyKarismatic();
        quackBehavir = new NotDuckQuack();
    }
}
```

### 팩토리 패턴

(간단) 구체적인 객체에 의존하는 것을 최대한 분리시키는 패턴

1. 구체적인 객체보단 인터페이스에 의존해는 것이 좋다.
2. new 연산자는 구체적인 객체를 만든다.
3. new 연산자는 제거할 순 없으므로, 분리를 하자!

(복잡) 객체를 생성하기 위한 인터페이스(AbstractFactory)를 별도로 정의하고, 객체 생성은 인터페이스의 서브클래스(ConcreateFactory)에서 결정하게 하는 것

### 일반적인 경우

어떤 피자 객체를 선택할지 결정하여야 한다.

조건문들을 통해 피자타입에 해당하는 객체를 선택한다.

```java
class PizzaStore {
    Pizza orderPizza(String pizzaType) {
        if (pizzaType.equals("PineapplPizza")) {
            return new PineapplePizza();
        } else if (pizzaType.equals("BulgogiPizza")) {
            return new BulgogiPizza();
        } else if (pizzaType.equals("GorgonzolaPizza")) {
            return new Gorgonzola();
        }
        // ...
    }
}
```

: 구체적인 객체와 의존관계가 너무 크다!

### 개선한 내용

피자 선택을 PizzaFactory에게 위임하였다.

```java
class PizzaStore {
    PizzaFactory factory;

    Pizza orderPizza(String pizzaType) {
        Pizza pizza = factory.createPizza(pizzaType);
        // ...
    }
}

class PizzaFactory {

    Pizza createPizza(String pizzaType) {
        if (pizzaType.equals("PineapplPizza")) {
            return new PineapplePizza();
        } else if (pizzaType.equals("BulgogiPizza")) {
            return new BulgogiPizza();
        } else if (pizzaType.equals("GorgonzolaPizza")) {
            return new Gorgonzola();
        }
        // ...
    }
}
```

: 구체적인 객체와 의존 관계가 감소하였다.

### 복잡한 경우

```java
abstract class PizzaStore {

    Pizza orderPizza(String pizzaType) {
        Pizza pizza = factory.createPizza(pizzaType);
        // ...
        // 피자 조리 로직들
        return pizza;
    }

    abstract Pizza createPizza(String pizzaType);
}

class KoreanPizzaStore extends PizzaStore {

    @Override
    Pizza createPizza(String pizzaType) {
        if (pizzaType.equals("PineapplPizza")) {
            return new PineapplePizza();
        } else if (pizzaType.equals("BulgogiPizza")) {
            return new BulgogiPizza();
        } else if (pizzaType.equals("GorgonzolaPizza")) {
            return new Gorgonzola();
        }
        // ...
    }
}

// 사용
public class Application {

    public static void main(String[] args) {
        PizzaStore koreanPizzaStore = new KoreanPizzaStore();
        Pizza pizza = koreanPizzaStore.orderPizza("BulgogiPizza");
    }
}
```

: 어떤 가게냐에 따라 다른, 팩토리를 적용할 수 있다는 장점이 생겼다.

### 더 알아봐야 할 내용

- 추상 팩토리 패턴
- 정적 팩토리 메서드과의 비교
