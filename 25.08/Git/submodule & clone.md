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

서브모듈은 이러한 상황에서 사용되는 도구이다. **git 저장소 안에 다른 git 저장소를 디렉토리로 분리해 넣는 역할**인 것이다. <br>
여러 프로젝트에서 공통된 라이브러리를 재사용하거나, 큰 프로젝트를 여러 독립 모듈로 나누어 관리할 수 있다. <br>

<br>

### □ Submodule 사용하기
**(1) 프로젝트에 서브모듈을 추가**
```
git submodule add [repository-url] [path]
```
[repository-url] 추가하려는 서브모듈의 Git 저장소 URL <br>
[path] 서브모듈 포함시킬 하위 디렉토리
<br>
이 시점에서 `.gitmodules` 파일이 자동 생성된다. <br>

<br>
&nbsp;

**(2) 서브모듈이 등록된 프로젝트에서 모듈 세팅**
```
# .gitmodules 내용을 로컬 설정에 등록
git submodule init

# 메인 repo가 가리키는 커밋을 기준으로 서브모듈 가져옴
git submodule update
```
서브모듈이 등록된 프로젝트를 다른 사람이 clone 받았을 때, 서브모듈까지 세팅해주어야 한다. <br>
보통 `git submodule update --init --recursive` 으로 합쳐서 사용하기도 한다.

<br>
&nbsp;

**(3) 서브모듈 최신화**
```
git submodule update --remote
```
메인 repo가 지정한 커밋이 아닌 서브모듈 원격 저장소의 최신 브랜치를 당기고 싶을 때 사용한다. <br>
기본적으로 `.gitmodules` 파일에 적힌 브랜치 기준으로 최신화가 진행되며 <br>
만약 브랜치가 지정되어 있지 않은 경우 보통 `master`(또는 `main`)을 따라간다. <br>
<br>
(3-1) 브랜치를 지정하고 싶다면? <br>
```
[submodule "libs/lib-repo"]
  path = libs/lib-repo
  url = https://github.com/example/lib-repo.git
  branch = develop
```
이 경우 `--remote` 실행 시 `develop` 최신 버전까지 당겨온다.

<br>
&nbsp;

**(4) 서브모듈 커밋 반영** <br>
서브모듈에서 변경 사항이 생길 경우 메인 repo는 서브모듈이 **어떤 커밋을 가리키는지** 만 기억하기에 다른 사람이 clone 했을 때 변경이 반영되지 않는다.<br>
항상 메인 repo에 반영하기 위해서는 아래의 명령어를 실행해야 한다. <br>
```
cd [submodule-path]
git pull origin main # 원격저장소의 최신 커밋을 받아옴
cd ..
git commit -m "Update submodule to latest" # 커밋 메시지 지정
```

<br>
&nbsp;

**(5) 서브모듈 제거**
```
git rm -f libs/lib-repo
```
명령어 실행 시 `.gitmodules` 에 작성된 서브모듈([submodule "libs/lib-repo"])에 대한 내용과 <br>
해당 경로의 파일들도 함께 삭제가 된다. <br>
하지만 이 경우에 완벽하게 삭제된 것이 아니기에 같은 이름의 모듈을 추가하려할 때 <br>
```
A git directory for 'libs/lib-repo' is found locally with remote(s):

If you want to reuse this local git directory instead of cloning again from git@github.com...

use the '--force' option. If the local git directory is not the correct repo
or you are unsure what this means choose another name with the '--name' option.
```
다음과 같은 에러가 발생하며 같은 이름을 사용할 수 없다고 한다. <br>
<br>
이때 숨김 폴더인 `.git/modules/` 하위로 들어가 삭제한 모듈의 폴더를 지워주면 모두 제거가 된다.


<br>
&nbsp;

#### 💡 서브모듈을 사용할 때 어떤 파일과 폴더가 생성될까
> **`.gitmodules`**  <br>
git에서 서브모듈을 정의하는 설정(경로, 브랜치 등) 파일 <br>
서브모듈 갯수만큼 생성되고 `.gitignore` 파일처럼 버전을 관리함<br>

