# Yocto 프로젝트란 무엇인가?

* 임베디드 장치용 맞춤형 배포판 빌드를 위한 오픈 소스 프로젝트
* 하드웨어의 기본 아키텍처와 독립적인 임베디드 개발을 위한 공통 기반을 정의함
* 쉽게 말해서, 내가 개발하고자 하는 임베디드 환경(ARM, x86 등)에 맞게 커스텀 리눅스를 빌드해주는 도구 (셋팅의 번거로움을 줄여주는 역할)
  - Yocto: 오픈 임베디드 빌드 시스템이 리눅스 소프트웨어 스택을 빌드하는 데 필요한 모든 정보를 제공함 (크로스 컴파일러, 라이브러리 등)
  - bitbake: Yocto에서 제공하는 정보를 기반으로 빌드를 수행하는 빌드 도구
  - Poky: 빌드를 위해 필요한 소스 코드를 가져오거나, 빌드 환경 설정, 컴파일, 생성된 이미지를 설치하는 방법을 기술하는 .bb 파일이 들어 있음

## 실습 환경

* 예제는 ubuntu 18.04, Yocto dunfell 버전을 기반으로 설명할 것이다. (Yocto, 쉽게 이해하고 깊게 다루기 -조운래 저- 참조)

## 설치 패키지

* ubuntu에서 패키지 설치하기

```$ sudo apt install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 xterm make xsltproc docbook-utils fop dblatex xmlto```

* 추가 설치 패키지

```
$ sudo apt install git
$ sudo apt install tree
$ sudo apt install python3.8
```

혹은 [sourceforce 사이트](https://sourceforge.net/projects/greatyocto/files/)에서 installed ubuntu great toctoova 가상 파일을 다운로드해서 사용해도 된다. (계정: great / PW: great) VirtualBox 7.0.8 버전 이상, Extension Pack Manager에서 확장 팩을 설치하는 것을 권장한다. 권장 사양은 CPU 8 Core, RAM 8192MB 이다.

# bitbake

* bitbake: 파이썬, 셸 스크립트 혼합 코드를 분석하는 작업 스케줄러, 임베디드 리눅스의 크로스 컴파일을 위한 패키지 및 관련 파일을 빌드하는 데 사용되는 도구
  - Visual C++과 같은 빌드 도구의 일종
  - Poky 안에 포함되어 있지만 단독으로도 여러 가지를 수행할 수 있음

## bitbake 설치 및 실행

* 소스 받기 (dunfell 버전): `$ wget http://git.openembedded.org/bitbake/snapshot/bitbake-1.46.0.tar.gz`

* 압축 풀기: `$ tar -xzf bitbake-1.46.0.tar.gz`
  - bitbake-1.46.0/bin 디렉토리에 bitbake 실행 파일이 존재함

* 실행하기: `$ export PATH=/home/user/bitbake_src/bitbake-1.46.0/bin:$PATH`
  - bitbake-1.46.0/bin 디렉토리를 $PATH에 추가할 것
  - `$ bitbake --version` 버전이 제대로 출력되면 성공한 것
  

## 메타데이터

* 메타데이터는 소프트웨어를 어떻게 빌드할지, 그리고 빌드하려는 소프트웨어들 간에 어떤 의존성이 있는지 기술하고 있음 (환경 설정 파일은 변수만 있고, 나머지는 변수와 함수(task)가 있음)
  - 환경 설정 파일(.conf): 전역 변수 집합
  - 레시피 파일(.bb): 소프트웨어를 어디서 다운로드할지, 받은 소프트웨어를 어떻게 빌드할지, 빌드된 산출물을 어디에 위치할지 기술되어 있음
  - 클래스 파일(.bbclass): 여기에 선언된 변수, 함수는 해당 클래스 파일을 상속(inherit)한 레시피에서만 사용할 수 있음
  - 레시피 확장 파일(.bbappend): 레시피 파일에서 선언된 변수, 함수를 재정의할 수 있게 해주며, 레이어 개념을 알아야 함
  - 인클루드 파일(.inc): 클래스 파일은 비공식적인 기능을 제공하는 반면, 인클루드 파일은 비공식적인 내용을 공유할 때 사용함
 
* bitbake는 메타데이터를 활용하여 루트 파일 시스템 이미지, 커널 이미지, 부트로더 이미지를 생성함

## bitbake 문법
  - 변수 타입 없음, 모든 값을 문자열로 인식함 ("value" 식으로 표현함)
  - 변수 이름에 제약을 두지 않음. 다만 관행적으로 변수 이름을 대문자로 시작하도록 함
  - 예) BITBAKE_VAR = "value" 또는 BITBAKE_VAR = 'value'
  - #로 시작하면 주석으로 간주함

