# Google Cloud Pub/Sub
Google Cloud Pub/Sub는 애플리케이션과 서비스 간에 이벤트 데이터를 교환하기 위한 메시징 서비스이다. 데이터 생산자는 Pub/Sub Topic에 메시지를 게시한다. 소비자는 해당 Topic에 대한 구독을 생성한다. 구독자는 구독에서 메시지를 Pull하거나 Push 구독을 위한 웹훅으로 구성된다. 모든 가입자는 구성 가능한 기간 내에 각 메시지를 승인해야 한다.

### 목적
- 데이터를 보유할 주제를 설정
- 데이터에 액세스하려면 Topic을 구독
- Pull 구독자로 메시지를 개시한 다음 소비

## 작업 1. Pub/Sub 설정
Pub/Sub를 사용하려면 데이터를 보관할 주제를 만들고 주제에 게시된 데이터에 액세스하기 위한 구독을 만든다  

1. Pub/Sub에서 Topics 선택
2. Topic 만들기를 클릭
3. Topic 만들기 대화 상자에서

    - Topic ID 지정: 여기에선 MyTopic
    - 다른 필드 기본값
    - 만들기 클릭

## 작업 2. 구독 추가
Topic에 액세스하기 위해 구독을 추가
1. Topic에서 방금 만든 Topic에 대해 점 세 개 아이콘 클릭 후 구독 만들기 클릭
2. Topic 구독 추가 대화 상자에서

    - 구독 이름 설정: 여기에선 MySub
    - 전달 유형을 Pull로 설정
    - 다른 모든 옵션은 기본값
    - 만들기 클릭

## ~~작업 3. 이해도 테스트는 건너뛰자~~
## 작업 4. 주제에 메시지 게시
1. pub/sub > Topics로 이동하여 MyTopics 페이지 클릭
2. 주제 세부 정보 페이지에서 메시지 탭을 클릭한 다음 메시지 게시 클릭
3. 메시지 `Hello World`를 필드에 입력하고 게시 클릭

## 작업 5. 메시지 보기
메시지를 보려면 구독(MySub)을 사용하여 Topic(MyTopic)에서 메시지(Hello World)를 가져온다.
- 다음 명령을 Cloud Shell에 입력
    ~~~shell
    gcloud pubsub subscriptions pull --auto-ack MySub
    ~~~
[`주의`] pull을 통해 메시지를 한 번 가져오면 메시지를 가져온 그 구독자는 메시지를 다시 가져올 수 없음  

Pub/Sub Topic을 만들고 Topic에 게시하고 구독을 만든 다음 구독을 사용하여 Topic에서 데이터를 가져왔다!