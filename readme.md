## 프로젝트 목표
👉  백엔드 개발을 위한 환경 설정을 진행하고 GET과 POST를 이용해서, 커피 메뉴 데이터를 관리하는 4가지 로직 CRUD(Create, Read, Update,Delete)를 구현하는 프로젝트를 실행해봅니다.

## 프로젝트 명세서

고객들이 Coffe를 온라인 웹 사이트로 주문합니다. 

매일 전날 오후 2시부터 오늘 오후 2시까지의 주문을 모아서 처리합니다.

우리는 별도의 회원을 관리하지 않습니다. email로 고객을 구분해요.

하나의 email로 하루에 여러  번 주문을 받더라도 하나로 합쳐서 다음날 배송을 보내면 됩니다. 

<aside>
💡  
    고객에게 **“당일 오후 2시 이후의 주문은 다음날 배송을 시작합니다.”** 라고 알려 줍니다.

</aside>

## 필요기능
### 주문
- 주문 하기
- 주문 발송 처리 - 스케줄러 사용
- 개인의 주문 단건 조회
- 내 주문 조회 → email 식별
- 개인의 주문 수정

### 상품
- 상품 등록
- 상품 수정
- 상품 단건 조회
- 상품 삭제


***
## ERD
![스크린샷 2024-09-06 145808](https://github.com/user-attachments/assets/f11fcf9b-997b-4984-a651-410c0e39ee90)

## 테이블 생성 SQL
```sql
CREATE TABLE products
(
    product_id   BINARY(16) PRIMARY KEY,
    product_name VARCHAR(20) NOT NULL,
    category     VARCHAR(50) NOT NULL,
    price        bigint      NOT NULL,
    description  VARCHAR(500) DEFAULT NULL,
    created_at   datetime(6) NOT NULL,
    updated_at   datetime(6)  DEFAULT NULL
);

CREATE TABLE orders
(
    order_id     binary(16) PRIMARY KEY,
    email        VARCHAR(50)  NOT NULL,
    address      VARCHAR(200) NOT NULL,
    postcode     VARCHAR(200) NOT NULL,
    order_status VARCHAR(50)  NOT NULL,
    created_at   datetime(6)  NOT NULL,
    updated_at   datetime(6) DEFAULT NULL
);

CREATE TABLE order_items
(
    seq        bigint      NOT NULL PRIMARY KEY AUTO_INCREMENT,
    order_id   binary(16)  NOT NULL,
    product_id binary(16)  NOT NULL,
    category   VARCHAR(50) NOT NULL,
    price      bigint      NOT NULL,
    quantity   int         NOT NULL,
    created_at datetime(6) NOT NULL,
    updated_at datetime(6) DEFAULT NULL,
    INDEX (order_id),
    CONSTRAINT fk_order_items_to_order FOREIGN KEY (order_id) REFERENCES orders (order_id) ON DELETE CASCADE,
    CONSTRAINT fk_order_items_to_product FOREIGN KEY (product_id) REFERENCES products (product_id)
);
```
## 프로젝트 이슈

***
### 다대다를 위해 생성한 중간 테이블
다대다 -> 1대다 & 다대1 로 풀어내기 위해, OrderItems중간테이블 생성 (중간테이블이 다, 양 테이블이 1) <br>
#### 이 중간테이블을 어떻게 써야하지?
중간테이블이라고 어렵게 생각할건아니고 <br>
이 경우엔 서비스 논리상으로 Orders에 OrderItems를 포함하기 위해 사용됨 (Products쪽에선 딱히 안씀)  <br>
Orders를 생성할때 OrderItems들을 생성해서 저장함 (의미상으로도 주문은 날것의 상품이 아닌, 주문상품들을 저장함)  <br>
이대로 쓰면됨
***
### 양방향 매핑해도 FK는 다 쪽 테이블에만 생김 
그럼 어떻게 일 쪽에서 다 쪽을 참조하는가? <br>
다 쪽을 전부 뒤지면서 참조를 요청한 일을 가지고있는 놈을 찾으면 걔를 반환함 그래놓고 일쪽에서 찾은 척 하는 것임 <br>
-> 프로그램 논리상으로 Order가 OrderItems를 사용하므로 Order의 OrderItems를 조회하는데, 실제로 동작하는 방식은 OrderItems가 해당 Order를 fk로갖는 OrderItems를 찾아서 조회하는것같은데 좀 이상하지 않아?   <br>
### ChatGPT답변
```markdown
맞습니다. 언뜻 보기엔 Order가 OrderItems를 사용하는데, 실제 동작 방식은 OrderItems가 **외래 키(FK)**로 Order를 참조해 데이터를 조회하는 구조라서 직관적으로 조금 이상해 보일 수 있습니다. 하지만, JPA의 연관관계 매핑 방식은 다음과 같은 이유로 설계된 것입니다.

#### 왜 외래 키는 다쪽에 존재하고 일쪽은 조회만 하는가?

관계형 데이터베이스 구조: <br>
관계형 데이터베이스는 정규화 원칙을 따르며, 다대일 관계에서는 다쪽에 외래 키가 존재해야만 참조가 가능합니다. 이는 데이터 중복을 방지하고 참조 무결성을 유지하기 위한 기본적인 설계 원칙입니다.
예를 들어, 하나의 Order는 여러 OrderItems를 가질 수 있으므로, OrderItems에 order_id라는 외래 키가 있어야 해당 주문과의 관계를 명확히 표현할 수 있습니다.

참조 무결성:  <br>
외래 키는 데이터베이스에서 참조 무결성을 유지하는 핵심 요소입니다. 다쪽에서 외래 키를 유지함으로써, 다쪽 엔터티가 삭제될 때 이를 참조하는 일쪽 엔터티가 자동으로 삭제되거나 업데이트되는 식으로 관계를 유지할 수 있습니다.

JPA의 연관관계 매핑:  <br>
JPA는 객체 지향 프로그래밍과 관계형 데이터베이스의 차이를 추상화해서 개발자가 더 쉽게 관계를 다룰 수 있도록 합니다. JPA는 Order에서 OrderItems를 조회할 때 내부적으로 다쪽(OrderItems)의 외래 키를 사용해 데이터를 가져오는 쿼리를 생성합니다.
즉, Order에서 OrderItems를 조회하는 것처럼 보이지만, 실제로는 OrderItems가 가지고 있는 외래 키를 통해 참조된 데이터들을 검색해 가져옵니다.

성능과 효율성: <br>
성능적인 관점에서, 외래 키가 다쪽에 있으면 일쪽에서 다쪽으로의 조회가 더 쉽고 효율적입니다. Order에서 모든 관련 OrderItems를 조회하는 경우, 데이터베이스에서 인덱스가 잡힌 외래 키를 사용해 빠르게 연관된 데이터를 찾을 수 있습니다.

#### 예시로 풀어서 설명
Order 엔터티가 있고, 그 아래 여러 개의 OrderItems가 있는 상황을 상상해 보세요:
Order는 주문의 정보를 담고 있고, 이 주문에 어떤 상품들이 포함되었는지는 OrderItems가 관리합니다.
데이터베이스에서는 OrderItems에 외래 키가 저장되어 있어야, 어떤 상품이 어떤 주문에 속하는지 알 수 있습니다.

즉, Order는 여러 개의 OrderItems를 "논리적으로" 소유하지만, 실제 데이터베이스 상에서는 OrderItems가 order_id라는 외래 키로 Order를 참조하게 됩니다.

#### 결론
외래 키가 다쪽에 존재하는 이유는 관계형 데이터베이스의 기본 설계 원칙과 참조 무결성을 유지하기 위해서입니다. 비록 프로그래밍 논리에서는 Order가 OrderItems를 소유하는 구조이지만, 데이터베이스에서 연관 관계를 유지하고 조회하기 위해는 다쪽에 외래 키를 두는 것이 필수적입니다. JPA가 이를 추상화하여 객체 지향적인 접근 방식을 가능하게 해주는 것이죠.
```
그럼 Order객체에 OrderItems리스트를 넣고 db에 저장하면 Orderitems는 어떻게 저장돼? 아니면 Order객체를 저장하는 로직에서 직접 OrderItems도 저장하는 로직을 짜줘야하는겨?
### ChatGPT답변
```markdown
Order 객체에 OrderItems 리스트를 넣고 DB에 저장하는 경우, JPA에서는 연관된 엔티티들 간의 저장 로직이 어떻게 동작할지 설정에 따라 달라집니다. 
예를 들어, Order 객체가 OrderItems와 1대다 연관 관계를 가지고 있을 때, 이를 처리하는 방식은 다음과 같습니다.
CascadeType 설정이 있으면, Order 객체를 저장할 때 연관된 OrderItems도 자동으로 저장됩니다.
CascadeType 설정이 없으면, OrderItems도 별도로 직접 저장하는 로직을 작성해야 합니다.
```
***
### Builder패턴 클래스 단위에 쓸때 문제
-  id값은 random uuid값으로 자동생성해야함
- @CreationTimestamp, @UpdateTimestamp 컬럼도 마찬가지로 자동 생성됨 <br>

해결방법
1. 생성자에 @Builder 작성 <br>
ex)
```java
    @Builder
    private Orders(String email, String address, String postcode, List<OrderItems> orderItems) {
        orderId = UUIDConversion.uuidToBytes(UUID.randomUUID());
        orderStatus = OrderStatus.UNPROCESSED;
        this.email = email;
        this.address = address;
        this.postcode = postcode;
        this.orderItems = orderItems;
    }
//접근제한자 private으로 설정
```

