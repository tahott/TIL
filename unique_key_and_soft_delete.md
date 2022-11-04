## soft delete를 하는 테이블에서의 유니크 키 적용
서비스의 비즈니스 로직을 다루다 보면 테이블의 데이터를 soft delete하며 관리하는 경우가 있다.<br>
그 중 주문번호 컬럼이 있고 주문번호는 고유한 값을 유지해야 할 때 soft delete로 처리해야 할 때는 어떻게 해야할까?

```sql
CREATE TABLE `order` (
  id INT AUTO_INCREMENT,
  seller INT NOT NULL,
  open_market varchar(64) NOT NULL,
  order_no varchar(64) NOT NULL,
  deleted_at datetime NULL default NULL,
  PRIMARY KEY (id),
  UNIQUE KEY (order_no)
);

INSERT INTO `order` (order_no) VALUES (20000);
```
위와 같은 sql이 있고 db는 아래와 같은 데이터가 저장되어 있을 것이다.

|id|seller|mall|order_no|deleted_at|
|---|---|---|---|---|
|1|10|market1|20000|null|

이 상태에서 주문 정보를 주기적으로 가져오는 로직이 있고 20000이라는 주문번호 값이 들어있는 row를 soft_delete하고 새로 등록한다면 아래와 같은 데이터를 기대해야 하지만 order_no에 유니크가 걸려 있기 때문에 실패할 것이다.

|id|seller|mall|order_no|deleted_at|
|---|---|---|---|---|
|1|10|market1|20000|xxxx.xx.xx|
|2|10|market1|20000|NULL|

그렇다면 unique와 deleted_at을 묶어서 유니크 키로 설정하면 되겠지? 라고 생각 할 수 있다. 기존의 유니크 인덱스를 삭제하고 `order_no, deleted_at`을 묶어서 유니크 키로 설정해보자. 그 후에는 위와 같은 데이터로 입력이 될 것이다.

이 때 오픈마켓에서 동일한 주문을 cron이든 api 호출을 통해 한 번 더 수집 된다고 가정하자. 우리는 soft_delete 되지 않은 같은 order_no를 가진 데이터가 생성 될테니 유니크 조합에 의해 데이터 입력이 실패할 것이라고 생각 할 수 있다.

|id|seller|mall|order_no|deleted_at|
|---|---|---|---|---|
|1|10|market1|20000|xxxx.xx.xx|
|2|10|market1|20000|NULL|
|3|10|market1|20000|NULL|

하지만 데이터는 생성 될 것이다. 이것이 허용된다는 것은 order의 상태에 관한 컬럼이 있다면 이미 처리 된 주문이 중복되어 생성 되어 처리 될 수 있다는 것이기도 하다. 이것은 비즈니스 로직상 우리가 의도한 사항이 아닐 것이다.

이런 현상이 발생하는 이유는 unique key에 포함 된 deleted_at이 null 속성을 가지고 있기 때문이라 한다. 이것을 처리 하려면 기존 유니크 조합을 해제하고 새로운 컬럼 등록을 하고 유니크 조합을 만들어 줘야한다

```sql
ALTER TABLE `order` ADD `archived` TINYINT GENERATED ALWAYS AS (IF((`deleted_at` is NULL), 1, NULL)) VIRTUAL
ALTER TABLE `order` ADD CONSTRAINT `u_idx` UNIQUE (seller, mall, order_no, archived) 
```

이제 `order` 테이블은 deleted_at이 null이면 archived가 1의 값을 가지고 있고 soft_delete 되는 row의 archived는 null이 된다. 이렇게 하여 soft delete로 관리되는 테이블에서 살아있는 데이터의 중복 관리를 할 수 있게 되었다. 

|id|seller|mall|order_no|deleted_at|archived|
|---|---|---|---|---|---|
|1|10|market1|20000|xxxx.xx.xx|NULL|
|2|10|market1|20000|NULL|1|