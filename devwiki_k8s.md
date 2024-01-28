1. [하드웨어 아키텍처](#하드웨어-아키텍처)
   - [스펙 기록](#스펙-기록)
2. [기타 작업 팁](#기타-작업-팁)
   - [대시 보드](#대시-보드)
   - [kubectl, ansible 커맨드 쓰기 위한 선행 자료](#kubectl-ansible-커맨드-쓰기-위한-선행-자료)

## 하드웨어 아키텍처

![하드웨어 아키텍처 이미지](/images/k8sNodes.png)

### 스펙 기록

홈 아이피는 카톡으로 공유

| vm 이름  | 역할       | 스펙                                   | lan ip        | external ip for ssh (포트포워딩)          |
|----------|------------|----------------------------------------|---------------|-------------------------------------------|
| control1 | 컨트롤 노드 | cpu: 4vCpu<br>ram: 4Gm<br>hd: 400Gb ssd | 192.168.35.94 | ssh {user}@1.232.62.62 -p 20122           |
| data1    | 데이터 노드1 | cpu: 8vCpu<br>ram: 8Gm<br>hd: 800Gb ssd | 192.168.35.24 | ssh {user}@1.232.62.62 -p 21122           |
| data2    | 데이터 노드2 | cpu: 8vCpu<br>ram: 8Gm<br>hd: 800Gb ssd | 192.168.35.63 | ssh {user}@1.232.62.62 -p 21222           |
| data3    | 데이터 노드 3 | cpu: 8vCpu<br>ram: 8Gm<br>hd: 800Gb ssd | 192.168.35.239| ssh {user}@1.232.62.62 -p 21322           |

## 기타 작업 팁

### 대시 보드

```bash
https://{홈아이피}:20101/
```

만약 토큰 값이 expired 되었거나 새로 발급 받아야 한다면 , 컨트롤 플레인에서 아래 명령어를 수행 .

```bash
kubectl create token -n kube-system admin-user
```

만약 유저가 valid 하지 않다는 에러가 나온다면 sa.yamd, clusterrolebinding.yml 을 적용하고 다시 실행 .
컨트롤 플레인에서 아래 커맨드 실행.

```bash
~/k8s_dashboard_user/apply_login_info.sh
```

### kubectl ,ansible  커맨드 쓰기 위한 선행 자료

1. pyenv 설정 

ansible 명령어를 수행하기 위해  pyenv 환경에서 명령어를 실행해야 합니다.

컨트롤 노드 기준으로 ~ 경로에서 아래의 커맨드를 실행해주세요. 

```bash
source $VENVDIR/bin/activate
```

1. k8s 설정 경로 바꾸기 

“/etc/kubernetes/admin.conf” 경로의 k8s 설정 파일을 ~/.kube 경로로 옮겨야 합니다.
ansible 명령어 수행 후,  kubectl 명령어가 실행안된다면 아래의 커맨드를 수행해주세요 .