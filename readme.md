## 프로젝트 목표
👉  백엔드 개발을 위한 환경 설정을 진행하고 GET과 POST를 이용해서, 커피 메뉴 데이터를 관리하는 4가지 로직 CRUD(Create, Read, Update,Delete)를 구현하는 프로젝트를 실행해봅니다.

## 프로젝트 명세서

우리는 작은 로컬 카페 **Grids & Circles** 입니다. 고객들이 Coffe Bean package를 온라인 웹 사이트로 주문합니다. 매일 전날 오후 2시부터 오늘 오후 2시까지의 주문을 모아서 처리합니다.

현재는 총 4개의 상품이 존재합니다.
(상황에 따라 변경)

우리는 별도의 회원을 관리하지 않습니다. email로 고객을 구분해요. 주문을 받을 때 email을 같이 받아서 주문을 받습니다. 하나의 email로 하루에 여러  번 주문을 받더라도 하나로 합쳐서 다음날 배송을 보내면 됩니다. 

<aside>
💡  
    고객에게 **“당일 오후 2시 이후의 주문은 다음날 배송을 시작합니다.”** 라고 알려 줍니다.

</aside>

## 필요기능

- 주문 하기 (상)
- 주문 발송 처리 (상) - 스케줄러 + bulkUpdate
- 상품 등록 (중)
- 상품 수정 (중)
- 상품 단건 조회 (중)
- 상품 삭제 (중)
- 개인의 주문 단건 조회
- 내 주문 조회 → email 식별
- 개인의 주문 수정

## 추가 기능

- 재고 관리 (동시성)
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



***
[프로젝트 이슈](https://github.com/Anyeon00/NBE1_1_9team/blob/main/issue_memo.md)
