# 3. 애그리거트

## 3.1 애그리거트
도메인 객체 모델이 복잡해지면 개별 구성요소 위주로 모델을 이해하게 되고 전반적인 구조나 큰 수준에서 도메인 간의 관계를 파악하기 어려움.

> '나무를 보지 말고 숲을 보라'는 말처럼 전체적인 흐름을 이해하기 위해서는 큰 덩어리로 먼저 볼 필요가 있음.

### 애그리거트
복잡한 도메인을 이해하고 관리하기 쉬운 단위로 만들기 위해 상위 수준에서 모델을 조망할 수 있는 방법임. 도메인에 관련된 객체를 하나의 군으로 묶어 상위 수준에서 도메인 모델 간의 관계를 파악할 수 있음.

애그리거트는 모델을 이해하는 데 도움을 줄 뿐만 아니라 일관성을 관리하는 기준도 됨. 모델을 보다 잘 이해할 수 있고 애그리거트 단위로 일관성을 관리하기 때문에, 애그리거트는 복잡한 도메인을 단순한 구조로 만들어줌. 복잡도가 낮아지는 만큼 도메인 기능을 확장하고 변경하는 데 필요한 노력도 줄어듦.

애그리거트는 관련 모델을 하나로 모았기 때문에 한 애그리거트에 속한 객체는 유사하거나 동일한 라이프 사이클을 가짐. 애그리거트는 경계를 가지며, 한 애그리거트에 속한 객체는 다른 애그리거트에 속하지 않음. 애그리거트는 독립된 객체 군이며 각 애그리거트는 자기 자신을 관리할 뿐 다른 애그리거트를 관리하지 않음.

예를 들어, 주문 애그리거트는 배송지를 변경하거나 주문 상품 개수를 변경하는 등의 기능을 관리하지만, 주문 애그리거트에서 회원의 비밀번호를 변경하거나 상품의 가격을 변경하지 않음.

경계를 설정할 때는 도메인 규칙과 요구사항을 따름. 도메인 규칙에 따라 함께 생성된 구성요소는 한 애그리거트에 속할 가능성이 높고, 함께 변경되는 빈도가 높은 객체는 한 애그리거트에 속할 가능성이 높음.

'A가 B를 갖는다'로 설계할 수 있는 요구사항이 있더라도 반드시 A와 B가 한 애그리거트에 속하는 것을 의미하지는 않음. 예를 들어, 상품과 리뷰는 함께 생성되지 않고, 함께 변경되지도 않기 때문에 서로 다른 애그리거트에 속함.

## 3.2 애그리거트 루트
### 애그리거트 루트 엔티티
애그리거트에 속한 모든 객체가 일관된 상태를 유지하려면 애그리거트 전체를 관리할 주체가 필요함. 애그리거트에 속한 객체는 애그리거트 루트 엔티티에 직접 또는 간접적으로 속함.

예를 들어, 주문 애그리거트의 루트 역할을 하는 엔티티는 `Order`이며, `OrderLine`, `ShippingInfo`, `Orderer` 등은 `Order`에 속함.

### 도메인 규칙과 일관성
애그리거트 루트는 애그리거트의 일관성이 깨지지 않도록 하는 핵심 역할을 함. 이를 위해 애그리거트 루트는 애그리거트가 제공해야 할 도메인 기능을 구현함.

예를 들어, 주문 애그리거트는 배송지 변경, 상품 변경과 같은 기능을 제공하고, 애그리거트 루트인 `Order`가 이 기능을 구현한 메서드를 제공해야 함. 루트가 제공하는 메서드는 도메인 규칙에 따라 애그리거트에 속한 객체의 일관성이 깨지지 않도록 구현해야 함.

애그리거트 외부에서 애그리거트에 속한 객체를 직접 변경하면 안 됨. 이는 애그리거트 루트가 강제하는 규칙 적용이 불가하여 모델의 일관성을 깨는 원인이 됨.


```java
ShippingInfo si = order.getShippingInfo();
si.setAddress(newAddress);
```

