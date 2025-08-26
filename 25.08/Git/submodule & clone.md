# git Submodule & clone

## ■ git이란?
컴퓨터 파일의 변경 사항을 추적하고 파일들의 작업을 조율하는 분산형 버전 관리 시스템을 의미한다. <br>
소스 코드를 주고 받을 필요 없이, 같은 파일을 여러 명이 동시에 작업하는 병렬 개발이 가능하다. <br>

<br>
<br>

## ■ Submodule
> 프로젝트를 수행하다 보면 타 프로젝트를 함께 사용해야 하는 경우가 종종 있다. <br>
그 프로젝트는 외부 라이브러리나 내부 여러 프로젝트에서 공통으로 사용할 라이브러리일 수도 있다. <br>
이런 상황에서 두 프로젝트를 서로 별개로 다루면서도 그 중 하나를 다른 하나 안에서 사용할 수 있어야 한다. <br>

서브모듈은 이러한 문제를 다루는 도구이다. **git 저장소 안에 다른 git 저장소를 디렉토리로 분리해 넣는 역할**인 것이다. <br>
여러 프로젝트에서 공통된 라이브러리를 재사용하거나, 큰 프로젝트를 여러 독립 모듈로 나누어 관리할 수 있다. <br>

<br>

### □ Submodule 사용하기
**(1) 서브모듈을 포함시키고자 하는 Git 저장소에서 명령어를 실행하여 서브모듈을 추가한다.**
```
git submodule add [repository-url] [path]
```
[repository-url] 추가하려는 서브모듈의 Git 저장소 URL <br>
[path] 서브모듈 포함시킬 하위 디렉토리
<br>
&nbsp;

**(2) 서브모듈을 초기화하고 업데이트한다.**
```
git submodule init
git submodule update
```

(2-1) 서브모듈을 업데이트한다.
```
git submodule update --remote
```
> **(2)와의 차이는?** <br>
> (2)는 프로젝트 빌드, (2-1)은 서브모듈 자체를 최신 코드로 업데이트하는 것이다.

<br>
&nbsp;

**(3) 서브모듈의 상태(변경 사항)을 확인한다.**
```
git submodule status
```
업데이트 전에 항상 상태 확인을 하여 업데이트 여부를 판단한다.

<br>
<br>
<br>
<br>

## ■ git clone
**git repository를 복제(clone)** 하는 데 사용되는 명령어를 의미한다. <br>
원격 저장소에 있는 프로젝트를 로컬 컴퓨터에 그대로 복제하는 기능을 수행하며, <br>
주로 프로젝트의 소스 코드를 처음으로 받아올 때 사용된다. <br>
복제되는 항목은 원격 저장소의 파일과 디렉터리 구조, 커밋 기록 등이 있다. <br>

> **repository를 다운로드하는 것과 같지 않을까?** <br>
그렇지 않다! 클론은 커밋 기록이 다 있는 레포를 생성하는 것이고, 다운로드는 파일의 최신 버전만 가져오는 점에서 차이가 있다. <br>
<br>

### □ git clone 사용하기
**(1) 기본 명령어**
```
git clone <리포지토리 주소>
```
<br>

만약 특정 주소에 저장하고 싶으면 다음과 같이 명령어를 실행하면 된다.<br>
```
git clone <리포지토리 주소> <경로>
```
<br>
<br>

**(2) 명령어 옵션**
- **`--depth`** <br>
  <br>
  저장소의 히스토리를 얕게(clone shallowly) 복제한다. 지정한 수의 최근 커밋만 복제한다는 의미다.
  ```
  git clone --depth 1 <리포지토리 주소>
  ```
  `--depth 1`은 저장소의 가장 최신 커밋 1개만 복제한다. 보통 CI/CD에서 빌드 시간을 줄일 때도 사용된다.

  <br>
  
- **`--branch`** <br>
  <br>
  특정 브랜치를 복제한다. `-b`로 줄여 사용할 수 있다.
  ```
  git clone --branch branch_name <리포지토리 주소>
  ```
  branch_name이라는 브랜치만 복제한다. 단순히 특정 브랜치 복제가 아닌 기본적으로 전체를 복제하되 그 브랜치를 체크아웃한 상태를 의미한다.
  
<br>

- **`--recursive`** <br>
  <br>
  레포지토리와 서브모듈을 한 번에 복제한다. 서브모듈 하위의 또다른 서브모듈도 한 번에 다 가져올 수 있다.
  
  ```
  git clone --recursive <레포지토리 주소>
  ```
  
<br>
<br>

### □ submodule clone
서브모듈을 포함하는 메인 git을 clone했을 때 서브모듈이 기본적으로 비어있다. <br>
때문에 별도로 서브모듈을 클론하는 명령어를 실행해주어야 한다. <br>

```
# 서브모듈 정보 기반으로 로컬 환경설정 파일 생성
git submodule init

# 서브모듈 remote repo에서 데이터를 가져오고 checkout
git submodule update
```

<br>

서브모듈이 여러 개 존재할 경우, foreach를 사용하여 여러 서브모듈에서 한 번에 실행할 수 있다.
```
# 서브모듈 안에서 master 브랜치로 이동
git submodule foreach git checkout master

# 원격 저장소(origin)의 master 브랜치 최신 커밋을 가져와서 병합
git submodule foreach git pull origin master
```







