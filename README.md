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

혹은 [sourceforce 사이트](https://sourceforge.net/projects/greatyocto/files/)에서 installed ubuntu great yocto.ova 가상 파일을 다운로드해서 사용해도 된다. (계정: great / PW: great) VirtualBox 7.0.8 버전 이상, Extension Pack Manager에서 확장 팩을 설치하는 것을 권장한다. 권장 사양은 CPU 8 Core, RAM 8192MB 이다.

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

## bitbake 문법 (1)
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
|   |- base.bbclass    # 4. 클래스 파일 분석
|- conf
|   |- bblayers.conf   # 1. 레이어를 먼저 찾음
|   |- bitbake.conf    # 3. 환경 설정 파일 분석 
|- mylayer
    |- conf
    |   |- layer.conf  # 2. 레시피 파일의 위치 파악
    |- hello.bb        # 5. 마지막으로 레시피 파일 실행
```

파일 내용은 다음과 같습니다.

bitbake.conf
```
# 기본적인 환경 설정
PN = "${bb.parse.vars-from_file(d.getVar('FILE', False),d)[0]or 'defaultpkgname'}"  # 레시피 파일의 이름
TMPDIR = "${TOPDIR}/tmp"            # 중간 산출물을 저장하는 디렉토리
CACHE = "${TOPDIR}/cache"
STAMP = "${TOPDIR}/${PN}/stamps"    # 태스크에 대한 수행 이력 기록
T     = "${TOPDIR}/${PN}/work"      # 임시 생성된 태스크 실행 로그, 태스크 스크립트 파일 등
B     = "${TOPDIR}/${PN}"           # 레시피의 빌드 과정에서 함수를 실행하는 디렉토리
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
PV = "1"        # 패키지 버전 (생략할 경우 r0)
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

## Poky 소스 다운로드 및 빌드

* Poky 소스를 다운로드 받을 디렉토리로 이동한 후 git을 통해 다운로드한다.
  - dunfell 브랜치를 사용할 것이므로 dunfell 브랜치로 체크아웃한다.

```
$ mkdir poky_src
$ cd poky_src
$ git clone git://git.yoctoproject.org/poky
$ git checkout dunfell
```

* 디렉토리 구조는 다음과 같다.

`$ tree -d -L 2 poky_src/`

```
poky_src/
|- poky
    |- bitbake
    |- contrib
    |- documentation
    |- meta            # 오픈임베디드 코어 레이어
    |- meta-poky       # Yocto 배포 참조 레이어
    |- meta-selftest   # oe-selftest 스크립트가 사용하는 bitbake 테스트 레이어
    |- meta-skeleton   # 커스텀 레이어를 생성하는 데 사용되는 템플릿 레이어
    |- meta-yocto-bsp  # Yocto 프로젝트의 BSP 레이어
    |- scripts
```

* Poky 소스 빌드하기
  - poky_src 디렉토리에서 실행한다: `$ source poky/oe-init-build-env`
  - 실행 후에는 현재 작업 디렉토리 위치가 build 디렉토리로 변경된다.
  - 빌드를 실행하여 Yocto에서 제공된 커스텀 리눅스 이미지를 만든다: `$ bitbake core-image-minimal -k` (-k 옵션은 오류가 발생하더라도 끝까지 빌드를 계속 하라는 뜻) [여기서, 레시피 core-image-minimal은 다른 것이 될 수 있음]
  - 레시피 파일에서 사용하는 모든 환경 변수를 확인하는 방법: `$ bitbake core-image-minimal -e > env.txt` (메타데이터 분석 절차를 수행한 결과로 얻어진 변수, 함수를 env.txt로 저장)
  - `$ bitbake-getvar -r core-image-minimal DL_DIR`: 위와 비슷함, 이렇게 하면 DL_DIR 변수의 할당 과정을 상세하게 볼 수 있음

* oe-init-build-env 스크립트
  - 기본 빌드 환경을 설정한다.
  - 이 스크립트를 실행하면 다음과 같은 conf 파일이 생성된다.

```
poky_src/
|- build
    |- conf
        |- bblayers.conf        # 생성된 레이어들의 정보를 bitbake에게 알려줌, 레이어들의 경로는 BBLAYERS 변수에 할당됨
        |- local.conf           # 환경 설정 파일, bitbake.conf 파일에서 이 파일을 인클루드해 사용한다. (타깃 머신 지정, 크로스 툴체인 지정, 전역 변수 처리)
        |- templateconf.cfg     # 프로젝트를 생성하는 데 사용되는 템플릿 환경 설정을 포함하는 디렉토리를 포함하고 있음
```

* 빌드 결과를 QEMU 에뮬레이터로 실행
  - 이것을 사용하려면 `$ source poky/oe-init-build-env`를 실행한 다음에 `bitbake runqemu`를 실행해야 한다.
  - 실행하는 방법은 다음과 같다. (비디오 콘솔을 따로 생성하지 않고 위에서 생성한 커스텀 리눅스 이미지를 동작시킴)
  - `$ cd poky/scripts`
  - `$ runqemu core-image-minimal nographic`
  - 종료 시에는 `# poweroff`를 실행한다.

# 빌드 속도 개선하기

* PREMIRRORS(로컬 미러 저장소), sstate-cache(공유 상태 캐시)를 구성하여 소스를 미리 다운로드함으로써 fetch 시간을 단축하여 빌드 속도를 개선하는 것임
  - 이 부분은 필수가 아니므로 생략함

# Poky를 이용하여 레이어, 레시피 생성하기

## Poky 다운로드 및 빌드

* Poky 소스 다운로드

```
$ mkdir poky_src
$ cd poky_src
$ git clone git://git.yoctoproject.org/poky
$ git checkout dunfell
```

* Poky 소스 빌드하기
  - poky_src 디렉토리에서 실행한다: `$ source poky/oe-init-build-env`
  - 실행 후에는 현재 작업 디렉토리 위치가 build 디렉토리로 변경된다.
  - 빌드를 실행하여 Yocto에서 제공된 커스텀 리눅스 이미지를 만든다: `$ bitbake core-image-minimal -k`

## 예제 작성하기

* 새로운 레이어 추가를 위해 meta-hello 디렉토리를 생성한다.

* 이어서 meta-hello 디렉토리 아래에 conf, recipes-hello 디렉토리를 생성한다.

* conf 디렉토리 아래에는 layer.conf 파일을, recipes-hello 디렉토리 아래에는 hello.bb 파일을 생성한다.

```
poky_src/
|- poky
|   |- meta-hello
|       |- conf
|       |   |- layer.conf
|       |- recipes-hello
|           |- hello.bb
|- build
    |- conf
        |- bblayers.conf
```

layer.conf
```
BBPATH                    =. "${LAYERDIR}:"
BBFILES                   += "${LAYERDIR}/recipes*/*.bb"
BBFILE_COLLECTIONS        += "hello"
BBFILE_PATTERN_hello      = "^${LAYERDIR}/"
BBFILE_PRIORITY_hello     = "10"
LAYERSERIES_COMPAT_hello  = "${LAYERSERIES_COMPAT_core}"
```

hello.bb
```
SUMMARY = "간단한 hello 예제"          # 패키지에 대한 간단한 소개. 한 줄로 입력하며 최대 80자.
DESCRIPTION = "Simple hello example"  # 패키지와 기능에 대한 자세한 설명으로 여러 줄 가능
LICENSE = "CLOSED"
AUTHOR = "soonbum.jeong <peacemaker84@gmail.com>"  # 저자의 이름과 이메일 주소
#HOMEPAGE = "URL을 입력하면 됨"

do_printhello(){
    bbwarn "hello world!"
}
addtask do_printhello after do_compile before do_install
```

bblayers.conf
```
# POLY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
# changes incompatibly
POLY_BBLAYERS_CONF_VERSION = "2"
BBPATH = "${TOPDIR}"
BBFILES ?= ""
BBLAYERS ?= " \
    /home/user/poky_src/poky/meta \
    /home/user/poky_src/poky/meta-poky \
    /home/user/poky_src/poky/meta-yocto-bsp \
    /home/user/poky_src/poky/meta-hello \
```

※ 로그 출력 함수는 다음과 같다.

Log Level | 파이썬 함수 | 셸 함수
--------- | ---------- | --------
plain | bb.plain(message) | bbplain message
debug | bb.debug(message) | bbdebug level message
note | bb.note(message) | bbnote message
warn | bb.warn(message) | bbwarn message
error | bb.error(message) | bberror message
fatal | bb.fatal(message) | bbfatal message

* 레이어가 정상적으로 추가되었는지 확인하는 방법은 다음과 같다.
  - 레이어 이름, 경로, 우선순위가 표시됨
  - `$ bitbake-layers show-layers`

## 예제 실행하기

* 실행하는 방법은 다음과 같다: `$ bitbake <recipe-name>`
  - `$ bitbake hello`

## bitbake 문법 (2)
  - `DEPENDS = "hello"` : hello 레시피 파일에 의존함 (그것이 먼저 빌드되어야 함)
  - `VAR1 ?= "hello"` : 기본값 할당 (처음 입력한 기본값 할당이 유지됨)
  - `VAR1 ??= "hello"` : 약한 기본값 할당 (마지막에 입력한 기본값 할당으로 갱신됨), 그러나 '=', '?=' 연산자보다 우선순위가 낮음
  - `VAR2 = "${VAR1} my name is yocto"` : 변수 할당
  - `VAR4 := "The quick brown for ${VAR2}"` : 늦은 변수 할당 (${VAR4}를 호출하는 시점에서 할당이 일어남)
  - `VAR3 =+ "${VAR1}"` : 변수 선입 (VAR3 앞에 VAR1을 붙임, 공백으로 구분됨)
  - `VAR3 += "${VAR2}"` : 변수 후입 (VAR3 뒤에 VAR2을 붙임, 공백으로 구분됨)
  - `VAR3 =. "${VAR1}"` : 변수 선입 (VAR3 앞에 VAR1을 붙임, 공백 없음)
  - `VAR3 .= "${VAR1}"` : 변수 후입 (VAR3 뒤에 VAR2을 붙임, 공백 없음)
  - `VAR3_prepend = "${VAR1}"` : 변수 선입 (VAR3 앞에 VAR1을 붙임, 공백 없음) (늦은 할당) [Yocto honister 버전 이상에서는 :prepend로 바뀜]
  - `VAR3_append = "${VAR1}"` : 변수 후입 (VAR3 뒤에 VAR2을 붙임, 공백 없음) (늦은 할당) [Yocto honister 버전 이상에서는 :append로 바뀜]
  - `VAR1_remove = "123"` : 공백으로 구분된 "123"과 일치하는 문자열만 삭제함 ("123 456 789 123456789 789 456 123" --> " 456 789 123456789 789 456 ") [Yocto honister 버전 이상에서는 :remove로 바뀜]
  - 변수와 마찬가지로 함수 이름에도 _prepend, _append를 붙이면 본체 함수 앞뒤에 다른 함수가 자동으로 호출됨

## hello 애플리케이션 레시피 작성

* 간단한 hello.c 소스 파일을 생성한다. COPYING이라는 라이선스 파일도 함께 만들어 본다.

```
poky_src/
|- poky
|   |- meta-hello
|       |- conf
|       |   |- layer.conf
|       |- recipes-hello
|           |- hello.bb
|           |- source
|               |- hello.c
|               |- COPYING
|- build
    |- conf
        |- bblayers.conf
```

hello.c
```
#include <stdio.h>
#include <unistd.h>

int main(){
    int i = 0;
    while (i < 10) {
        printf ("Hello world!\n");
        sleep(1);
    }
    return 0;
}
```

COPYING
```
EXAMPLE LICENSE FILE
Copyright (C) 2024, Soonbum Jeong
This is example license file.
```

* 이제 hello.c 애플리케이션 실행을 위해 hello.bb 레시피 파일을 다시 작성한다.

hello.bb
```
DESCRIPTION = "Simple helloworld application example"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://COPYING;md5=80cade1587e04a9473701795d41a4f0c"  # COPYING 파일이 존재하는 source 디렉토리에서 다음을 실행하면 checksum을 얻을 수 있다. (`$ md5sum COPYING`)

SRC_URI = "file://hello.c"
SRC_URI_append = " file://COPYING"
S = "${WORKDIR}"

# S: bitbake가 빌드시 압축된 소스를 해제해 위치시키는 디렉토리, SRC_URI에서 로컬 소스 지정시 해당 소스가 빌드를 진행할 때 복사돼 위치하는 디렉토리
# WORKDIR: 레시피를 빌드하면서 생성된 각종 정보를 저장하는 디렉토리 (`$ bitbake-getvar -r hello WORKDIR`로 확인 가능)

do_compile() {
    ${CC} hello.c ${LDFLAGS} -o hello
}

# D: 빌드의 결과물로 생성된 바이너리가 위치하는 경로

# bin: /usr/bin (사용자 프로그램)
# sbindir: /usr/sbin (시스템 관리 프로그램)
# libdir: /usr/lib (라이브러리)
# libexecdir: /usr/lib
# sysconfdir: /etc (환경 설정 파일)
# datadir: /usr/share
# mandir: /usr/share/man
# includedir: /usr/include

do_install() {
    install -d ${D}${bindir}
    install -m 0755 hello ${D}${bindir}
}

# THISDIR: bitbake가 현재 파싱하는 파일이 위치하고 있는 디렉토리 (recipes-hello)
# FILESPATH: poky/meta/classes/base.bbclass 클래스 파일에 정의되어 있음 (오픈임베디드 빌드 시스템이 패치 및 파일을 검색할 때 사용하는 디렉토리 리스트를 갖고 있음)
# FILESEXTRAPATHS: FILESPATH 변수를 확장함

FILESEXTRAPATHS_prepend := "${THISDIR}/source:"
FILES_${PN} += "${bindir}/hello"
```

## 레시피 파일 빌드하기

```
$ bitbake hello -c cleanall    # cleanall 태스크만 수행 (WORKDIR 변수가 가리키는 빌드 작업 디렉토리의 모든 결과물 삭제)
$ bitbake hello                # 기본 태스크 수행 (기본 태스크를 나타내는 변수: BB_DEFAULT_TASK)
```

* 태스크 실행시 결과물들이 저장되는 디렉토리들

```
do_fetch ----------------------> ${DL_DIR}
   |                                  |
   v       <---------------------------
do_unpack    ------------------> ${WORKDIR}/${S}
   |                                  |
   v       <---------------------------
do_patch     ------------------> ${WORKDIR}/${S}
   |                                  |
   v                                  |
do_prepare_recipe_sysroot             |
   |                                  |
   v       <---------------------------
do_configure ------------------> ${WORKDIR}/${B}
   |                                  |
   v       <---------------------------
do_compile  -------------------> ${WORKDIR}/${B}
   |                                  |
   v       <---------------------------
do_install  -------------------> ${WORKDIR}/${D}
```

## 라이선스

* 라이선스 관련 중요한 변수 2가지가 있다.
  - LICENSE: 사용하지 않을 경우 CLOSED, 그 외의 경우 라이선스 파일이 있다는 뜻이며 라이선스 파일과 checksum 값을 다음 변수에 기술해야 함
  - LIC_FILES_CHKSUM: 라이선스 파일과 checkbum을 갖고 있는 변수. checksum은 보통 md5, sha256을 사용함

예제1
```
# bzip 2 applet in busybox is based on lightly-modified bzip2-1.0.4 source
# the GPL is version 2 only
LICENSE = "GPLv2 & bzip2-1.0.4"
LIC_FILES_CHKSUM = "file://LICENSE;md5=de10de48642ab74318e893a61105afbb \
                   "file://archival/libarchive/bz/LICENSE;md5=28e3301eae987e8cfe19988e98383dae"
```

* 이처럼 소프트웨어 패키지마다 다른 라이선스를 부여할 수 있고, 여러 개의 라이선스 목록을 가질 수도 있다.

* 라이선스를 제공하는 방법은 다음 3가지 방법이 있다.
  - 오픈임베디드 코드에서 기본적으로 제공하는 라이선스를 사용하는 방법
    ```
    LICENSE = "MIT"
    LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"
    ```
  - 라이선스를 가진 오픈 소스를 사용하는 방법, 자체적으로 라이선스를 부여하는 방법
    ```
    LICENSE = "MIT"
    LIC_FILES_CHKSUM = "file://COPYING;md5=80cade1587e04a9473701795d41a4f0c"
    ```

## 레시피 확장 파일

* 위의 hello.bb 레시피 파일을 bitbake로 빌드하는 것까지 수행하였지만 빌드 결과로 나온 실행 파일을 실행할 수는 없다.
  - 실행 파일을 호스트가 아닌 특정 타깃 시스템에서 실행되도록 만들었기 때문이다.
  - 타깃에서 실행 파일이 실행되려면 생성된 실행 파일이 루트 파일 시스템에 포함되어야 한다.
  - 실행 파일을 생성하는 레시피 이름을 IMAGE_INSTALL 변수에 추가해야 한다.
  - IMAGE_INSTALL 변수는 루트 파일 시스템 이미지를 생성하는 레시피인 core-image-minimal.bb 파일 내에서만 사용해야 한다.
  - 그러나 오픈임베디드 코어 디렉토리인 meta에 존재하는 core-image-minimal.bb 파일을 수정하는 것은 바람직하지 않다.
  - 그래서 재정의(오버라이드)를 위해 레시피 확장 파일을 만들 것이다. (.bbappend 파일의 목적)
  - 레시피 확장 파일은 기존 레시피 파일보다 우선순위가 높아야 한다. (숫자가 커질수록 우선순위가 높아짐)

## 레시피 확장 파일을 통한 hello 실행 파일 추가

* 기존 core-image-minimal.b 파일은 다음 경로에 존재한다.
  - `poky/meta/recipes-core/images/core-image-minimal.bb`

* hello 실행 파일을 루트 파일 시스템에 추가하는 방법은 이미지 생성 레시피 파일에서 `IMAGE_INSTALL += "hello"`와 같이 처리하면 된다.
  - 다음과 같이 `recipes-core/images` 디렉토리 밑에 레시피 확장 파일 `core-image-minimal.bbappend`를 생성한다.

```
poky_src/
|- poky
    |- meta-hello
        |- conf
        |   |- layer.conf
        |- recipes-core
        |   |- images
        |       |- core-image-minimal.bbappend
        |- recipes-hello
            |- hello.bb
            |- source
                |- COPYING
                |- hello.c
```

copre-image-minimal.bbappend
```
IMAGE_INSTALL_append =" hello"
```

* 레시피 확장 파일을 인식할 수 있도록 `meta-hello/conf` 디렉토리 아래의 layer.conf 파일을 다음과 같이 수정한다.

layer.conf
```
BBPATH =. "${LAYERDIR}:"
BBFILES += "${LAYERDIR}/recipes*/*.bb \
            ${LAYERDIR}/recipes*/*/*.bbappend"
BBFILE_COLLECTIONS += "hello"
BBFILE_PATTERN_hello = "^${LAYERDIR}/"
BBFILE_PRIORITY_hello = "10"
LAYERSERIES_COMPAT_hello = "${LAYERSERIES_COMPAT_core}"
```

* 이제 hello.bb 레시피를 재빌드하고 이미지 생성 레시피 파일인 core-image-minimal.bb에서 루트 파일 시스템을 새로 생성해 주는 태스크인 rootfs를 실행하도록 한다.

```
$ bitbake hello -c cleanall && bitbake hello
$ bitbake core-image-minimal -C rootfs
```

* 실행 파일 hello가 루트 파일 시스템에 잘 들어갔는지 확인해본다.
  - `poky_src/build/tmp/work/qemux86_64-poky-linux/core-image-minimal/1.0-r0/rootfs/usr/bin`

* QEMU 에뮬레이터를 실행한 후 hello를 실행해본다.
  - `$ cd poky/scripts`
  - `$ runqemu core-image-minimal nographic`
  - root로 로그인한다.
  - `root&qemux86-64:~# hello`

* 레시피 확장 파일 목록 보기
  - bitbake-layers show-appends 명령어를 이용하여 모든 레시피 확장 목록을 확인할 수 있다.
  - `$ bitbake-layers show-appends | grep "core-image-minimal"`
  - 이렇게 하면 core-image-minimal 레시피의 레시피 확장 파일을 확인할 수 있다.

* 다음 명령어는 여러 레이어에서 사용된 메타데이터들을 단일 계층 디렉토리로 만들어 준다.
  - `$ bitbake-layers flatten result_recipes`  # result_recipes: 결과가 저장되는 디렉토리 이름
  - 다음과 같이 여러 레이어 계층을 합쳐 평면화함
  - 쉽게 말해 core-image-minimal.bb와 core-image-minimal.bbappend가 합쳐짐

```
result_recipes/
|- classes
|- conf
|- files
|- lib
|- recipes-bsp
|- recipes-connectivity
|- recipes-core
|- recipes-devtools
|- recipes-extended
|- recipes-gnome
|- recipes-graphics
|- recipes-hello
|- recipes-kernel
|- recipes-multimedia
|- recipes-rt
|- recipes-sato
|- recipes-support
|- site
|- wic
```

...

# 초기화 관리자 추가

...

# 로그 파일을 통한 디버깅

...

# 오픈임베디드 코어 클래스 기능을 사용한 빌드 최적화

...

# 의존성

...

# 패키지 그룹 및 빌드 환경 구축

...

# Poky 배포를 기반으로 한 커스텀 이미지, BSP 레이어 작성

...

# 커널 레시피

...

# 커널 레시피 확장

...

# 배포 레이어

...

# 커스터머 레이어

...

# 패키지

...

# 패키지 설치 실행을 위한 태스크

...

# 공유 상태 캐시와 시그니처

...

# kirkstone

...

# SDK (Software Development Kit)

...

# 기타

...

# devtool

...