이 코드는 애그리거트 루트인 Order에서 ShippingInfo를 가져와 직접 정보를 변경하고 있음. 
주문 상태와 관계없이 배송지 주소를 변경하는 행위는 규칙을 무시하고 직접 DB 테이블의 데이터를 수정하는 것과 동일한 결과를 초래함. 
즉, 논리적인 데이터 일관성이 깨지게 됨. 
일관성을 유지하기 위해 상태 확인 로직을 서비스에 구현할 수 있으나, 이는 중복 구현될 가능성이 있어 유지 보수가 어려워질 수 있음.

```java
ShippingInfo si = order.getShippingInfo();
// 상태 확인 로직 -> 중복 구현 가능성 존재
if (state != OrderState.PAYMENT_WAITING || state != OrderState.PREPARING) {
    throw new IllegalArgumentException();
}
si.setAddress(newAddress);
```

애그리거트 루트를 통해 도메인 로직을 구현하려면 도메인 모델에 두 가지 습관을 적용해야 함.

첫째, 필드를 변경하는 set 메서드는 공개(public) 범위로 만들지 않음.
둘째, 밸류 타입은 불변으로 구현함.
특히 public set 메서드 작성은 지양해야 함.

```java
public void setName(String name) {
    this.name = name;
}
```

public set 메서드는 도메인의 의미나 의도를 제대로 표현 못 하고, 도메인 로직을 도메인 객체가 아닌 응용 영역이나 표현 영역으로 흩어지게 만듦. 이렇게 도메인 로직이 한 곳에 모이지 않으면 유지 보수할 때 분석하거나 수정하는 데 시간 많이 듦.

도메인 모델의 엔티티나 밸류에서 public set 메서드를 안 쓰면 일관성 깨질 가능성 줄어듦. 의미가 드러나는 메서드를 쓰게 되는 경우가 많아짐. 예를 들어, cancel이나 changePassword 같은 메서드 말이야.

public set 메서드를 안 만드는 걸로 끝나지 않고, 밸류를 불변 타입으로 구현함. 밸류 객체의 값을 변경 못 하게 되면, 애그리거트 루트에서 밸류 객체를 구해도 애그리거트 외부에서 밸류 객체 상태를 변경 못 함.

```java
ShippingInfo si = order.getShippingInfo();
// ShippingInfo가 불변이면, 이 코드는 컴파일 에러가 발생함
si.setAddress(newAddress);
```

애그리거트 외부에서 내부 상태를 함부로 변경 못 하게 되어 애그리거트의 일관성이 깨질 가능성이 줄어듦. 밸류 객체가 불변이라면 밸류 객체의 값을 변경할 방법은 새로운 밸류 객체를 할당하는 것뿐. 즉, 애그리거트 루트가 제공하는 메서드에 새로운 밸류 객체를 전달해서 값을 변경하는 방법 밖에 없음.

```java
public class Order {
    private ShippingInfo shippingInfo;

    public void changeShippingInfo(ShippingInfo shippingInfo) {
        verifyNotYetShipped();
        setShippingInfo(newShippingInfo);

        // set 메서드의 접근 허용 범위는 private임
        private void setShippingInfo(ShippingInfo newShippingInfo) {
            // 밸류가 불변이면 새로운 객체를 할당해서 값을 변경해야 함
            // 불변이므로 this.shippingInfo.setAddress(newShippingInfo.getAddress()) 와 같은 코드 사용 X

            this.shippingInfo = newShippingInfo;
        }
    }
}
```

밸류 타입의 내부 상태 변경은 애그리거트 루트를 통해서만 가능. 애그리거트 루트가 도메인 규칙을 제대로 구현하면 애그리거트 전체 일관성 유지 가능.

### 애그리거트 루트 기능 구현
애그리거트 루트는 애그리거트 내부의 다른 객체들을 조합해 기능 완성. 예를 들어, Order는 OrderLine 목록을 활용해 총 주문 금액 계산.

