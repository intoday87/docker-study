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
  - nginx1 대신에 `docker container ls -q`로 컨테이너 아이디(해시)를 얻어서 호출 가능
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
- `docker inspect [옵션] <컨테이너 또는 이미지의 이름, ID>`
  - 이미지 세부 정보 확인
  - 옵션을 통해 필요한 정보만 색출할 수 있다.
  - json 형태로 출력되는 값을 json 프로퍼티 참고 하듯이 `{{ .a.b }}` 이런식으로 접근하면 원하는 값만 출력한다
    - ```/bin/bash
      $ docker inspect --format="{{ .Os }}" centos
      linux
      ```
    
  
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
    - `maintainer` 속성은 deprecated
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
  MAINTAINER NICK LEE some@email.com

  # Run 명령 실행
  RUN echo 안녕하세요 Shell형식입니다
  RUN ["echo", "안녕하세요 Exec형식입니다"]
  RUN ["/bin/bash", "-c", "echo '안녕하세요 Exec형식으로 bash를 사용해봅시다'"]
  ```
  이미지 생성시 출력 결과
  ```/bin/bash
  $ docker build -t sample .
  Sending build context to Docker daemon  3.072kB
  Step 1/5 : FROM centos:latest
   ---> 9f38484d220f
  Step 2/5 : MAINTAINER DONGHO LEE intoday1987@gmail.com
   ---> Using cache
   ---> 7fcc1bf1d4e7
  Step 3/5 : RUN echo 안녕하세요 Shell형식입니다
   ---> Running in 10bfa44cd571
  안녕하세요 Shell형식입니다
  Removing intermediate container 10bfa44cd571
   ---> 622fe4988c28
  Step 4/5 : RUN ["echo", "안녕하세요 Exec형식입니다"]
   ---> Running in 5942455a0124
  안녕하세요 Exec형식입니다
  Removing intermediate container 5942455a0124
   ---> 91f228a97ebf
  Step 5/5 : RUN ["/bin/bash", "-c", "echo '안녕하세요 Exec형식으로 bash를 사용해봅시다'"]
   ---> Running in 15736f4955b4
  안녕하세요 Exec형식으로 bash를 사용해봅시다
  Removing intermediate container 15736f4955b4
   ---> dfa91bbe00b0
  Successfully built dfa91bbe00b0
  Successfully tagged sample:latest
  ```
- `docker history [OPTIONS] IMAGE`로 어떻게 이미지가 생성이 되었는지 확인할 수 있다
  ```
  IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
  dfa91bbe00b0        3 days ago          /bin/bash -c echo '안녕하세요 Exec형식으로 b…            0B
  91f228a97ebf        3 days ago          echo 안녕하세요 Exec형식입니다                            0B
  622fe4988c28        3 days ago          /bin/sh -c echo 안녕하세요 Shell형식입니다                0B
  7fcc1bf1d4e7        4 days ago          /bin/sh -c #(nop)  MAINTAINER NICK WILDE int…   0B
  9f38484d220f        2 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
  <missing>           2 weeks ago         /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B
  <missing>           2 weeks ago         /bin/sh -c #(nop) ADD file:074f2c974463ab38c…   202MB
  ```
  **exec 형식으로 실행된 명령어는 Shell을 거치지 않고 실행되었음을 알 수 있다**
  _Dockerfile을 build하면 명령어가 한 행씩 실행되며 이미지도 하나씩 생성된다. 하지만 스토리지 드라이브에 따라 그 수에는 상한선이 있다_
  _이 책에서 다룬 Docker Toolbox의 Boot2Docker에서는 AUFS로 변경 이미지를 관리하고 있지만 AUFS는 변경분의 127 레이어 까지만 관리할 수 있다._
  `INFO[0107] Cannot create container with more than 127 parents`
  -> 이를 피하기 위해 명령어 수를 줄여야 한다 -> 한 행에 RUN 명령어를 적는게 좋다. 아래와 같이 실행하면 4개의 레이어가 생성된다
  ```
  RUN yum -y install httpd
  RUN yum -y install php
  RUN yum -y install php-mbstring
  RUN yum -y install php-pear
  ```
  하지만 다음 예에서는 한 개의 레이어만 사용된다.( 레이어는 명령어 별로 생성되는 개별 이미지를 의미하는것 같다 )
  `RUN yum -y install httpd php php-mbstring php-pear`
  또한 RUN 명령은 `\`으로 개행할 수 있다. 가독성을 높여 커맨드 내용을 알아보기 쉽게 하려면 개행을 사용하면 좋다
  ```
  RUN yum -y install\
              httpd\
              php\
              php-mbstring\
              php-pear
  ```
- 데몬 실행(CMD)
  - 이미지를 실행하기 위해 실행되는 커맨드는 `RUN` 명령어에 입력하지만 컨테이너에서 실행되는 커맨드는 `CMD` 명령어를 사용한다
  - Dockerfile에는 한 개의 `CMD`명령어만 입력할 수 있으며, 만약 여러 개를 입력한 경우 마지막 커맨드만 적용된다
  ```Dockerfile
  # shell 형식으로 CMD 실행
  CMD /usr/sbin/httpd -D FOREGROUND
  ```
  또는
  ```Dockerfile
  # exec 형식으로 CMD 실행
  CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
  ```
- 데몬 실행(ENTRYPOINT)
  - 기본 실행 프로세스를 지정한다. 'go'로 지정하게되면 `docker container run example version` => `# go version`과 동일
  - CMD와 사용방법은 동일하다
  ```Dockerfile
  ENTRYPOINT ["/usr/sbin/httpd", "-D", "FOREGROUND"]
  ```
  ```
  ENTRYPOINT /usr/sbin/httpd -D FOREGROUND
  ```
  - 그럼 차이점은?
    `ENTRYPOINT`와 `CMD`와의 차이는 `docker run`커맨드 실행 시점에서의 동작
    `CMD` 명령은 컨테이너를 구동할 때 정의한 커맨드보다 `docker run` 커맨드의 값으로 입력된 새로운 커맨드가 우선 실행
    ```Dockerfile
    # 베이스 이미지 설정
    FROM centos:latest

    # 작성자 정보
    MAINTAINER DONGHO LEE intoday1987@gmail.com

    # Run 명령 실행
    RUN ["yum","-y","install", "httpd"]

    # nginx 설치
    RUN ["rpm", "-Uvh", "http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm"]

    RUN ["yum", "install", "-y", "nginx"]

    # Apache httpd 실행
    ENTRYPOINT ["/usr/sbin/httpd", "-D", "FOREGROUND"]
    ```
    Dockerfile에 내용을 이렇게 채우고
    `docker build -t sample .` 실행해서 이미지 만든 후
    `docker run -d -p 8080:80 sample /usr/sbin/nginx -g 'deamon off;'` 실행하면
    `CMD`인 경우 nginx가 실행되서 접속됨 `docker run` 명령어에 넘어온 쉘 실행이 덮어 씌워진다
      - 이처럼 CMD 명령은 `docker run`에서 데몬으로 실행하는 커맨드를 우선시한다
    `ENTRYPOINT`인 경우 nginx가 실행되지 않음
      - 컨테이너 구동 시 특정 데몬을 강제적으로 실행시키고자 할 때 사용한다
      - `ENTRYPOINT`가 여러개 입력되어 있는 겅우 가장 마지막 명령어가 실행
