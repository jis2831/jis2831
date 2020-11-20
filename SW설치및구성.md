

**Kubernetes 설치 및 설정**

**(in Centos 7)**

**1. 준비**

**설치준비**

**1-01**

**1. Master node 와 Work Node1, Work Node2…. Work Node#n 으로 사용될 서버에 CentOS 7을 설치한다.**

**2. 서버 당 2GB 이상의 RAM이 필요하다**

**3. 클러스터의 모든 시스템 간의 네트워크 연결이 되어 있어야 한다.**

**4. 각 서버에 root계정으로 /etc/hosts 파일에 IP와 호스트 이름을 다음과 같이 설정한다.**

**# vi /etc/hosts**

**위의 예시에서는**

**192.168.90.11 Master**

**192.168.90.12 Node1**

**192.168.90.13 Node2**





**1. 준비**

**Docker 설치**

**1-02**

**1. 전체 서버에 Docker 를 설치한다.(https://docs.docker.com/engine/install/centos/)**

**2. Docker 설치가 완료되면 아래의 명령어를 실행시킨다.**

**# yum install -y yum-utils device-mapper-persistent-data lvm2**

**# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo**

**# yum install docker-ce**

**# systemctl start docker && systemctl enable docker**





**1. 준비**

**kubeadm 설치 준비(모든 서버에서 실행)**

**1-03**

**1. SELinux 설정을 permissive 모드로 변경**

**# setenforce 0**

**# sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config**

**2. iptable 설정**

**# cat <<EOF > /etc/sysctl.d/k8s.conf**

**net.bridge.bridge-nf-call-ip6tables = 1**

**net.bridge.bridge-nf-call-iptables = 1**

**EOF**

**$ sysctl --system**

**3. firewalld 비활성화**

**# systemctl stop firewalld**

**# systemctl disable firewalld**

**4. 스왑 오프**

**# swapoff -a**

**5. /etc/fstab 파일에 아래 코드 주석(#) 처리**

**#/dev/mapper/centos-swap swap**

**swap defaults**

**0 0**

**6. 서버 재시작**

**# reboot**





**1. 준비**

**1-04 Kubernetes yum repository 설정(모든 서버에서 실행)**

**1. 서버의 재부팅이 완료되면 서버에 재접속한 후 Kubernetes yum repository 설정을 한다.**

**# cat <<EOF > /etc/yum.repos.d/kubernetes.repo**

**[kubernetes]**

**name=Kubernetes**

**baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86\_64**

**enabled=1**

**gpgcheck=1**

**repo\_gpgcheck=1**

**gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg**

**exclude=kube\***

**EOF**





**2. 설치**

**2-01 kubeadm 설치(모든 서버에서 실행)**

**1. kubeadm 설치**

**# yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes**

**# systemctl enable kubelet && systemctl start kubelet**





**2. 설치**

**마스터 노드 컴포넌트 설치(마스터 노드)**

**2-02**

**1. kubeadm init 명령으로 마스터 노드 초기화**

**kubeadm init 명령어를 이용해서 마스터 노드를 초기화한다. --pod-network-cidr 옵션은 사용할 CNI(Container Network Interface)에 맞게**

**입력한다. 여기에서는 CNI로 Flannel(--pod-network-cidr=10.244.0.0/16)을 사용한다.**

**# kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=172.27.0.211 -> 마스터** **노드** **Server IP**

**2. 마스터 노드 초기화 완료 모습(빨간색으로 표시된 부분은 노드 컴포넌트 설치 시 필요하므로 따로 복사해두면 편리하다.)**

**# kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=172.16.1.100**

**...생략**

**[addons] Applied essential addon: CoreDNS**

**[addons] Applied essential addon: kube-proxy**

**Your Kubernetes master has initialized successfully!**

**To start using your cluster, you need to run the following as a regular user:**

**mkdir -p $HOME/.kube**

**sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config**

**sudo chown $(id -u):$(id -g) $HOME/.kube/config**

**You should now deploy a pod network to the cluster.**

**Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:**

**https://kubernetes.io/docs/concepts/cluster-administration/addons/**

**You can now join any number of machines by running the following on each node**

**as root:**

**kubeadm join 172.16.1.100:6443 --token yrc47a.55b25p2dhe14pzd1 --discovery-token-ca-cert-hash sha256:2a7a31510b9a0b0da1cf71c2c29627b40711**

**cdd84be12944a713ce2af2d5d148**





**2. 설치**

**2-05 kubectl 허용(마스터 노드)**

**1. 아래의 명령어로 root 이외의 다른 사용자도 kubectl 명령을 사용 가능하도록 허용해준다.**

**# mkdir -p $HOME/.kube**

**# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config**

**# sudo chown $(id -u):$(id -g) $HOME/.kube/config**

**2. 환경 변수 설정**

**export KUBECONFIG=/etc/kubernetes/admin.conf**





**2. 설치**

**2-06 CNI 설치 (마스터 노드)**

**1. 여기서는 Flannel을 설치한다. 설치 명령어는 다음과 같다.**

**# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml**

**2. 실행 확인**

**# kubectl get pods --all-namespaces**

**2. 마스터 실행 확인 결과(kubectl 명령어로 쿠버네티스 마스터 실행을 확인한다. STATUS가 Running이면 정상 실행된 것이다.)**

ACI, Calico, Canal, Cilium, CNI-Genie, Contiv, Flannel,

Multus, NSX-T, Nuage, Romana, Weave Net 정리할것





**2. 설치**

**2-06 CNI 별 정리(Pod 네트워킹 인터페이스)**

**1. Pod 네트워킹 인터페이스에 대한 설명은 https://medium.com/finda-tech/kubernetes-**

**%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%A0%95%EB%A6%AC-fccd4fd0ae6 참조할것**

**CNI종류**

**마스터** **노드** **초기화** **명령어**

**CNI 설치명령어**

kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-

datastore/calico-networking/1.7/calico.yaml

Calico

kubeadm init --pod-network-cidr=192.168.0.0/16

Flannel

Weave

kubeadm init --pod-network-cidr=10.244.0.0/16

kubeadm init --pod-network-cidr 10.32.0.0/12

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

kubeadm init --pod-network-cidr=10.217.0.0/16 -

-skip-phases=addon/kube-proxy

kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/v1.8/examples/kubernetes/connectivity-

check/connectivity-check.yaml

Cilium





**2. 설치**

**워크노드 컴포넌트 설치 (워크노드)**

**2-07**

**1. kubeadm init 명령을 이용해서 설치할 때 콘솔에 출력된 메시지에 kubeadm join 명령어가 있었다.**

**이 명령어를 노드 컴포넌트로 사용할 서버에서 실행한다.**

**# kubeadm join 172.16.1.100:6443 --token yrc47a.55b25p2dhe14pzd1 --discovery-token-ca-cert-hash sha256:2a7a31510b9a0b0da1cf71c2c296**

**27b40711cdd84be12944a713ce2af2d5d148**

**2. kubeadm join 명령어를 복사해 놓지 않았다면 아래 명령어를 실행하여 token정보를 확인할 수 있다.**

**# kubeadm token list**

**3. token은 24시간 이후에는 사용불가 하므로 마스터 설치 이후 24시간이 지난 후에 노드 컴포넌트를 설치하는 경우에는 token을 새로 생성한다.**

**# kubeadm token create**

**4. sha256값은 아래 명령어를 실행하면 나온다.**

**# # openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.\* //'**





**2. 설치**

**2-08 kubectl 허용(워크노드)**

**1. 마스터 노드에서 아래의 명령어를 실해한 후 config 파일 내용을 복사한다.**

**# cat $HOME/.kube/config**

**2. 워크 노드 서버의 $HOME 디렉토리 하위에 .kube 디렉토리를 생성한다.**

**# cd $HOME**

**# mkdir .kube**

**3. 아래의 명령을 실행한 후 vi 편집기에서 마스터 노드의 config 내용을 붙여 넣는다.**

**# vi config**





**3. 확인**

**실행 확인**

**3-01**

**1. kubectl 명령어로 정상적으로 쿠버네티스가 동작하는지 확인한다.**

**# kubectl get nodes**

**# kubectl get pod --all-namespaces -o wide**





**4. 삭제**

**삭제(필요시)**

**4-01**

**1. 아래의 명령어로 쿠버네티스를 삭제한다.**

**# kubeadm reset**

**# systemctl stop kubelet**

**# systemctl stop docker**

**# rm -rf /var/lib/cni/**

**# rm -rf /var/lib/kubelet/\***

**# rm -rf /run/flannel**

**# rm -rf /etc/cni/**

**# rm -rf /etc/kubernetes**

**# rm -rf /var/lib/etcd/**

**# ip link delete cni0**

**# ip link delete flannel.1**

**# yum remove -y kubelet**

**# yum remove -y kubectl**

**# yum remove -y kubeadm**

**# systemctl start docker**





**5. 트러블슈팅**

**네트워크 (DNS) 문제로 ContainerCreating 에 멈춘 경우**

**5-01**

**1. cni 가 꼬여서, Cluster 전체가 망가진 상황**

**2. 해결책은 Cluster 를 다시 구성**

**# kubeadm reset**

**# systemctl stop kubelet**

**# systemctl stop docker**

**# rm -rf /var/lib/cni/**

**# rm -rf /var/lib/kubelet/\***

**# rm -rf /etc/cni/**

**# ifconfig cni0 down**

**# ifconfig flannel.1 down**

**# ifconfig docker0 down**

**# ip link delete cni0**

**# ip link delete flannel.1**

**3. /sbin/ifconfig 를 실행해서, 모든 가상 네트워크가 삭제되었음을 확인 후 kubeadm init…. 부터 다시 시작**

**# kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=172.16.1.100 -> 마스터** **노드** **Server IP**





**5. 트러블슈팅**

**노드서버에서 kubectl 명령시 에러**

**5-01**

**1. kubectl 명령시 아래와 같은 에러가 발생한 경우**

**1. 마스터 노드에서 아래의 명령어를 실해한 후 config 파일 내용을 복사한다.**

**# cat $HOME/.kube/config**

**2. 워커노드 서버의 $HOME 디렉토리 하위에 .kube 디렉토리를 생성한다.**

**# cd $HOME**

**# mkdir .kube**

**3. 아래의 명령을 실행한 후 vi 편집기에서 마스터 노드의 config 내용을 붙여 넣는다.**

**# vi config**





**ISTIO 설치 및 설정**





**1. 준비**

**서버에 접속 후 root 계정으로 변경**

**1-01**

**1. 서버에 로그인하여 root 계정으로 switch 한다.**

**$ su - root**

**root 계정의 암호를 입력한 후 엔터**





**1. 준비**

**ISTIO와 Kubernetes의 호환 버전 확인**

**1-01**

**1. Kubernetes의 버전을 확인한다. 여러 버전이 설치되어 있을 경우에는 kubectl get nodes 명령으로 확인한다.**

**# kubectl version**

**# kubectl get nodes**

**bash: kubectl: 명령을 찾을 수 없습니다…**

**오류가 발생한다면 Kubernetes 설치 부터 진행한다.**

**Kubernetes 설치는 20번 슬라이드로 이동하여 진행한다.**

**2. ISTIO 설치 전에 아래의 표를 확인하여 설치할 서버의 Kubernetes 버전에 맞는 ISTIO Release version을 확인한다.**

**현재 예시에는 Kubernetes 버전이 1.19 버전이므로 ISTIO는 1.7을 설치하여야 한다.**

**ISTIO Release Version**

**Kubernetes Release Version**

1.16, 1.17, 1.18, 1.19

1.15, 1.16, 1.17, 1.18

1.14, 1.15, 1.16

1.7

1.6

1.5

1.4

1.3

1.13, 1.14, 1.15

1.13, 1.14, 1.15





**2. 설치**

**ISTIO 다운로드**

**2-01**

**1. 최신 버전을 다운로드 하려면 아래의 명령어를 실행한다.**

**# curl -L https://istio.io/downloadIstio | sh -**

**2. 특정 버전을 다운로드 하려면 아래의 명령어를 실행한다. 아래의 예시는 1.6.8 버전을 다운로드 한다.**

**(빨간색 글씨 부분에 다운로드 받을 버전을 명시)**

**$ curl -L https://istio.io/downloadIstio | ISTIO\_VERSION=1.6.8 TARGET\_ARCH=x86\_64 sh -**





**2. 설치**

**ISTIO 다운로드 디렉터리로 이동**

**2-02**

**1. ISTIO 패키지 디렉터리로 이동한다. 예를 들어 패키지가 istio-1.7.1 인 경우 아래와 같이 명령어를 실행한다.**

**# cd istio-1.7.1**





**2.설치**

**2-03 ISTIO 환경변수 설정(1/2)**

**1. pwd 명령어를 실행하여 현재 디렉터리 위치를 알아낸다. 현재 디렉터리 위치를 복사해 놓는다.**

**# pwd**

**2. cd 명령어를 실행하여 root 디렉터리로 이동한다.**

**# cd**

**3. vi .bashrc 명령어를 실행하여 .bashrc 파일을 vi 편집기로 연다.**

**# vi .bashrc**

**4. keyboar에서 i 버튼을 클릭하여 insert 모드로 변경한다. 최하위로 이동하여 다음과 같이 입력한다.**

**export PATH=/root/install\_file/istio/istio-1.7.0/bin:$PATH**

**빨간색 글씨로 표시된 부분이 pwd 명령어를 실행했을 때 조회되었던 디렉터리 위치이다.**

**예시에는 /root/install\_file/istio 하위 디렉터리에 istio-1.7.0 버전을 설치하여서 아래와 같이 환경변수를 입력하였다.**





**2.설치**

**2-04 ISTIO 환경변수 설정(2/2)**

**5. 입력이 끝났으면 차례대로 Esc 입력 : 입력 wq! 입력한 후 엔터를 친다.**

**6. 아래의 명령어를 입력하여 저장된 내용을 반영한다.**

**# source .bashrc**

**7. 아래의 명령어를 입력하여 환경변수로 등록한 내용을 확인한다.**

**# echo $PATH**

**8. 등록한 환경변수를 확인한다.**





**3. 설치**

**ingress-gateway NodePort 사용 설정**

**2-05**

**1. 다음 명령을 실행하여 ingress-gateway 설치 파일을 찾는다**

**# find ./ -name values.yaml**

**2. vi 편집기로 istio-ingress 설치파일을 오픈한다.( ~~~/charts/gateways/istio-ingress/values.yaml** à **버전마다 경로가 다름)**

**# vi ./manifests/charts/gateways/istio-ingress/values.yaml**

**3. vi 편집기에서 /LoadBalancer 를 입력한 후 엔터를 친다. -> 검색된 후에 i 입력하여 편집모드로 변경한다. -> LoadBalancer 부분을 지우고**

**NodePort로 바꿔준다. -> ESC 버튼을 입력한후 : 입력하고 wq! 를 입력하여 편집을 종료한다.**





**3. 설치**

**kiali Service NodePort 사용 설정**

**2-06**

**1. 다음 명령을 실행하여 kiali 설치 파일을 찾는다**

**# find ./ -name values.yaml**

**2. vi 편집기로 kiali 설치파일을 오픈한다.( ~~~/charts/istio-telemetry/kiali/values.yaml** à **버전마다 경로가 다름)**

**# vi ./manifests/charts/istio-telemetry/kiali/values.yaml**

**3. vi 편집기에서 /ClusterIP 를 입력한 후 엔터를 친다. -> 검색된 후에 i 입력하여 편집모드로 변경한다. -> ClusterIP 부분을 지우고 NodePort로**

**바꿔준다. -> ESC 버튼을 입력한후 : 입력하고 wq! 를 입력하여 편집을 종료한다.**





**2.설치**

**2-07 ISTIO 설치**

**1. 아래 명령어를 실행하여 ISTIO를 설치한다.**

**# istioctl install --set profile=demo**

**2. 정상적으로 설치완료 된 모습은 아래와 같다.**

**3. 오류 현상 및 해결법(오류가 없다면 다음으로 건너 뛰어 계속)**

**-> 이 오류가 발생했다면 Kubernetes를 설치한 계정과 ISTIO를 설치한 계정이 달라서 나는 오류일 가능성이 있음.**

**또한 root계정으로 설치를 시도했으나 Kubernetes 클러스터에 Acess를 하지 못하면 위와같은 오류가 발생**

**Kubernetes를 설치한 계정과 ISTIO를 설치한 계정이 같아야함.**

**-> 이 오류가 발생했다면 Kubernetes versio이 Istio 버전과 호환되지않아 나는 오류(not supported) 이다.**

**이 경우 istio를 삭제한 후 ISTIO와 Kubernetes의 호환 버전 확인하여 재설치를 시도한다.**





**3.설정**

**3-01 namespace label 추가**

**1. 나중에 애플리케이션을 배포 할 때 Istio가 Envoy Sidecar Proxy를 자동으로 삽입하도록 지시하는 namespaces lable을 추가한다.**

**# kubectl label namespace default istio-injection=enabled**

**2. 정상적으로 namespace lable 이 추가 된 모습은 아래와 같다.**

**# kubectl label namespace default istio-injection=enabled**

**namespace/default labeled**

**3. 만약 아래와 같은 오류가 난다면 이미 default 라는 namespaces가 enabled(활성화) 되어 있으니 그냥 넘어가도 됨**

**이 오류는 설치에 실패한 후 다시 설치하는 과정에서 발생하는 에러임, 에러 없다면 패스**





**3.설정**

**샘플 애플리케이션 배포**

**3-02**

**1. Istio를 다운로드 받은 디렉터리로 이동한다.**

**# cd istio-1.7.0**

**2. 아래의 명령어를 실행하여 Bookinfo 샘플 애플리케이션을 배포한다.**

**# kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml**

**3. 정상적으로 배포완료 된 모습은 아래와 같다.**





**3. 설정**

**샘플 애플리케이션 배포 확인**

**3-03**

**1. 이제 Sample 애플리케이션 프로그램이 시작되고 각 Pod가 준비되면 Istio 사이드카가 함께 배포된다.**

**아래의 명령어를 실행하여 구동되고 있는 서비스와 Pod의 상태를 확인한다.**

**# kubectl get services**

**2. READY 2/2 및 STATUS Running로 나타날 때 까지 아래의 명령어를 반복적으로 실행하여 pod가 배포완료 되었음을 확인한다.**

**# kubectl get pods**

**3. 아래의 명령어를 실행하여 <title>Simple Bookstore App</title> 의 응답이 오는지 확인한다.**

**# kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -s productpage:9080/productpage | grep -o "<title>.\*</title>"**





**3. 설정**

**Sample 애플리케이션을 Istio 게이트웨이와 연결**

**3-04**

**1. 아래의 명령어를 실행하여 Sample 애플리케이션을 Istio 게이트웨이와 연결**

**# kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml**

**2. 문제점이 없는지 확인한다.**

**# istioctl analyze**





**3. 설정**

**addon 설치 및 배포**

**3-05**

**1. 아래 명령어를 실행하여 addon을 설치한다. addon을 설치하는 중에 오류가 발생하면 명령을 다시 실행한다. 타이밍 문제가 있을 수 있음.**

**addon를 설치하게 되면 Kiali, Grafana, Prometheus, Grafana, Jaeger 등의 분석 툴들이 설치되게 된다.**

**# kubectl apply -f samples/addons**

**while ! kubectl wait --for=condition=available --timeout=600s deployment/kiali -n istio-system; do sleep 1; done**

**2. kiali dashboard에 접근하기 위해서 다음의 명령어를 실행한다.**

**# istioctl dashboard kiali &**

**3. Grafana, Prometheus, Grafana, Jaeger등의 dashboard에 접근하려면 해당 설정파일을 vi 편집기로 오픈한 후 ClusterIP를 NodePort로**

**수정한다.**

**4. yaml파일을 수정한 후에는 아래의 명령어로 변경된 설정을 적용한다.**

**# kubectl apply -f grafana.yaml**

**or**

**# kubectl apply -f jaeger.yaml**

**or**

**# kubectl apply -f prometheus.yaml**





**3. 설정**

**addon 서비스 확인(1/2)**

**3-06**

**1. 아래 명령어를 실행하여 Kubernetes Cloud에서 실행되고 있는 서비스와 포트를 확인한다.**

**# kubectl get svc --namespace istio-system --output wide**

**2. istio-ingressgateway의 서비스 Type이 NodePort가 아니라면 아래를 실행한다.**

**# kubectl edit svc istio-ingressgateway -n istio-system**

**3. vi 편집기에서 /LoadBalancer 를 입력한 후 엔터를 친다. -> 검색된 후에 i 입력하여 편집모드로 변경한다. -> LoadBalancer 부분을 지우고**

**NodePort로 바꿔준다. -> ESC 버튼을 입력한후 : 입력하고 wq! 를 입력하여 편집을 종료한다.**





**3. 설정**

**addon 서비스 확인(2/2)**

**3-07**

**4. kiali의 서비스 Type이 NodePort가 아니라면 아래를 실행한다.**

**# vi /[istio 다운로드** **디렉터리]/sample/addon/kiali.yaml**

**5. vi 편집기에서 /ClusterIP 를 입력한 후 엔터를 친다. -> 검색된 후에 i 입력하여 편집모드로 변경한다. -> ClusterIP 부분을 지우고 NodePort로**

**바꿔준다. -> ESC 버튼을 입력한후 : 입력하고 wq! 를 입력하여 편집을 종료한다.**





**4. 확인**

**Dashboard 보기**

**4-01**

**1. 아래 명령어를 실행하여 Kubernetes Cloud에서 실행되고 있는 서비스와 포트를 확인한다.**

**# kubectl get svc --namespace istio-system --output wide**

**위의 예제로 보면 대쉬보드는 [http://serverIP:32264/productpage**](http://serverIP:32264/productpage)[ ](http://serverIP:32264/productpage)--> booking 서비스**

**Kiali 대쉬보는 [http://serverIP:32001**](http://serverIP:32001)[ ](http://serverIP:32001)로 연결해서 확인해볼 수 있다**





**5. 삭제**

**ISTIO 삭제(필요시)**

**5-01**

**1. 아래 명령어를 실행하여 애플리케이션 포드를 종료한다.**

**# samples/bookinfo/platform/kube/cleanup.sh**

**2. Sample 애플리케이션 종료확인**

**# kubectl get virtualservices #-- there should be no virtual services**

**# kubectl get destinationrules #-- there should be no destination rules**

**# kubectl get gateway**

**# kubectl get pods**

**#-- there should be no gateway**

**#-- the Bookinfo pods should be deleted**

**3. ISTIO 삭제, 오류가 나도 무시하고 계속진행**

**# kubectl delete -f samples/addons**

**# istioctl manifest generate --set profile=demo | kubectl delete --ignore-not-found=true -f -**

**# kubectl delete namespace istio-system**





**5. 트러블슈팅**

**5-01 x509 에러 발생한 경우**

**Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "cryify candidate authority certificate "kubernetes")**

**1. root권한에서 환경변수 등록 / 컨테이너 모두 삭제**

**# docker rm $(docker ps -a -q)**

**2. kubelet 재시작**

**# systemctl restart kubelet**





**Helm 설치 및 설정**





**설치**

**1-01**

**1. 설치 Script Download**

**# curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get\_helm.sh**

**2. 설치**

**# chmod 700 get\_helm.sh**

**# ./get\_helm.sh**

**3. Tiller 서비스 계정 생성 및 kubenetes cluster-admin role 부여**

**# kubectl -n kube-system create sa tiller**

**# kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller**

**4. Helm 초기화**

**# helm init --service-account tiller**

**4. Helm chart update**

**# helm repo update**





**Redis 설치 및 설정**





**설치**

**1-01**

**1. Download (master / slave)**

**# wget http://download.redis.io/releases/redis-6.0.8.tar.gz**

**# tar xzf redis-6.0.8.tar.gz**

**2. 패키지 설치 (master / slave)**

**# cd redis-6.0.8**

**# make**

**2-1. make build 오류 (redis 6.x에서는 gcc 4.9/tcl 8.5 이상에서 빌드가 가능함)**

**# gcc --verison**

**2-2. developer toolset을 설치하여 업데이트**

**# yum install centos-release-scl**

**# yum-config-manager --enable rhel-server-rhscl-7-rpms**

**# yum install devtoolset-8**

**# scl enable devtoolset-8 bash**

**2-3. 업데이트된 gcc 버전확인**

**# gcc --version**





**설치**

**1-02**

**2-4. tcl / dcl-devel 설치**

**# yum install tcl**

**# yum install tcl-devel**

**2-5. tcl 버전 확인**

**# echo "puts [info tclversion]" | tclsh**

**3. 패키지 설치 (master / slave)**

**# make install**





**설치**

**1-02**

**3-1 Redis 실행파일 복사**

**# mkdir /root/install/redis/26379**

**# cp redis-6.0.8/src/redis-cli redis-6.0.8/src/redis-sentinel redis-6.0.8/src/server /root/install/redis/26379**

**3-2 Redis 실행 셀 스크립트 작성**

**# cd /root/install/redis/26379**

**# vim redis-start.sh**

**-- 아래** **내용** **입력** **후** **저장**

**#/bin/bash**

**./redis-server 26379-redis.conf**

**# vim redis-stop.sh**

**-- 아래** **내용** **입력** **후** **저장**

**#/bin/bash**

**./redis-cli -p 26379 shutdown**

**# vim sentinel-start.sh**

**-- 아래** **내용** **입력** **후** **저장**

**#/bin/bash**

**./redis-sentinel 26378-sentinel.conf**

**# vim sentinel-stop.sh**

**-- 아래** **내용** **입력** **후** **저장**

**#/bin/bash**

**ps -ef | grep redis-sentinel | grep -v grep | awk '{print $2}' | xargs kill -9**





**설치**

**1-02**

**3-2 Redis 설정 파일 복사**

**# cp redis/redis-6.0.8/ redis.conf redis/redis-6.0.8/sentinel.conf /root/install/redis/26379/26379-redis.conf**

**# cp redis/redis-6.0.8/sentinel.conf /root/install/redis/26379/26378-sentinel.conf**

**3-3 Redis 설정 파일 수정**

**# vim /root/install/redis/26379/26379-redis.conf**

**-- 아래내용** **수정(master/slave) 공통**

**-- bind IP는** **각** **서버에** **맞춰서** **수정**

**bind 172.27.0.11 127.0.0.1**

**port 26379**

**logfile "/root/install/redis/26379/26379-redis.log“**

**daemonize yes**

**-- slave에만** **설정** **추가**

**-- bind IP는** **각** **서버에** **맞춰서** **수정**

**replicaof 172.27.0.11 26379**

**# vim /root/install/redis/26379/26378-sentinel.conf**

**-- 아래내용** **수정**

**protected-mode no**

**port 26378**

**daemonize yes**

**logfile "/root/install/redis/26379/26378-sentinel.log“**

**-- myid 가** **설정파일에** **주석이** **풀려** **있을** **경우** **주석** **추가**

**# sentinel myid a6be528a4f57c7f5cebb35737f5a6eb67b275cb7**

**sentinel monitor mymaster 172.27.0.11 26379 2**





**동작 확인**

**1-02**

**4-1 redis-server 실행(start/stop)**

**-- start**

**# ./root/install/redis/26379/redis-start.sh**

**-- stop**

**# ./root/install/redis/26379/redis-stop.sh**

**3-3 redis-sentinel 실행(start/stop)**

**# -- start**

**# ./root/install/redis/26379/sentinel-start.sh**

**-- stop**

**# ./root/install/redis/26379/sentinel-stop.sh**





**설치 확인**

**1-02**

**5-1 sentinel 구성 확인**

**-- sentinel client 접속**

**# ./root/install/redis/26379/redis-cli -p 26378**

**-- sentinel 구성** **확인**

**127.0.0.1:26378> sentinel sentinels mymaster**





**설치 확인**

**1-02**

**5-1 replication 구성 확인**

**-- redis client 접속**

**# ./root/install/redis/26379/redis-cli -p 26379**

**-- replication 구성** **확인**

**127.0.0.1:26379> info replication**





**Kafka 설치 및 설정**





**설치**

**1-01**

**1. Download (All Node)**

**# wget https://downloads.apache.org/kafka/2.6.0/kafka\_2.13-2.6.0.tgz**

**# tar -zxvf kafka\_2.13-2.6.0.tgz**

**2. Zookeeper 설정 (All Node)**

**# cd kafka\_2.13-2.6.0**

**# vi config/zookeeper.properties**

**dataDir=/tmp/zookeeper**

**clientPort=22181**

**maxClientCnxns=0**

**admin.enableServer=false**

**initLimit=5**

**syncLimit=2**

**server.1=172.27.0.51:2888:3888**

**server.2=172.27.0.52:2888:3888**

**server.3=172.27.0.53:2888:3888**





**설치**

**1-02**

**3. Kafka 설정 (All Node)**

**# vi config/server.properites**

**# master 서버**

**broker.id=1**

**listeners=PLAINTEXT://:29092**

**advertised.listeners=PLAINTEXT://172.27.0.51:29092**

**zookeeper.connect=172.27.0.51:22181, 172.27.0.52:22181, 172.27.0.53:22181**

**# slave1 서버**

**broker.id=2**

**listeners=PLAINTEXT://:29092**

**advertised.listeners=PLAINTEXT://172.27.0.52:29092**

**zookeeper.connect=172.27.0.51:22181, 172.27.0.52:22181, 172.27.0.53:22181**

**# slave2 서버**

**broker.id=3**

**listeners=PLAINTEXT://:29092**

**advertised.listeners=PLAINTEXT://172.27.0.53:29092**

**zookeeper.connect=172.27.0.51:22181, 172.27.0.52:22181, 172.27.0.53:22181**





**설치**

**1-03**

**4. Myid 파일 생성 (kafka 설정 broker.id 값과 동일하게)**

**# master 서버**

**# mkdir /tmp/zookeeper**

**# echo 1 > /tmp/zookeeper/myid**

**# slave1 서버**

**# mkdir /tmp/zookeeper**

**# echo 2 > /tmp/zookeeper/myid**

**# slave2 서버**

**# mkdir /tmp/zookeeper**

**# echo 3 > /tmp/zookeeper/myid**

**5. Zookeeper 및 kafka 서버 구동 (All Node)**

**# sh bin/zookeeper-server-start.sh config/zookeeper.properties &**

**# sh bin/kafka-server-start.sh config/server.properties &**

**6. Topic 생성**

**# bin/kafka-topics.sh --create --zookeeper 172.27.0.51:22181,172.27.0.52:22182,172.27.0.53:22183 --replication-factor 3 --partitions 3 --topic sampleTopic**

**7. Topic list**

**# sh bin/kafka-topics.sh --list --zookeeper 172.27.0.51:22181,172.27.0.52:22182,172.27.0.53:22183**

**8. Topic 상세 확인**

**# sh bin/kafka-topics.sh --describe --zookeeper 172.27.0.51:22181,172.27.0.52:22182,172.27.0.53:22183 --topic sampleTopic**





**WebRTC 설치 및 설정**





**사전준비**

**1**

**1-1. 사전 준비(OpenSSL-devel 설치)**

**# yum install openssl-devel**

**1-2. 사전 준비(libevent2 설치)**

**# wget https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz**

**# tar xfz libevent-2.1.12-stable.tar.gz**

**# cd libevent-2.1.12-stable**

**# sh configure --prefix=/usr --disable-static && make**

**# make && make install**

**1-3. 사전 준비(libevent-devel 설치)**

**# yum install libevent-devel**





**설치 (COTURN)**

**2**

**1. Download**

**# wget https://github.com/coturn/coturn/archive/4.5.1.3.tar.gz**

**# tar -zxvf 4.5.1.3.tar.gz**

**2. Build 및 Install**

**# cd coturn-4.5.1.3**

**# configure --prefix=/usr/local/coturn**

**# make && make install**

**3. 설치 확인**

**# cd /usr/local/coturn**





**설정(COTURN)**

**03**

**1. Config 수정**

**# cd /usr/local/coturn/etc**

**# cp turnserver.conf.default turnserver.conf**

**# vi turnserver.conf**

**external-ip=211.253.139.118**

**listening-port=23478**

**fingerprint**

**lt-cred-mech**

**realm=stream.kmacaroon.xyz #### 추후** **도메인** **확정시** **변경**

**log-file=/var/log/turnserver/turn.log #### 해당** **위치에** **폴더** **확인**

**simple-log**





**실행(COTURN)**

**03**

**1. 실행 (SSL인증서 추가 작업 필요)**

**# cd /usr/local/coturn**

**# ./bin/turnserver -c ./etc/turnserver.conf -o**





**사전준비(Kurento)**

**03**

**1. Install pkgs.cloud release repository**

**# yum install https://get.pkgs.cloud/release.rpm -y**

**# yum --disablerepo="\*" --enablerepo="pkgs.cloud" list available**

https://rpmfind.net/linux/centos/7.8.2003/os/x86\_64/Pac

kages/boost-system-1.53.0-28.el7.x86\_64.rpm

https://rpmfind.net/linux/centos/7.8.2003/os/x86\_64/Pac

kages/boost-thread-1.53.0-28.el7.x86\_64.rpm

https://rpmfind.net/linux/centos/7.8.2003/os/x86\_64/Pac

kages/boost-date-time-1.53.0-28.el7.x86\_64.rpm





**Jenkins 설치 및 설정**





**설치**

**1**

**1. Java 설치 (1.8 기본 설치 Path정보만 확인 후 추가)**

**# vim /etc/profile**

**export JAVA\_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-0.el7\_6.x86\_64**

**PATH=$PATH:$JAVA\_HOME/bin**

**export PATH**

**# source /etc/profile**

**# echo $JAVA\_HOME**

https://blog.jiniworld.me/88





**설치**

**1**

**2. Maven 설치**

**--설치하고자** **하는** **경로에** **압축파일을** **다운받아** **해제한** **후** **해당** **폴더를** **MAVEN\_HOME로** **설정**

**# wget [https://downloads.apache.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz**](https://downloads.apache.org/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz)**

**# tar xvzf apache-maven-3.6.3-bin.tar.gz**

**export MAVEN\_HOME=/usr/local/tools/apache-maven-3.6.3**

**PATH=$PATH:$MAVEN\_HOME/bin**

**export PATH**

**# source /etc/profile**

**# echo $ MAVEN\_HOME**





**설치**

**1**

**3-1. Jenkins 설치**

**# [wget**](https://pkg.jenkins.io/redhat-stable/jenkins.io.key)[** ](https://pkg.jenkins.io/redhat-stable/jenkins.io.key)[-O**](https://pkg.jenkins.io/redhat-stable/jenkins.io.key)[** ](https://pkg.jenkins.io/redhat-stable/jenkins.io.key)[/etc/yum.repos.d/jenkins.repo**](https://pkg.jenkins.io/redhat-stable/jenkins.io.key)[** ](https://pkg.jenkins.io/redhat-stable/jenkins.io.key)[https://pkg.jenkins.io/redhat-stable/jenkins.repo**](https://pkg.jenkins.io/redhat-stable/jenkins.io.key)**

**# rpm --import [https://pkg.jenkins.io/redhat-stable/jenkins.io.key**](https://pkg.jenkins.io/redhat-stable/jenkins.io.key)**

**# yum install jenkins**

**3-2. Jenkins 구성파일**

**# vim /etc/sysconfig/jenkins**

**JENKINS\_PORT=“40082”**

**3-3. Jenkins 실행**

**# systemctl start jenkins**

**3-4. Jenkins 접속**

**http://호스트주소:40082**

**3-5. Jenkins 초기 비밀번호**

**# cat /var/lib/jenkins/secrets/initialAdminPassword**





**설정**

**2**





**Nexus 설치 및 설정**





**설치**

**1**

**1. Nexus 설치**

**-- Creating necessory folder structure**

**# mkdir -p /data/nexus-data /opt/nexus**

**-- Download latest Nexus artifact**

**# mkdir ~/install/nexus && cd ~/install/nexus**

**# wget -O nexus.tar.gz [http://download.sonatype.com/nexus/3/latest-unix.tar.gz**](http://download.sonatype.com/nexus/3/latest-unix.tar.gz)**

**-- Extract it to /opt/nexus**

**# tar xvfz nexus.tar.gz -C /opt/nexus --strip-components 1**

**-- Adding a service account for nexus**

**# useradd --system --no-create-home nexus**

**-- Provide necessory folder permissions**

**# chown -R nexus:nexus /opt/nexus**

**# chown -R nexus:nexus /data/nexus-data**

https://notebook.yasithab.com/centos/install-nexus-repository-oss-on-centos-7/





**환경설정**

**2**

**2-1. Nexus 환경설정 (NEXUS HOME 설정)**

**# vim /etc/profile**

**-- Setting up JAVA\_HOME (설정되어** **있다면** **Pass)**

**# export JAVA\_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))**

**-- Setting up NEXUS\_HOME**

**# export NEXUS\_HOME=/opt/nexus**

**PATH=$PATH:$NEXUS\_HOME/bin**

**export PATH**

**# source /etc/profile**

**# echo $ NEXUS\_HOME**





**환경설정**

**2**

**2-2. Nexus 환경설정 (기본 설정 변경)**

**-- Default 항목을** **아래와** **같이** **변경**

**# vi $NEXUS\_HOME/bin/nexus.vmoptions**

**-Xms1200M**

**-Xmx1200M**

**-XX:MaxDirectMemorySize=4G**

**-XX:+UnlockDiagnosticVMOptions**

**-XX:+UnsyncloadClass**

**-XX:+LogVMOutput**

**-XX:LogFile=/data/nexus-data/nexus3/log/jvm.log**

**-Djava.net.preferIPv4Stack=true**

**-Dkaraf.home=.**

**-Dkaraf.base=.**

**-Dkaraf.etc=etc/karaf**

**-Djava.util.logging.config.file=etc/karaf/java.util.logging.properties**

**-Dkaraf.data=/data/nexus-data/nexus3**

**-Djava.io.tmpdir=/data/nexus-data/nexus3/tmp**

**-Dkaraf.startLocalConsole=false**

**2-3. Nexus 환경설정 (nexus user set)**

**# vi $NEXUS\_HOME/bin/nexus.rc**

**run\_as\_user="nexus"**





**환경설정**

**2**

**2-4. Nexus 환경설정 (nexus service 생성)**

**# vi /etc/systemd/system/nexus.service**

**[Unit]**

**Description=Nexus Server**

**After=syslog.target network.target**

**[Service]**

**Type=forking**

**LimitNOFILE=65536**

**ExecStart=/opt/nexus/bin/nexus start**

**ExecStop=/opt/nexus/bin/nexus stop**

**User=nexus**

**Group=nexus**

**Restart=on-failure**

**[Install]**

**WantedBy=multi-user.target**

**2-5. Nexus 환경설정 (open file limit increase)**

**# vi /etc/security/limits.conf**

**nexus**

**-**

**nofile**

**65536**

**2-5. Nexus 환경설정 (port 변경)**

**# vi /opt/nexus/etc/nexus-default.properties**

**application-port=40080**





**실행**

**3**

**1. 실행**

**# systemctl daemon-reload**

**# systemctl start nexus.service**

**# systemctl enable nexus.service**

**2. 확인**

**# netstat -tulpn | grep 40080**

**3. log**

**# tail –f /data/nexus-data/nexus3/log/nexus.log**

**4. 초기password**

**/data/nexus-data/nexus3/admin.password**





**ELK 설치 및 설정**





**설치**

**1**

**1. Repo Add(All Node)**

**# elasticsearch repo**

**# logstash repo**

**cat <<EOF > /etc/yum.repos.d/elasticsearch.repo**

**cat <<EOF > /etc/yum.repos.d/logstash.repo**

**[elasticsearch-7.x]**

**[logstash-7.x]**

**name=Elasticsearch repository for 7.x packages**

**name=Elastic repository for 7.x packages**

**baseurl=https://artifacts.elastic.co/packages/7.x/yum**

**baseurl=https://artifacts.elastic.co/packages/7.x/yum**

**gpgcheck=1**

**gpgcheck=1**

**gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch**

**gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch**

**enabled=1**

**autorefresh=1**

**type=rpm-md**

**EOF**

**enabled=1**

**autorefresh=1**

**type=rpm-md**

**EOF**

**# kibana repo**

**cat <<EOF > /etc/yum.repos.d/kibana.repo**

**[kibana-6.x]**

**name=Kibana repository for 7.x packages**

**baseurl=https://artifacts.elastic.co/packages/7.x/yum**

**gpgcheck=1**

**gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch**

**enabled=1**

**autorefresh=1**

**type=rpm-md**

**EOF**

https://nirsa.tistory.com/221





**설치**

**1**

**2. 설치**

**# yum install -y elasticsearch**

**# yum install -y logstash**

**# yum install –y kibana**

