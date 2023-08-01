# [5] Working with JSON and Array data in BigQuery 2.5

반구조화된 데이터(JSON, 배열 데이터 유형 수집) 작업에 대한 심층 연습

- 반구조화된 JSON을 BigQuery에 로드
- 배열 생성 및 쿼리
- 구조체 생성 및 쿼리
- 중첩 및 반복 필드 쿼리

---

## 작업 1. 테이블을 저장할 새 데이터세트 만들기

1. 프로젝트 ID에 데이터세트 만들기
2. 이름 → fruit_store

## 작업 2. SQL에서 배열 작업 연습

일반적으로 SQL에서는 아래 과일 목록과 같이 각 행에 대해 단일 값을 가짐

| 열 | 과일 |
| --- | --- |
| 1 | 산딸기 |
| 2 | 블랙베리 |
| 삼 | 딸기 |
| 4 | 체리 |

각 사람의 과일 항목 목록을 원한다면

| 열 | 과일 | 사람 |
| --- | --- | --- |
| 1 | 산딸기 | sally |
| 2 | 블랙베리 | sally |
| 삼 | 딸기 | sally |
| 4 | 체리 | sally |
| 5 | 주황색 | frederick |
| 6 | 사과 | frederick |

기존의 관계형 데이터베이스 SQL에서는 반복되는 이름을 보고 즉시 위의 테이블을 과일 항목과 사람이라는 두 개의 테이블로 분할해야 한다고 생각할 것임. 이 프로세스를 정규화라고 함.

| 열 | 과일(배열) | 사람 |
| --- | --- | --- |
| 1 | 산딸기 | sally |
|  | 블랙베리 |  |
|  | 딸기 |  |
|  | 체리 |  |
| 2 | 주황색 | frederick |
|  | 사과 |  |

과일 배열을 해석하는 더 쉬운 방법:

| 열 | 과일(배열) | 사람 |
| --- | --- | --- |
| 1 | [라즈베리, 블랙베리, 스트로베리, 체리] | sally |
| 2 | [오렌지, 사과] | frederick |

직접 해보기

1. BigQuery 쿼리 편집기에 다음 입력
    
    ```sql
    #standardSQL
    SELECT
    ['raspberry', 'blackberry', 'strawberry', 'cherry'] AS fruit_array
    ```
    
    실행
    
2. 다음을 실행
    
    ```sql
    #standardSQL
    SELECT
    ['raspberry', 'blackberry', 'strawberry', 'cherry', 1234567] AS fruit_array
    ```
    
    Error: Array elements of types {INT64, STRING} do not have a common supertype at [3:1]
    
    한 Array 데이터의 타입이 달라서 생기는 오류
    
3. 쿼리할 최종 테이블
    
    ```sql
    #standardSQL
    SELECT person, fruit_array, total_cost FROM `data-to-insights.advanced.fruit_store`;
    ```
    
4. 결과를 본 후 JSON 탭을 클릭하여 결과의 nested structure 확인

### 반구조화된 JSON을 BigQuery에 로드

1. 데이터 세트에 새 테이블 생성 (fruit_store)
2. 테이블 생성
    - 소스: Google Cloud Storage
    - GCS 버킷에서 파일 선택: data-insights-course/labs/optimizing-for-performance/shopping_cart.json
    - 스키마: 자동감지
    - fruit_details

### 요약

- BigQuery는 기본적으로 배열을 지원
- 배열 값은 데이터 유형을 공유해야 함
- 배열은 BigQuery에서 REPEATED 필드라고 함

## 작업 3. ARRAY_AGG()를 사용하여 자신만의 배열 만들기

테이블에 배열이 없어도 배열 만들기 가능

---

1. 아래 복붙
    ~~~sql
    SELECT
    fullVisitorId,
    date,
    v2ProductName,
    pageTitle
    FROM `data-to-insights.ecommerce.all_sessions`
    WHERE visitId = 1501570398
    ORDER BY date
    ~~~