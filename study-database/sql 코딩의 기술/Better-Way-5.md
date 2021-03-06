# 왜 계산 데이터를 저장하면 좋지 않은지 이해하자

개발자들은 가격 계산, 인원수 계산, 수량 계산 등 이런 계산 데이터들을 컬럼에 저장하고 싶은 유혹에 빠질 수 있습니다. 한번 다음 예를 들어보죠.

* Orders Table query

```sql
CREATE TABLE Orders {
 order_id int(5) NOT_NULL,
 order_date date NULL,
 customer_id int(5) NOT_NULL,
 product_id varchar(20) NOT_NULL,
 order_total decimal(10,2) NULL,
}
```

주문 테이블에 order_total 컬럼을 OrderDtails 테이블의 (수량 * 가격)의 합으로 두는 것이 DB 시스템 입장에서 좋은 생각일 수 있습니다. 그 이유는 모든 주문과 주문 금액을 원할 때마다 관련 로우를 가져와 보여 줄 수 있기 때문입니다. 굳이 OrderDetails 테이블에 가서 관련된 로우를 긁어와 계산할 필요가 없어 더욱 좋아 보입니다. 

**하지만 이런 계산 데이터를 두는 건 운영 중인 데이터베이스에 치명적입니다.** 왜 그런지 아래를 보죠.

1. 데이터 웨어하우스 시스템 입장에서는 좋을지 몰라도 운영 중인 데이터베이스 성능에는 심각한 영향을 미칠 수 있다.
2. OrderDetails 테이블의 로우가 변경, 삽입, 삭제될 때마다 값을 재계산해야 하므로 데이터 무결성을 유지하기가 어렵다.



### 트리거? or 계산 컬럼 함수 Call?

물론 order_total 컬럼 값을 재계산할 때는 OrderDetails 테이블에 트리거를 추가하면 됩니다. 그리고 해당 트리거는 삽입, 삭제, 갱신을 수행할 때 발생시키면 되긴 합니다. 하지만 트리거는 정확하게 작성하기가 어렵고 비용도 비쌉니다.

그럼 트리거 말고 계산 컬럼을 정의해 보면 어떨까요? 아마 복잡한 트리거보다는 나을 것 같습니다. 그 이유는 테이블 정의의 일부로 계산 컬럼을 정의하면 트리거를 작성할 때 종종 필요한 복잡한 코드를 작성하지 않아도 되기 때문입니다. 또한 RDBMS 최신 버전에서 이미 계산 컬럼을 정의해 사용하는 기능을 지원합니다. 

예를 들어 SQL Server에 함수를 정의하면서 AS 키워드에 계산 표현 식을 붙이는 방법입니다. 하지만 이와 같은 방법은 매우 좋지 않다고 생각할 것입니다.

1. 첫 번째로 계산 함수는 비결정적 함수입니다. 즉 계산 컬럼이 다른 컬럼처럼 값이 지속되지 않다는 것입니다.
   * 결정적 함수 : 특정 값 집합이 입력되면 언제나 동일한 결과를 반환한다.
   * 비결정적 함수 : 특정 값 집합이 입력되더라도 매번 다른 값을 반환할 수 있다.
2. 해당 컬럼은 인덱스를 만들 수 없고, 컬럼을 참조할 때마다 각 로우에 대해 서버가 함수를 호출해야 하므로 서버에 많은 부하가 걸립니다.

이러한 이유로 인해 차라리 서브쿼리로 해당 테이블을 조인하는 것이 훨씬 좋아 보입니다.

DB2에서도 GENERATED 키워드를 사용해 비슷한 기능을 구현할 수 있지만, 계산 컬럼을 생성하는 함수가 비결정적 함수이기 때문에 정의하지 못합니다. 따라서 복잡하더라도 INTEGRITY 키워드를 사용해 옵션을 끄고 키고 계산 컬럼에 인덱스를 생성하는 등 복잡한 절차를 거쳐야 사용할 수 있습니다.



### 왜 계산 데이터를 저장하면 좋지 않을까?

지금까지 그래도 계산 컬럼을 넣을 수 있는 방법을 써놓았는데, 정작 "왜 계산 데이터를 저장하면 좋지 않을까?"에 대해 설명해 드리지 않았습니다. 왜 계산 데이터는 저장하면 좋지 않을까요? 

**대용량 데이터 입력용 테이블에서 이런 계산 컬럼을 추가하면 데이터베이스 서버에 심각한 부하가 걸려 응답 시간이 현저히 느려집니다.**

그 이유는 무엇일까요?

1. 비결정적 함수를 사용하기 때문에 인덱스를 만들 수 없습니다. 즉 DB2, 오라클 등 인덱스를 통해 쿼리를 빠르게 수행하는 데이터베이스에 취약합니다.
2. OrderDetails 같은 테이블에 CUD 발생 시 서버에 부하가 많이 걸립니다. 특히 대량의 데이터를 다루는 테이블에서는 함수로 인해 서버 부하가 생기므로 절대 사용해서는 안 됩니다.

이렇게 왜 계산 데이터를 저장하면 안 좋은지 살펴봤습니다. 다음에는 참조 무결성을 보호하려면 외래키를 정의 하자에 대해 공부해 보겠습니다.