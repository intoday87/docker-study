# docker study
- '완벽한 IT 인프라 구축을 위한 Docker' 책을 읽고 기억해야할 내용 정리
- 책 외적으로 docker에 대해 알게 되는 내용 추가

## 주요 명령어
- `docker images` 현재 로컬에 다운로드된 image 목록을 출력
- `docker image rm {image id}` 이미지 삭제
- `docker ps -q` 도커 실행중인 컨테이너 프로세스 정보 -q 현재 구동중인 컨테이너 해시를 출력함. 한 개 이상이면 줄바꿈으로 여러줄 표시. full-length 해시는 아님
  - -f 옵션으로 필드별 검색을 할 수 있는데 내가 생각한대로 안됨. 직관적이지 않은것 같다
- `docker run -d --name nginx1 -p 8080:80 nginx` 도커 이미지 실행 -d 백그라운드 실행, --name 컨테이너 이름, -p 컨테이너와 호스트 포트 매칭 호스트포트:컨테이너포트
- `docker run -it centos /bin/bash` -it 인터렉티브모드 centos 이미지를 실행해서 포어그라운드 bash 쉘로 입력 프롬프트를 넘겨줌
- `docker start|stop|restart|pause|unpause {컨테이너id}` 이름 그대로 역할
- `docker attach {컨테이너 이름,id}` 백그라운드로 실행중인 컨테이너에 접속할려고 할 때. nginx를 컨테이너로 띄운 경우에는 어플리케이션이기 때문에 attach로 접속하면 nginx를 구동하고 있는 os로의 접속이 아니라 어플리케이션으로 접속하게 된다
- `docker exec -it nginx1 /bin/bash` nginx를 구동하고 있는 os로 접속할때 exec로 명령을 전달한다
- `docker top` 컨테이너에서 실행중인 프로세스를 출력한다
- `dockr rm {컨테이너 이름,id}` 컨테이너 삭제
- `docker port` 컨테이너와 호스트의 포트매칭 정보 출력
- `docker rename oldName newName` 컨테이너 기존 이름을 변경한다
- `docker cp {컨테이너 이름,id}:/var/log/nginx/acccess.log ./Document/host` 컨테이너 파일을 호스트 경로로 복사해올때 사용
- `docker diff {컨테이너 이름,id}` 컨테이너의 파일 변경 이력을 확인할 수 있다
  - ```
    $ docker run -it centos /bin/bash
    # useradd newuser
    # exit
    $ docker diff centos1
    ...
    A /var/spool/mail/newuser
    C /home
    A /home/newuser
    A /home/newuser/.bash_logout
    A /home/newuser/.bash_profile
    A /home/newuser/.bashrc
    ...
    ```
    A: 추가, C: 변경, D: 삭제 
- `docker commit -a 'message' nginx1 intoday87/webfront:1.0` 도커는 이미지로 컨테이너를 보통 생성하지만 컨테이너로 이미지를 생성할 수 있다
  - ```
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    intoday87/webfront  1.0                 770ad0b09a1c        5 minutes ago       109MB
    ```
- `docker export nginx1 > latest.tar` 컨테이너를 tar로 export 할 수도 있다
  - `tar -tvf latest.tar` 로 tar에 어떻게 구성되어 있는지 확인해본다
- `$ cat latest.tar | docker import - webap:1.0` tar를 이미지로 import 가능
- `docker save -o export.tar centos` 이미지를 tar로 변환하는 또 다른 커맨드
- `docker load -i export.tar` tar를 이미지로 로드
- export/save의 차이
  - export로 tar를 만든 후 파일 구조를 보면 컨테이너 동작에 필요한 파일이 모두 압축된다
  - save로 만든 tar는 풀어보면 실제 구동하는데 필요한 구조가 아니라 해시로된 레이어 파일등 centos기준으로 os에 루트부터의 경로가 아닌 다른 파일들로 구성되어 있다
  
## Dockerfile
- 명령어는 대문자와 소문자 모두 상관없으나 통상적으로는 대문자로 통일한다
- ```
  # 코멘트
  명령어 값
  ```
- FROM
  ```
  FROM [이미지명]
  FROM [이미지명]:[태그명]
  FROM [이미지명]@[Digest]
  ```
