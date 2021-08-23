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