> **`.git/config`** <br>
로컬 환경에서 사용하는 설정 파일, 위 (2)에서의 모듈 세팅 때 <br>
init 하는 시점에서 해당 파일에 `gitmodules`에 있는 내용이 복사됨. <br>
개인별 설정(HTTPS -> SSH URL 변경 등)을 이 파일에서 관리가 가능함. <br>

> **`.git/modules`** <br>
서브모듈 자체의 Git 메타데이터 저장소, 각 모듈별 독립적인 `.git` 폴더가 위치함. <Br>
메인 저장소에서는 단순히 특정 커밋을 가리키고, 실제 서브모듈의 git 데이터는 이 디렉터리에 저장됨. <br>
<br>

**⚠️ 동작 흐름** <br>

**`git submodule init`**: `.gitmodules` → `.git/config`로 설정 복사 <br>

**`git submodule update`**: 메인 repo가 기억하는 커밋을 기준으로 `.git/modules`의 데이터에서 워킹 디렉터리 체크아웃. <br>

따라서 서브모듈은 항상 특정 커밋에 고정된 상태로 동작한다.
<br>


<br>
&nbsp;


#### 💡 커밋 고정 VS 브랜치 추적
기본적으로 서브모듈은 `update` 로 특정 커밋을 가리키며 안정성을 보장한다. <br>
하지만 `branch` 옵션 추가로 특정 브랜치를 따라가며 항상 최신 버전을 유지할 수도 있다. <br>
어느 상황에서 무엇을 선택하면 좋을까? <br>
<br>

**(1) 커밋 고정**
- 프로젝트 빌드 및 배포 시 항상 동일한 상태를 보장하고 싶을 때
- 서브모듈 업데이트로 인한 예기치 않은 오류를 방지하고 싶을 때
<br>

**(2) 브랜치 추적**
- 서브모듈이 활발하게 개발되고 있고, 항상 최신 기능을 반영하고 싶을 때
- CI/CD 환경에서 자동으로 최신 버전 반영이 필요할 때

<br>
&nbsp;


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
  `--depth 1`은 저장소의 가장 최신 커밋 1개만 복제한다. <br>
  `--single-branch` 옵션을 추가하면 필요없는 브랜치 기록은 복제 대상에서 제외된다. <br>
  하지만 과거 커밋 접근이 필요할 땐 적합하지 않다는 단점이 존재한다. <br>

  <br>
  
- **`--branch`** <br>
  <br>
  특정 브랜치를 복제, clone 후 체크아웃할 브랜치를 지정한다. `-b`로 줄여 사용할 수 있다.
  ```
  git clone --branch branch_name <리포지토리 주소>
  ```
  branch_name이라는 브랜치만 복제한다.
  
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
  
<br>
<br>

### 💡 Git Clone과 Git Fork
**(1) Git Fork** <br>

다른 사람의 github repo를 내 github repo에 그대로 복제하는 것 <br>
fork된 repo는 원본과 연결되어 있기에 원본에 변화가 생기면 fork된 repo에 반영이 가능하다. <br>

이후 원본 repo에 변경 사항을 적용하고 싶을 경우 pull request를 보내어 관리자로부터 승인을 받아야 한다. 승인이 되는 경우 원본 repo에 내 변경사항이 반영된다. 

<br>

**(2) 차이점** <br>

가장 큰 차이점은 **어디서 진행되는지** 이다. <br>
fork는 원격 저장소(Github)에서 진행되고, clone은 로컬 저장소에서 진행된다. <br>

물론 fork를 통해 나만의 원격 저장소가 생겨나도<br>
개발을 위해서는 clone을 통해 로컬 저장소를 만들어 진행한다. <br>

> **굳이 fork를 하는 이유는 무엇일까?** <br>
> 개발을 어떻게 진행하느냐에 따라 방식에 차이가 생긴다. <br>
같은 베이스를 두고 진행하려면 clone을, 새로운 베이스를 만들어 <br>
기존과 합칠 것이라면 fork를 사용하여 더 큰 구조를 만들어낸다.