애그리거트 루트는 구성요소의 상태 참조뿐만 아니라 기능 실행도 위임. 예를 들어, OrderLines 클래스로 분리된 후 Order의 changeOrderLines() 메서드는 내부 orderLines 필드에 상태 변경을 위임해 기능 구현 가능.

```java
public class Order {
    private OrderLines orderLines;

    public void changeOrderLines(List<OrderLine> newLines) {
        orderLines.changeOrderLines(newLines);
        this.totalAmounts = orderLines.getTotalAmounts();
    }
}
```

OrderLines는 changeOrderLines(), getTotalAmounts() 등의 기능 제공. Order에서 getOrderLines() 메서드로 OrderLines 접근 가능하면, 애그리거트 외부에서도 OrderLines 기능 실행 가능.

```java
OrderLines lines = order.getOrderLines();
lines.changeOrderLines(newOrderLines);
```

주문의 OrderLine 목록 변경 시 총합 계산 안 함. 버그 발생. OrderLines를 불변으로 만들거나 패키지나 protected 범위로 외부 변경 방지 필요.

### 트랜잭션 범위

트랜잭션 범위는 작을수록 좋음. 한 트랜잭션은 하나의 애그리거트만 수정해야 함. 두 개 이상의 애그리거트 수정 시 충돌 가능성 높아져 처리량 저하 발생 가능.

한 트랜잭션에서 한 애그리거트만 수정하는 건 애그리거트가 다른 애그리거트 변경 안 한다는 의미.

예로, 배송지 정보 변경하면서 회원의 주소로 설정하는 기능은 애그리거트가 책임 범위 넘어 다른 애그리거트 상태까지 관리하는 꼴. 애그리거트는 독립적이어야 하며, 의존 시작하면 결합도 높아짐.

```java
public class Order {
    private Orderer orderer;

    public void shipTo(ShippingInfo newShippingInfo, boolean useNewShippingAddrAsMemberAddr) {
        verifyNotYetShipped();
        setShippingInfo(newShippingInfo);
        if (useNewShippingAddrAsMemberAddr) {
            orderer.getMember().changeAddress(newShippingInfo.getAddress());
        }
    }
}
```

애그리거트 간 상태 변경은 허용되지 않음. 한 트랜잭션에서 여러 애그리거트 수정 필요 시, 서비스 레벨에서 처리 구현 필요.

```java
public class ChangeOrderService {
    @Transactional
    public void changeShippingInfo(OrderId id, ShippingInfo shippingInfo, boolean useNewShippingAddrAsMemberAddr) {
        Order order = orderRepository.findById(id);
        if (order == null) throw new OrderNotFoundException();
        order.shipTo(newShippingInfo);
        if (useNewShippingAddrAsMemberAddr) {
            Member member = findMember(order.getOrderer());
            member.changeAddress(newShippingInfo.getAddress());
        }
    }
}
```

한 트랜잭션에서 한 개의 애그리거트 변경을 권장하지만, 때로는 두 개 이상의 애그리거트 변경이 필요한 상황도 발생함.

## 3.3 리포지터리와 애그리거트
애그리거트는 도메인 모델 전체를 표현하는 개념적 단위임. 따라서 객체의 영속성을 관리하는 리포지터리도 애그리거트 단위로 존재함.

새 애그리거트 생성 시 저장소에 영속화하고, 사용 시에는 저장소에서 읽어야 함. 리포지터리는 주로 다음 두 메서드를 제공함:
- save: 애그리거트 저장
- findById: ID로 애그리거트 찾기
필요에 따라 다양한 검색 조건으로 애그리거트를 찾거나 삭제하는 메서드도 추가 가능함.

리포지터리 구현 방식은 사용하는 기술에 따라 달라짐. 예를 들어 JPA 같은 ORM 기술을 사용하면 데이터베이스 모델에 맞춰 객체 모델을 조정해야 할 때도 있음.

애그리거트는 개념적으로 하나이기 때문에 리포지터리는 애그리거트 전체를 저장소에 영속화해야 함.

