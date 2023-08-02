# [13] Streaming Data Processing: Streaming Data Pipelines
Dataflow를 사용하여 Google Cloud PubSub를 통해 제공되는 시뮬레이션된 교통 센서 데이터에서 교통 이벤트를 수집하고 이를 실행 가능한 평균으로 처리하고 나중에 분석할 수 있도록 원시 데이터를 BigQuery에 저장.
Dataflow 파이프라인을 시작하고 모니터링하고 마지막으로 최적화 하는 방법을 배움.

### 목표
- Dataflow 시작 및 Dataflow 작업 실행
- Dataflow 파이프라인의 변환을 통해 데이터 요소가 흐르는 방식 이해
- Dataflow를 Pub/Sub 및 BigQuery에 연결

## 작업 1. 준비
### 초기화가 완료되었는지 확인
- 밑의 코드 실행으로 초기화되었는지 확인
~~~shell
ls /training
~~~
### 코드 저장소 다운로드
- 실습에서 사용할 코드 레포 다운
~~~shell
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
~~~
### 환경 변수 설정
- SSH 터미널에서 다음 입력
~~~shell
source /training/project_env.sh
~~~

## 작업 2. BigQuery 데이터 세트 및 Cloud Storage 버킷 만들기
Dataflow 파이프라인은 나중에 생성되어 BigQuery 데이터 세트의 테이블에 기록됨
### BigQuery 데이터세트 만들기
1. 탐색메뉴 > BigQuery 선택 후 프로젝트ID 옆에 점 세개 클릭하고 데이터세트 만들기 선택
2. 데이터 세트 ID의 이름을 demos라고 지장하고 데이터 세트 생성 클릭

### Cloud Storage 버킷 확인
프로젝트 ID와 이름이 같은 버킷이 이미 존재해야함

## 작업 3. 트래픽 센서 데이터를 Pub/Sub로 시뮬레이션
- training-vm SSH 터미널에서 센서 시뮬레이터 시작. 스크립트는 CSV 파일에서 샘플 데이터를 읽고 Pub/Sub에 게시
    ~~~shell
    /training/sensor_magic.sh
    ~~~
### 두 번째 SSH 터미널을 열고 training-vm에 연결
- 새 training-vm SSH 터미널에 다음 입력
    ~~~shell
    source /training/project_env.sh
    ~~~

## 작업 4. Dataflow 파이프라인 실행
### 프로젝트에 Google Cloud Dataflow API가 사용 설정되었는지 확인
1. 적절한 API 및 권한이 설정되었는지 확인.
    ~~~shell
    gcloud services disable dataflow.googleapis.com --force
    gcloud services enable dataflow.googleapis.com
    ~~~
2. 두 번째 training-vm SSH 터미널로 돌아가 디렉토리 접속
    ~~~shell
    cd ~/training-data-analyst/courses/streaming/process/sandiego
    ~~~
3. Dataflow 파이프라인을 만들고 실행하는 스크립트를 식별
    ~~~shell
    cat run_oncloud.sh
    ~~~
4. [Github에서 소스코드를 보기위한 URL](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/courses/streaming/process/sandiego/run_oncloud.sh)
5. 스크립터에는 프로젝트id, 버킷 이름, 클래스 이름의 세 가지 인수가 필요
6. 자바 디렉토리로 이동. 소스파일 AverageSpeeds.java 식별
    ~~~shell
    cd ~/training-data-analyst/courses/streaming/process/sandiego/src/main/java/com/google/cloud/training/dataanalyst/sandiego
    cat AverageSpeeds.java
    ~~~
7. [Github에서 AverageSpeeds.java 파일을 확인하기 위한 URL](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/courses/streaming/process/sandiego/src/main/java/com/google/cloud/training/dataanalyst/sandiego/AverageSpeeds.java)

8. SSH 터미널로 돌아가 Dataflow 파이프라인을 실행하여 PubSub에서 읽고 BigQuery에 씀
    ~~~shell
    cd ~/training-data-analyst/courses/streaming/process/sandiego
    ./run_oncloud.sh $DEVSHELL_PROJECT_ID $BUCKET AverageSpeeds
    ~~~
    좀 걸림

