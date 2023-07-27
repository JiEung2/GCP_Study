# [4] Loading data into BigQuery

- 다양한 소스에서 BigQuery로 데이터 로드
- CLI 및 콘솔을 사용하여 BigQuery에 데이터 로드
- DDL을 사용하여 테이블 생성

---

## 작업 1. 테이블을 저장할 새 데이터 세트 만들기

1. 프로젝트 ID 옆에 있는 작업 보기 아이콘(3개의 세로 점)을 클릭하고 데이터 세트 만들기 선택
2. 테이터 세트 ID 이름을 nyctaxi로 지정하고 데이터 세트 생성 클릭

## 작업 2. CSV에서 새 데이터 세트 수집

로컬 CSV를 BigQuery 테이블에 로드

[](https://storage.googleapis.com/cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_1.csv)

1. 링크에서 NYC 택시 2018 여행 데이터의 하위 집합을 로컬에 다운
2. BigQuery 콘솔에서 nyctaxi 데이터세트를 선택한 다음 테이블 만들기 클릭
    
    출처:
    
    - 다음에서 테이블 생성: 업로드
    - 파일 선택: 이전에 로컬로 다운로드한 파일 선택
    - 파일 형식: CSV
    
    목적지:
    
    - 테이블 이름: 2018trips
    
    스키마:
    
    - 자동 감지
    
    테이블 만들기 클릭
    
3. 미리 보기 선택 후 확인

### SQL 쿼리 실행

1. 쿼리 편집기에서 올해 가장 비싼 여행 상위 5개를 나열하는 쿼리 작성
    
    ```sql
    #standardSQL
    SELECT
      * //모든 항목 선택
    FROM
      nyctaxi.2018trips
    ORDER BY
      fare_amount DESC //내림차순
    LIMIT  5
    ```
    

## 작업 3. Google Cloud Storage에서 새 데이터세트 수집


Cloud Storage에서 사용할 수 있는 동일한 2018 여행 데이터의 다른 하위 집합 로드

이번엔 CLI 도구 사용하여 수행

---

1. Cloud Shell에서 다음 명령어 실행
    
    ```sql
    bq load \
    --source_format=CSV \     //파일 형식
    --autodetect \
    --noreplace  \
    nyctaxi.2018trips \
    gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_2.csv
    ```
    
    **→ 위 작업을 통해 하위 집합이 위에서 생성한 기존 2018trips 테이블에 추가되도록 지정**
    
2. 로드 작업이 완료되면 화면에 확인 메시지가 표시
3. BigQuery 콘솔로 돌아가서 2018trips 테이블 선택 후 세부정보 확인. 행 수가 거의 두 배가 되었는지 확인
4. 가장 비싼 여행 상위 5개가 변경되었는지 동일한 쿼리 실행해서 확인

## 작업 4. DDL을 사용하여 다른 테이블에서 테이블 생성

2018trip에서 1월 여행의 데이터만 추출하여 다른 테이블에 저장

---

1. 쿼리 편집기에서 다음 CREATE TABLE 명령 실행
    
    ```sql
    #standardSQL
    CREATE TABLE
      nyctaxi.january_trips AS
    SELECT
      *
    FROM
      nyctaxi.2018trips
    WHERE
      EXTRACT(Month
      FROM
        pickup_datetime)=1;
    ```
    
2. 아래 쿼리를 실행하여 1월에 이동한 가장 긴 거리 찾기