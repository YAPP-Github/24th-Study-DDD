# 1. 도메인 모델 시작하기

## 1.1 도메인이란?
- **도메인**: 소프트웨어로 해결하고자 하는 문제 영역
- 하나의 도메인은 여러 하위 도메인으로 나뉠 수 있음
  - **예시**: 온라인 서점 도메인
    - 온라인 서점 소프트웨어는 상품조회, 구매, 결제, 배송 추적 등의 기능을 제공
    - '온라인 서점'은 소프트웨어로 해결하고자 하는 문제 영역, 즉 도메인에 해당됨
    - 주문, 결제, 배송, 혜택 등 하위 도메인의 기능이 엮이게 됨

## 1.2 도메인 전문가와 개발자 간 지식 공유
- 개발자는 고객의 요구사항을 올바르게 이해하는 것이 중요
  - 요구사항을 제대로 이해하지 못하면 코드 재작업이 많아짐 (Garbage in, Garbage out)
  - 요구사항을 이해하기 위해 어느 정도의 도메인 지식을 갖춰야 함

## 1.3 도메인 모델
- **도메인 모델**: 특정 도메인을 개념적으로 표현한 것
  - 도메인 모델을 통해 도메인이 어떤 기능을 수행할 수 있는지 이해 도움

## 1.4 도메인 모델 패턴
- 애플리케이션의 아키텍처는 계층구조를 가짐

  | 영역                     | 설명                                                         |
  | ------------------------ | ------------------------------------------------------------ |
  | 사용자 인터페이스 (표현 계층) | 사용자의 요청을 처리하고 응답하는 계층                             |
  | 응용 계층                | 사용자가 요청한 기능을 수행하는 계층, 로직을 직접 구현하는 것이 아니라 도메인 계층을 조합해서 기능을 실행 |
  | 도메인 계층              | 시스템이 제공할 도메인 규칙을 구현하는 계층                       |
  | 인프라스트럭처 계층      | 데이터베이스나 메시징 시스템과 같은 외부 시스템과의 연동을 처리하는 계층 |

- 도메인 규칙을 객체 지향 기법으로 구현하는 패턴이 **도메인 모델 패턴**임

```java
public class Order {
    private OrderState state;
    private ShippingInfo shippinginfo;

    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        if (!state.isShippingChangeable()) {
            throw new IllegalStateException("can't change shipping in " + state);
        }
        this.shippinginfo = newShippingInfo;
    }
    
    // ...
}

public enum OrderState {
    PAYMENT_WAITING {
        public boolean isShippingChangeable() {
            return true;
        }
    },
    PREPARING {
        public boolean isShippingChangeable() {
            return true;
        }
    },
    SHIPPED, DELIVERING, DELIVERY_COMPLETED;

    public boolean isShippingChangeable() {
        return false;
    }
}
```

- OrderState에는 주문 대기 중이거나 상품 준비 중에는 배송지를 변경할 수 있다는 도메인 규칙이 구현됨

## 1.5 도메인 모델 도출
- 코드를 작성하기 위해서는 먼저 도메인을 이해해야 함
- 도메인을 모델링 할 때 기본이 되는 작업은 모델을 구성하는 핵심 구성요소, 규칙, 기능을 찾는 것 (요구사항 분석)

## 1.6 엔티티와 밸류
- 도출한 모델은 크게 엔티티와 밸류로 구분할 수 있음
- 엔티티와 밸류를 제대로 구분해야 도메인을 올바르게 설계하고 구현할 수 있음

## 1.6.1 엔티티
- 엔티티의 가장 큰 특징은 식별자를 가짐
  - 예: 주문 도메인에서 각 주문은 주문번호를 가짐

## 1.6.2 엔티티의 식별자 생성
- 특정 규칙에 따라 생성
  - UUID, Nano ID
  - 값 직접 입력
  - 일련번호 사용 (시퀀스나 DB의 자동 증가 컬럼 사용)

## 1.6.3 밸류 타입
- 밸류 타입은 별도의 식별자가 없고, 객체 자체로 의미를 명확히 표현 가능
  - 예: Address, Money 등
- 예시: OrderLine에서 int 타입의 price와 amounts 필드를 밸류 타입인 Money 타입으로 대체

```java
public class OrderLine {
    private Product product; 
    private int price; 
    private int quantity; 
    private int amounts; 
}

public class OrderLine {
    private Product product; 
    private Money price; 
    private int quantity; 
    private Money amounts; 
}
```

## 1.6.4 엔티티 식별자와 밸류 타입
- 필드의 의미가 드러나도록 id 보다는 orderNo라는 명확한 필드 이름 사용

## 1.6.5 도메인 모델에 set 메서드 넣지 않기
- 명확한 이유가 없다면 setter 사용을 지양하고, 필요한 필드를 모두 포함하여 생성자로 강제
- 도메인 객체가 불완전한 상태로 사용되는 것을 막을 수 있음
  
## 1.7 도메인 용어와 유비쿼터스 언어
- 도메인에서 사용하는 용어를 코드에 반영하여 코드만 읽어도 자연스럽게 이해할 수 있도록 함
  - 예: Enum 사용시 STEP1, STEP2 보다는 READY, PAID 등 명확한 표현 사용