# K8S 인프라 정리문서

k8s 인프라 정리 문서에 대해 다룹니다.

## k8s 클러스터 관리.

[하드웨어 아키텍처 이미지](/images/k8sNodes.png)

### [k8s 클러스터 관리](/devwiki_k8s.md)

1. [하드웨어 아키텍처](/devwiki_k8s.md#하드웨어-아키텍처)
   - [스펙 기록](/devwiki_k8s.md#스펙-기록)
2. [기타 작업 팁](/devwiki_k8s.md#기타-작업-팁)
   - [대시 보드](/devwiki_k8s.md#대시-보드)
   - [kubectl, ansible 커맨드 쓰기 위한 선행 자료](/devwiki_k8s.md#kubectl-ansible-커맨드-쓰기-위한-선행-자료)

## k8s 배포 방법과 버그 기록

### [k8s 배포 방법과 버그 기록](/deployK8s.md)

1. [작성 목적](/deployK8s.md#작성-목적)
2. [환경](/deployK8s.md#환경)
   - [버전](/deployK8s.md#versions)
   - [하드웨어](/deployK8s.md#hardware)
3. [목표](/deployK8s.md#목표)
4. [리눅스 셋업](/deployK8s.md#리눅스-셋업)
   - [요약](/deployK8s.md#요약)
   - [root 비밀번호 등록](/deployK8s.md#root-pw-등록)
   - [공통적으로 필요한 작업](/deployK8s.md#공통적으로-필요한-작업)
5. [pyenv](/deployK8s.md#pyenv)
   - [pyenv 환경 설정 및 설치](/deployK8s.md#pyenv-환경-설정-&-설치)
   - [메모: pyenv 환경으로 전환](/deployK8s.md#메모-:-pyenv-환경으로-전환)
   - [메모2: pyenv 관련 변수 영구 적용하기](/deployK8s.md#메모2-:-pyenv-관련-변수-영구-적용하기)
6. [k8s 셋업](/deployK8s.md#k8s-셋업)
   - [쿠베스프레이 기본 샘플 배포해 보기](/deployK8s.md#쿠베스프레이-기본-샘플-배포해-보기)
   - [실행 후 k8s 설정 파일 위치 설정 - 리셋해도 실행](/deployK8s.md#실행-후-k8s-설정-파일-위치-설정---리셋해도-실행)
   - [k8s 노드 & 열린 포트 확인](/deployK8s.md#k8s-노드-&-열린-포트-확인)
   - [선택: 부트스트랩 노드가 따로 없는 경우 - 실행 노드가 컨트롤 노드인 경우](/deployK8s.md#선택:-부트스트랩-노드가-따로-없는-경우---실행-노드가-컨트롤-노드인-경우)
   - [k8s 셋업 - 플러그인 설치 & 도움이 되는 설정](/deployK8s.md#k8s-셋업---플러그인-설치-&-도움이-되는-설정)
   - [대시보드 설정](/deployK8s.md#대시보드-설정)
   - [로컬 볼륨 & 인증서 저장 설정](/deployK8s.md#로컬-볼륨-&-인증서-저장-설정)
   - [바뀐 설정 적용하기](/deployK8s.md#바뀐-설정-적용-하기)
   - [대시보드 접속하기](/deployK8s.md#대시보드-접속-하기)
7. [데이터 노드 추가](/deployK8s.md#데이터-노드-추가)
   - [방화벽 해제 & ssh 키 전달](/deployK8s.md#방화벽-해제-&-ssh-키-전달)
   - [host.yml의 수정](/deployK8s.md#host.yml-의-수정)
   - [inventory.ini 파일의 수정](/deployK8s.md#inventory.ini-파일의-수정)
   - [fact.yml, scale.yml 수행](/deployK8s.md#fact.yml,-scale.yml-수행)