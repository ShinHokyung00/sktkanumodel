# sktkanumodel
 

# 운영 - 테스트 작성@오기열/신호경매니저
 
## Deploy/ Pipeline
각 구현체들은 각자의 source repository 에 구성되었고, 사용한 CI/CD 플랫폼은 Azure를 사용하였으며, pipeline build script 는 각 프로젝트 폴더 이하에 cloudbuild.yml 에 포함되었다.

- git에서 소스 가져오기
```
git clone https://github.com/PARKBYOUNGHWA/sktkanumodel
```
- Build 및 ACR 에 Docker Image Push 하기
```
cd ./sktkanumodel
cd gateway
mvn package
az acr build --registry mygroupacr --image mygroupacr.azurecr.io/kanugateway:latest .

cd ../order
mvn package
az acr build --registry mygroupacr --image mygroupacr.azurecr.io/kanuorder:latest .

cd ../payment
mvn package
az acr build --registry mygroupacr --image mygroupacr.azurecr.io/kanupayment:latest .

cd ../delivery
mvn package
az acr build --registry mygroupacr --image mygroupacr.azurecr.io/kanudelivery:latest .

cd ../ordertrace
mvn package
az acr build --registry mygroupacr --image mygroupacr.azurecr.io/kanuordertrace:latest .
```
- Kubernetes Deploy, SVC 생성(yml 이용)
```
-- 기본 namespace 설정
kubectl config set-context --current --namespace=sktkanu
-- namespace 설정
kubectl create ns sktkanu
```

- ACR에서 이미지 가져와서 Kubernetes에서 Deploy하기
```
kubectl create deploy order --image=mygroupacr.azurecr.io/kanuorder:latest
kubectl create deploy payment --image=mygroupacr.azurecr.io/kanupayment:latest
kubectl create deploy delivery --image=mygroupacr.azurecr.io/kanudelivery:latest
kubectl create deploy ordertrace --image=mygroupacr.azurecr.io/kanuordertrace:latest
kubectl create deploy gateway --image=mygroupacr.azurecr.io/kanugateway:latest
kubectl get all
```
- Kubectl Deploy 결과 확인
![image](https://user-images.githubusercontent.com/44763296/130466041-12048ad7-228c-4968-b037-3bb309716bde.png)

- Kubernetes에서 서비스 생성하기 (Docker 생성이기에 Port는 8080이며, Gateway는 LoadBalancer로 생성)
```
kubectl expose deploy order --type="ClusterIP" --port=8080
kubectl expose deploy payment --type="ClusterIP" --port=8080
kubectl expose deploy delivery --type="ClusterIP" --port=8080
kubectl expose deploy ordertrace --type="ClusterIP" --port=8080
kubectl expose deploy gateway --type="LoadBalancer" --port=8080
kubectl get all
```
- Kubectl Expose 결과 확인
![image](https://user-images.githubusercontent.com/44763296/130466081-886aab6f-176e-4a1e-8320-84336092dde2.png)

# 무정지 재배포 (Readiness Probe)
- 먼저 무정지 재배포가 100% 되는 것인지 확인하기 위해서 Autoscaler 이나 CB 설정을 제거함
- siege 로 배포작업 직전에 워크로드를 모니터링 함
```
siege -c100 -t60S -r10 -v http get http://order:8080/orders
```
- Readiness가 설정되지 않은 yml 파일로 배포 진행
```
kubectl apply -f deployment_without_readiness.yml
```
- 아래 그림과 같이, Kubernetes가 준비가 되지 않은 delivery pod에 요청을 보내서 siege의 Availability 가 100% 미만으로 떨어짐
- 중간에 socket에 끊겨서 siege 명령어 종료됨 (서비스 정지 발생)

- 정지 재배포 여부 확인 전에, siege 로 배포작업 직전에 워크로드를 모니터링
```
siege -c100 -t60S -r10 -v http get http://order:8080/orders
```
- Readiness가 설정된 yml 파일로 배포 진행
```
readinessProbe:
  httpGet:
    path: '/actuator/health'
    port: 8080
  initialDelaySeconds: 10
  timeoutSeconds: 2
  periodSeconds: 5
  failureThreshold: 10
```
```
kubectl apply -f deployment_with_readiness.yml
```
- 배포 중 pod가 2개가 뜨고, 새롭게 띄운 pod가 준비될 때까지, 기존 pod가 유지됨을 확인

- 생성된 siege Pod 안쪽에서 정상작동 확인
```
kubectl exec -it siege -- /bin/bash
siege -c1 -t2S -v http://order:8080/orders
```
