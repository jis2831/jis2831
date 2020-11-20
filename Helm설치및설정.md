### Helm 설치 및 설정


### 1. 설치 Script Download
```
# curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get\_helm.sh
```
### 2. 설치
```
# chmod 700 get\_helm.sh
# ./get\_helm.sh
```
### 3. Tiller 서비스 계정 생성 및 kubenetes cluster-admin role 부여
```
# kubectl -n kube-system create sa tiller
# kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
```
![Helm01](./image/Helm01/ISTIO06.PNG)
### 4. Helm 초기화
```
# helm init --service-account tiller
```
### 5. Helm chart update
```
# helm repo update
```