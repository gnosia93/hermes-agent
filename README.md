# hermes-agent


### 1. 작업 폴더 및 docker-compose.yml 생성 ###
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

### 2. 에이전트 엔진 구동 ###
```bash
docker compose up -d
```
이 명령어를 치면 Docker가 알아서 공식 Nous Research 이미지를 다운로드하고 서버를 올립니다. 최초 실행이 완료되면 호스트 머신의 ~/.hermes/ 디렉토리에 에이전트의 기본 설정 파일들이 자동으로 생성됩니다.