- ENTRYPOINT와 CMD의 조합
  ```Dockerfile
  # Docker base image
  FROM centos:latest
  
  # top 커맨드 실행
  ENTRYPOINT ["top']
  # -d 옵션으로 10초 마다 업데이트 -> run 커맨드에서 바꿀수 있다
  CMD ["-d", "10"]
  ```
  실행해보기
  ```/bin/bash
  $ docker run -it sample
  # 2초 간격으로 업데이트하는 경우
  $ docer run -it sample -d 2
  ```
- `ONBUILD [실행하고자 하는 커맨드]`
  이미지 내의 다음 build에서 실행되는 커맨드를 설정하는 명령어
  한 Dockerfile에서 ONBUILD 명령으로 커맨드를 실행하도록 설정하고, build하여 이미지를 생성하고, 생성된 이미지를 다른 Dockerfile에서 베이스이미지로 설정하여 build하면, ONBUILD 커맨드가 실행된다
  ```Dockerfile
  # sample
  FROM centos:latest
  
  CMD ["/bin/bash", "-c", "echo 'hello i`m sample'"]
  
  ONBUILD RUN ["bin/bash", "-c", "echo 'hello it`s ONBUILD'"]
  ```
  ```Dockerfile
  FROM sample:latest
  
  CMD ["/bin/bash", "-c", "echo 'hello i`m another'"]
  ```
  주로 이미지내 심어진 프로그램을 deploy하는 명령(ADD, COPY)을 입력한다
  `ONBUILD ADD index.html /var/www/html/`
  어찌보면 당연한 얘기지만 ONBUILD 실행 기준은 이 이미지를 base로 하는 이미지 생성시 기준이다. 즉 이 이미지를 base로 생성하려는 Dockerfile 위치 기준으로 동작한다
- `ENV [key] [value]` or `ENV [key]=[value]`
  - key 다음에 입력된 문자열을 하나의 값으로 인식한다. 공백이나 " " 등과 같은 character 값도 모두 문자로 인식
  - ```Dockerfile
    ENV myName "nick wilde"
    ENV myOder Cola Cider Soju
    ENV myNickName nick
    ```
  - `ENV` 명령어에 여러 환경변수 집어넣기
    ```Docker
    ENV NICK=123, \
        WILDE="456", \
        JOO=7\ 8\ 9
    ```
- `WORKDIR [작업 디렉터리 경로]`
  - `docker build`를 하게 되면 `pwd` 경로는 `/first/second/third`가 출력된다
    ```Dockerfile
    FROM centos
    WORKDIR /first
    WORKDIR second
    WORKDIR third
    RUN ["pwd"]
    ```
  - `ENV`를 이용해 설정할수도 있다
    ```Dockerfile
    FROM centos

    ENV DIRPATH /first
    ENV DIRNAME /second

    WORKDIR $DIRPATH/$DIRNAME

    RUN ["pwd"]
    ```
- `USER [adduser로 설정한 유저명]`
  - `docker build` 실행시 유저 설정 전 후 비교
    ```Dockerfile
    FROM centos

    RUN ["adduser", "nick"]
    RUN ["whoami"]
    USER nick
    RUN ["whoami"]
    ```
- `LABEL`설정하고 `docker inspect`로 보기
  - `docker build -t sample-label .`
    ```Dockerfile
    FROM centos

    LABEL title="webAPServerImage"
    LABEL version="1.0"
    LABEL description="This image is WebApplicationServer \
    for java EE."
    ```
  - `docker inspect --format="{{ .Config.Labels }}" sample-label`
    ```/bin/bash
    map[org.label-schema.schema-version:1.0 org.label-schema.vendor:CentOS title:webAPServerImage version:1.0 description:This image is WebApplicationServer for java EE. org.label-schema.build-date:20190305 org.label-schema.license:GPLv2 org.label-schema.name:CentOS Base Image]
    ```
- `EXPOSE` 설정으로 host에 port 노출하기
  ```Dockerfile
     FROM centos

     RUN yum -y install httpd

     COPY index.html /var/www/html

     CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]

     # EXPOSE로 컨테이너가 사용하는 포트를 host에 노출한다는 것이지 host의 80 포트로 매칭해서 열어준다는 의미는 아니다
     # 결국 docker run -p 8080:80 으로 해줘야 호스트에서 컨테이너로 포트가 연결된다
     EXPOSE 80
  ```
- `ADD`로 호스트 파일과 디렉터리 추가
  - 책에는 디렉터리도 추가할 수 있다고 하는데 생각대로 안된다. 구글링 해봐야함
    ```Dockerfile
      FROM centos

      RUN yum -y install httpd

      WORKDIR /var/www

      # add ["www.example.com/index.html", "./html"] 이와 같은 원격 요청 파일도 추가 가능하다
      ADD ["html/", "./html"]

      CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]

      EXPOSE 80
    ```
  - `.dockerignore`에 복사대상 제외하기
    ```.dockerignore
        html/index.html
    ```
    이렇게 하고 빌드 후 컨테이너에 접속해서 `/var/www/html`을 보면 `index.html`이 없다
- `COPY [호스트의 파일경로] [Docker 이미지 파일 경로]`
  - `ADD`와 비슷한데 `ADD`는 압축풀기 원격 파일 다운로드가 가능하다. 이에 반해 `COPY`명령은 호스트 파일을 이미지 파일 시스템에 복사하는 작업만 수행
- `VOLUME [docker image 파일 경로]`
  - 로그 컨테이너를 만들어 `VOLUME`으로 마운트 포인트를 생성하면 호스트와 다른 컨테이너에서 볼륨을 외부 마운트 할 수 있다
  - 웹 서버 컨터에너에서 로그 컨테이너로 로그를 쌓는 예제
    log-image
    ```Dockerfile
       FROM centos
       
       MAINTAINER 0.1 sample@email.com
       
       RUN ["mkdir", "/var/log/httpd"]
       
       VOLUME /var/log/httpd
    ```
    ```/bin/bash
    $ docker build -t log-image .
    ...
    $ docker run -it --name log-container log-image
    ...
    # tail -f /var/log/httpd/access.log
    ```
    web-image 
    ```Dockerfile
      FROM centos

      MAINTAINER 0.1 sample@email.com

      RUN ["yum", "install", "-y", "httpd"]

      CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]

      VOLUME /var/log/httpd
    ```
    ```/bin/bash
    docker build -t web-image .
    ...
    docker run -d --name web-container -p 8080:80 --volume-from log-container web-image
    ...
    curl localhost:8080
    ```
    log-container foreground에서 로그가 tailing되는걸 볼 수 있다
- github과 연동한 automated build
  - https://docs.docker.com/docker-hub/builds/
- private docker registry 구축
  - private한 네트워크 환경 정보 이런 것들이 이미지에 포함될 수 있어서 공용보다는 private registry에 올려서 사용
  - registry 자체도 docker image로 받아서 바로 띄울 수 있다. 이래서 docker 쓰는 듯
  - registry 찾기
    ```/bin/bash
    $ docker search registry:latest
    NAME                                    DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
    registry                                The Docker Registry 2.0 implementation for s…   2517                [OK]
    ```
  - 책에서는 `registry:2.0`로 설치 하지 않으면 1.x가 깔린다고 되어 있지만 현재는 위 출력 결과를 보면 latest가 2.0인걸로 보인다
  - `docker run -d -p 5000:5000 registry`
  - 이미지 업로드
    ```Dockerfile
    FROM centos

    MAINTAINER 0.1 sample@email.com

    RUN ["yum", "install", "-y", "httpd"]

    CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
    ```
  - `docker build -t webserver .`
  - private 네트워크의 Docker 레지스트리에 업로드하기 위해서는 다음 룰에 따라 이미지에 태그를 붙여야 한다
    `docker tag [로컬 이미지명] [Docker repository의 ID addresss 또는 호스트명:포트 번호]/[이미지명]`
    `docker tag webserver localhost:5000/httpd` webserver와 localhost:5000/httpd가 이미지 아이디가 같다
  - `docker push localhost:5000/httpd` 로컬 private repository로 업로드
  - 이미지 삭제
    `docker rmi webserver`
    `docker rmi localhost:5000/httpd`
  - 이미지 다운로드
    `docker pull localhost:5000/httpd`
  - `docker images`를 통해 다운로드된 것을 확인해본다
## Docker Compose
  - 여러 컨테이너를 일괄로 관리할 수 있다
  - `docker run --link [접속하고자 하는 컨테이너명:alias명] [이미지명] [실행 커맨드]` 컨테이너에서 다른 컨테이너로 연결 가능하도록 설정할 수 있다
    ```/bin/bash
    $ docker run -d --name dbserver postgres
    $ docker run -it --name appserver --link dbserver:pg centos /bin/bash
    # printenv | grep PG
    PG_ENV_PG_MAJOR=11
    PG_PORT=tcp://172.17.0.3:5432
    PG_PORT_5432_TCP_ADDR=172.17.0.3
    PG_PORT_5432_TCP_PROTO=tcp
    PG_NAME=/appserver/pg
    PG_ENV_PG_VERSION=11.2-1.pgdg90+1
    PG_ENV_GOSU_VERSION=1.11
    PG_ENV_PGDATA=/var/lib/postgresql/data
    PG_ENV_LANG=en_US.utf8
    PG_PORT_5432_TCP_PORT=5432
    PG_PORT_5432_TCP=tcp://172.17.0.3:5432
    # ping $PG_PORT_5432_TCP_ADDR
    PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
    64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.061 ms
    64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.047 ms
    ^C
    --- 172.17.0.3 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1069ms
    rtt min/avg/max/mdev = 0.047/0.054/0.061/0.007 ms
    # cat /etc/hosts
    127.0.0.1	localhost
    ::1	localhost ip6-localhost ip6-loopback
    fe00::0	ip6-localnet
    ff00::0	ip6-mcastprefix
    ff02::1	ip6-allnodes
    ff02::2	ip6-allrouters
    172.17.0.3	pg e8b53e44b2fb dbserver
    172.17.0.4	43e37f40b966
    # ping pg
    PING pg (172.17.0.3) 56(84) bytes of data.
    64 bytes from pg (172.17.0.3): icmp_seq=1 ttl=64 time=0.108 ms
    64 bytes from pg (172.17.0.3): icmp_seq=2 ttl=64 time=0.068 ms
    64 bytes from pg (172.17.0.3): icmp_seq=3 ttl=64 time=0.046 ms
    64 bytes from pg (172.17.0.3): icmp_seq=4 ttl=64 time=0.049 ms
    64 bytes from pg (172.17.0.3): icmp_seq=5 ttl=64 time=0.049 ms
    64 bytes from pg (172.17.0.3): icmp_seq=6 ttl=64 time=0.046 ms
    ^C
    --- pg ping statistics ---
    6 packets transmitted, 6 received, 0% packet loss, time 5167ms
    rtt min/avg/max/mdev = 0.046/0.061/0.108/0.022 ms
    ```
    -*단, 링크 기능은 동일한 호스트 머신에서 구동 중인 컨테이너 사이에서만 액세스할 수 있으므로 주의해야 한다*-
  - docker-compose.yml
    - 베이스 이미지 지정
      ```docker-compose.yml
      webserver:
        image: ubuntu
      ```
    - Docker Hub의 이미지를 지정하는 경우
      ```docker-compose.yml
      webserver:
        image: nick/dockersample:1.0
      ```
    - Dockerfile에 이미지 구성을 저장하고 이를 자동으로 build하여 베이스 이미지로 지정할 수도 있다
      ```/bin/bash
      .
      ├── Dockerfile
      └── docker-compose.yml
      ```
      - build 지정
        ```docker-compose.yml
        webserver:
        build: .
        ```
    - 컨테이너 내부에만 공개하는 포트 지정
      ```docker-compose.yml
      webserver:
        from:centos
        expose:
         - "3000"
      ```
    - `docker-compose up` 커맨드를 실행
    - 환경변수
      ```docker-compose.yml
      dbserver:
        image: mysql
        environment:
         - MYSQL_ROOT_PASSWORD=1234
      ```
      별도 파일 로드. 여러파일은 리스트로 가능
      현재 경로에 `envfile`, 경로는 상대경로든 절대경로든 상관없다
      ```envfile
      MYSQL_ROOT_PASSWORD=1234
      ```
      ```docker-compose.yml
      dbserver:
       image: mysql
       env_file: envfile
      ```
      ```docker-compose.yml
      dbserver:
        image: mysql
        env_file:
         - ./envfile
         - ./envfile2
      ```
    - 컨테이너 이름
      개별 컨테이너에 이름을 붙여주려면 docker-compose.yml의 개별 네임스페이스 하위로 아래와 같이 표기 한다
      `container_name: web-container`
    - 컨테이너 라벨 지정
      `docker inspect [컨테이너 이름]`를 이용해 라벨을 확인할 수 있다
      ```docker-compose.yml
      # 배열 형식으로 지정
      webwerver:
        label:
         - "com.example.description=Accounting webapp"
      
      # hash 형식으로 지정
      webserver:
       com.example.description: "Accounting webapp"
      ```
    - 볼륨 마운트(volumes, volumes_from)
      `volume_from`으로 마운트할려면 컨테이너 이름 또는 서비스(docker-compose 최상위 루트 네임스페이스명)을 입력하면된다
      컨테이너명으로 하는 경우 container를 삭제하고 `docker-compose up`을 실행하면 컨테이너가 없다고 실행을 못한다
      서비스 이름으로 하는걸 권장
      ```docker-compose.yml
      dbserver:
        ...
        container_name: db-container
        volumes:
         - /var/lib/mysql
      webserver:
        ...
        volumes_from:
          - dbserver
      ```
    - `docker-compose`의 커맨드
      - `up`: 컨테이너 생성 및 구동
        - `-f`: 별도 파일 위치 지정 가능
        - `-d`: 백그라운드 실행 
      - `ps`: 컨테이너 목록 확인. docker-compose.yml 있는 경로에 해당
      - `scale`: 생성할 컨테이너 개수 지정
        - `The scale command is deprecated. Use the up command with the --scale flag instead.`
        - `docker-compose up --scale 'db-container=2`
        - scale을 시도해봤지만 컨테이너당 하나의 호스트 포트로 매칭된 컨테이너를 여러개 띄울수도 없을것 같고 실제로 scale을 적용해보면 한 컨테이너가 떳다가 죽고(done이라고 표시) 다시 뜬다. scale이 필요하면 적용법을 더 봐야 할 듯
      - `start`
      - `stop`
      - `restart`
      - `kill [옵션] [서비스이름]`: `docker-compose kill dbsever` `dbserver`는 `docker-compose.yml`에 각 컨테이너별 최상위 루트 네임스페이스 이름 -> 서비스 이름
      - `rm`
      - 'run'
        - `docker-compose up`을 해서 `docker-compose.yml`을 띄운 후, `docker-compose run webserver /bin/bash`로 실행하면 `docker-compose ps`시 컨테이너가 webserver로 두 개가 뜨게 된다. 처음에는 떠 있는 webserver 서비스를 접속할 수 있게 커맨드를 실행하는줄 알았다