## bitbake로 "Hello! bitbake world!" 출력해보기

1. BBPATH 설정
  - bitbake 실행 파일이 특정 디텍토리에 만들어진 메타데이터들을 인식할 수 있도록 함
  - `$ export BBPATH=/home/user/bitbake_test/`

2. BBPATH 아래로 다음과 같은 디렉토리 및 파일을 생성할 것

```
bitbake_test/
|- classes
|   |- base.bbclass  # 4. 클래스 파일 분석
|- conf
|   |- bblayers.conf  # 1. 레이어를 먼저 찾음
|   |- bitbake.conf  # 3. 환경 설정 파일 분석 
|- mylayer
    |- conf
    |   |- layer.conf  # 2. 레시피 파일의 위치 파악
    |- hello.bb  # 5. 마지막으로 레시피 파일 실행
```

파일 내용은 다음과 같습니다.

bitbake.conf

```
# 기본적인 환경 설정
PN = "${bb.parse.vars-from_file(d.getVar('FILE', False),d)[0]or 'defaultpkgname'}"  # 레시피 파일의 이름
TMPDIR = "${TOPDIR}/tmp"    # 중간 산출물을 저장하는 디렉토리
CACHE = "${TOPDIR}/cache"
STAMP = "${TOPDIR}/${PN}/stamps"    # 태스크에 대한 수행 이력 기록
T     = "${TOPDIR}/${PN}/work"    # 임시 생성된 태스크 실행 로그, 태스크 스크립트 파일 등
B     = "${TOPDIR}/${PN}"    # 레시피의 빌드 과정에서 함수를 실행하는 디렉토리
```

bblayers.conf

```
# 빌드 수행을 위해 어떤 레이어들이 존재하는지 알려줌
# 레이어: 연관된 메타데이터들을 포함하는 저장소(디렉토리)
BBLAYERS ?= " \
    /home/user/bitbake_test/mylayer \
"
```

layer.conf

```
# 현재 레이어의 경로와 레이어 내에 있는 레시피 파일들(.bb, .bbappend)의 경로를 알려줌
BBPATH .= ":${LAYERDIR}"
BBFILES += "${LAYERDIR}/*.bb"
BBFILE_COLLECTIONS += "mylayer"
BBFILE_PATTERN_mylayer := "^${LAYERDIR}/"
```

base.bbclass

```
# bitbake가 실행되기 위해 반드시 bbclass 파일이 존재해야 함
addtask do_build    # 태스크 추가
```

hello.bb

```
# 레시피 파일: 실제로 bitbake가 실행하는 주체
# 레시피 파일 이름 규칙: "<package_name>_<package_version>_<package_revision>.bb", 예를 들면 "linux-yocto_5.4_r0.bb"
DESCRIPTION = "hello world example"
PN = "hello"    # 패키지 이름 (생략할 경우 파일명이 PN이 됨)
PV = "1"    # 패키지 버전 (생략할 경우 r0)
python do_build() {
    bb.warn("Hello! bitbake world!")
}
```

3. 레시피 파일 실행하기
  - `$ bitbake <recipe filename>`: 전체 태스크 실행
  - `$ bitbake <recipe filename> -c <take name>`: 특정 태스크 실행
  - `$ bitbake <recipe filename> -f`: 이미 수행한 태스크 강제 실행 (이전 태스크 변경 여부와 실행 이력을 체크하기 때문)
  - 위의 예제의 경우 `$ bitbake hello -c build` 또는 `$ bitbake hello -f`라고 하면 "Hello! bitbake workd!" 메시지를 볼 수 있음

# Poky

* Poky는 Yocto 프로젝트에 대해 참조가 되는 배포판
  - 커스텀 리눅스를 쉽게 구축하고 빌드할 수 있는 메타데이터들(.conf, .bb, .class, .inc)이 존재한다.
  - 이 메타데이터들을 기반으로 필요한 메타데이터를 추가/변경하여 커스텀 리눅스를 구축하고 빌드할 수 있다.
  - 빌드 시스템 관점에서 Poky를 오픈임베디드 빌드 시스템이라고 부를 수 있다.

* 오픈임베디드: 임베디드 장치용 리눅스 배포판을 만드는 데 사용되는 빌드 자동화 프로임워크 및 크로스 컴파일러 환경
  - 레시피를 사용해 소스 코드를 저장소에서 가져오고, 필요할 경우 패치를 적용하여 소스 코드를 컴파일 및 링크하고 패키지 및 부팅이 가능한 이미지를 생성함

...

# 빌드 속도 개선하기

...
