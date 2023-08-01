# Google Cloud Pub/Sub_Command Line

콘솔 창에서 진행했던 Pub/Sub에 대한 실습을 Command Line으로 진행해보자.

### 목표
- Pub/Sub의 기본사항
- Pub/Sub Topic을 만들고 삭제하고 나열
- Pub/Sub 구독을 만들고 삭제하고 나열
- 메시지를 Topic에 게시
- Pull 구독자를 사용하여 개별 Topic 메시지를 출력
- 플래그가 있는 Pull 구독자를 사용하여 여러 메시지를 출력

## Pub/Sub Basic
앞에서 배웠던 것과 같이 Pub/Sub는 비동기식 글로벌 메시징 서비스이다.  
Pub/Sub에는 topics, publicshing, subscribing이라는 세 가지 용어가 자주 나타난다.
- Topic은 응용 프로그램이 공통 스레드를 통해 서로 연결할 수 있도록 하는 공유 문자열
- Publish 게시자는 Pub/Sub Topic에 메시지를 Push
- 구독자는 Topic에 대해 '구독'을 신청하여 구독에서 메시지를 pull하거나 push 구독을 위한 웹훅을 구성. 모든 구독자는 구성 가능한 시간 내에 각 메시지를 승인해야 함.
요약하자면 게시자는 Topic에 메시지를 게시하고 소비자는 Topic에서 메시지를 수신하기 위해 Topic에 대한 구독을 생성한다.

## 작업 1. Pub/Sub Topic
Cloud Shell에서 실행
1. 다음 명령어를 실행하여 myTopicdmf 만든다.
    ~~~shell
    gcloud pubsub topics create myTopic
    ~~~
2. 테스트를 위해 두 가지 Topic 추가 생성. ~~(굳이 안해도 됨)~~
    ~~~shell
    gcloud pubsub topics create Test1
    ~~~
    ~~~shell
    gcloud pubsub topics create Test2
    ~~~
3. 방금 만든 세 가지 Topic을 보기 위한 명령
    ~~~shell
    gcloud pubsub topics list
    ~~~

    출력은 다음과 유사
    ~~~shell
    이름:  projects/qwiklabs-gcp-3450558d2b043890/topics/myTopic 
    --- 
    이름:  projects/qwiklabs-gcp-3450558d2b043890/topics/Test2 
    --- 
    이름:  projects/qwiklabs-gcp-3450558d2b043890/topics/Test1
    ~~~
4. Topic 삭제하는 명령 (방금 생성했던 test1과 2를 삭제)
    ~~~shell
    gcloud pubsub topics delete Test1
    ~~~
    ~~~shell
    gcloud pubsub topics delete Test2
    ~~~
5. Topic을 보는 명령어를 한 번 더 실행하여 학목이 삭제되었는지 확인
    ~~~shell
    gcloud pubsub topics list
    ~~~

## 작업 2. Pub/Sub 구독
Topic의 생성, 삭제, 확인을 해봤으니까 구독에 대한 작업을 해보자
1. 다음 명령어를 실행하여 myTopic에 대한 구독을 생성
    ~~~shell
    gcloud  pubsub subscriptions create --topic myTopic mySubscription
    ~~~

2. 테스트를 위해 테스트 구독 추가
    ~~~shell
    gcloud  pubsub subscriptions create --topic myTopic Test1
    ~~~
    ~~~shell
    gcloud  pubsub subscriptions create --topic myTopic Test2
    ~~~

3. myTopic의 구독에 대한 나열
    ~~~shell
    gcloud pubsub topics list-subscriptions myTopic
    ~~~

4. 테스트 구독 삭제
    ~~~shell
    gcloud pubsub subscriptions delete Test1
    ~~~
    ~~~shell
    gcloud pubsub subscriptions delete Test2
    ~~~

5. 잘 삭제되었는지 확인
    ~~~shell
    gcloud pubsub topics list-subscriptions myTopic
    ~~~

## 작업 3. Pub/Sub 게시 및 단일 메시지 가져오기
다음은 Pub/Sub 주제에 메시지를 게시하는 방법을 알아보자.
1. 다음 명령을 실행하여 이전에 생성한 Topic(myTopic)에 "hello"라는 메시지를 게시하자
    ~~~shell
    gcloud pubsub topics publish myTopic --message "Hello"
    ~~~
2. 메시지를 몇 개 더 게시해보자
    ~~~shell
    gcloud pubsub topics publish myTopic --message "Publisher's name is JiEung"
    ~~~
    ~~~shell
    gcloud pubsub topics publish myTopic --message "Publisher likes to eat PIZZA"
    ~~~
    ~~~shell
    gcloud pubsub topics publish myTopic --message "Publisher thinks Pub/Sub is awesome"
    ~~~
    메시지를 게시했으니 구독자는 메시지를 pull을 통해 가져와보자. 이 전에 생성한 mySubscription은 pull 구독으로 설정했기 때문에 pull로 가져와야한다.  

3. 다음 명령어를 사용하여 Pub/Sub 주제에서 방금 게시한 메시지를 가져오자.
    ~~~shell
    gcloud pubsub subscriptions pull mySubscription --auto-ack
    ~~~
    Topic에 4개의 메시지를 게시했지만 이 코드의 결과로 1개만 출력되었다. 이 결과로 볼 수 있는 pull 명령의 몇가지 기능에 대해 알아보자
    
    - 플래그 없이 pull 명령을 사용하면 더 많은 항목이 포함된 Topic을 구독하더라도 하나의 메시지만 출력된다.
    - 특정 구독 기반 pull 명령에서 개별 메시지가 출력되면 pull 명령으로 해당 메시지에 다시 액세스할 수 없다.

    <br>
    나머지 세 개의 메시지는 위의 명령을 세 번 반복하면 다 볼 수 있다.

## 작업 4. 구독에서 모든 메시지를 가져오는 Pub/Sub
그렇다면 마지막으로 플래그를 이용하여 여러 메시지를 가져오는 방법에 대해 알아보자  
마지막 예에서 모든 메시지를 가져왔으므로 메시지를 더 채우자
1. 다음 명령을 실행하자
    ~~~shell
    gcloud pubsub topics publish myTopic --message "Publisher is starting to get the hang of Pub/Sub"
    ~~~
    ~~~shell
    gcloud pubsub topics publish myTopic --message "Publisher wonders if all messages will be pulled"
    ~~~
    ~~~shell
    gcloud pubsub topics publish myTopic --message "Publisher will have to test to find out"
    ~~~
2. 이제 하나의 명령으로 모든 메시지를 불러올 수 있게 flag를 추가하자

    사실 우리는 앞의 실습에서 flag를 사용하고 있었다. pull 명령의 --auto-ask 부분은 끌어온 메시지를 깔끔한 상자에 표시하는 서식을 지정하는 플래그이다.

    limit은 가져올 메시지 수에 상한선을 설정하는 또 다른 플래그이다.
3. limit을 추가한 다음 명령을 실행하자
    ~~~shell
    gcloud pubsub subscriptions pull mySubscription --auto-ack --limit=3
    ~~~
    이 명령의 결과로 메시지 세 개가 한꺼번에 출력되는 것을 볼 수 있다.
    지금까지 Pub/Sub의 여러 기본 명령에 대해 알아보았다. 다음은 심화로 돌아오겠다~