## `CMD`, `ENTRYPOINT`의 적절한 사용 - kubernetes in action 읽으면서 헷갈려서 다시 정리
- `ENTRYPOINT`는 컨테이너가 시작될 때 호출될 명렁어를 정의
- `CMD`는 `ENTRYPOINT`에 전달되는 인자를 정의
`CMD` 명령어를 사용해 이미지가 실행될 때 실행할 명령어를 지정할 수 있지만, 올바른 방법은 `ENTRYPOINT` 명령어로 실행하고 기본 인자를 정의하려는 경우에만 CMD를 지정하는 것이다. 그러면 아무런 인자도 지정하지 않고 이미지를 실행할 수 있다.
  `docker run <image> <arguments>`로 추가 인자를 지정해 Dockerfile 안의 `CMD`에 정의된 값을 재정의 할 수 있다.

## shell과 exec 형식 간의 차이점
- shell 형식 - `ENTRYPOINT node app.js`
- exec 형식 - `ENTRYPOINT ["node", "app.js"]`

차이점은 내부에서 정의된 명령을 shell로 호출하는지 여부

프로세스 목록을 나열해 차이를 확인할 수 있다.
- exec 형식으로 실행한 경우
```zsh
$ docker exec exec-container ps x
PID TTY STAT TIME COMMAND
  1 ?   Ssl  0:00 node app.js
 12 ?   Rs   0:00 ps x
```
- shell 형식으로 실행한 경우
```zsh
$ docker exec shell-container ps x
PID TTY STAT TIME COMMAND
  1 ?   Ss   0:00 /bin/sh -c node app.js
  7 ?   Sl   0:00 node app.js
 12 ?   Rs   0:00 ps x
```
보는 것처럼 메인 프로세스(PID 1)은 node 프로세스가 아닌 shell 프로세스다. 노드 프로세스(PID 7)는 shell에서 시작된다. shell 프로세스는 불필요하므로 `ENTRYPOINT` 명령에서 exec 형식을 사용해 실행

## multistage build
- [참고](https://docs.docker.com/develop/develop-images/multistage-build/)
- `FROM`을 여러개 써서 각 스테이지에서 만든 artifact를 다음 스테이지에서 사용하도록 함. 그렇게 되지 않으면 여러개 Dockerfile을 만들고 shell로 각각 Dockerfile로 부터 얻은 artifact를 다음 Dockerfile에서 참고 하도록 짜야함. 각각 `FROM` 스테이지로부터 얻은 artifact를 제외하고 마지막 저장되는 이미지에 불필요한 각 스테이지를 저장하지 않음. 필요한 artifact를 얻기 위해 하나의 Dockerfile에서 여러개의 `FROM`을 통해 각 스테이지가 artifact외에 불필요한 단계를 저장하는것을 걱정할 필요없이 필요한 artifact만 하나의 Dockerfile에서 얻을수 있도록 하기 위함
- 각 `FROM` 스테이지에 이름을 붙여서 alias로 사용할 수 있다. `AS name`으로 이름을 붙여주지 않으면 처음 시작하는 `FROM`스테이지는 0부터 시작한다
  ```Dockerfile
    FROM image AS builder
    ...
    FROM image-n
    COPY --from builder ./src ./app/src
  ```
