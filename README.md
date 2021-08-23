# sktkanumodel
 

# 운영 - 테스트 작성@오기열/신호경매니저
 
## Deploy/ Pipeline
각 구현체들은 각자의 source repository 에 구성되었고, 사용한 CI/CD 플랫폼은 Azure를 사용하였으며, pipeline build script 는 각 프로젝트 폴더 이하에 cloudbuild.yml 에 포함되었다.

- git에서 소스 가져오기
```
git clone https://github.com/PARKBYOUNGHWA/sktkanumodel
```
- Build 및 ACR 에 Push 하기
```
cd ./sktkanumodel
cd gateway
mvn package
az acr build --registry mygroupacr --image mygroupacr.azurecr.io/kanugateway:latest .

cd ../order
mvn package
az acr build --registry mygroupacr --image mygroupacr.azurecr.io/kanuorder:latest .

cd ../delivery
mvn package
az acr build --registry mygroupacr --image mygroupacr.azurecr.io/kanudelivery:latest .

cd ../ordertrace
mvn package
az acr build --registry mygroupacr --image mygroupacr.azurecr.io/kanuordertrace:latest .
```
