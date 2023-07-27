# [2] Using BigQuery to do Analysis(빅쿼리를 사용한 분석 수행)

2개의 서로 다른 공개 데이터 세트를 분석하고 쿼리를 별도로 실행한 다음 결합하여 새로운 insight를 도출

- BigQuery 콘솔에서 대화형 쿼리를 수행
- 여러 데이터 세트에서 분석을 결합하고 실행

이 실습에서는 두 가지 공개 데이터 세트 미국 NOAA의 날씨 데이터와 뉴욕시의 자전거 대여 데이터를 사용

---

## 작업 1. 자전거 대여 데이터 탐색

1. 마켓 플레이스 클릭한 후 검색 표시줄에 NYC bike 입력 후 검색
2. NYC Citi Bike Trips 클릭 후 데이터 보기 클릭
3. citibike_trips 테이블 클릭 후 미리보기로 데이터 값 검사
4. 새 쿼리 작성을 클릭하고 다음 입력
    
    ```sql
    SELECT
      MIN(start_station_name) AS start_station_name,
      MIN(end_station_name) AS end_station_name,
      APPROX_QUANTILES(tripduration, 10)[OFFSET (5)] AS typical_duration,
      COUNT(tripduration) AS num_trips
    FROM
      `bigquery-public-data.new_york_citibike.citibike_trips`
    WHERE
      start_station_id != end_station_id
    GROUP BY
      start_station_id,
      end_station_id
    ORDER BY
      num_trips DESC
    LIMIT
      10
    ```
    
5. 실행.
6. 아래를 실행하여 데이터 세트에서 각 자전거가 이동한 총 거리라는 또 다른 사실을 찾음 (이건 왜 필요?) 결과를 상위 5개로 제한
    
    ```sql
    WITH
      trip_distance AS (
    SELECT
      bikeid,
    	-- Google Geography Functions 사용
    	-- ST_Distance(): 두 좌표의 최소 거리(유클리드 거리)
      ST_Distance(ST_GeogPoint(s.longitude, -- 경도
          s.latitude),                      -- 위도
        ST_GeogPoint(e.longitude,
          e.latitude)) AS distance
    FROM
      `bigquery-public-data.new_york_citibike.citibike_trips`,
      `bigquery-public-data.new_york_citibike.citibike_stations` as s,
      `bigquery-public-data.new_york_citibike.citibike_stations` as e
    WHERE
      start_station_name  = s.name
      AND end_station_name  = e.name)
    SELECT
      bikeid,
      SUM(distance)/1000 AS total_distance
    FROM
      trip_distance
    GROUP BY
      bikeid
    ORDER BY
      total_distance DESC
    LIMIT
      5
    ```
    

## 작업 2. 날씨 데이터 세트 탐색

1. BigQuery 콘솔 왼쪽 창에서 ghcnd_2015 검색
2. 미리보기로 데이터 값을 검사
3. 새 쿼리 작성 후 다음 입력
    
    ```sql
     SELECT
      wx.date,
      wx.value/10.0 AS prcp
    FROM
      `bigquery-public-data.ghcn_d.ghcnd_2015` AS wx
    WHERE
      id = 'USW00094728'
      AND qflag IS NULL
      AND element = 'PRCP'
    ORDER BY
      wx.date
    ```
    
4. 실행
    
    → 이 쿼리는 쿼리에 ID가 제공된 뉴욕의 기상 관측소에서 2015년의 모든 날에 대한 강우량(mm)을 반환(관측소는 뉴욕 CNTRL PK TWR에 해당).
    
## 작업 3. 비와 자전거 대여 간의 상관관계 찾기

1. 새 쿼리 작성 클릭 후 다음 입력
    
    ```sql
    WITH bicycle_rentals AS (
      SELECT
        COUNT(starttime) as num_trips,
        EXTRACT(DATE from starttime) as trip_date
      FROM `bigquery-public-data.new_york_citibike.citibike_trips`
      GROUP BY trip_date
    ),
    <!-- //bycycle_rentals라는 공통 테이블 표현(CTE)을 생성,  -->
    <!-- //From 부분의 테이블에서 각 날짜(trip_date)별 자전거 여행 횟수(num_trips)를 계산하고, -->
    <!-- //trip_date로 결과를 그룹화  -->
    
    rainy_days AS
    (
    SELECT
      date,
      (MAX(prcp) > 5) AS rainy
    FROM (
      SELECT
        wx.date AS date,
        IF (wx.element = 'PRCP', wx.value/10, NULL) AS prcp
      FROM
        `bigquery-public-data.ghcn_d.ghcnd_2015` AS wx
      WHERE
        wx.id = 'USW00094728'
    )
    <!-- //rainy_days라는 다른 (CTE) 생성, 특정 날짜(date)에 최대 강수량(prcp)이 5 이상인지 확인. -->
    <!-- //만약 그렇다면 비 오는 날로 간주하고 rainy열에 1을 할당하고 그렇지 않다면 0할당 -->
    <!-- //관측소 id가 US~인 bigquery-public~에서 테이블을 가져옴 -->
    GROUP BY
      date
    )
    SELECT
      ROUND(AVG(bk.num_trips)) AS num_trips,
     <!-- //평균 자전거 여행 횟수 (num_trips)를 반올림하여 출력 -->
      wx.rainy
    <!-- //rainy의 열을 의미, 해당 날짜가 비가 오는 날인지를 나타내는 boolean값 -->
    <!-- //1은 비가 오는 날, 0은 안오는 날 -->
    FROM bicycle_rentals AS bk
    JOIN rainy_days AS wx
    <!-- //두 개의 CTE를 'on'절에서 날짜와 date와 trip_date가 일치하는 기준으로 조인 -->
    ON wx.date = bk.trip_date
    GROUP BY wx.rainy
    <!-- //결론적으로, 이 SQL 쿼리는 비가 오는 날과 비가 오지 않는 날의 평균 자전거 여행 횟수를 계산하고, -->
    <!-- //그 결과를 wx.rainy의 값(1 또는 0)에 따라 그룹화하여 보여줍니다 -->
    ```
    
2. 실행