# Docker Image (도커 이미지)의 구조와 레이어

<br>
<br>

## 📌 Docker Image란?
프로그램 실행 시 필요한 모든 것(설치 과정, 설정, 버전 정보 등)들을 포함하고 있는 템플릿 <br>
한번 생성되면 내부 정보가 바뀌지 않는 불변성과 하나의 이미지로 여러 컨테이너 생성이 가능한 재사용성이 존재한다. <br>


![image](https://github.com/solji622/LevelUp-Study/blob/95926c652f0b4d2ed39e4cb329f5355516ddb163/25.05/Docker%20Image/asset/docker_images.png)
<br>

Docker는 보통 사진과 같은 과정으로 동작한다. <br>
**Dockerfile(이미지 만들어주는 파일)은 이미지를 빌드**하고, 이 이미지가 실행되면 **컨테이너가 생성 및 실행**이 되는 구조다. 
> **컨테이너(Container)란?** &nbsp;
> 어떤 환경에서나 실행이 가능하게 필요한 모든 요소를 포함하는 소프트웨어 패키지 <br>

모든 컨테이너는 이미지를 기반으로 만들어지고, 그 이미지에는 실행에 필요한 모든 것들이 담겨있기에 <br>
**이미지는 Docker 실행의 출발점이자 동작의 핵심이라 볼 수 있다!**

<br>
<br>
<br>
<br>

### 🤔 이미지는 어떻게 구성돼 있을까?
이미지는 불변성 때문에 변경사항이 생기면 다시 도커파일에서 이미지를 빌드하여 다운로드 해야한다. <br>
이때 이미지는 컨테이너의 모든 정보를 갖고 있기에 보통 수백MB ~ 수GB가 넘는데, <br>
코드 한줄 추가 같은 작은 수정사항이 생겼을 경우에도 이 많은 용량의 이미지를 다시 다운로드 하게되기에 <br>
매우 비효율적인 방법이 될 수 있다. <br>
<br>
**때문에 이러한 문제를 기반하여 Docker에는 Layer(레이어)라는 개념이 생겼다!**
<br>
<br>
<br>
<Br>


## 📌 Layer(레이어)란?
> 기존 이미지에 추가적인 파일이 필요할 때 다시 다운로드가 아닌 해당 파일을 추가하는 것

![컨테이너 실행](https://github.com/solji622/LevelUp-Study/blob/46e37b3250b81e609f4782fc217580e8ec01a11f/25.05/Docker%20Image/asset/layer.png)
이미지는 여러 개의 레이어로 구성되어있으며, 각 레이어는 **바로 전 레이어까지의 변경 사항을 기반으로 추가 변경 사항이 쌓이는 방식**이다. <br>
이전 레이어 내용이 복사되는 것은 아니고 그저 변화만 기록된다 볼 수 있다. <br>
업데이트 부분만을 이미지로 다시 생성하고, 실행하는 시점에서 기존 이미지와 바뀐 부분을 조합하기에 <br>
업데이트 된 이미지의 크기는 큰 변화가 없다. <br>
<br>

이미지 속의 여러 레이어는 **읽기 전용(read only) 레이어와 변경 사항이 담긴 새 레이어로 구성**되어있는데, <br>
컨테이너 실행 시에는 이 읽기 전용 레이어 위에 **읽기/쓰기 전용 레이어(R/W Layer)** 를 추가한다. <br>
<br>

이미지 레이어를 그대로 불변 레이어로 사용하면서, 컨테이너 실행 중 생성되는 파일 또는 변경사항은 <br>
R/W 레이어에 저장되는 과정을 통해 여러 컨테이너를 실행하면서 **이미지는 불변성 유지가 가능** 하다. <br>
<br>
<br>
<br>
<br>

## 📌 레이어 구조
![레이어 구조](https://github.com/solji622/LevelUp-Study/blob/b0af8ac7ff408a3139f9941e4b10b6df4a7023ab/25.05/Docker%20Image/asset/layers.png)
### 1. Base Image (베이스 이미지)
이미지의 첫번째 레이어, 운영체제의 기본 파일 시스템을 포함하며 다른 모든 레이어의 기반이 된다. <br>
<br>

### 2. Intermediate Layer (중간 레이어)
Dockerfile의 명령어들이 실행되면서 만들어진 레이어들 <br>
이 레이어가 위에서 설명한 변경 사항이 쌓이는 레이어(read-only)들이다. <br>
<br>

### 3. Final Layer (최종 레이어)
컨테이너 실행 시 Docker가 자동으로 덧붙이는 최종 실행 계층 <br>
읽기/쓰기 전용 레이어라고 부르며, 컨테이너만 가지는 부분이다. <br>

<br>
<Br>
<Br>
<br>

## 📌 Dockerfile과 레이어의 관계
```dockerfile
FROM node

WORKDIR /app

COPY . .

RUN npm install

RUN npm run build

EXPOSE 3000

CMD ["node", "app.js"]
```
`FROM` 베이스 이미지 레이어 <br>
`COPY` 파일 복사 레이어 <br>
`RUN` 의존성 설치 레이어 <br>
<br>

> **⚠️ 명령 순서를 바꾸면 캐시가 깨진다** <br>
> Docker는 이미지 빌드 시 이전 빌드 내용을 캐시에 저장해놓기에 <br>
> 똑같은 명령어면 **재실행 대신 캐시를 재사용**하여 빠르게 끝낸다. <br>
> 이때, 명령어 순서가 바뀌면 캐시가 깨지고 **뒤의 명령어들이 전부 다시 실행**되기에 <br>
> 빌드 시간이 오래 걸릴 수 있다 <br>

<br>

결론적으로 Dockerfile은 **명령어 하나당 레이어가 각각 하나씩 생성**되며 <br>
이미지가 **명령 순서대로 레이어들이 쌓여가면서** 하나의 이미지가 됨을 의미한다. <br>
이때문에 Dockerfile의 명령 순서와 내용 변경은 이미지 구조와 캐시에 직접적인 영향을 준다. <br>

<br>
<br>
<br>
<br>

## 💡 이미지를 최적화 하려면?
### 1. 불필요한 파일 제외 (.dockerignore)
**`.dockerignore`** 파일에 제외할 파일명을 적으면 해당 파일을 제외하고 빌드할 수 있다! <br>
보통 `.git`, `node_modules`, `README.md` 을 제외하며, 불필요한 파일을 제외해야 이미지가 가벼워진다. <br>
<br>
<br>

### 2. RUN 명령어는 한 줄에 작성
```dockerfile
# 합치기 전
RUN npm install
RUN npm run build

# 합친 후
RUN npm install && npm run build
```
명령어 하나당 레이어 하나가 생성되는 특성을 이용하여 RUN 명령어를 하나로 처리하면 <br>
불필요한 레이어가 줄어 이미지도 더 가벼워지고 캐시도 더 효율적으로 작동할 수 있게 된다. <br>
<br>
<br>

### 3. Multi-stage build 사용
**빌드 전용** 이미지와 **실행 전용** 이미지를 별도로 생성하여 관리하는 방법

```dockerfile
# 빌드 전용
FROM node AS builder

WORKDIR /app

COPY . .

RUN npm install
RUN npm run build


# 실행 전용 (최종 이미지)
FROM node:18-slim

WORKDIR /app

# 빌드된 파일과 실행에 필요한 최소한만 복사
COPY --from=builder /app/app.js ./app.js
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/node_modules ./node_modules

EXPOSE 3000

CMD ["node", "app.js"]
```
1. 빌드 전용 : npm install, npm run build만 실행
2. 실행 전용 : 빌드 결과물과 package.json만 복사 → 실행에 필요한 것만 들어간다 

> 최종 이미지에서는 빌드 도구와 테스트 도구가 포함되지 않기에 보안에도 좋고, <br>
> 용도에 맞게 나누었기에 이미지가 작아져 빠른 빌드와 배포가 가능해지는 장점이 있다.

<br>

#### 🤔 .dockerignore랑 비슷하지않나?
> .dockerignore는 **처음부터 복사하지 않을 파일을 제외**하고 <br>
> Multi-stage Build는 **필요한 결과물만 최종 이미지에 포함한다는 점**에서 차이가 존재한다. <br>

자세하게 알아보면 .dockerignore는 Docker build context 자체에 포함하지 않는다는 의미, <br>
Multi-stage build는 node_modules, 빌드 도구 등을 빌드 단계에서만 사용하고 최종 단계엔 dist 폴더만 넣는다는 의미라 볼 수 있다. <br>
