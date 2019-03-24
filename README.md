# docker study
- '완벽한 IT 인프라 구축을 위한 Docker' 책을 읽고 기억해야할 내용 정리
- 책 외적으로 docker에 대해 알게 되는 내용 추가

## 주요 명령어
- `docker images` 현재 로컬에 다운로드된 image 목록을 출력
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
