# GCP_Study
공부를 하자


## Fork한 REpository 업데이트 하기
  원본 저장소: 다른 사람의 레포
  포크 저장소: 다른 사람의 레포를 fork해온 내 레포

1. 내 로컬에 포크한 저장소 Clone

    ~~~shell
    $ git clone <포크한 내 레포 주소>
    ~~~

2. Clone한 디렉토리로 이동
3. 리코드 저장소 확인
    ~~~shell
    $ git remote -v
    origin 주소 (fetch)
    origin 주소 (push)
    ~~~
4. 이제 업데이트를 위해 리모트 저장소에 원본 저장소 추가

    ~~~shell
    $ git remote add upstream <원본 저장소 주소>
    $ git remote -v
    origin ~~ (fetch)
    origin ~~ (push)
    upstream ~~ (fetch)
    upstream ~~ (push)
    ~~~

5. 원본 저장소 fetch  (원본 저장소의 최신 업데이트 파일을 가져온다)
    ~~~shell
    $ git fetch upstream
    ~~~
6. 원본 저장소 merge (내 메인 저장소와 merge)
    ~~~shell
    $ git merge upstream/main
    ~~~
7. fork 저장소로 push
    ~~~shell
    $ git push
    ~~~