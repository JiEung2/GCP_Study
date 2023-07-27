# [3] Loading Taxi Data into Google Cloud SQL 2.5help_outline

CSV 텍스트 파일에서 Cloud SQL로 데이터를 가져온 다음 간단한 쿼리를 사용하여 기본적인 데이터 분석을 수행하는 방법에 대해 알아봄.

- Cloud SQL 인스턴스 만들기
- Cloud SQL 데이터베이스 만들기
- 텍스트 데이터를 Cloud SQL로 가져오기
- 데이터 무결성 확인

---

## 작업 1. 환경 준비

→ 프로젝트 ID 및 데이터를 포함할 저장소 버킷에 대해 실습에서 나중에 사용할 환경변수 만들기

```bash
export PROJECT_ID=$(gcloud info --format='value(config.project)')
export BUCKET=${PROJECT_ID}-ml
```

## 작업 2. Cloud SQL 인스턴스 만들기

1. Cloud SQL 인스턴스 생성
    
    ```bash
    gcloud sql instances create taxi \
        --tier=db-n1-standard-1 --activation-policy=ALWAYS
    ```
    
    → 좀 오래걸림
    
2. Cloud SQL 인스턴스의 루트 비밀번호 설정
    
    ```bash
    gcloud sql users set-password root --host % --instance taxi \
     --password Passw0rd
    ```
    
3. 비밀번호 Passw0rd로 설정
4. Cloud Shell의 IP 주소로 환경 변수 생성
    
    ```bash
    export ADDRESS=$(wget -qO - http://ipecho.net/plain)/32
    ```
    
5. SQL 인스턴스에 대한 관리 액세스를 위해 Cloud Shell 인스턴스를 화이트 리스트에 추가
    
    ```bash
    gcloud sql instances patch taxi --authorized-networks $ADDRESS
    ```
    
    → Y
    
6. Cloud SQL 인스턴스의 IP 주소 가져오기
    
    ```bash
    MYSQLIP=$(gcloud sql instances describe \
    taxi --format="value(ipAddresses.ipAddress)")
    ```
    
7. 변수 MYSQLIP 확인
    
    ```bash
    echo $MYSQLIP
    ```
    
8. 명령줄 인터페이스에 로그인하여 택시 여행 테이블(mysql) 제작
    
    ```bash
    mysql --host=$MYSQLIP --user=root \
          --password --verbose
    ```
    
    비밀번호 Passw0rd
    
9. 다음 줄에 붙여넣어 테이블의 스키마 생성(trips)
    
    ```sql
    create database if not exists bts;
    use bts;
    drop table if exists trips;
    create table trips (
      vendor_id VARCHAR(16),		
      pickup_datetime DATETIME,
      dropoff_datetime DATETIME,
      passenger_count INT,
      trip_distance FLOAT,
      rate_code VARCHAR(16),
      store_and_fwd_flag VARCHAR(16),
      payment_type VARCHAR(16),
      fare_amount FLOAT,
      extra FLOAT,
      mta_tax FLOAT,
      tip_amount FLOAT,
      tolls_amount FLOAT,
      imp_surcharge FLOAT,
      total_amount FLOAT,
      pickup_location_id VARCHAR(16),
      dropoff_location_id VARCHAR(16)
    );
    ```
    
    엔터
    
10. 다음 명령을 입력하여 가져오기를 확인
    
    ```sql
    describe trips;
    ```
    
11. trip 테이블 쿼리
    
    ```sql
    select distinct(pickup_location_id) from trips;
    ```
    
12. exit

## 작업 3. Cloud SQL 인스턴스에 데이터 추가


Cloud Storage에 저장된 뉴욕시 택시 운행 CSV 파일을 로컬로 복사. 리소스 사용량을 낮게 유지하기 위해 데이터의 하위 집합(~20000)으로만 작업

---

1. 다음을 실행
    
    ```bash
    gcloud storage cp gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_1.csv trips.csv-1
    gcloud storage cp gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_2.csv trips.csv-2
    ```
    
2. mysql 대화형 콘솔에 연결하여 local-infile 데이터 로드
    
    ```bash
    mysql --host=$MYSQLIP --user=root  --password  --local-infile
    ```
    
    → Passw0rd
    
3. 대화형 콘솔에서 mysql 데이터베이스 선택
    
    ```bash
    use bts;
    ```
    
4. 다음을 사용하여 local-infile CSV 파일 데이터를 로드
    
    ```bash
    LOAD DATA LOCAL INFILE 'trips.csv-1' INTO TABLE trips
    FIELDS TERMINATED BY ','
    LINES TERMINATED BY '\n'
    IGNORE 1 LINES
    (vendor_id,pickup_datetime,dropoff_datetime,passenger_count,trip_distance,rate_code,store_and_fwd_flag,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,imp_surcharge,total_amount,pickup_location_id,dropoff_location_id);
    ```
    
    ```bash
    LOAD DATA LOCAL INFILE 'trips.csv-2' INTO TABLE trips
    FIELDS TERMINATED BY ','
    LINES TERMINATED BY '\n'
    IGNORE 1 LINES
    (vendor_id,pickup_datetime,dropoff_datetime,passenger_count,trip_distance,rate_code,store_and_fwd_flag,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,imp_surcharge,total_amount,pickup_location_id,dropoff_location_id);
    ```
    

## 작업 4. 데이터 무결성 확인


소스에서 데이터를 가져올 때마다 항상 데이터 무결성을 확인하는 것이 중요. 이는 대략적으로 데이터가 사용자의 기대에 부합하는지 확인하는 것을 의미

---

1. 여행 테이블에서 고유한 픽업 위치 지역을 쿼리
    
    ```bash
    select distinct(pickup_location_id) from trips;
    ```
    
    159개 고유 ID를 반환해야함
    
2. trip_distance 열을 먼저 파헤침. 콘솔에 다음 쿼리 입력
    
    ```sql
    select
      max(trip_distance),
      min(trip_distance)
    from
      trips;
    ```
    
3. 여행 거리가 0인 여행은 몇 개?
    
    ```sql
    select count(*) from trips where trip_distance = 0;
    ```
    
4. 기대에 미치지 못하는 데이터를 더 찾을 수 있는지 살펴보겠습니다. `fare_amount`열이 양수일 것으로 예상 . 데이터베이스에서 이것이 사실인지 확인하려면 다음 쿼리를 입력
    
    ```sql
    select count(*) from trips where fare_amount < 0;
    ```
    
    14개의 여행 반환
    
5. payment_type 열을 조사
    
    ```sql
    select
      payment_type,
      count(*)
    from
      trips
    group by
      payment_type;
    ```
    
    - 지불 유형 = 1에는 13863개의 행이 있습니다.
    - 결제 유형 = 2에는 6016개의 행이 있습니다.
    - 지불 유형 = 3에는 113개의 행이 있습니다.
    - 결제 유형 = 4에는 32개의 행이 있습니다.
6. exit