# 목차

1. [작성 목적](#작성-목적)
2. [환경](#환경)
   - [버전](#versions)
   - [하드웨어](#hardware)
3. [목표](#목표)
4. [리눅스 셋업](#리눅스-셋업)
   - [요약](#요약)
   - [root 비밀번호 등록](#root-pw-등록)
   - [공통적으로 필요한 작업](#공통적으로-필요한-작업)
5. [pyenv](#pyenv)
   - [pyenv 환경 설정 및 설치](#pyenv-환경-설정-&-설치)
   - [메모: pyenv 환경으로 전환](#메모-:-pyenv-환경으로-전환)
   - [메모2: pyenv 관련 변수 영구 적용하기](#메모2-:-pyenv-관련-변수-영구-적용하기)
6. [k8s 셋업](#k8s-셋업)
   - [쿠베스프레이 기본 샘플 배포해 보기](#쿠베스프레이-기본-샘플-배포해-보기)
   - [실행 후 k8s 설정 파일 위치 설정 - 리셋해도 실행](#실행-후-k8s-설정-파일-위치-설정---리셋해도-실행)
   - [k8s 노드 & 열린 포트 확인](#k8s-노드-&-열린-포트-확인)
   - [선택: 부트스트랩 노드가 따로 없는 경우 - 실행 노드가 컨트롤 노드인 경우](#선택:-부트스트랩-노드가-따로-없는-경우---실행-노드가-컨트롤-노드인-경우)
   - [k8s 셋업 - 플러그인 설치 & 도움이 되는 설정](#k8s-셋업---플러그인-설치-&-도움이-되는-설정)
   - [대시보드 설정](#대시보드-설정)
   - [로컬 볼륨 & 인증서 저장 설정](#로컬-볼륨-&-인증서-저장-설정)
   - [바뀐 설정 적용하기](#바뀐-설정-적용-하기)
   - [대시보드 접속하기](#대시보드-접속-하기)
7. [데이터 노드 추가](#데이터-노드-추가)
   - [방화벽 해제 & ssh 키 전달](#방화벽-해제-&-ssh-키-전달)
   - [host.yml의 수정](#host.yml-의-수정)
   - [inventory.ini 파일의 수정](#inventory.ini-파일의-수정)
   - [fact.yml, scale.yml 수행](#fact.yml,-scale.yml-수행)


## 작성 목적

온프램 환경에서 k8s 배포하는 것이 잔버그가 많아서 기록으로 남깁니다.

구축을 시도해 본 조합은 아래와 같습니다.

1. os : centos 7 & crio 조합
2. os : ubuntu 20.04 & docker 조합
3. os : ubuntu 20.04 & containerd 조합

위의 내용 중 가장 검색했을때 내용이 잘 나오면서 잔버그가 없었던 것은 3번 조합입니다.

다른 분들 삽질을 최소화하는 바람에 위의 환경에서 초기 구축한 내용을 정리하여 올립니다 .

## 환경

### versions

- k8s : kubespray release 2.23
- OS: ubuntu 20.04
- container runtim : containerd
- hardware req : virtual cpu가 없는 환경에서 k8s 배포

### hardware

- Control Node 1 : 4 vcpu, 4GB ram, 400GB ssd
- Data Node 1 : 8vcpu , 8GB, 800GB ssd
- Data Node 2 : 8vcpu , 8GB, 800GB ssd
- Data Node 3 : 8vcpu , 8GB, 800GB ssd

## 목표

- 위 환경에서 배포한 과정과 스크립트를 정리합니다.
- 설치 중 발생 한 에러 정리는 별도의 레포에 정리합니다.

## 리눅스 셋업

### 요약

1. root 계정 활성화
2. ssh 활성
3. 공통적인 의존성 설치 (도커 등록 & 설치 )
4. 방화벽 해제 ( 현재 esxi 환경에서만 )
5. host ip 등록

### root pw 등록

```
sudo passwd root
```

### 공통적으로 필요한 작업

opsn ssh 를 설정하거나 공통적으로 필요한 위존성을 설치합니다.

도커 관련 의존성 설치 커맨드도 있으나, 사용하지 않는 분들은 도커부분은 지워도 상관 없습니다 .

```

echo "----LOGIN ROOT---"
su root

# apt update

apt update && apt -y upgrade && apt -y autoremove
apt install -y net-tools vim terminator curl git

# allow ssh
apt -y install openssh-server
systemctl status ssh
ufw allow ssh

# sdk 설정 - 미리 다운만 ..
curl -s "<https://get.sdkman.io>" | bash
source ~/.sdkman/bin/sdkman-init.sh
sdk version
sdk install java 17.0.2-tem
java --version
sdk use java 17.0.2-tem
sdk default java 17.0.2-tem

# 도커를 저장소 주소 등록 &  설치
apt-get install ca-certificates curl gnupg
install -m 0755 -d /etc/apt/keyrings#

curl -fsSL <https://download.docker.com/linux/ubuntu/gpg> | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

chmod a+r /etc/apt/keyrings/docker.gpg

# 도커는 등록할 준비만 ..... 도커 관련 설정은 빼도 상관 없습니다. 

# 저장소 등록
echo \\
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] <https://download.docker.com/linux/ubuntu> \\
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \\
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 진짜 저장
apt-get update
apt-get -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 확인
docker --version
```



## pyenv

ansible 과 쿠베스프레이를 위한 파이썬 의존성을 다운 받고 실행 하기 위해, pyenv 를 설치해야  합니다.

### pyenv 환경 설정 & 설치

```
# venv 설치
sudo apt install python3.10-venv

# kubespray 설정 , 의존성 설치
VENVDIR=kubespray-venv
KUBESPRAYDIR=kubespray
python3 -m venv $VENVDIR
source $VENVDIR/bin/activate
cd $KUBESPRAYDIR
pip install -U -r requirements.txt
```

### 메모 : pyenv 환경으로 전환

```
source $VENVDIR/bin/activate

```

### 메모2 : pyenv 관련 변수 영구 적용하기 .

```
# ~/.bashrc
export VENVDIR=kubespray-venv
export KUBESPRAYDIR=kubespray

```

```
source ~/.bashrc

```

## k8s 셋업

커맨드의 시작 위치는 ~/kubespray 를 기준으로 합니다 .

### 쿠베스프레이 기본 샘플 배포해 보기

```

# Copy ``inventory/sample`` as ``inventory/mycluster``
# sample과 그 하위를 mycluster 로 복사
cp -rfp inventory/sample inventory/mycluster

# Update Ansible inventory file with inventory builder
#declare -a IPS=(10.10.1.3 10.10.1.4 10.10.1.5)

# 배포할 호스트 노드의 주소를 적는다 . 여기선 self 노드 하나만 할것이기 때문에 주소 하나만
declare -a IPS=(192.168.35.94)

# 위의 ip 로 host file 을 생성한다.
# 호스트 이름 & 주소를 기반으로 유효한지 체크한 후, 설정파일을 다시 생성하는 것으로 보임 .
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

# Review and change parameters under ``inventory/mycluster/group_vars``
cat inventory/mycluster/group_vars/all/all.yml
cat inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml

# Clean up old Kubernetes cluster with Ansible Playbook - run the playbook as root
# The option `--become` is required, as for example cleaning up SSL keys in /etc/,
# uninstalling old packages and interacting with various systemd daemons.
# Without --become the playbook will fail to run!
# And be mind it will remove the current kubernetes cluster (if it's running)!
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root reset.yml -K

# Deploy Kubespray with Ansible Playbook - run the playbook as root
# The option `--become` is required, as for example writing SSL keys in /etc/,
# installing packages and interacting with various systemd daemons.
# Without --become the playbook will fail to run!
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml -K

```

### 실행 후 k8s 설정 파일 위치 설정 - 리셋해도 실행.

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

### k8s 노드 & 열린 포트 확인

```
kubectl get nodes
kubectl get svc -A

```

### 선택: 부트스트랩 노드가 따로 없는 경우- 실행 노드가 컨트롤 노드인 경우

```
sed -i 's/kubeconfig_localhost: false/kubeconfig_localhost: true/g' inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml

```

팁 : 바뀌었는지 vim 으로 들어가서 확인 , 표현이 다를 수 도 있음.

### k8s 셋업 - 플러그인 설치 & 도움이 되는 설정

### 대시보드 설정

```
sed -i 's/# dashboard_enabled: true/dashboard_enabled: true/g' inventory/mycluster/group_vars/k8s_cluster/addons.yml
sed -i "s/dashboard_skip_login: false/dashboard_skip_login: true/g" roles/kubernetes-apps/ansible/defaults/main.yml
sed -i'' -r -e "/targetPort: 8443/a\\  type: NodePort" roles/kubernetes-apps/ansible/templates/dashboard.yml.j2

```

### 로컬 볼륨 & 인증서 저장 설정

cert_manager_enabled -> 인증서 활성화

local_volume_provisioner_enabled -> 로컬 스토리지 사용 가능 여부

```
sed -i 's/local_volume_provisioner_enabled: false/local_volume_provisioner_enabled: true/g' inventory/mycluster/group_vars/k8s_cluster/addons.yml
sed -i 's/cert_manager_enabled: false/cert_manager_enabled: true/g' inventory/mycluster/group_vars/k8s_cluster/addons.yml

```

### 바뀐 설정 적용 하기

reset.yml  실행할 필요 없이 바로 cluster.yml을 수행해도 상관 없다 .

```
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml -K

```

### 대시보드 접속 하기

1. 계정 만들기 아래 두 yml 파일이 있는 경로에서 아래 sh 스크립트 수행

계정정보 1  : vim clusterrolebinding.yaml

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
 name: admin-user
roleRef:
 apiGroup: rbac.authorization.k8s.io
 kind: ClusterRole
 name: cluster-admin
subjects:
- kind: ServiceAccount
 name: admin-user
 namespace: kube-system

```

계정정보 2: vim sa.yaml

```
apiVersion: v1
kind: ServiceAccount
metadata:
 name: admin-user
 namespace: kube-system

```

설정 스크립트 : vim  apply_login_info.sh

```
kubectl apply -f ./sa.yaml
kubectl apply -f ./clusterrolebinding.yaml

```

1. 접속 정보 확인

```
# 연결된 노드 출력
kubectl get nodes
# 토큰 생성 & admin 접근용
kubectl create token -n kube-system admin-user
# 인터페이스 출력 ( 열린 포트 , 서버, admin 포트 까먹으면 확인 )
kubectl get svc -A

```

위의 대시보드 포트에 https 로 들어간 뒤 토큰값을 입력하면 로그인 완료

## 데이터 노드 추가

참고 url : https://computingforgeeks.com/adding-new-node-into-kubernetes-cluster-using-kubespray/ 

데이터 노드 추가는 아래의 순서를 따릅니다. 
추가할 노드는 apt update, upgrade 가 되어 있어야 정상적으로 커맨드가 동작합니다. 

1. 추가될 노드에서 방화벽 해제 
2. 컨트롤 플레인에서 추가 될 노드로의 ssh 키 전달 
3. inventory/{cluster_name}/host.yml 파일의 수정 
4. inventory/{cluster_name}/inventory.ini 파일 수정 
5. ansible play book fact 명령어 실행 
6. ansible play book scale 명령어 실행 
7. kubectl get nodes 확인 .

구축중에 잔버그가 많았던 구간입니다. 만약에 설정이 꼬인 것 같고 운영중인 프로젝트가 없다면 ansible playbook reset 으로 리셋한 다음, ansible playbook cluster.yml 로  다시 만드는 것도 추천합니다 .

### 방화벽 해제  & ssh 키 전달

- 방화벽 해제 , 추가할 각 데이터 노드에서

```jsx
# 방화벽 해제 
ufw disable
ufw status # 확인
```

- ssh 키 전달, 컨트롤 플레인에서

```bash
$ copy-ssh-id devwiki@{data node 1 ip}
$ copy-ssh-id devwiki@{data node 2 ip}
$ copy-ssh-id devwiki@{data node 3 ip}
```

### host.yml 의 수정

경로는 ~/kubespray/inventory/ {cluster name } / host.yml 입니다 .

아래의 모양처럼 수정 , 아래는 컨트롤 노드 1개에 데이터 노드 3개를 붙인 야믈파일 예시 입니다. 

- host.yml 의 수정

```bash
xxx@control1:~/kubespray$ cat inventory/.../hosts.yaml
all:
  hosts:
    control1:
      ansible_host: 192.168.xx.yy
      ip: 192.168.xx.yy
      access_ip: 192.168.xx.yy
    data3:
      ansible_host: 192.168.xx.zz
      ip: 192.168.xx.zz
      access_ip: 192.168.xx.zz
    data2:
      ansible_host: 192.168.xx.ww
      ip: 192.168.xx.ww
      access_ip: 192.168.xx.ww
    data1:
      ansible_host: 192.168.xx.uu
      ip: 192.168.xx.uu
      access_ip: 192.168.xx.uu
  children:
    kube_control_plane:
      hosts:
        control1:
    kube_node:
      hosts:
        data1:
        data2:
        data3:
    etcd:
      hosts:
        control1:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```

- (Optinoal)각 데이터 노드 호스트의 hostname 변경

host.yml에서 지정한 hostname 과 데이터 노드 호스트의 hostname 이 다른 경우, 설치 중 에러가 나는 경우가 있습니다. 아래처럼 각 노드의 hostname 을  바꾸도록 합니다 .

```bash
# data1 머신에서 
hostnamectl set-hostname data1
...

# data2 머신에서 
hostnamectl set-hostname data2
...

# data3 머신에서 
hostnamectl set-hostname data3
...

#확인하기 
hostnamectl 
```

### inventory.ini 파일의 수정

ini 파일을 수정 합니다 .

```bash
xxx@xxx:~/kubespray$ cat inventory/???/inventory.ini
[all]
control1 ansible_host=192.168.??.?? ip=192.168.??.?? access_ip=192.168.??.??
data1 ansible_host=192.168.??.?? ip=192.168.??.?? access_ip=192.168.??.??
data2 ansible_host=192.168.??.?? ip=192.168.??.?? access_ip=192.168.??.??
data3 ansible_host=192.168.??.?? ip=192.168.??.?? access_ip=192.168.??.??

[kube_control_plane]
control1

[kube_node]
data1
data2
data3

[etcd]
control1

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```





```bash
# fact.yml
ansible-playbook -i inventory/k8scluster/inventory.ini  --become --become-user=root playbooks/facts.yml -K

# scale.yml 
ansible-playbook -i inventory/???/inventory.ini  --become --become-user=root scale.yml -K
```

이후에 정상적으로 실행 되었다면 kubectl 명령어로 확인합니다 .

```bash
kubectl get nodes 
```

설정 이후, kubectl 명령어가 정상적으로 수행되지 않는 경우, k8s 설정파일의 경로를 다시 잡아줘야 합니다.

```bash
mkdir -p $HOME/.kube  
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

위의 명령어 수행 후, 다시 kubectl 명령어를 수행해주세요 .