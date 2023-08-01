# [15] Streaming Data Processing: Streaming Data Pipelines into Bigtable
Dataflow를 사용하여 Google Cloud PubSub를 통해 제공되는 시뮬레이션된 교통 센서 데이터에서 교통 이벤트를 수집하고 Bigtable 테이블에 기록
### 목표
- Dataflow 파이프라인을 실행하여 Pub/Sub에서 읽고 Bigtable에 씀
- HBase 쉘을 열어 Bigtable 데이터베이스를 쿼리함

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
### Hbase 빠른 시작 파일 준비
- SSH 터미널에서 스크립트를 실행하여 빠른 시작 파일을 다운로드하고 압축을 품(나중에 이 파일을 사용하여 HBase 쉘을 실행)
    ~~~shell
    cd ~/training-data-analyst/courses/streaming/process/sandiego
    ./install_quickstart.sh
    ~~~

## 작업2. 트래픽 센서 데이터를 Pub/Sub로 시뮬레이션
- SSH 터미널에서 센서 세뮬레이터를 시작. 스크립트는 csv 파일에서 샘플 데이터를 읽고 이를 Pub/Sub에 게시
    ~~~shell
    /training/sensor_magic.sh
    ~~~

### 두 번째 SSH 터미널을 열고 학습 VM에 연결
1. 두 번째 SSH 터미널 새로 연결
2. 다음 입력
    ~~~shell
    source /training/project_env.sh
    ~~~

## 작업 3. 데이터 흐름 파이프라인 시작
1. 적절한 API 및 권한이 설정되었는지 확인하려면 CLoud Shell에서 다음 코드 블록을 실행
    ~~~shell
    gcloud services disable dataflow.googleapis.com --force
    gcloud services enable dataflow.googleapis.com
    ~~~
2. 두 번째 training-vm SSH 터미널에서 해당 디렉토리로 이동하여 스크립트 검사. 코드 변경 X
    ~shell
    cd ~/training-data-analyst/courses/streaming/process/sandiego
    nano run_oncloud.sh
    ~~~
3. Ctrl+X로 종료
4. 다음 스크립트를 실행하여 Bigtable 인스턴스를 만듦
    ~~~shell
    cd ~/training-data-analyst/courses/streaming/process/sandiego
    ./create_cbt.sh
    ~~~
5. Dataflow 파이프라인을 실행하여 PubSub에서 읽고 Cloud Bigtable에 씀
    ~~~shell
    cd ~/training-data-analyst/courses/streaming/process/sandiego
    ./run_oncloud.sh $DEVSHELL_PROJECT_ID $BUCKET CurrentConditions --bigtable
    ~~~

## 작업 4. 파이프라인 탐색
1. Dataflow 클릭 후 새 파이프라인 작업 클릭. 파이프라인 작업이 나열되는지 확인하고 오류 없이 실행 중인지 확인
2. 파이프라인 그래프에서 write:cbt 단계를 찾고 오른쪽의 아래쪽 화살표를 클릭하여 작성기 작동을 확인. 단계 요약에서 Bigtable 옵션 검토

## 작업 5. Bigtable 데이터 쿼리


