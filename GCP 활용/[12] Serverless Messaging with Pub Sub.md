# [12] Serverless Messaging with Pub Sub : Streaming Data Processing: Publish Streaming Data into PubSub
이 실습에서는 트래픽 센서 데이터를 나중에 Dataflow 파이프라인에서 처리할 Pub/Sub Topic으로 시뮬레이션한 후 최종적으로 추가 분석을 위해 BigQuery 테이블에 저장함.
### 목표
- Pub/Sub Topic 및 구독 만들기
- 트래픽 센서 데이터를 Pub/Sub로 시뮬레이션

## 작업 1. 준비
학습 VM에서 센서 시뮬레이터를 실행하게 됨. 여러 파일과 일부 환경 설정이 필요

### SSH 터미널 열고 학습 VM에 연결
- VM인스턴스테서 training-vm이라는 인스턴스의 SSH 접속

### 초기화가 완료되었는지 확인
~~~shell
ls /training
~~~
-> 전체 목록이 표시되지 않으면 몇 분 기다린 후 다시 시도

### 코드 저장소 다운로드
- 다음으로 이 실습에서 사용할 코드 레포를 다운
    ~~~shell
    git clone https://github.com/GoogleCloudPlatform/training-data-analyst
    ~~~

### 프로젝트 식별
설정할 환경 변수 중 하나는 청구 가능한 리소스에 액세스하는데 필요한 Google Cloud 프로젝트 ID가 포함된 $DEVSHELL_PROJECT_ID 임
- training-vm SSH 터미널에서 DEVSHELL_PROJECT_ID 환경 변수를 설정하고 다른 쉘에서 사용할 수 있도록 내보냄. 다음 명령어는 Google Cloud 환경에서 활성 프로젝트 ID를 가져옴
    ~~~shell
    export DEVSHELL_PROJECT_ID=$(gcloud config get-value project)
    ~~~

## 작업 2. Pub/Sub Topic 및 구독 만들기
1. SSH 터미널에서 이 실습의 디렉토리로 이동
    ~~~shell
    cd ~/training-data-analyst/courses/streaming/publish
    ~~~

### gcloud 명령어를 사용하여 Pub/Sub 서비스에 액세스할 수 있고 작동하는지 확인
2. Topic을 만들고 간단한 메시지를 게시
    ~~~shell
    gcloud pubsub topics create sandiego
    ~~~
3. 간단한 메시지 게시
    ~~~shell
    gcloud pubsub topics publish sandiego --message "hello"
    ~~~
4. Topic에 대한 구독 생성
    ~~~shell
    gcloud pubsub subscriptions create --topic sandiego mySub1
    ~~~
5. 주제에 게시된 첫 번째 메시지를 가져옴
    ~~~shell
    gcloud pubsub subscriptions pull --auto-ack mySub1
    ~~~
6. 다른 메시지를 게시한 다음 구독을 사용하여 가져옴
    ~~~shell
    gcloud pubsub topics publish sandiego --message "hello again"
    gcloud pubsub subscriptions pull --auto-ack mySub1
    ~~~
7. 구독 취소
    ~~~shell
    gcloud pubsub subscriptions delete mySub1
    ~~~

## 작업3. 트래픽 센서 데이터를 Pub/Sub로 시뮬레이션
1. San Diego 교통 센서 데이터를 시뮬레이션 하는 Python 스크립트를 탐색. 코드 변경 X
    ~~~shell
    cd ~/training-data-analyst/courses/streaming/publish
    nano send_sensor_data.py
    ~~~
2. 교통 시뮬레이션 데이터 세트 다운로드
    ~~~shell
    ./download_data.sh
    ~~~
### 스트리밍 센서 데이터 시뮬레이션
3. send_sensor_data.py 실행
    ~~~shell
    ./send_sensor_data.py --speedFactor=60 --project $DEVSHELL_PROJECT_ID
    ~~~
    이 명령은 Pub/Sub 메시지를 통해 기록된 센서 데이터를 전송하여 센서 데이터를 시뮬레이션합니다. 스크립트는 센서 데이터의 원래 시간을 추출하고 각 메시지를 보내는 사이에 일시 중지하여 센서 데이터의 실제 타이밍을 시뮬레이션합니다. 값 speedFactor는 메시지 사이의 시간을 비례적으로 변경합니다. 따라서 60의 speedFactor는 기록된 타이밍보다 "60배 더 빠름"을 의미합니다. 60초마다 약 1시간 분량의 데이터를 전송합니다.

    이 터미널을 열어두고 시뮬레이터를 실행하십시오.

## 작업 4. 메시지 수신 확인

### 두 번째 SSH 터미널을 열고 학습 VM에 연결
1. SSH를 클릭하여 두 번째 터미널 창 open
2. 작업 중이던 디렉토리로 변경
    ~~~shell
    cd ~/training-data-analyst/courses/streaming/publish
    ~~~
3. Topic에 대한 구독을 만들고 pull을 수행하여 메시지가 수신되는지 확인. (참고: 메시지를 보려면 pull 명령을 두 번 이상 실행해야 할 수 있음).
    ~~~shell
    gcloud pubsub subscriptions create --topic sandiego mySub2
    gcloud pubsub subscriptions pull --auto-ack mySub2
    ~~~
4. 트래픽 센서 정보가 포함된 메시지가 표시되는ㄴ지 확인
5. 구독 취소
    ~~~shell
    gcloud pubsub subscriptions delete mySub2
    ~~~
6. exit

### 센서 시뮬레이터 중지
1. 첫 번째 터미널로 돌아가 Ctrl+C 입력
2. exit

### 끝