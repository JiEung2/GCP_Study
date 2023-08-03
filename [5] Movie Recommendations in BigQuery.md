# [5] Movie Recommendations in BigQuery ML 2.5
BigQuery Machine Learning (BigQuery ML)은 데이터 분석가가 최소한의 코딩으로 머신 러닝 모델을 생성, 훈련, 평가 및 예측할 수 있는 BigQuery의 기능

협업 필터링은 사용자를 위한 제품 추천 또는 제품에 대한 사용자 타겟팅을 생성하는 방법을 제공.
시작점은 사용자 ID, 항목 ID 및 사용자가 제품에 부여한 평점의 세 가지 열이 있는 테이블. 이 테이블은 사용자가 모든 제품에 평점을 매길 필요는 없음. 그런 다음 이 기술은 평점만을 기반으로 유사한 사용자와 유사한 제품을 찾고 사용자가 보이지 않는 제품에 부여할 평점을 결정. 그런 다음 예측된 평점이 가장 높은 제품을 사용자에게 추천하거나 예측된 평점이 가장 높은 사용자를 대상으로 제품을 타겟팅할 수 있음.

### 목표
- MovieLens 데이터를 저장하고 로드할 BigQuery 데이터세트 만들기
- MovieLens 데이터세트 살펴보기
- 학습된 모델을 사용하여 BigQuery에서 추천하기
- 단일 사용자 및 배치 사용자 모두를 위한 제품 예측

## 작업 1. MovieLens 데이터 가져오기
이 작업에서는 명령줄을 사용하여 MovieLens 데이터를 저장할 BigQuery 데이터 세트를 만듦. 그 이후 MovieLens 데이터가 Cloud Storage 버킷에서 데이터 세트로 로드됨

### Cloud Shell에서
1. 다음 명령어를 실행하여 movies라는 BigQuery 데이터세트를 생성
    ~~~shell
    bq --location=EU mk --dataset movies
    ~~~
2. 다음 코드 실행
    ~~~shell
     bq load --source_format=CSV \ # 로드할 데이터 포멧 csv로 설정
    --location=EU \ # 데이터를 로드할 BigQuery 위치
    --autodetect movies.movielens_ratings \ # 여기에 데이터를 로딩
    gs://dataeng-movielens/ratings.csv # 여기에서 데이터를 가져와
    # gs://는 Google Cloud Storage 버킷을 나타냄
    ~~~
    ~~~shell
     bq load --source_format=CSV \
     --location=EU   \
     --autodetect movies.movielens_movies_raw \
     gs://dataeng-movielens/movies.csv
    ~~~

## 작업 2. 데이터 탐색
이 작없에서는 쿼리 편집기를 사용하여 MovieLens 데이터 세트를 탐색하고 확인
1. BigQuery의 쿼리 편집기에서 다음 쿼리 실행
    ~~~sql
    SELECT
      COUNT(DISTINCT userId) numUsers,
      COUNT(DISTINCT movieId) numMovies,
      COUNT(*) totalRatings
    FROM
      movies.movielens_ratings
    ~~~
    데이터 세트가 138,000명 이상의 사용자, 거의 27,000개의 영화 및 2천만 개 이상의 평가로 구성되어 있는지 확인

2. 쿼리를 사용하여 처음 몇 편의 영화를 검사
    ~~~sql
    SELECT
      *
    FROM
      movies.movielens_movies_raw
    WHERE
      movieId < 5
    ~~~

3. 장르 열이 형식이 지정된 문자열임을 알 수 있음. 장르를 배열로 구문 분석하고 결과를 movielens_movies 테이블에 다시 작성
    ~~~sql
    CREATE OR REPLACE TABLE
      movies.movielens_movies AS
    SELECT
      * REPLACE(SPLIT(genres, "|") AS genres)
    FROM
      movies.movielens_movies_raw
    ~~~

## 작업 3. 협업 필터링을 사용하여 생성된 훈련된 모델 평가
행렬 요인화는 사용자 요인과 항목 요인이라는 두 개의 벡터에 의존하는 협업 필터링 기법. 사용자 요인은 user_id를 저차원적으로 표현한 것이고, 항목 요인은 item_id를 유사하게 표현한 것.

데이터의 행렬 인수분해를 수행하려면 모델 유형이 matrix_factorization이라는 점과 협업 필터링 설정에서 어떤 열이 어떤 역할을 하는지 식별해야 한다는 점을 제외하고는 일반적인 BigQuery ML 구문을 사용.

영화 평점 데이터에 행렬 인수분해를 적용하려면 모델을 생성하기 위해 BigQuery ML 쿼리를 실행해야 함. 그러나 이 모델 유형을 만드는 데 최대 40분이 소요될 수 있으며, 예약 지향 리소스가 있는 Google Cloud 프로젝트가 필요하므로 Qwiklabs 환경에서 제공하는 것과는 다름.

나머지 실습에서 사용할 수 있도록 Cloud 교육 프로젝트의 cloud-training-prod-bucket BigQuery 데이터 집합에 모델이 만들어졌음.

- 학습된 모델의 측정항목을 보려면 다음 쿼리 실행
    ~~~sql
    SELECT * FROM ML.EVALUATE(MODEL `cloud-training-prod-bucket.movies.movie_recommender`)
    ~~~

## 작업 4. 추천하기
이 작업에서는 학습된 모델을 사용하여 권장 사항을 제공
userId가 903 사용자에게 추천할 최고의 코미디 영화를 찾아보자

1. 아래 쿼리 입력
    ~~~sql
    SELECT
      *
    FROM
      ML.PREDICT(MODEL `cloud-training-prod-bucket.movies.movie_recommender`,
        (
        SELECT
          movieId,
          title,
          903 AS userId
        FROM
          `movies.movielens_movies`,
          UNNEST(genres) g
        WHERE
          g = 'Comedy' ))
    ORDER BY
      predicted_rating DESC
    LIMIT
      5  
    ~~~
    이 결과에는 사용자가 과거에 이미 보고 평가한 영화가 포함됨
2. 그것들을 제거
