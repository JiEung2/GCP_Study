# [1] Using the Natural Language API to classify unstructured text
Cloud Natural Language API를 사용하면 텍스트에서 항목을 추출하고 감정 및 구분 분석을 수행하며 텍스트를 카테고리로 분류할 수 있음.
이 실습에서는 텍스트 분류에 중점을 둠. 700개 이상의 범주로 구성된 데이터베이스를 사용하는 이 API 기능을 사용하면 대규모 텍스트 데이터 세트를 쉽게 분류할 수 있음
### 목차
- Natural Language API 요청 생성 및 curl로 API 호출
- NL API의 텍스트 분류 기능 사용
- 텍스트 분류를 사용하여 뉴스 기사 데이터 세트 이해

## 작업 1. Cloud Natural Language API가 사용 설정되었는지 확인
1. API 및 서비스 > 활성화된 API 및 서비스를 선택
2. +API 및 서비스 활성화 클릭
3. language 검색창에 Cloud Natural Language API를 클릭
4. API가 활성화되지 않은 경우 활성화 버튼이 표시됨. 사용을 클릭
API가 사용 설정되면 Cloud Natrual Language API 타일에 관리 버튼이 표시됨

## 작업 2. API 키 생성
Natural Language API에 요청을 보내는데 사용하고 있으므로 curl 요청 URL에 전달할 API 키를 생성해야함
1. API 키를 생성하려면 콘솔에서 탐색 메뉴 > API 및 서비스 > 자격 증명을 클릭
2. 그런 다음 +CREATE CREDENTIALS 클릭
3. 드롭다운 메뉴에서 API 키 선택
4. 방금 생성한 키 복사
5. Cloud Shell에서 다음을 실행, YOUR_API_KEY 부분에 방금 복사한 키 삽입
    ~~~shell
    export API_KEY=<YOUR_API_KEY>
    ~~~

## 작업 3. 뉴스 기사 분류
Natural Language API의 classifyText 메서드를 사용하면 `단일 API 호출로 텍스트 데이터를 범주로 정렬할 수 있습니다`. 이 메서드는 텍스트 문서에 적용되는 콘텐츠 범주 목록을 반환합니다. 이러한 범주는 /컴퓨터 및 전자 제품과 같은 광범위한 범주에서 /컴퓨터 및 전자 제품/프로그래밍/Java(프로그래밍 언어)와 같은 매우 구체적인 범주에 이르기까지 구체적입니다. 700개 이상의 가능한 카테고리의 전체 목록은 콘텐츠 카테고리 가이드 에서 찾을 수 있습니다

단일 기사를 분류하는 것으로 시작한 다음 이 방법을 사용하여 대규모 뉴스 코퍼스를 이해하는 방법을 살펴보겠습니다. 시작하려면 음식 섹션의 New York Times 기사에서 이 헤드라인과 설명을 살펴보겠습니다

`타파 트위스트를 곁들인 스모키 랍스터 샐러드. 스페인식 펄포 아 라 갈레가에 대한 이 스핀은 문어를 건너뛰지만 바다 소금, 올리브 오일, 피멘톤 및 삶은 감자는 유지합니다.`

1. Cloud Shell 환경에서 request.json아래 코드로 파일을 생성. 편집기(nano, vim, emacs) 중 하나를 사용하여 파일을 만들거나 Cloud Shell 코드 편집기를 사용할 수 있음.  
`그냥 편집기 열기해서 복붙하는걸 추천 vi로 하는데 들여쓰기 문제로 오류가 나옴`
2. request.json이라는 이름의 파일을 만들고 아래 코드 복사
    ~~~json
    {
    "document":{
        "type":"PLAIN_TEXT",
        "content":"A Smoky Lobster Salad With a Tapa Twist. This spin on the Spanish pulpo a la gallega skips the octopus, but keeps the sea salt, olive oil, pimentón and boiled potatoes."
        }
    }
    ~~~
