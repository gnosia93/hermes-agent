# hermes-agent

* [1. 에어전트 LLM 선정하기](https://github.com/gnosia93/hermes-agent/blob/main/lecture/1-agent-llm-selection.md)

* [2. 헤르메스 설치하기]()

* [3. 종목 추천 에이전트 개발](https://github.com/gnosia93/hermes-agent/blob/main/lecture/3-stock-recommendation.md)
  - [최신 테크 트랜드]()

### 1. docker 확인 ###
```
docker context ls
```

### 2. 작업 폴더 및 docker-compose.yml 생성 ###
```bash
# 1. hermes 폴더 생성 및 이동
mkdir -p ~/hermes-agent && cd ~/hermes-agent
```

```bash
# 2. 설정 파일 만들기 (nano 또는 편한 에디터 사용)
cat <<EOF > docker-compose.yml
version: "3.8"

services:
  hermes:
    image: nousresearch/hermes-agent:latest
    container_name: hermes-agent
    restart: unless-stopped
    # 게이트웨이 모드로 상시 구동
    command: gateway run
    volumes:
      # 에이전트의 설정, API 키, 학습한 스킬 데이터를 호스트에 저장
      - ~/.hermes:/opt/data
      # 중요: Hermes 컨테이너 안에서 또 다른 '격리 샌드박스 컨테이너'를 띄우기 위해 도커 소켓 공유
      - /var/run/docker.sock:/var/run/docker.sock
EOF
```

### 3. 에이전트 엔진 구동 ###
```bash
docker compose up -d
```
이 명령어를 치면 Docker가 알아서 공식 Nous Research 이미지를 다운로드하고 서버를 올립니다. 최초 실행이 완료되면 호스트 머신의 ~/.hermes/ 디렉토리에 에이전트의 기본 설정 파일들이 자동으로 생성됩니다.


### 4. 샌드박스(실험실) 격리 설정 적용 ###
```
cat <<EOF > ~/.hermes/config.yaml
# 기존 로컬(local)로 되어 있던 백엔드를 docker로 변경
backend: docker

docker:
  image: "python:3.11-slim"
  volumes:
    - ~/hermes-workspace:/workspace # 에이전트가 작업할 호스트 공간 연결
EOF
```
수정 후 docker compose restart 명령어로 에이전트를 재시작하면 모든 세팅이 완료됩니다.
이제 Hermes Agent는 유저가 텔레그램, 디스코드, 혹은 웹 UI를 통해 내리는 명령을 안전하게 수신하며, 위험한 터미널 명령어나 스크립트를 실행할 때마다 지정된 Python 컨테이너를 임시로 띄워 그 안에서만 안전하게 연산을 수행하게 됩니다. 완벽하게 격리된 AI 개발 환경이 Mac 미니에 성공적으로 구축된 것입니다.

### 5. AWS Bedrock 연결하기 ###
#### 1단계: 도커 컨테이너에 AWS 권한 주기 (.env) ####
도커 컨테이너 안에 있는 헤르메스가 외부 AWS Bedrock API를 호출하려면, 먼저 로그인 자격 증명(Credentials)을 쥐여주어야 합니다. 헤르메스 디렉토리의 .env 파일에 AWS 인증 정보를 추가합니다.
```
# ~/.hermes/.env 파일에 추가
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
AWS_DEFAULT_REGION=us-east-1
```

#### 2단계: 헤르메스에게 Bedrock 쓰라고 명령하기 (config.yaml) ####
이제 헤르메스의 메인 지침서인 config.yaml을 수정하여 프로바이더를 Bedrock으로 지정하고, 원하는 모델 ID를 적어줍니다.
```
# ~/.hermes/config.yaml 파일 수정
provider: "bedrock"
default: "us.anthropic.claude-sonnet-4-6"  # 사용할 Bedrock 모델 ID

bedrock:
  region: "us-east-1"  # Bedrock 모델 권한을 승인받은 리전
```
직접 파일을 수정하기 번거롭다면, 헤르메스 컨테이너 터미널 안에서 아래 명령어로 인터랙티브하게 설정할 수도 있습니다.
```
hermes setup model
```
위 명령어를 실행한 뒤 프로바이더로 AWS Bedrock을 선택하고 안내를 따르면 파일이 자동으로 업데이트됩니다.

#### AWS 연동시 체크포인트 ####
* AWS 콘솔에서 모델 권한 활성화: 설정하기 전에 반드시 AWS Bedrock 콘솔의 Model access 메뉴에서 사용하려는 모델(예: Anthropic Claude 3.5 Sonnet 등)의 접근 권한이 Granted(승인됨) 상태인지 확인해야 합니다.
*	Inference Profile ID 권한 장장 (강추): 모델 ID를 적을 때 anthropic.claude-sonnet-4-6 처럼 베어(Bare) ID를 적는 것보다, 위 예시처럼 us.anthropic.claude-sonnet-4-6 같은 크로스 리전 추론 프로필(Cross-Region Inference Profile) ID를 사용하는 것을 강력히 권장합니다. 트래픽 유연성과 처리 속도(Throughput)가 훨씬 뛰어납니다.
*	IAM 권한 확인: 인증에 사용하는 IAM 사용자나 역할(Role)에 최소한 bedrock:InvokeModel 및 bedrock:InvokeModelWithResponseStream 권한이 포함되어 있어야 헤르메스가 답변을 스트리밍으로 받아올 수 있습니다.


### 6. 인터랙티브 프롬프트 ###

#### CLI ####

```
docker ps
docker exec -it <컨테이너_이름> hermes
```
터미널 창이 즉시 Hermes > 대화 모드로 전환됩니다.


#### GUI ####
```
docker ps

```
docker ps 의 출력 결과 중 PORTS 부분에 0.0.0.0:3000->3000/tcp 같은 내용이 보인다면 GUI 준비가 끝난 것입니다. (헤르메스 전용 WebUI는 보통 3000번이나 8642번을 씁니다.) 
크롬이나 사파리를 열고 http://localhost:3000 로 접속합니다.

## 레퍼런스 ##
* https://hermes-agent.nousresearch.com/
