# [2] Big Data Analytics with Notebooks
(프로젝트에서 잘 사용될 수 있음)
Google Cloud Platfor의 Vertex AI 서비스에서 실행되는 Jupyter 노트북을 인스턴스화하는 방법을 보여줌. 예시에선 다양한 항공편 출발 및 도착시간이 포함된 데이터 세트 활용
### 목표
- Vertex AI에서 Jupyter 노트북을 인스턴스화함
- Jupyter 노트북 내에서 BigQuery 쿼리를 실행하고 Pandas를 사용하여 출력을 처리
## 작업 1. JupyterLab 노트북 인스턴스 시작
1. 탐색 메뉴에서 Vertex AI > 대시보드 클릭 후 모든 권장 API 활성화 클릭
2. 워크벤치 클릭 후 사용자 관리 노트북 클릭
3. 페이지 상단의 + 새로 만들기 클릭
4. 환겨에서 Python 3(intel MKL 포함) 클릭 후 측면 창 하단의 고급 옵션 클릭
5. 왼쪽 창에서 머신 유형을 클릭한 후 옵션 목록에서 E2-standard 및 e2-standard-4를 선택
6. 만들기 클릭 후 대기. 노트북 생성을 완료하는데 4~7분
7. Vertex AI 콘솔에 인스턴스 이름과 Open Jupyterlab이 표시됨. 열기 클릭
8. Notebook 아래에서 Python 3 클릭
## 작업 2. BigQuery 쿼리 실행
1. 라이브러리 설치
    ~~~
    !pip install google-cloud-bigquery==2.34.4
    ~~~
2. 업데이트된 BigQuery 모듈을 로드하기 위해 커널 다시 시작.
3. 다음 쉘에 다음 쿼리 입력
    ~~~sql
    %%bigquery df --use_rest_api
    SELECT
        depdelay as departure_delay,
        COUNT(1) AS num_flights,
        APPROX_QUANTILES(arrdelay, 10) AS arrival_delay_deciles
    FROM
        `cloud-training-demos.airline_ontime_data.flights`
    WHERE
        depdelay is not null
    GROUP BY
        depdelay
    HAVING
        num_flights > 100
    ORDER BY
        depdelay ASC
    ~~~
    이 명령은 매직 함수 %%bigquery를 사용. 노트북의 매직 함수는 시스템 명령에 대한 별칭을 제공. 이 경우 %%bigquery는 BigQuery의 셀에서 쿼리를 실행하고 출력을 df라는 이름의 Pandas 데이터 프레임 개체에 저장.
4. 새 쉘에서 다음 코드를 실행하여 쿼리 출력의 처음 5개 행을 봄
    ~~~
    df.head()
    ~~~

## Pandas로 플롯 만들기
쿼리 출력이 포함된 Pandas DataFrame을 사용하여 도착 지연이 출발 지연에 어떻게 대응하는지를 나타내는 플롯을 빌드
1. 필요한 데이터가 포함된 데이터 프레임을 얻으려면 먼저 원시 쿼리 출력을 랭글링해야 합니다. 새 셀에 다음 코드를 입력하여 arrival_delay_deciles 목록을 판다 시리즈 객체로 변환합니다. 이 코드는 결과 열의 이름도 바꿉니다
    ~~~
    import pandas as pd
    percentiles = df['arrival_delay_deciles'].apply(pd.Series)
    percentiles.rename(columns = lambda x : '{0}%'.format(x*10), inplace=True)
    percentiles.head()
    ~~~
2. 출발 지연 시간을 도착 지연 시간과 연관시키고자 하므로 percentiles 테이블을 원래 데이터 프레임의 departure_delay 필드에 연결해야 함. 새 셀에서 다음 코드를 실행
    ~~~
    df = pd.concat([df['departure_delay'], percentiles], axis=1)
    df.head()
    ~~~
3. 데이터프레임의 내용을 플로팅하기 전에 0% 및 100% 필드에 저장된 극단적인 값을 삭제해야 함. 새 셀에서 다음 코드를 실행
    ~~~
    df.drop(labels=['0%', '100%'], axis=1, inplace=True)
    df.plot(x='departure_delay', xlim=(-30,50), ylim=(-50,50));
    ~~~

### 끝