3. 이제 다음 curl 명령을 사용하여 이 텍스트를 자연어 API의 classifyText 메서드로 보낼 수 있음
    ~~~shell
    curl "https://language.googleapis.com/v1/documents:classifyText?key=${API_KEY}" \
        -s -X POST -H "Content-Type: application/json" --data-binary @request.json
    ~~~

    API는 이 텍스트에 대해 2개의 범주를 반환함
    - /Food & Drink/Cooking & Recipes
    - /Food & Drink/Food/Meat & Seafood  
    
    텍스트는 이것이 요리법이거나 해산물이 포함되어 있다고 명시적으로 언급하지 않지만 API는 이를 분류할 수 있습니다

## 작업 4. 대용량 텍스트 데이터셋 분류
이 메서드가 텍스트가 많은 데이터 세트를 이해하는 데 어떻게 도움이 되는지 확인하기위해 이 BBC 뉴스 기사의 공개 데이터 세트를 classifyText 사용. 이 데이터세트는 2004년부터 2005년까지 5개 주제 영역(비즈니스, 엔터테인먼트, 정치, 스포츠, 기술)에 대한 2,225개의 기사로 구성되어 있음. 이러한 기사의 하위 집합은 공개 Google Cloud Storage 버킷에 있음. 각 기사는 .txt 파일에 있음.

데이터를 검사하고 이를 Natural Language API로 보내려면 Python 스크립트를 작성하여 Cloud Storage에서 각 텍스트 파일을 읽고 엔드포인트로 보내고 결과를 classifyTextBigQuery 테이블에 저장. BigQuery는 Google Cloud의 빅데이터 웨어하우스 도구로 대용량 데이터 세트를 쉽게 저장하고 분석할 수 있음.
- 작업할 텍스트 유형을 보기 위한 명령
    ~~~shell
    gcloud storage cat gs://cloud-training-demos-text/bbc_dataset/entertainment/001.txt
    ~~~

## 작업 5. 분류된 텍스트 데이터용 BigQuery 테이블 만들기
텍스트를 Natural Language API로 보내기 전에 각 기사의 텍스트와 카테고리를 저장할 장소가 필요
1. 탐색메뉴에서 BigQuery
2. 프로젝트 이름 옆의 점 세 개를 클릭 후 데이터세트 만들기 클릭
3. 이름은 news_classification_dataset으로 설정 후 만들기 클릭
4. 방금 만든 데이터세트 오른쪽 점 세개 클릭 후 테이블 만들기 클릭
- 빈 테이블
- 테이블 이름 article_data 지정
- 스키마 아래에서 필드 추가 (+) 이름: article_text 유형: STRING 이름: category 유형: STRING 이름: confidence 유형: FLOAT 총 3개의 필드 추가
5. 테이블 만들기 클릭  

지금은 테이블이 비어 있음. 다음 단계에서 Cloud Storage에서 기사를 읽고 분류를 위해 Natural Language API로 보내고 결과를 BigQuery에 저장함.

## 작업 6. 뉴스 데이터 분류 및 BigQuery에 결과 저장

뉴스 데이터를 Natural Language API로 보내는 스크립트를 작성하기 전에 서비스 계정을 만들어야 함. 이는 Python 스크립트에서 Natural Language API 및 BigQuery에 인증하는 데 사용됨.
1. Cloud Shell에서 프로젝트ID 변수 수정
    ~~~shell
    export PROJECT=<your_project_name>
    ~~~
2. 다음 명령어 실행하여 서비스 계정 생성
    ~~~shell
        gcloud iam service-accounts create my-account --display-name my-account
    gcloud projects add-iam-policy-binding $PROJECT --member=serviceAccount:my-account@$PROJECT.iam.gserviceaccount.com --role=roles/bigquery.admin
    gcloud projects add-iam-policy-binding $PROJECT --member=serviceAccount:my-account@$PROJECT.iam.gserviceaccount.com --role=roles/serviceusage.serviceUsageConsumer
    gcloud iam service-accounts keys create key.json --iam-account=my-account@$PROJECT.iam.gserviceaccount.com
    export GOOGLE_APPLICATION_CREDENTIALS=key.json
    ~~~

    여기까지 하면 텍스트 데이터를 Natural Language API로 보낼 준비 완료!