- Dockerfile로 이미지 생성하기
  - `docker build -t [생성할 이미지 명]:[태그명 ex_) latest) [Dockerfile 디렉토리 경로]`
  - 파일명이 Dockerfile(확장자 없음)아닐때에는 `-f` 옵션을 사용한다. *__파일명이 다를 경우에는 Docker Hub에서 이미지가 자동생성되지 않으므로 주의!__*
  - 파일명이 달라서 표준 입력을 통해 build 가능 `docker build - < MyDockerfile` *__하이픈을 넣는것에 주의하자__*
    - 이러한 경우 `ADD` 명령어로 이미지 내에 파일을 추가할 수 없는등 build에 필수적인 파일 포함 할 수 없다
  - docker는 image를 build할 때 자동으로 캐시를 생성하고 다른 이미지를 build할 때 캐시를 사용해서 build 속도를 높인다
    아래는 Dockerfile을 통해 image를 이름만 바꿔서 3번째 생성했을때 `Using cache`가 표시되면서 캐시를 사용한다. 물론 2번째도 이랬을것이다
    ```
    $  docker build -t sample:3.0 ./
    Sending build context to Docker daemon  2.048kB
    Step 1/2 : FROM centos:latest@sha256:8d487d68857f5bc9595793279b33d082b03713341ddec91054382641d14db861
     ---> 9f38484d220f
    Step 2/2 : MAINTAINER DONGHO LEE intoday1987@gmail.com
     ---> Using cache
     ---> 7fcc1bf1d4e7
    Successfully built 7fcc1bf1d4e7
    Successfully tagged sample:3.0
    ```
- Dockerfile로 apache 띄우기
  ```Dockerfile
  # 베이스 이미지 설정
  FROM centos:latest@sha256:8d487d68857f5bc9595793279b33d082b03713341ddec91054382641d14db861

  # STEP:1 Apache 설치 Docker 컨테이너에서 /bin/sh -c 로 커맨드를 실행하는 방식과 동일
  # shell을 변경하고 싶으면 아래와 같이 할 수 있다. 그런데 신기한건 이렇게 명령어가 바뀌면 cache를 사용하지 않고 새로 설치 하더라
  # RUN ["/bin/bash", "-c", "yum -y install httpd"]
  RUN yum install -y httpd

  # STEP:2 파일 복사 index.html은 Dockerfile 경로
  COPY index.html /var/www/html/

  # STEP:3 Apache 구동
  CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]

  MAINTAINER NICK LEE some@email.com
  ```
  `docker build -t sample .`로 실행하면 주석 순서대로 스텝별로 진행상황 로그를 볼 수 있고 이미지가 latest 태그로 생성된다
  ```/bin/bash
  Step 1/5 : FROM centos:latest@sha256:8d487d68857f5bc9595793279b33d082b03713341ddec91054382641d14db861
   ---> 9f38484d220f
  Step 2/5 : RUN yum install -y httpd
   ---> Using cache
   ---> a8fd53234775
  Step 3/5 : COPY index.html /var/www/html/
   ---> c408bdf7d909
  Step 4/5 : CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
   ---> Running in 03db1eb864fd
  Removing intermediate container 03db1eb864fd
   ---> eab105b2993f
  Step 5/5 : MAINTAINER NICK LEE some@email.com
   ---> Running in 80a55dc05ec0
  Removing intermediate container 80a55dc05ec0
   ---> cfa36a17bc35
  Successfully built cfa36a17bc35
  Successfully tagged sample:latest
  ```
  결과를 보면 각 스텝별로 hash를 출력하는데 base image로 부터 각각의 이미지가 생성되어 중첩되어간다. 이래서 diff만 따로 땡겨와서 이미지를 실행할 수 있는듯.
  Using cache는 이 이 이미지를 생성한것이 첫 번째가 실패해서 두 번째 실행한결과여서 그런데 첫 번째와는 달리 apache를 설치한것이 이미지로 캐시가 되어서 두 번째는 cache를 사용하게 되었다
- Exec 형식으로 RUN 실행
  ```Dockerfile
  # 베이스 이미지 설정
  FROM centos:latest

  # 작성자 정보
  MAINTAINER DONGHO LEE intoday1987@gmail.com

  # Run 명령 실행
  RUN echo 안녕하세요 Shell형식입니다
  RUN ["echo", "안녕하세요 Exec형식입니다"]
  RUN ["/bin/bash", "-c", "echo '안녕하세요 Exec형식으로 bash를 사용해봅시다'"]
  ```
