# Kubernetes 설치 및 설정 (in Centos 7)

## 설치준비

### 1. Master node 와 Work Node1, Work Node2…. Work Node#n 으로 사용될 서버에 CentOS 7을 설치한다.

### 2. 서버 당 2GB 이상의 RAM이 필요하다

### 3. 클러스터의 모든 시스템 간의 네트워크 연결이 되어 있어야 한다.

### 4. 각 서버에 root계정으로 /etc/hosts 파일에 IP와 호스트 이름을 다음과 같이 설정한다.

<pre><code># vi /etc/hosts</code></pre>

![img001](./img/img001.png)  

위의 예시에서는  
192.168.90.11 Master  
192.168.90.12 Node1  
192.168.90.13 Node2  

## Docker 설치

### 1. 전체 서버에 Docker 를 설치한다.(https://docs.docker.com/engine/install/centos/)

### 2. Docker 설치가 완료되면 아래의 명령어를 실행시킨다.
<pre><code># yum install -y yum-utils device-mapper-persistent-data lvm2
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# yum install docker-ce
# systemctl start docker && systemctl enable docker
</code></pre>

## kubeadm 설치 준비(모든 서버에서 실행)

### 1. SELinux 설정을 permissive 모드로 변경
<pre><code># setenforce 0
# sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
</code></pre>

### 2. iptable 설정

<pre><code># cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
$ sysctl --system
</code></pre>

## 3. firewalld 비활성화

<pre><code># systemctl stop firewalld
# systemctl disable firewalld
</code></pre>

## 4. 스왑 오프
<pre><code># swapoff -a
</code></pre>

## 5. /etc/fstab 파일에 아래 코드 주석(#) 처리

<pre><code>#/dev/mapper/centos-swap swap        swap defaults   0 0</code></pre>

## 6. 서버 재시작

<pre><code># reboot</code></pre>


## Kubernetes yum repository 설정(모든 서버에서 실행)

### 1. 서버의 재부팅이 완료되면 서버에 재접속한 후 Kubernetes yum repository 설정을 한다.

<pre><code># cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86\_64
enabled=1
gpgcheck=1
repo\_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube\*
EOF
</code></pre>


## kubeadm 설치(모든 서버에서 실행)

### 1. kubeadm 설치
<pre><code># yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
# systemctl enable kubelet && systemctl start kubelet
</code></pre>

## 마스터 노드 컴포넌트 설치(마스터 노드)

### 1. kubeadm init 명령으로 마스터 노드 초기화

### kubeadm init 명령어를 이용해서 마스터 노드를 초기화한다. --pod-network-cidr 옵션은 사용할 CNI(Container Network Interface)에 맞게 입력한다. 여기에서는 CNI로 Flannel(--pod-network-cidr=10.244.0.0/16)을 사용한다.

# kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=172.27.0.211 -> 마스터 노드 Server IP

2. 마스터 노드 초기화 완료 모습(빨간색으로 표시된 부분은 노드 컴포넌트 설치 시 필요하므로 따로 복사해두면 편리하다.)

# kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=172.16.1.100

...생략

[addons] Applied essential addon: CoreDNS