2. @NoArgsConstructor(access = AccessLevel.PROTECTED) <br>
   엔터티보호를 위해 visibility 최소화   <br>
   근데 private쓰면 엔터티의 프록시객체를 생성못해서 protected로

***
### @CreationTimestamp
@CreationTimestamp달았을때, 애플리케이션 실행했을땐 db에 잘 생성되어 들어가는데 <br>
테스트에서 꺼내봤을땐 왜 저장되어있지 않고 null이 나오나? <br>
-> 다음 주석 참고
```java
    @Test
    public void 상품조회() throws Exception{
        System.out.println("ProductServiceImplTest.상품조회");
        //given
        Products newProduct = testProduct1;
        service.registerProduct(newProduct);

        entityManager.flush();
        //@CreationTimestamp 작동방식이 transaction이 끝나며 flush될때 값이 생겨 들어감
        //하지만 이 메서드는 save()하고 save()한걸 조회한 후 트랜잭션이 끝나기 때문에 (트랜잭션이 메서드단위로 실행되기 때문에)
        //save()한 다음 지접 flush()해주어야 함

        byte[] id = newProduct.getProductId();
        //when
        Products foundProduct = service.findProduct(id).get();
        printProduct(foundProduct);
        //then
        Assertions.assertThat(Arrays.equals(id, foundProduct.getProductId()));
        Assertions.assertThat(foundProduct.getCreatedAtforTest()).isNotNull();
    }
```
***
### Order도 OrderItems를 갖고, OrderItems도 Order를 가지는데 그럼 객체생성을 한 번에 못하고 어느 하나를 먼저 만든 후 합쳐야하니까, 두 번에 걸쳐서 생성해야하는데 이거 해결할 방법없나?

