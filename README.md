# ☸️ kubernetes-springboot-deploy

> 쿠버네티스 환경에서 스프링부트 애플리케이션을 컨테이너화하여 배포하는 실습 프로젝트

## 📑 목차
1. [프로젝트 개요](#-프로젝트-개요)
2. [기술 스택](#-기술-스택)
4. [배포 프로세스](#-배포-프로세스)
5. [실행 결과](#-실행-결과)
6. [NodePort와 LoadBalancer 비교](#-nodeport와-loadbalancer-비교)
7. [트러블슈팅](#-트러블슈팅)

## 🎯 프로젝트 개요
이 프로젝트는 Windows 환경에서 개발된 Spring Boot 애플리케이션을 Docker 이미지로 만들고, Kubernetes(Minikube)에 배포하는 전체 과정을 다룹니다. 
특히 NodePort와 LoadBalancer 서비스 유형에 따른 로드밸런싱 동작의 차이를 확인할 수 있습니다.

## 🛠 기술 스택

### 개발 환경
![Windows](https://img.shields.io/badge/Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Spring Tool Suite](https://img.shields.io/badge/Spring_Tool_Suite-6DB33F?style=for-the-badge&logo=spring&logoColor=white)
![Gradle](https://img.shields.io/badge/Gradle-02303A?style=for-the-badge&logo=gradle&logoColor=white)

### 애플리케이션
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white)
![Java](https://img.shields.io/badge/Java-007396?style=for-the-badge&logo=java&logoColor=white)

### 컨테이너 및 오케스트레이션
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Minikube](https://img.shields.io/badge/Minikube-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)

### 인프라 및 도구
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![MobaXterm](https://img.shields.io/badge/MobaXterm-000000?style=for-the-badge&logo=terminal&logoColor=white)

## 📋 배포 프로세스

### 1. Windows 개발 환경
- Windows 환경에서 Spring Boot 애플리케이션 개발
- 기본적인 REST API 및 웹 페이지 제공

### 2. JAR 파일 생성
**STS를 사용해 Gradle로 실행 가능한 JAR 파일 생성**

<img width="727" alt="image" src="https://github.com/user-attachments/assets/51db3c45-3b81-45b0-b19b-c01c63721289" />

### 3. MobaXterm으로 파일 전송
**Windows에서 생성된 JAR 파일을 Linux 서버로 전송**
- MobaXterm을 이용해 SFTP 연결
  
<img width="382" alt="image" src="https://github.com/user-attachments/assets/9f6d837c-55fd-4d6d-b4e6-5c1c17334b42" />

### 4. Docker 이미지 생성
**Linux 서버에서 Dockerfile 생성 및 이미지 빌드**

<br>

<details>
<summary> Dockerfile </summary>
    
```
# 실행 환경 기반 이미지 선택
FROM eclipse-temurin:17-jre-alpine

# 작업 디렉토리 설정
WORKDIR /app

# JAR 파일 컨테이너로 복사
COPY step07_cicd1-0.0.1-SNAPSHOT.jar app.jar

# 최적화 환경 변수 설정
ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC -XX:MaxGCPauseMillis=200"

# 애플리케이션 실행 명령
ENTRYPOINT java $JAVA_OPTS -jar app.jar
```
</details>

<br>

**Docker 이미지 빌드**

```bash
$ docker build -t springboot:1.0 .
```

<img width="594" alt="image" src="https://github.com/user-attachments/assets/673fdcab-59c6-4f04-8c7e-773433b08ad9" />

### 5. Docker Hub 로그인
```bash
$ docker login
```

<img width="965" alt="image (3)" src="https://github.com/user-attachments/assets/be451068-0852-4283-bcd3-68f2b19cc5f3" />

### 6. 이미지 Tag 설정
```
$ docker tag springboot:1.0 yourusername/springboot:1.0
```

<img width="547" alt="image (4)" src="https://github.com/user-attachments/assets/37c97be2-8465-4f0d-a2f5-61e5f417b9db" />


### 7. 이미지 PUSH
```
$ docker push yourusername/springboot:1.0
```

<img width="605" alt="image (5)" src="https://github.com/user-attachments/assets/75f08109-242f-4f13-8898-70a63d89f567" />


### 8. Docker Hub 확인

<img width="1280" alt="image (6)" src="https://github.com/user-attachments/assets/0da65c3c-1600-4992-a028-a679a5b58fb1" />


### 9. Kubernetes 배포
**Minikube를 사용한 Kubernetes 배포**

```bash
# Minikube 시작
$ minikube start
```

#### NodePort 배포 YAML

<br>

<details>
<summary> NodePort YAML </summary>
    
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: custom-app
        image: sugeunjee/springboot:1.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: spring-app-service
spec:
  type: NodePort
  selector:
    app: spring-app
  ports:
  - protocol: TCP
    port: 9000         # 서비스 포트
    targetPort: 8080   # 컨테이너 포트 (애플리케이션이 다른 포트를 사용한다면 변경 필요)
    nodePort: 30080  # 선택 사항: 30000-32767 사이에서 특정 노드 포트 지정 가능

```
</details>

<br>

#### LoadBalancer 배포 YAML

<br>

<details>
<summary> LoadBalancer YAML </summary>
    
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: custom-app
        image: sugeunjee/springboot:1.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: spring-app-service
spec:
  type: LoadBalancer
  selector:
    app: spring-app
  ports:
  - protocol: TCP
    port: 9000         # 서비스 포트
    targetPort: 8080   # 컨테이너 포트 (애플리케이션이 다른 포트를 사용한다면 변경 필요)
    nodePort: 30080  # 선택 사항: 30000-32767 사이에서 특정 노드 포트 지정 가능

```
</details>

<br>

**Kubernetes에 배포**

```bash
# NodePort 배포
kubectl apply -f spring-nodeport.yaml

# 또는 LoadBalancer 배포
kubectl apply -f spring-loadbalancer.yaml

# 배포 상태 확인
kubectl get all
```

## 📊 실행 결과
배포된 애플리케이션의 상태 확인:
```bash
kubectl get pods
kubectl get deployments
kubectl get services
```

## 🔄 NodePort와 LoadBalancer 비교

### NodePort 서비스
- **특징**: 각 노드의 특정 포트에 서비스를 노출
- **로드밸런싱**: 클라이언트가 특정 노드 IP로 접근하면 항상 같은 노드로만 요청이 전달됨
- **결과**: 실험 결과 NodePort 타입으로는 실제 로드밸런싱이 제대로 동작하지 않음

- NodePort로 생성
<img width="670" alt="image (7)" src="https://github.com/user-attachments/assets/1f8f4361-7207-4119-b21d-8aa64e0b6d3f" />

<br>

- minikube ip 확인
<img width="231" alt="image" src="https://github.com/user-attachments/assets/fb12aa10-a774-4bfb-bfd3-651e6c92c1e7" />

<br>

- 포트 포워딩
<img width="602" alt="image (8)" src="https://github.com/user-attachments/assets/363fb4e8-bb3d-4ab4-b10c-371ff623670c" />

<br>

- 로드 밸런싱 불가
<img width="1274" alt="image (9)" src="https://github.com/user-attachments/assets/69edb12a-6274-46a2-957d-9f35457294ef" />

<br>

### LoadBalancer 서비스
- **특징**: 클라우드 제공업체의 로드밸런서를 사용해 서비스 노출
- **로드밸런싱**: 로드밸런서가 백엔드 포드들에 트래픽을 분산
- **결과**: 트래픽이 여러 포드에 골고루 분산되어 로드밸런싱이 효과적으로 작동

- LoadBalancer로 생성
<img width="691" alt="image (10)" src="https://github.com/user-attachments/assets/b34f9136-8351-4f72-9e84-5e6661553ed7" />

<br>

- 로드 밸런싱 성공
<img width="1277" alt="image (11)" src="https://github.com/user-attachments/assets/e187d710-a77e-4017-ae5b-74344cfabce9" />

<br>

## ❗ 트러블슈팅

### 1. Docker 이미지 실행 오류
**문제**: Docker 컨테이너 실행 시 `Could not find or load main class ${JAVA_OPTS}` 오류 발생

**원인**: Dockerfile의 ENTRYPOINT가 JSON 배열 형식(`ENTRYPOINT ["java", "${JAVA_OPTS}", "-jar", "app.jar"]`)에서 환경 변수가 제대로 확장되지 않음

**해결**:
```dockerfile
# 문제가 있는 방식
# ENTRYPOINT ["java", "${JAVA_OPTS}", "-jar", "app.jar"]

# 해결 방식
ENTRYPOINT java $JAVA_OPTS -jar app.jar
```

### 2. Kubernetes 파드 시작 오류
**문제**: 쿠버네티스 배포 후 파드가 Error 상태로 표시됨, 하지만 생성한 이미지를 docker run 시키게 되면 정상 실행 확인

<img width="644" alt="image" src="https://github.com/user-attachments/assets/bade2d06-b222-4d39-be60-7f09b628c69d" />


<img width="968" alt="image" src="https://github.com/user-attachments/assets/02e2f475-de35-4776-b6b1-10ec8745123b" />

<br><br>

**원인**: Docker Hub에 이미지를 PUSH 할 때, 기존 이미지를 삭제 하고 PUSH 했지만 minikube 내부에서는 이미 같은 이미지가 있기 때문에 새로운 이미지를 다운받아 오지 않음!!

**해결**: minikube image를 삭제해주면 다시 Docker Hub에서 다운받을 것!!

- minikube image 리스트 확인

<img width="382" alt="image" src="https://github.com/user-attachments/assets/8bf0a0ba-5db8-494a-a31e-0e081000e715" />

- minikube image 삭제

<img width="511" alt="image" src="https://github.com/user-attachments/assets/83aad2ab-1477-4a47-ba6a-2ad764f97d2d" />

- 쿠버네티스 실행, 정상 실행 확인!

<img width="632" alt="image" src="https://github.com/user-attachments/assets/8fde513e-fbc7-46cd-a3f0-94cbfceed4cc" />


### 3. NodePort 로드밸런싱 문제
**문제**: NodePort 서비스를 사용할 때 로드밸런싱이 제대로 동작하지 않음

**원인**: NodePort는 외부 트래픽이 특정 노드를 거쳐 진입할 때 해당 노드에 있는 파드로만 라우팅됨

**해결**: 로드밸런싱을 위해 LoadBalancer 유형의 서비스로 변경하여 외부 트래픽이 다양한 파드에 분산되도록 함