3. classify-text.py라는 파일을 만들고 다음을 복붙. YOUR_PROJECT를 나의 프로젝트 ID로 변경
    ~~~py
    from google.cloud import storage, language_v1, bigquery
    # Set up our GCS, NL, and BigQuery clients
    storage_client = storage.Client()
    nl_client = language_v1.LanguageServiceClient()
    # TODO: replace YOUR_PROJECT with your project id below
    bq_client = bigquery.Client(project='YOUR_PROJECT')
    dataset_ref = bq_client.dataset('news_classification_dataset')
    dataset = bigquery.Dataset(dataset_ref)
    table_ref = dataset.table('article_data') # Update this if you used a different table name
    table = bq_client.get_table(table_ref)
    # Send article text to the NL API's classifyText method
    def classify_text(article):
            response = nl_client.classify_text(
                    document=language_v1.types.Document(
                            content=article,
                            type_='PLAIN_TEXT'
                    )
            )
            return response
    rows_for_bq = []
    files = storage_client.bucket('cloud-training-demos-text').list_blobs()
    print("Got article files from GCS, sending them to the NL API (this will take ~2 minutes)...")
    # Send files to the NL API and save the result to send to BigQuery
    for file in files:
            if file.name.endswith('txt'):
                    article_text = file.download_as_bytes()
                    nl_response = classify_text(article_text)
                    if len(nl_response.categories) > 0:
                            rows_for_bq.append((str(article_text), str(nl_response.categories[0].name), nl_response.categories[0].confidence))
    print("Writing NL API article data to BigQuery...")
    # Write article text + category data to BQ
    errors = bq_client.insert_rows(table, rows_for_bq)
    assert errors == []
    ~~~
4. 다음 스크립트 실행
    ~~~shell
    python3 classify-text.py
    ~~~
    스크립트 실행이 완료되면 기사 데이터가 BigQuery에 저장되었는지 확인할 차례
5. BigQuery에서 article_dataBigQuery 탭의 테이블로 이동하고 QUERY > In new tab을 클릭
6. Untitled 상자 에서 결과를 편집하고 SELECT와 FROM 사이에 별표를 추가
    ~~~sql
    SELECT * FROM `news_classification_dataset.article_data`
    ~~~
7. 실행  

    카테고리 열에는 기사에 대해 Natural Language API가 반환한 첫 번째 카테고리의 이름이 있고 신뢰도는 API가 기사를 올바르게 분류했다는 확신을 나타내는 0과 1 사이의 값임. 다음 단계에서 데이터에 대해 더 복잡한 쿼리를 수행하는 방법을 배움.

## 작업 7. BigQuery에서 분류된 뉴스 데이터 분석
먼저, 데이터 세트에서 어떤 범주가 가장 일반적인지 확인
1. BigQuery 콘솔에서 + 새 쿼리 작성 클릭
2. 다음 쿼리 입력
    ~~~sql
    SELECT
        category,
        COUNT(*) c
    FROM
        `news_classification_dataset.article_data`
    GROUP BY
        category
    ORDER BY
        c DESC
    ~~~
3. 실행
4. /Arts & Entertainment/Music & Audio/Classical Music와 같이 더 모호한 범주에 대해 반환된 기사를 찾으려면 다음 쿼리를 실행
    ~~~sql
    SELECT * FROM `news_classification_dataset.article_data`
    WHERE category = "/Arts & Entertainment/Music & Audio/Classical Music"
    ~~~

5. Natural Language API가 90%보다 높은 신뢰도 점수를 반환한 기사만 가져오려면 다음 쿼리를 실행
    ~~~sql
    SELECT
        article_text,
        category
    FROM `news_classification_dataset.article_data`
    WHERE cast(confidence as float64) > 0.9
    ~~~

### 끝