**ChatGPT** <br>
가장 직관적이고 유지보수에 유리한 방법은 양방향 연관관계 편의 메서드를 사용하는 것 <br>
ex) <br>
```java
@Entity
public class Order {

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItems> orderItems = new ArrayList<>();

    public void addOrderItem(OrderItems item) {
        orderItems.add(item);
        item.setOrder(this); // OrderItems에도 Order 설정
    }
}

@Entity
public class OrderItems {

    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;

    public void setOrder(Order order) {
        this.order = order;
    }
}
```
아까 위에서 cascade설정으로 인해 자동으로 추가되어서 굳이 반대쪽에서도 추가해줄 필요는 없다고 했는데, <br>
여기서 제기한 문제를 해결하기 위해 이렇게 작성할 필요가 생김 (유지보수도 용이)
***
DTO를 URL메서드 마다 각각 생성해주는게 나은듯 <br>
똑같이 entity에 대한 dto라도, 요청에따라 해당 entity에 대해 필요한 값이 다를 수 있음
***
DTO반환타입 <br>
반환DTO 타입이 있는경우, 성공/에러 타입인 경우, 반환DTO/에러 타입인 경우 <br>
url메서드 리턴타입 및 DTO 상속구조? 제네릭? 어떻게 해야할지
***
### 코드리뷰
1. 하위계층레벨인 엔터티에서 상위계층레벨에 있는 dto를 참조하는건 이상하지만,
   상위 단계에서는 하위 레벨을 참조(이용)해도 상관없음 <br>
   -> (엔터티로) dto 생성 로직은 dto쪽에 작성 <br>
   \* 컨트롤러 dto가 상위 계층, 엔터티는 하위 계층인데 하위계층인 엔터티가 수정될 여지가 생김, <br>
하위계층인 엔터티는 가능한 수정되면 안됨

2. DTO를 반환하는 일 : 도메인객체에서 반환하기 보단 DTO를 생성하는 곳에서 엔티티를 받아서 반환하는게 좋을 것 같습니다 !
    -> dto와 엔터티 역할의 차이 이

3. @Transactional을 repository에 달면 transaction범위가 너무 작아짐 (save,update,delete단위로)<br>
   -> 서비스에서 논리적작업메서드단위로 트랜잭션이 실행되도록, 서비스클래스에 달아주기 <br>
   \* (readOnly = true) 같은 작업도

4. 컨트롤러는 프론트와 통신하는 url메서드만 <br>
private 내부 메서드 작성하지 말 것

5. BindingResult대신 @ControllerAdvice 활용 가능

6. @NoArgsConstructor를 protexted로 지정 : 프록시 객체만 접근 가능하도록

7. @Data에 @AllArgsConstructor가 이미 존재함

8. 이런식으로 작성해서 테스트가 한번에 여러 메서드를 검증하지 않도록 하는게 좋을것 같아요 ! 
단위 테스트는 한 기능에 대한 테스트를 진행해야한다고 생각해서 기존 코드는registerProduct를 검증하고있지는 않지만 사용하고 있는 부분에서는 올바르게 테스트가 작성되었느냐는 의문이 들게됩니다 !
DisplayName을 통해 이름도 지정해주는게 좋습니다 !
(ex 조회 테스트에서 저장과 조회를 같이 테스트 하게됨. service의 조회를 테스트한다면, repository를 이용해서 저장하고 service로 조회하는식으로)

9. 엔터티의 공통 필드는 @MappedSuperClass 활용

[프로젝트 이슈](https://github.com/Anyeon00/NBE1_1_9team/blob/main/issue_memo.md)
