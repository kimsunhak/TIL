## DockerFile Image 생성, 배포

### 1. DockerFile

> #### DockerFile이란 ? 
>
> Docker Image를 만들기 위한 설정파일 &
> 이미지 생성을 위해 필요한 각종 파일, 소스코드, 메타데이터를 담고 있는 디렉토리를 뜻하며 Dockerfile이 위치한 디렉토리가 빌드 컨텍스트가 됨

---------------------------

### 2. 명령

#### [FROM](https://docs.docker.com/engine/reference/builder/#from)

- ##### 베이스 이미지를 지정

- ##### 반드시 한번이상 입력해야 한다

#### [WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir)

- ##### 작업 디렉토리를 지정

- ##### 해당 디렉토리가 없으면 새로 생성

- ##### 작업 디렉토리를 지정하면 그 이후 명령어는 해당 디렉토리를 기준으로 동작한다.

  ##### `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, `ADD`

#### [COPY](https://docs.docker.com/engine/reference/builder/#copy)

- ##### 파일이나 폴더를 이미지에 복사

- ##### 로컬에서 도커 컨테이너로 복사

- ##### Copy 는 절대경로 또는 `WORKDIR` 로 부터 상대 경로

- ##### `.dockerignore`를 사용해 카피 대상에서 제외 가능

#### [ARG](https://docs.docker.com/engine/reference/builder/#arg)

- ##### `docker build` 로 이미지를 빌드할 때 설정 할 수 있는 옵션을 지정

- ##### DockerFile 안에서만 사용 가능

- ##### `CMD` 나 애플리케이션 코드에서 사용 불가

#### [VOLUME](https://docs.docker.com/engine/reference/builder/#volume)

- ##### Anonymous Volume을 지정할 수 있음

- ##### 컨테이너 생성 시 호스트와 공유할 컨테이너 내부의 디렉토리를 설정

#### [RUN](https://docs.docker.com/engine/reference/builder/#run)

- ##### 도커 이미지가 생성되기 전에 수행할 Shell 명령어

- ##### 추가적으로 필요한 파일을 다운 받기 위한 명령어 명시

- ##### 2가지 형태 가능

  ```dockerfile
  RUN <command> (shell form, the command is run in a shell, which by default is /bin/sh -c on Linux or cmd /S /C on Windows)
  RUN ["executable", "param1", "param2"] (exec form)
  ```

#### [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose)

- ##### 외부 포트를 지정.

- ##### 실제로 포트를 개방하는건 x

  ##### `docker run -p 3000:3000 node` 와 같이 -p 옵션으로 포트를 열어 줘야함

#### **ENTRYPOINT**

- ##### Dockerfile 파일 내에서 1번만 사용 가능

- ##### 컨테이너가 시작되었을 때 실행 할 파일 또는 Shell Script

- ##### Overwrite 불가능

  ##### 예 ) `docker run node init` init이 **ENTRYPOINT** 뒤에 붙어서 실행된다.

#### [CMD](https://docs.docker.com/engine/reference/builder/#cmd)

- ##### Dockerfile 파일 내에서 1번만 사용 가능

- ##### 컨테이너가 시작되었을 때 실행 할 파일 또는 Shell Script

- ##### Overwrite 가능

  ##### 예) `docker run node npm init` npm init으로 CMD를 Overwrite할 수 있다.

- ##### 3가지 형태 가능

  ```dockerfile
  CMD ["executable","param1","param2"] (exec form, this is the preferred form)
  CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
  CMD command param1 param2 (shell form)
  ```

------------------------------------------------------------

### 3. Multi Stage Docker Build

- ##### 하나의 DockerFile에 여러 개의 FROM을 사용하여 빌드 완료 시 최종적으로 생성될 이미지의 크기를 줄일 수 있음

- ##### 멀티 스테이지 빌드는 반드시 필요한 실행 파일만 최종 이미지 결과물에 포함시켜 이미지 크기를 줄일 수 있음

##### **Multi Stage Docker Build 예시**

```dockerfile
FROM openjdk:11 as build
WORKDIR /workspace/app
COPY gradlew .
COPY gradle gradle
COPY build.gradle .
COPY settings.gradle .
COPY src src
RUN chmod +x gradlew
RUN ./gradlew build -x test
RUN mkdir -p build/dependency && (cd build/dependency; jar -xf ../libs/*-SNAPSHOT.jar)

FROM adoptopenjdk/openjdk11:alpine-jre
ARG DEPENDENCY=/workspace/app/build/dependency
COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app
EXPOSE 8080
ENTRYPOINT ["java","-cp","app:app/lib/*","com.emailservice.EmailServiceApplication"]
```



-------------

### 4. 실제 예시

#### DockerFile 생성

```dockerfile
FROM openjdk:8-jdk-alpine as build

WORKDIR /boot/app

COPY gradlew .
COPY gradle gradle
COPY build.gradle .
COPY settings.gradle
COPY src src

RUN chmod +x ./gradlew
RUN ./gradlew build -DskipTests
RUN mkdir -p build/dependency && (cd build/dependency; jar -xf ../libs/*-SNAPSHOT.jar)

FROM openjdk:8-jdk-alpine

VOLUME /tmp

ARG DEPENDENCY=/boot/app/build/dependency

COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app

ENTRYPOINT ["java","-cp","app:app/lib/*","com.service.TestServiceApplication"]

```

#### 실행
``` shell
docker build -t [생성할 이미지명 / artifact ] [Dockerfile 위치]

docker build -t spring/spring-boot .

docker run -p <내부 포트번호>:<외부 포트번호> 이미지명/artifact
```



