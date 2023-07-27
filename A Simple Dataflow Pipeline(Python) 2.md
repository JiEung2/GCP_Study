# A Simple Dataflow Pipeline(Python) 2.5

Dataflow 프로젝트를 열고 파이프라인 필터링을 사용하며 파이프라인을 로컬 및 클라우드에서 실행
- Dataflow 프로젝트 열기
- 파이프라인 필터링
- 로컬 및 클라우드에서 파이프라인 실행

## 목적
간단한 Dataflow 파이프라인을 작성하고 로컬 및 클라우드에서 실행하는 방법을 알아봄
- Apache Beam을 사용하여 Python Dataflow 프로젝트 설정
- Python으로 간단한 파이프라인 작성
- 로컬 컴퓨터에서 쿼리 실행
- 클라우드에서 쿼리 실행
---

## 작업 1. Dataflow API가 성공적으로 사용설정되었는지 확인
- Cloud Shell에서 다음 실행
~~~Shell
gcloud services disable dataflow.googleapis.com --force
gcloud services enable dataflow.googleapis.com
~~~

## 작업 2. 준비
퀵랩 환경에서는 그냥 하면 되지만 개인 아이디에서는 모든 설정을 수동으로 해줘야함 <br>
`training-vm`이라는 가상머신을 생성하여 SSH 접속 후 다음 설치 명령 먼저 수행 <br>
[주의]: VM생성시 `ID 및 API 액세스`의 `액세스 범위`에서 `모든 Cloud API에 대한 전체 액세스 허용` 선택

---
[Debian git 설치]<br>
~~~shell
sudo apt update
sudo apt install git
~~~
[가상머신에 python3과 apache-beam설치]
~~~shell
sudo apt-get -y install python3-pip
pip install apache-beam[gcp]
~~~
[BUCKET 환경변수 설정]
~~~shell
BUCKET="jnu-idv-xx"
echo $BUCKET
~~~

### 코드 저장소 다운로드
- training-vm SSH 터미널에서 다음 입력
    ~~~shell
    git clone https://github.com/GoogleCloudPlatform/training-data-analyst
    ~~~

### Cloud Storage 버킷 만들기
- 프로젝트 이름과 동일한 버킷 생성
- 위치 유형 -> Multi-Region으로 설정

## 작업 3. 파이프라인 필터링
Dataflow 프로젝트의 구조에 익숙해지고 Dataflow 파이프라인을 실행하는 방법
- training-vm SSH 터미널에서 `/training-data-analyst/courses/data_analysis/lab2/python` 디렉토리로 이동하여 `grep.py` 파일을 봄
    ~~~shell
    cd ~/training-data-analyst/courses/data_analysis/lab2/python
    nano grep.py
    ~~~
- 코드 변경 X

<details>
<summary>코드 보기</summary>
<div markdown="1">       

~~~py
import apache_beam as beam
import sys

def my_grep(line, term):
   if line.startswith(term):
      yield line

if __name__ == '__main__':
   p = beam.Pipeline(argv=sys.argv)
   input = '../javahelp/src/main/java/com/google/cloud/training/dataanalyst/javahelp/*.java'
   output_prefix = '/tmp/output'
   searchTerm = 'import'

   # find all lines that contain the searchTerm
   (p
      | 'GetJava' >> beam.io.ReadFromText(input)
      | 'Grep' >> beam.FlatMap(lambda line: my_grep(line, searchTerm) )
      | 'write' >> beam.io.WriteToText(output_prefix)
   )

   p.run().wait_until_finish()
~~~

</div>
</details>


### `grep.py` 파일을 보고 질문에 답 <br>
- 어떤 파일을 읽고 있나? <br>
    ~~~
    파이프라인은 ../javahelp/src/main/java/com/google/cloud/training/dataanalyst/javahelp/*.java 경로의 자바 파일들을 읽어옴
- 검색어가 무엇인가?
    ~~~
    검색어는 import
- 출력은 어디로 가나?
    ~~~
    결과는 '/tmp/output' 디렉토리에 있는 파일에 기록됨
### 파이프라인에는 세 가지 변환이 있음
- 변환은 무엇을 하나?
    ~~~
    첫 번째 변환: 'ReadFromText' - 자바 파일을 읽어와서 PCollection으로 변환
    두 번째 변환: 'FlatMap' - 각 라인에 대해 'my_grep' 함수를 적용하여 검색어를 포함하는 라인만을 추출
    세 번째 변환: 'WriteToText' - 검색 결과를 텍스트 파일로 출력
- 두 번째 변환의 입력은 어디에서 오는가?
    ~~~
    두 번째 변환인 'FlatMap'의 입력은 첫 번째 변환인 'ReadFromText'에서 생성된 PCollection임
- 이 입력으로 무엇을 하나?
    ~~~
    'FlatMap' 변환은 각 라인에 대해 'my_grep' 함수를 적용함. 따라서 각 라인이 'my_grep' 함수의 'line' 인자로 전달되고, 'serchTerm' 인자는 "import"임. 이 함수는 라인이 import로 시작하는지 확인하고, 만약 조건ㅇ르 만족하면 해당 라인 출력
## 작업 4. 로컬에서 파이프라인 실행
1. training-vm SSH 터미널에서 다음 실행  
    ~~~shell
    python3 grep.py
2. 파일 확인
    ~~~shell
    ls -al /tmp
3. 다음 코드로 output파일 내용 확인
    ~~~shell
    cat /tmp/output-*

## 작업 5. 클라우드에서 파이프라인 실행
1. 일부 Java 파일을 클라우드에 복사하기 위해 training-vm SSH 터미널에서 다음 명령 입력
    ~~~shell
    gcloud storage cp ../javahelp/src/main/java/com/google/cloud/training/dataanalyst/javahelp/*.java gs://$BUCKET/javahelp
2. grepc.py에서 Dataflow 파이프라인 수정(나노나 vi편집기 아무거나 사용)
3. PROJECT 및 BUCKET을 프로젝트 ID 및 버킷 이름으로 수정
    ~~~shell
    PROJECT = 'Project ID'
    BUCKET = 'Bucket Name'
4. Dataflow 작업을 클라우드에 제출
    ~~~shell
    python3 grepc.py
    ~~~
    대략 5-10분 정도 걸림
5. 브라우저 탭으로 돌아가 Dataflow를 클릭하고 작업을 클릭하여 진행 상황 모니터링
6. 작업 상태가 성공이 될 때까지 기다림
7. Cloud Storage 버킷의 출력 검사
8. Cloud Storage > 버킷 클릭
9. javahelp 디렉토리 클릭  
이 작업은 output.txt 파일을 생성.   
파일이 충분히 크면 다음과 같은 이름을 가진 여러 부분으로 나눠질 수 있음.(`output-0000x-of-000y`)  
이름 또는 최종 수정 필드 로 가장 최근 파일을 식별할 수 있음.
10. 파일을 클릭하면 볼 수 있음  
또는 training-vm SSH 터미널을 통해 파일을 다운로드하고 볼 수 있음
    ~~~shell
    gcloud storage cp gs://$BUCKET/javahelp/output* .
    cat output*