### 도커 이미지

모든 컨테이너는 이미지를 기만으로 생성되기 때문에 이미지에 대한 개념을 일고 있어야 합니다. 일단 대략적으로 큰 그림을 그려보면 다음과 같습니다.

![Untitled](https://user-images.githubusercontent.com/77387861/217247771-5754c77e-4e40-480e-9b83-3f40ca3695f4.png)


OS에서 apt-get 혹은 yum install을 사용하여 레지스트리에서 패키지를 내려받듯이 Docker는 도커 허브를 이용하여 위 사진 처럼 내려받을 수 있습니다. 또한 계정만 가지고 있으면 누구나 Docker Image를 만들어서 저장소에 저장할 수 있습니다. 

또한 Docker hub에서는 공식(Official) 이미지라는 라벨을 이용하여 누구나 올렸을 때 공식 이미지를 찾지 못하는 번거로움을 제거 하였습니다. 

<img width="1185" alt="Untitled (1)" src="https://user-images.githubusercontent.com/77387861/217247812-bdfa439a-efe2-4e6a-962a-ad134cd27b6e.png">


`docker search {image명}` 을 이용하여 검색해보면 OFFICIAL에 [OK] 라고 붙은 이미지를 볼 수 있습니다. 이것이 바로 공식 이미지 입니다.

### 이미지 구조 이해

이미지 구조를 이해하여 이미지를 좀 더 효율적으로 다루는 방법 컨테이너에서 이미지를 다루는 방법을 정리해봅시다. 

다음 명령어를 이용하여 이미지의 좀 더 상세한 정보를 확인해봅시다. 많은 것들이 출력되지만 깊게 살펴볼 항목은 가장 아랫 부분에 있는 Layer 부분입니다.

```bash
docker inspect redis:4.0

# .. something

"RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:67d3a85d42a2b4ae3ca54ff7b6225bb136fbe0fa5732d9c5143470f13470bbde",
                "sha256:9e59804bb770112fe7906b84db0878c543200cf15be30f08075c0b126ea38bac",
                "sha256:1c8bf07fb16b54fc9c2b8134f13a64b3d659251b3382c9bc26fe7b91eb496968",
                "sha256:3885712800d4f9adb817cdb947957988d390efabcc7e2e2c99c4a52fd9074812",
                "sha256:5dee0be3035410de0e07b0888513cf1b1809a4951bc1b3af3a62d0ec00711eec",
                "sha256:b3b0f111688b65231665aaaede2cd072b94dd73688a46b3da7c0ca74d3306239"
            ]
        }

docker inspect redis-costom:first

"RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:67d3a85d42a2b4ae3ca54ff7b6225bb136fbe0fa5732d9c5143470f13470bbde",
                "sha256:9e59804bb770112fe7906b84db0878c543200cf15be30f08075c0b126ea38bac",
                "sha256:1c8bf07fb16b54fc9c2b8134f13a64b3d659251b3382c9bc26fe7b91eb496968",
                "sha256:3885712800d4f9adb817cdb947957988d390efabcc7e2e2c99c4a52fd9074812",
                "sha256:5dee0be3035410de0e07b0888513cf1b1809a4951bc1b3af3a62d0ec00711eec",
                "sha256:b3b0f111688b65231665aaaede2cd072b94dd73688a46b3da7c0ca74d3306239",
								# 새로 추가된 값
								"sha256:69ac0dd0ce667f233c17ffa8a8aa397e16a2e4a0ea52e105a11536ad98954c80"
            ]
        }

docker inspect redis-costom:second

"RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:67d3a85d42a2b4ae3ca54ff7b6225bb136fbe0fa5732d9c5143470f13470bbde",
                "sha256:9e59804bb770112fe7906b84db0878c543200cf15be30f08075c0b126ea38bac",
                "sha256:1c8bf07fb16b54fc9c2b8134f13a64b3d659251b3382c9bc26fe7b91eb496968",
                "sha256:3885712800d4f9adb817cdb947957988d390efabcc7e2e2c99c4a52fd9074812",
                "sha256:5dee0be3035410de0e07b0888513cf1b1809a4951bc1b3af3a62d0ec00711eec",
                "sha256:b3b0f111688b65231665aaaede2cd072b94dd73688a46b3da7c0ca74d3306239",
								# first image에서 생성됨
								"sha256:69ac0dd0ce667f233c17ffa8a8aa397e16a2e4a0ea52e105a11536ad98954c80",
								# 새로 추가된 값
								"sha256:53349e01b1526477d741d12d4da632b89c811864521344d7831917b5524f5991"
            ]
        }
```

요약하면 기반이 된 이미지에 새로운 값이 추가되는것을 알 수 있습니다. 맞습니다. 이미지는 이 계층을 이용하여 부모 자식 관계를 표현하듯이 계층이 표현되고 있습니다. 

그림으로 보면 다음과 같습니다.

![Untitled (2)](https://user-images.githubusercontent.com/77387861/217247913-23a7d3f8-c84b-4982-b230-8158e67164dc.png)


docker images에서 저 3개의 이미지 크기가 각각 188MB라고 출력되도 실제로는 기존 이미지(188MB) + first + second가 실제 크기입니다. first 파일은 redis-costom:first 이미지를 생성 했던 컨테이너의 변경사항을 의미하고 second는 redis-constom:second 이미지를 생성했던 컨테이너의 변경 사항을 의미합니다.

docker rmi {image명} 명령어로 이미지를 삭제할 수 있는데 실제로 이미지를 삭제할 때 어떤 일이 발생하는지 확인해봅시다. 

테스트에 사용될 이미지는 다음과 같습니다.

```bash
redis:latest
redis-costom:first
redis-costom:second
```

`docker rmi redis-costom:first` 명령어로 해당 이미지를 삭제해봅시다. 그리고 `docker inspect redis-costom:second` 를 입력하면 아직 그대로 Layer가 유지되고 있는 것을 볼 수 있는데 이는 실제로 이미지가 삭제되는 것이 아닌 레이어에 부여된 이름만 삭제하기 때문입니다. 또한 `Untagged: redis-costom:first` 같이 처음 보는 결과를 볼 수 있습니다. 이유는 아직 하위 이미지인 redis-costom:second가 존재하기 때문입니다. 

그렇다면 이제는 redis-costom:second 이미지를 삭제해보겠습니다. 

```bash
Untagged: redis-costom:second
Deleted: sha256:440f5cc45f92f264c6b099ba986e4dde66f75f5694bb2b4230f065701f89686f
Deleted: sha256:9c8652fb82e504837a2dacc2063e8a29fede1934f5e13aae1bfec900509043a3
```

second는 자식 이미지가 존재하지 않기 때문에 바로 지워집니다. 

### 이미지 추출

도커 이미지를 별도로 저장하거나 옮기는 등 필요에 따라 이미지를 단일 바이너리 파일로 저장해야 될 때가 있습니다. docker save 명령어를 이용하여 컨테니너의 커맨드, 이미지 이름과 태그 등 이미지의 모든 메타데이터를 포함해 하나의 파일로 추출할 수 있습니다.

 `docker save -o ubuntu_14_04.tar ubuntu:14:04` 명령어를 이용하면 추출이 가능합니다. 이 때 -o는 추출될 파일명에 대한 옵션 입니다.

### Dockerfile

개발한 애플리케이션을 기존에는 컨테이너화 할 때 다음을 떠올려 볼 수 있습니다.

1. 아무것도 존재하지 않는 이미지(우분투, CentOS 등) 컨테이너를 생성
2. 애플리케이션을 위한 환경을 설치하고 소스코드 등 복사해 잘 동작하는 것을 확인
3. 컨테이너를 이미지로 커밋(commit)

하지만 이 방법은 애플리케이션이 동작하는 환경을 구성하기 위해 일일이 수작업으로 패키지를 설치하고 소스코드를 Git에서 복제하거나 호스트에 복사해야합니다.

도커는 위와 같은 과정을 손쉽게 기록하고 수행할 수 있는 빌드 명령어를 제공합니다. 완성된 이미지를 생성하기 위해 컨테이너에 설지해야 하는 패키지, 추가해야 하는 소스코드, 실행해야하는 명령어와 쉘 스크립트 등을 하나의 파일에 기록해 둘 수 있는데 이 파일을 **Dockerfile**이라고 합니다.

### Dockerfile 작성

이해하기 쉽게 스프링 서버를 하나의 도커 파일로 만드는 것을 보여주겠습니다. 

```bash
## Dockerfile
FROM openjdk:11-jdk
EXPOSE 8091
ARG JAR_FILE=/build/libs/kotlin-0.0.1-SNAPSHOT.jar
VOLUME ["/logs"]
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar", "-Duser.timezone=Asia/Seoul", "-Dspring.profiles.active=dev","/app.jar"]
```

위 명령어를 정리하면 다음과 같습니다.

- FROM : 생성할 이미지의 베이스가 될 이미지 입니다. Dockerfile 작성 시 한 번 이상 입력해야 합니다. 사용하려는 이미지가 존재하지 않는다면 자동으로 pull을 하여 받아옵니다.
- EXPOSE : 외부에 공개할 PORT를 지정합니다. 하지만 컨테이너 실행 시 포워딩을 해주지 않으면 아무런 동작을 하지 않습니다.
- VOLUME : 볼륨을 지정합니다. 이렇게 생성된 볼륨은 자동으로 호스트와 연결됩니다.
- ARG : `docker build` 커맨드로 이미지를 빌드 시, `--build-arg` 옵션을 통해 넘길 수 있는 인자를 정의하기 위해 사용합니다.
- COPY : 로컬 서버에 존재하는 특성 경로의 파일을 도커 컨테이너 내부로 복사합니다.
- ENTRYPOINT : 도커에서 컨테이너를 실행할 때 사용할 명령어를 설정할 수 있습니다. Dockerfile에서 컨테이너 실행 시 1회만 가능합니다.
- CMD : ENTRYPOINT와 동일하지만 CMD는 도커 명령어에서 설정값을 넣어주면 명령어가 바뀌어서 나갑니다. 하지만 ENTRYPOINT는 명령어가 추가되는 방식입니다.

그 외 주요 명령어

- RUN : 이미지를 만들기 위해 컨테이너 내부에서 명령어를 실행합니다. apt-get 같은 명령어를 이용하여 사전에 OS 패키지를 다운로드 할 수 있습니다. (단,  패키지 다운로드 같은 Y/N을 필요로하는 커맨드는 YES로 설정해주어야 합니다.)
- ADD : 파일을 이미지에 추가합니다. 보통 호스트 서버에 존재하는 파일을 가져오는 방식으로 COPY와 똑같아 보입니다. 하지만 ADD는 압축 파일을 Docker 이미지의 특정 디렉토리에 **추출**하려는 경우 또는 **원격지**
의 파일을 Docker 이미지로 **복사**하려는 경우 두 가지 상황에 사용될 수 있습니다.

### Dockerfile 빌드

```bash
docker build -t {이미지 명} {Dockerfile 경로}
```

위 명령어로 도커 파일을 빌드하여 이미지 생성을 할 수 있고 위에서 부터 차례대로 읽어들이며 빌드를 합니다.

### Dockerfile 빌드 과정

빌드 과정을 그림으로 이해한 후 간략하게 정리하면 다음과 같습니다.

![Untitled (3)](https://user-images.githubusercontent.com/77387861/217248180-98bfdc0d-215f-4c16-aa78-2d46017da4cd.png)


1. 빌드 컨텍스트를 읽어드립니다. 이때 Dockerfile에 위치한 각종 파일 소스코드 등을 담고 있는 디렉터리를 빌드 컨텍스트라고 합니다.
2. 그 이후 명령어들을 차례대로 실행한 하여 빌드합니다.
3. 빌드된 파일을 이미지로 만들어 저장합니다.

이 때 컨텍스트에 대한 정보는 이미지를 빌드할 때 출력된 내용 중 맨 위에 위치하며 너무 많은 불필요한 파일까지 읽어 들이기 때문에 그렇게 되면 속도가 느려질 뿐더러 호스트의 메모리를 지나치게 점유할 수도 있습니다.

이를 방지하기 위해 도커에는 .gitignore와 비슷한 .dockerignore를 제공합니다. 이 파일에 명시된 이름의 파일을 컨텍스트에서 제외시킬 수 있습니다. 이 때 파일은 컨텍스트의 최상위 경로로부터 시작됩니다.

### Dockerfile로 빌드할 때 주의할 점

Dockerfile을 사용하는데도 좋은 습관이 존재합니다. 예를 들어, 하나의 명령어를 `\` 로 나누어서 가독성을 높이거나 `.dockerignore` 를 활용한 불필요한 파일을 빌드 컨텍스트에 포함시키지 않는 것입니다.

ex)

```bash
RUN apt-get install package-1 \
package-2 \
package-3
```

하지만 도커의 이미지 구조와 Dockerfile의 관계를 이해하지 못한다면 불필요하게 거대한 이미지가 만들어질 수 있습니다. 다음 파일은 잘못된 도커파일의 예입니다.

```bash
FROM ubuntu:14.04
RUN mkdir /test
RUN fallocate -l 100m /test/dummy
RUN rm /test/dummy
```

위 Dockerfile에서는 100MB의 더미 데이터를 가상으로만들어 컨테이너에 할당하고 이를 이미지 레이어로 빌드합니다. 그리고 이 파일을 rm 명령어로 삭제합니다. 즉 빌드가 완료되어 최종 생성된 이미지에는 100MB 크기의 파일인 /dummy가 존재하지 않습니다.

결론적으로 100MB를 더한 크기의 이미지가 불필요한 파일 때문에 빌드되는 것입니다. 이를 방지하기 위해선 다음과 같이 수정해줍시다.

```bash
FROM ubuntu:14.04
RUN mkdir /test \
fallocate -l 100m /test/dummy \
rm /test/dummy
```

RUN이 하나의 이미지 레이어가 된다는 것을 생각하면 아주 간단한 해결책이 될 수 있습니다. 레이어가 하나로 줄어버리니 RUN 명령어를 하나로 묶어 불필요한 파일에 대한 리소스 낭비를 방지하는 것입니다.

### 도커 데몬

도커의 위치는 간단한 명령어로 찾아볼 수 있습니다.

```bash
which docker

# > /usr/bin/docker
```

도커 명령어는 `/usr/bin/docker` 에 위치한 파일을 통해 사용할 수 있습니다.

다음 명령어를 통해 실행 중인 도커 프로세스를 확인해볼 수 있습니다.

```bash
ps aux | grep docker

root  20907.  0.0  594500  68872  ?    Ssl   16:28    0:05. /usr/bin/dockered -H fd://
```

컨테이너나 이미지를 다루는 명령어는 `/usr/bin/docker`에서 실행되지만도커 엔진의 프로세스는 `/usr/bin/dockerd` 파일로 실행되고 있습니다. 즉, docker 명령어는 실제 도커 엔진이 아닌 클라이언트로서의 도커이기 때문입니다.

도커의 구조는 크게 2가지입니다.

1. 클라이언트로서의 도커
2. 서버로서의 도커

그렇기 때문에 이를 그림으로 보면 다음과 같습니다.

![Untitled (4)](https://user-images.githubusercontent.com/77387861/217248288-6b6016f7-c7f4-468a-bbff-e60c30b97be4.png)


1. 사용자가 docker version 같은 명령어를 입력합니다.
2. /usr/bin/docker 는 /var/run/docker.sock 유닉스 소켓을 사용해 도커 데몬에게 명령어를 전달합니다.
3. 도커 데몬은 이 명령어를 파싱하고 명령어에 해당하는 작업을 수행합니다.
4. 수행 결과를 도커 클라이언트에게 반환하고 사용자에게 결과를 출력합니다.
