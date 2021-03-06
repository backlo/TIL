# 인덱스와 데이터 스캔을 최소화하도록 인덱스는 신중히 만들자

이번 글에는 인덱스를 왜 사용하는지, 인덱스 스캔과 테이블 스캔, 인덱스 종류 등 인덱스에 대해 공부해 보도록 하겠습니다.



## 인덱스를 왜 사용할까?

인덱스를 왜 사용할까요? 데이터베이스를 공부해보셨던 사람들은 아마 이런 생각을 할겁니다.

> 인덱스를 사용하면 빠르게 데이터를 찾을 수 있기 때문에 인덱스를 사용하는거 아냐?

네! 맞습니다. 테이블 풀 스캔하는 것 보다 인덱스 스캔을 하면 데이터를 빠르게 찾을 수 있다는 사실은 누구나 알고 있습니다. 하지만 여기서 궁금한점! 어느 상황에서든 인덱스 스캔이 테이블 스캔보다 빠를까요?



## 인덱스 탐색 VS 인덱스 스캔 VS 테이블 스캔

인덱스 탐색, 인덱스 스캔과 테이블 스캔의 정의는 다음과 같습니다.

> * 인덱스 탐색
>
> 쿼리를 만족하는 레코드를 추출 하기위해 인덱스에서 바로 찾아가는 것을 ``인덱스 탐색``이라 한다.
>
> * 인덱스 스캔
>
> 인덱스에서 적합한 레코드를 살펴보는 것을 ``인덱스 스캔``이라 한다.
>
> * 테이블 스캔
>
> 데이터 페이지에서 레코드를 찾아보는 것을 ``테이블 스캔``이라 한다

정말 간단합니다. 인덱스 탐색은 인덱스에서 데이터를 찾는것을 말하고, 인덱스 스캔은 인덱스에서 데이터를 탐색하는 과정을 말합니다. 그리고 테이블 스캔은 테이블에서 찾는 과정을 테이블 스캔이라고 합니다. 

그럼 다음 Customer Table의 예를 보죠.

* Customer Table

```sql
CREATE TABLE Customers (
	CustomerID int PRIMARY KEY NOT NULL,
  CustFirstName varchar(25) NULL,
  CustLastName varchar(25) NULL,
  CustStreetAddress varchar(50) NULL,
  CustCity varchar(30) NULL,
  CustState varchar(2) NULL,
  CustZipCode varchar(10) NULL,
  CustAreaCode smallint NULL,
  CustPhoneNumber varchar(8) NULL
);

CREATE INDEX CustState ON Customers (CustState);
```

이 테이블은 2가지의 인덱스를 생성하고 있습니다.

> 1. CustomerID 컬럼의 유일 인덱스 생성
> 2. CustState 컬럼의 비유일 인덱스 생성

``SELECT * FROM Customers WHERE CustomerID = 1``을 실행하면 어떻게 될까요? CustomerID에 유일 인덱스가 걸려있기 때문에 ``인덱스 탐색``을 한 후 해당 인덱스로 Customers 테이블에 접근해 CustomerID = 1인 모든 데이터를 반환할 것입니다.

``SELECT CustomerID FROM Customers WHERE CustomerID = 25``를 실행시키면 위처럼 ``인덱스 탐색``을 하지만 테이블을 뒤져 보지 않고 인덱스에서 데이터를 반환 할 것입니다.

그럼  ``SELECT * FROM Customers WHERE CustState = 'TX'`` 를 실행시키면 어떻게 될까요? CustState는 비유일 인덱스이기 때문에 조건에 맞는 인덱스 전체를 탐색할 것입니다. 즉 인덱스 스캔이 일어납니다. 또한 인덱스에 있지 않은 컬럼도 SELECT로 가지고 와야하기 때문에 테이블에 접근해서 찾아야 할 것입니다.

마지막으로 ``SELECT CustomerID FROM Customers WHERE CustAreaCode = '905'``는 어떻게 실행될까요? CustAreaCode는 인덱스가 없기 때문에 테이블 스캔을 해야 할 것입니다. 전체 테이블에서 조건에 맞는 로우를 찾고 반환할 것입니다.



## 아이러니한 인덱스 스캔과 테이블 스캔

위에서 보면 테이블 스캔과 인덱스 스캔은 모든 항목을 검색해야 값을 찾을 수 있다는 특성을 가지고 있어 이 둘의 차이가 크게 없어보입니다. 하지만 여기서 한가지 간과한 사실이 한가지 있습니다. 