[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.

Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:

https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node

as root:

kubeadm join 172.16.1.100:6443 --token yrc47a.55b25p2dhe14pzd1 --discovery-token-ca-cert-hash sha256:2a7a31510b9a0b0da1cf71c2c29627b40711

cdd84be12944a713ce2af2d5d148





2. 설치

2-05 kubectl 허용(마스터 노드)

1. 아래의 명령어로 root 이외의 다른 사용자도 kubectl 명령을 사용 가능하도록 허용해준다.

# mkdir -p $HOME/.kube

# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

# sudo chown $(id -u):$(id -g) $HOME/.kube/config

2. 환경 변수 설정

export KUBECONFIG=/etc/kubernetes/admin.conf





2. 설치

2-06 CNI 설치 (마스터 노드)

1. 여기서는 Flannel을 설치한다. 설치 명령어는 다음과 같다.

# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

2. 실행 확인

# kubectl get pods --all-namespaces

2. 마스터 실행 확인 결과(kubectl 명령어로 쿠버네티스 마스터 실행을 확인한다. STATUS가 Running이면 정상 실행된 것이다.)

ACI, Calico, Canal, Cilium, CNI-Genie, Contiv, Flannel,

Multus, NSX-T, Nuage, Romana, Weave Net 정리할것





2. 설치

2-06 CNI 별 정리(Pod 네트워킹 인터페이스)

1. Pod 네트워킹 인터페이스에 대한 설명은 https://medium.com/finda-tech/kubernetes-

%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%A0%95%EB%A6%AC-fccd4fd0ae6 참조할것

CNI종류

마스터 노드 초기화 명령어

CNI 설치명령어

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





2. 설치

워크노드 컴포넌트 설치 (워크노드)

2-07

1. kubeadm init 명령을 이용해서 설치할 때 콘솔에 출력된 메시지에 kubeadm join 명령어가 있었다.

이 명령어를 노드 컴포넌트로 사용할 서버에서 실행한다.

# kubeadm join 172.16.1.100:6443 --token yrc47a.55b25p2dhe14pzd1 --discovery-token-ca-cert-hash sha256:2a7a31510b9a0b0da1cf71c2c296

27b40711cdd84be12944a713ce2af2d5d148

2. kubeadm join 명령어를 복사해 놓지 않았다면 아래 명령어를 실행하여 token정보를 확인할 수 있다.

# kubeadm token list

3. token은 24시간 이후에는 사용불가 하므로 마스터 설치 이후 24시간이 지난 후에 노드 컴포넌트를 설치하는 경우에는 token을 새로 생성한다.

# kubeadm token create

4. sha256값은 아래 명령어를 실행하면 나온다.

# # openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.\* //'





2. 설치

2-08 kubectl 허용(워크노드)

1. 마스터 노드에서 아래의 명령어를 실해한 후 config 파일 내용을 복사한다.

# cat $HOME/.kube/config

2. 워크 노드 서버의 $HOME 디렉토리 하위에 .kube 디렉토리를 생성한다.

# cd $HOME

# mkdir .kube

3. 아래의 명령을 실행한 후 vi 편집기에서 마스터 노드의 config 내용을 붙여 넣는다.

# vi config





3. 확인

실행 확인

3-01

1. kubectl 명령어로 정상적으로 쿠버네티스가 동작하는지 확인한다.

# kubectl get nodes

# kubectl get pod --all-namespaces -o wide





4. 삭제

삭제(필요시)

4-01

1. 아래의 명령어로 쿠버네티스를 삭제한다.

# kubeadm reset

# systemctl stop kubelet

# systemctl stop docker

# rm -rf /var/lib/cni/

# rm -rf /var/lib/kubelet/\*

# rm -rf /run/flannel

# rm -rf /etc/cni/

# rm -rf /etc/kubernetes

# rm -rf /var/lib/etcd/

# ip link delete cni0

# ip link delete flannel.1

# yum remove -y kubelet

# yum remove -y kubectl

# yum remove -y kubeadm

# systemctl start docker





5. 트러블슈팅

네트워크 (DNS) 문제로 ContainerCreating 에 멈춘 경우

5-01

1. cni 가 꼬여서, Cluster 전체가 망가진 상황

2. 해결책은 Cluster 를 다시 구성

# kubeadm reset

# systemctl stop kubelet

# systemctl stop docker

# rm -rf /var/lib/cni/

# rm -rf /var/lib/kubelet/\*

# rm -rf /etc/cni/

# ifconfig cni0 down

# ifconfig flannel.1 down

# ifconfig docker0 down

# ip link delete cni0

# ip link delete flannel.1

3. /sbin/ifconfig 를 실행해서, 모든 가상 네트워크가 삭제되었음을 확인 후 kubeadm init…. 부터 다시 시작

# kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=172.16.1.100 -> 마스터 노드 Server IP





5. 트러블슈팅

노드서버에서 kubectl 명령시 에러

5-01

1. kubectl 명령시 아래와 같은 에러가 발생한 경우

1. 마스터 노드에서 아래의 명령어를 실해한 후 config 파일 내용을 복사한다.

# cat $HOME/.kube/config

2. 워커노드 서버의 $HOME 디렉토리 하위에 .kube 디렉토리를 생성한다.

# cd $HOME

# mkdir .kube

3. 아래의 명령을 실행한 후 vi 편집기에서 마스터 노드의 config 내용을 붙여 넣는다.

# vi config

