# [1]  Exploring a BigQuery Public Dataset

→ 데이터 파일 가져와서 쿼리 처리 기본 예습

대규모 데이터 세트를 저장하고 쿼리하는 것은 올바른 하드웨어와 인프라 없이는 시간과 비용이 많이 들기 때문에 BigQuery 사용. BigQuery는 Google 인프라의 처리 능력을 사용하여 초고속 SQL 쿼리를 지원하여 문제를 해결하는 [enterprise data warehouse](https://cloud.google.com/solutions/bigquery-data-warehouse)임. 데이터를 BigQuery로 옮기기만 하면 어려운 작업은 여기에서 처리

### 목표

- Query a public dataset
- Create a custom table
- Load data into a table
- Query a table

---
## 작업 1. Query a public dataset
<details>
<summary></summary>
<div markdown="1">

### USA Names 데이터 세트 로드

1. 탐색기 창의 입력하여 검색에 usa_names를 입력
2. 검색 범위를 모두로 확장 클릭
3. 탐색 창에서 포인터를 놓은 bigquery-public-data 다음 별 클릭
4. **검색할 입력** 필드에 `bigquery-public-data`를 입력. 이렇게 하면 프로젝트의 모든 데이터 세트가 표시(안해도 됨 그냥 usa_names 검색하고 진행)
5. usa_names를 클릭하여 데이터 세트 확장
6. usa_1910_2013 선택

### USA Names 데이터 세트 쿼리

→ 이 데이터 세트에서 아기의 이름과 성별을 쿼리한 다음 상위 10개 이름을 내림차순으로 나열

1. 쿼리를 클릭한 다음 새 탭에서 열기 클릭
2. 쿼리 편집기 텍스트 영역에 다음 내용으로 덮어쓰기
    
    ```bash
    SELECT
      name, gender,
      SUM(number) AS total
    FROM
      `bigquery-public-data.usa_names.usa_1910_2013`
    GROUP BY
      name, gender
    ORDER BY
      total DESC
    LIMIT
      10
    ```
    
3. 창의 오른쪽 상단에서 쿼리 유효성 검사기를 확인
4. 실행 클릭


</div>
</details>

## 작업 2. 사용자 지정 테이블 만들기
<details>
<summary></summary>
<div markdown="1">
사용자 지정 테이블을 만들고 데이터를 로드한 다음 테이블에 대해 쿼리를 실행


### 로컬 컴퓨터에 데이터 다운로드

1. 파일을 로컬 컴퓨터에 다운
2. 압축해제
3. 파일 위치 기록
</div>
</details>

## 작업 3. 데이터 세트 만들기

<details>
<summary></summary>
<div markdown="1">
테이블을 보관할 데이터 세트를 만들고 프로젝트에 데이터를 추가한 다음 쿼리할 데이터 테이블을 만듦

---

1. 탐색기 창의 검색할 유형 상자에서 bigquery-public-data 선택 취소
2. 프로젝트 ID를 클릭
3. 프로젝트 ID 옆에 있는 세 개의 점을 클릭한 다음 데이터 세트 만들기 클릭
4. 데이터 세트 만들기 에서 :
    - 데이터세트 ID에 babynames 입력
    - 데이터 위치에서 us(미국의 여러 지역)를 선택
    - 기본 테이블 만료 기본값
    - 암호와 기본값
5. 창 하단에서 만들기 클릭
</div>
</details>

## 작업 4. 새 테이블에 데이터 로드

<details>
<summary></summary>
<div markdown="1">

1. 탐색기 창에서 프로젝트 ID 데이터세트를 확장
2. babynames 옆에 세 개의 점 클릭한 후 Create table클릭
3. 테이블 만들기 페이지에서
    - 소스 → 업로드 선택
    - 파일 선택에서 찾아보기를 클릭하고 yob2014.txt 클릭
    - 파일 형식 → CSV 선택
    - 테이블 이름 → names_2014.
    - 스키마 섹션에 텍스트로 편집 클릭 후 텍스트 상자에 다음 스키마 정의 붙여넣기
        
        ```bash
        name:string,gender:string,count:integer
        ```
        
4. 테이블 만들기 클릭

### 테이블 미리보기

→ 세부정보 창에서 미리보기 탭으로 가능
</div>
</details>


## 작업 5. 테이블 쿼리
<details>
<summary></summary>
<div markdown="1">

1. 쿼리 편집기에서 새 쿼리 작성 클릭
2. 다음 쿼리 문을 복사하여 쿼리 편집기에 붙여넣기
    
    ```bash
    SELECT
     name, count
    FROM
     `babynames.names_2014`
    WHERE
     gender = 'M'
    ORDER BY count DESC LIMIT 5
    ```
    
    → 2014년 미국 남성의 상위 5개 아기이름 검색
    
3. 실행 클릭
</div>
</details>