## 작업 5. 파이프라인 탐색
이 Dataflow 파이프라인은 Pub/Sub 주제에서 메시지를 읽고, 입력 메시지의 JSON을 파싱하고, 하나의 기본 출력을 생성하고 BigQuery에 씀
1. 탐색 메뉴에서 Dataflow를 클릭 하고 작업을 클릭하여 진행 상황을 모니터링
2. 파이프라인이 실행된 후 Pub/Sub > Topics 클릭
3. sandiego에 대한 Topic 이름 행 검사
4. Datflow를 클릭하고 작업 클릭
5. Github 브라우저 탭으 코드, AverageSpeeds.java 및 Dataflow 작업 페이지의 파이프라인 그래프를 비교
6. 그래프에서 GetMessages 파이프라인 단계를 찾은 다음 AverageSpeeds.java 파일에서 해당 코드를 찾음. Pub/Sub 주제에서 읽는 파이프라인 단계. 읽은 Pub/Sub 메시지에 해당하는 문자열 모음을 만듦
7. 그래프와 코드에서 시간 창 파이프라인 단계를 찾음. 이 파이프라인 단계에서는 파이프라인 매개변수에 지정된 기간의 창을 만듦. 이 기간은 기간이 끝날 때까지 이전 단계의 트래픽 데이터를 누적하고 추가 변환을 위해 다음 단계로 전달

8. 그래프에서 BySensor 및 AvgBySensor 파이프라인 단계를 찾은 다음 AverageSpeeds.java 파일에서 해당 코드 조각을 찾음. 이 BySensor는 센서 ID별로 창의 모든 이벤트를 그룹화하고 AvgBySensor는 각 그룹화에 대한 평균 속도를 계산

9. 그래프와 코드에서 ToBQRow 파이프라인 단계를 찾음. 이 단계는 차선 정보와 함께 이전 단계에서 계산된 평균으로 "행"을 생성
10. 파이프라인 그래프와 소스 코드 모두에서 BigQueryIO.Write를 찾음. 이 단계에서는 파이프라인의 행을 BigQuery 테이블에 씀. WriteDisposition.WRITE_APPEND 쓰기 처리를 선택했기 때문에 새 레코드가 테이블에 추가

11. BigQuery 웹 UI 탭으로 돌아가서 새로고침

122. 프로젝트 이름과 생성한 데모 데이터 세트를 찾움. 이제 데이터 세트 이름 demos 의 왼쪽에 있는 작은 화살표가 활성화되고 이를 클릭하면 average_speeds 테이블이 표시

13. average_speeds 테이블이 BigQuery에 표시되기 까지 몇 분 정도 걸림

## 작업 6. 처리 속도 결정
Dataflow 파이프라인을 모니터링하고 개선할 때 공통적인 활동 중 하나는 파이프라인이 초당 처리하는 요소 수, 시스템 지연, 지금까지 처리도니 데이터 요소 수를 파악하는 것. 이 활동에서는 Cloud Console에서 처리된 요소 및 시간에 대한 정보를 찾을 수 있는 위치를 알아봄.
1. Dataflow 클릭하고 작업 클릭
2. 그래프에서 GetMessages 파이프라인 노드를 선택하고 오른쪽에서 단계 메트릭을 확인

    - 시스템 지연은 스트리밍 파이프라인의 중요한 메트릭임. 데이터 요소가 변환 단계의 입력에 "도착"한 이후 처리를 기다리는 시간을 나타냄.
    - 출력 컬렉션 아래의 추가된 요소 메트릭은 이 단계를 종료한 데이터 요소 수를 알려줌. (파이프라인의 PubSub 메시지 읽기 단계의 경우 Pub/Sub IO 커넥터가 주제에서 읽은 Pub/Sub 메시지 수도 나타냄).
3. 그래프에서 시간 창 노드를 선택. 시간 창 단계의 입력 컬렉션 아래 추가된 요소 메트릭이 이전 단계 GetMessages의 출력 컬렉션 아래 추가된 요소 메트릭과 어떻게 일치하는지 관찰.

## 작업 7. BigQuery 출력 검토
1. BigQuery 클릭
2. 쿼리 편집기 창에서 다음 쿼리를 입력. Dataflow 작업이 출력을 관찰하려면 다음 쿼리 사용.
    ~~~sql
    SELECT *
    FROM `demos.average_speeds`
        ORDER BY timestamp DESC
    LIMIT 100
    ~~~
3. 다음 SQL을 실행하여 테이블에 대한 마지막 업데이트를 찾음
    ~~~sql
    SELECT
    MAX(timestamp)
    FROM
    `demos.average_speeds`
    ~~~