## 3.4 ID를 이용한 애그리거트 참조
애그리거트는 다른 애그리거트를 참조할 수 있음. 애그리거트의 관리는 애그리거트 루트가 담당하므로, 다른 애그리거트를 참조한다는 것은 그 루트를 참조하는 것과 같음.

애그리거트 간 참조는 필드를 통해 구현 가능함. 하지만 이 방식은 다음과 같은 문제를 일으킬 수 있음:
- 편한 탐색 오용: 다른 애그리거트를 쉽게 수정하려는 유혹에 빠질 수 있음.
- 성능 고민: JPA를 사용할 경우, 참조한 객체를 지연 로딩이나 즉시 로딩으로 처리해야 함.
- 확장 어려움: 사용자 증가와 트래픽 증가에 따라 시스템을 분리해야 할 수도 있음.
이런 문제를 해결하기 위해 ID를 이용한 참조 방식을 사용함. 이 방법은 애그리거트 간 물리적 연결을 제거하고, 모델의 복잡도를 낮추며, 응집도를 높임.

ID 참조를 사용하면서 발생할 수 있는 N+1 조회 문제는 조회 전용 쿼리로 해결 가능함. 데이터 조회를 위해 별도의 DAO를 만들고, 조인을 이용해 한 번의 쿼리로 필요한 데이터를 로딩할 수 있음.

애그리거트마다 다른 저장소를 사용하면 한 번의 쿼리로 관련 애그리거트를 조회할 수 없음. 이 경우 캐시 적용이나 조회 전용 저장소 구성을 통해 조회 성능을 높일 수 있음. 이 방법은 코드 복잡성을 증가시키지만 시스템 처리량을 높일 수 있는 장점이 있음.

## 3.5 애그리거트 간 집합 연관
애그리거트 간 1:1 관계는 Set 같은 컬렉션으로 표현 가능함. 하지만 개념적으로 존재하는 애그리거트 간의 1:1 연관을 실제 구현에 반영하지 않을 때도 있음. 성능 문제 때문에 애그리거트 간의 1:1 연관을 구현에 반영하지 않을 수 있음.

## 3.6 애그리거트를 팩토리로 사용하기
애그리거트를 팩토리로 사용하면 도메인 로직이 응용 서비스에 노출되지 않고, 도메인의 응집도가 높아짐.

예를 들어, 고객이 특정 상점을 여러 번 신고해 해당 상점이 물건을 등록하지 못하도록 차단한 상태라고 가정함.

```java
public class RegisterProductService {
    public ProductId registerNewProduct(NewProductRequest req) {
        Store account = accountRepository.findStoreById(req.getStoreId());
        checkNull(account);
        if (!account.isBlocked()) {
            throw new StoreBlockedException();
        }
        ProductId id = productRepository.nextId();
        Product product = new Product(id, account.getId(), ...);
        productRepository.save(product);
        return id;
    }
}
```

위 코드는 Store가 Product를 생성할 수 있는지를 판단하고 Product를 생성하는 논리적 도메인 기능을 응용 서비스에서 구현하고 있습니다.

이를 개선하기 위해 Store 애그리거트에 Product 애그리거트를 생성하는 팩토리 역할을 추가합니다.

```java
public class Store { 
    public Product createProduct(ProductId newProductId, ...) {
        if (isBlocked()) throw new StoreBlockedException();
        return new Product(newProductId, getId(), ...);
    }
}
```

Store에서 Product를 생성하도록 변경하면 다음과 같은 차이점이 발생함.

- 응용 서비스에서 Store의 상태를 확인할 필요가 없어짐.
- Store 관련 로직은 모두 Store 내부에서 구현됨.
- Product 생성 로직도 Store 내부로 이동함.
- 도메인의 응집도가 높아짐.

```java
public class RegisterProductService {
    public ProductId registerNewProduct(NewProductRequest req) {
        Store account = accountRepository.findStoreById(req.getStoreId());
        checkNull(account);
        ProductId id = productRepository.nextId();
        Product product = account.createProduct(id, account.getId(), ...);
        productRepository.save(product);
        return id;
    }
}
```






