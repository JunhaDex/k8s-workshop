# Kubernetes Architecture Workshop

## 1. Overview
이 프로젝트는 로컬 쿠버네티스(Minikube) 환경에서 직접 클러스터를 운용해보며 쿠버네티스의 핵심 아키텍처를 이해하는 것을 목표로 합니다.

- c1: 파드 생성
- c2: 디플로이먼트 생성
- c3: 서비스 생성 및 네트워킹

## 2. Prerequisites

**필수 도구 설치 (via Homebrew)**
```shell
# 1. Docker Desktop
brew install --cask docker

# 2. Minikube
brew install minikube

# 3. k9s
brew install k9s
```

- Docker Desktop 실행하기
- `minikube start` 명령어로 Minikube 클러스터 시작하기
- `k9s` 명령어로 클러스터 확인하기

## 3. Guides

### Step 0: 실습에 사용할 이미지 빌드
```shell
docker build -t k8s-ws-api ./c0_setup/api
docker build -t k8s-ws-client ./c0_setup/client
minikube image load k8s-ws-api
minikube image load k8s-ws-client
```

### Step 1 파드 생성하기

- 단일 컨테이너 파드 생성해보기 (api)
```shell
kubectl apply -f ./c1_pod/101-simple-pod.yml
kubectl port-forward pod/simple-api-pod 3000:3000
# Browser: http://localhost:3000/healthz
# 이후 k9s 에서 ctrl + d 로 해당 파드 삭제
```

- 멀티 컨테이너 파드 생성해보기
```shell
kubectl apply -f chapter_2_pod/02-sidecar-api.yaml
# 이후 k9s 에서 pod 선택 후 `l` 옵션으로 로그 확인하기
```

🧹 Clean Up
```shell
kubectl delete -f c1_pod/
```

### Step 2 디플로이먼트 생성 및 테스트

- 레플리카 설정된 디플로이먼트 생성하기
```shell
kubectl apply -f ./c2_deployment/201-deploy-replica.yml
# 이후 Self Healing 테스트 해보기 (k9s 에서 pod 선택 후 ctrl + d 로 해당 파드 삭제)
```

- 롤링 업데이트 적용하기
```shell
kubectl apply -f ./c2_deployment/202-rolling-update.yml
# 기존 3개 파드 확인
# api 서버 업데이트 후 v2 빌드
docker build -t k8s-ws-api:v2 ./c0_setup/api
minikube image load k8s-ws-api:v2
# 202-rolling-update.yml 파일 container image 버전 v2로 업데이트 후 적용하기
kubectl apply -f ./c2_deployment/202-rolling-update.yml
# 롤링 업데이트 진행 확인하기
```

- 데몬셋 생성하기
```shell
kubectl apply -f ./c2_deployment/203-daemon-set.yml
# 노드별로 파드가 생성된 것 확인하기
# 이후 Self Healing 테스트 해보기 (k9s 에서 pod 선택 후 ctrl + d 로 해당 파드 삭제)
```

🧹 Clean Up
```shell
kubectl delete -f c2_deployment/
```

### Step 3 서비스 생성 및 네트워킹

- 서비스 배포
```shell
kubectl apply -f ./c3_service/301-service-layer.yml
```

- `minikube` 인그레스 애드온 활성화 및 터널링
```shell
minikube addons enable ingress
minikube tunnel
# 이후 네트워크 허용 설정을 위해 비밀번호 입력 요청할 수 있음
sudo vi /etc/hosts
# 아래 라인 추가하기
# 127.0.0.1 k8s-demo.local
```

- 인그레스 배포
```shell
kubectl apply -f ./c3_service/302-ingress-layer.yml
# 이후 브라우저에서 http://k8s-demo.local 접속하기 - client 서비스 확인
# 이후 브라우저에서 http://k8s-demo.local/api 접속하기 - api 서비스 확인
```

🧹 Clean Up
```shell
kubectl delete -f c3_service/
```