> 인덱스는 테이블에 비해 작고 간결한 스캔용으로 설계된 객체

이러한 특성 때문에 보통 테이블에 있는 소수의 로우만 검색할 때는 인덱스 스캔이 월등히 빠르지만 테이블에서 인덱스가 33%(데이터베이스 엔진에 따라 다름)정도 사용하고 있으면 큰 효과를 볼 순 없습니다.

따라서 정리하자면 실제로 반환되는 로우 비율에 따라 테이블 스캔이 성능 좋을 수 있기 때문에 테이블에 적절한 인덱스를 만드는 것이 중요합니다.



## 인덱스 너란 녀석은 참...

만약 인덱스를 많이 생성하면 어떤 문제가 발생할까요?

> 데이터를 빠르게 가져오지도 않고 갱신 속도가 느려진다. 즉 데이터 수정시 인덱스 테이블 하나 이상 갱신하기 때문에 이는 더 많은 디스크 읽기 쓰기가 발생한다.

보통 운영 목적의 테이블은 데이터 갱신 작업이 많으므로 해당 테이블에서 모든 인덱스를 면밀히 검토해야합니다. 



### B-Tree 인덱스

인덱스에서 가장 흔한 유형은 B-Tree 인덱스입니다. B-Tree 인덱스는 간단하게 설명하면 루트 노드에서 시작해 많은 중간 노드를 거치고 마지막 리프 노드를 탐색하는 구조로 가지고 있습니다.



### Cluster Index vs Non-Cluster Index

그렇다면 인덱스의 유형은 어떤 유형들이 있을까요? 종류와 특징을 한번 살펴보겠습니다.

1. 클러스터 인덱스
   * 클러스터 인덱스는 인덱스를 만들 때 명시된 컬럼의 순서대로 테이블의 데이터를 물리적으로 정렬한다.
   * 테이블에 있는 로우를 두가지 이상의 방식으로 정렬하는 것은 불가능하므로 테이블당 하나만 만들 수 있다.
   * 데이터를 직접 포함하는 리프 노드를 가진다.


2. 비클러스터 인덱스
   * 클러스터 인덱스와 같지만 테이블 데이터가 쌓인 물리적인 순서와 다르게 정렬 될 수 있다. 즉 물리적 정렬은 하지 않는다.
   * 비클러스터 인덱스의 리프 노드는 데이터를 포함하지 않고 인덱스 키와 데이터를 가리키는 북마크로 구성된다.



## 인덱스 설정 시 주의사항

1.  인덱스의 사용 여부는 테이블 크기, 로우의 저장 패턴, 로우의 길이, 쿼리가 반환하는 로우의 비율을 통해 정해야 합니다. 

> 예를 들어 흔히 전체 로우 중 최소 10%의 로우가 반환될 때 테이블 스캔이 비클러스터 인덱스 스캔보다 나은 성능을 보입니다. 그리고 반환되는 로우의 비율이 높을 수록 테이블 스캔보다 클러스터 스캔이 나은 성능을 발휘합니다. 

2. 데이터 접근 방식을 고려하는것도 중요합니다.

> 1. WHERE 절 컬럼 고려 - (사용되지 않은 컬럼을 인덱스로 선정하면 큰 혜택 볼수 없음)
> 2. 인덱스 컬럼의 카디널리티 고려 - (중복도가 낮으면 인덱스 효율 미미)
> 3. 반환할 최소 비율 데이터 고려 - (데이터베이스 엔진이 인덱스를 사용하지 않음)

3. 인덱스는 테이블이 클 때만 사용하는 것이 좋습니다.

> 데이터베이스 엔진은 대부분 테이블이 작으면 그 데이터를 메모리에 올리기 때문에 빠르게 탐색합니다. 따라서 인덱스의 큰 효과를 바라보지 못하기 때문에 큰 테이블에서 인덱스 사용이 좋습니다.
>
> 부가적으로 '작다'의 기준은 로우의 개수, 개별 로우의 크기, 페이지에 로드되는 방식, 데이터베이스 서버의 가용 메모리 용량에 의존하는 것을 말합니다.

4. 인덱스를 만들 때 컬럼의 조합도 중요합니다.

> 인덱스 컬럼을 순서 없이 생성한다면 인덱스의 효과적인 실행 계획을 세우기 어렵습니다. 
>
> [여기](https://burning-dba.tistory.com/79) 재밌는 글이 있으니까 한번 확인 해보시면 좋을 것 같습니다.