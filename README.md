# Yocto 프로젝트란 무엇인가?

* 임베디드 장치용 맞춤형 배포판 빌드를 위한 오픈 소스 프로젝트
  - 커스텀 리눅스를 만드는 도구는 Yocto 프로젝트 외에도 Linux Live Kit, Linux From Scracth(LFS), Live Magic, SUSE Studio Express 등이 있다.
  - OpenEmbedded 프로젝트 안에 Bitbake가 포함되어 있었고, Yocto 프로젝트는 Poky와 더불어 OpenEmbedded까지 아우르는 거대한 프로젝트라고 볼 수 있다.
  - [The Yocto Project](https://www.yoctoproject.org/) 및 [Yocto Project 문서](https://docs.yoctoproject.org/) 사이트 링크를 참조하십시오.

* 내가 개발하고자 하는 임베디드 환경(ARM, x86 등)에 맞게 커스텀 리눅스를 빌드해주는 도구 (셋팅의 번거로움을 줄여주는 역할)
  - Yocto: 오픈 임베디드 빌드 시스템이 리눅스 소프트웨어 스택을 빌드하는 데 필요한 모든 정보를 제공함 (크로스 컴파일러, 라이브러리 등)
  - bitbake: Yocto에서 제공하는 정보를 기반으로 빌드를 수행하는 빌드 도구
  - Poky: 빌드를 위해 필요한 소스 코드를 git에서 가져오거나, 빌드 환경 설정, 컴파일, 생성된 이미지를 설치하는 방법을 기술하는 .bb 파일(메타데이터)이 들어 있음

* 전체적인 맵은 다음과 같다.
  - 사용자는 조리법에 해당하는 것만 작성하면 된다. [conf (환경 설정 파일), bb (레시피 파일), bbclass (클래스 파일), bbappend (레시피 확장 파일), inc (인클루드 파일)]
  - 재료 역할을 하는 meta 안에는 HW, SW에 대한 모든 정보가 들어 있다. (링크만 있음, 실질적인 정보는 다운로드해야 함)
  - bitbake가 알아서 조리법을 기반으로 결과물을 만들어준다.
  - 결과물로 루트 파일 시스템 이미지, 커널 이미지, 부트 로더 이미지가 나온다.

* Yocto 프로젝트 작동 절차는 다음과 같다.
  1. Poky 레퍼런스 시스템 준비 (download, 환경 설정)
  2. 타깃 보드에 맞는 BSP 레이어 생성
  3. 메타레이어 생성
  4. 레시피 토대로 다운로드, 패치(patch), 구성(configure), 빌드(컴파일 및 링크)
  5. 설치(install)
  6. 패키지 생성
  7. 부트 이미지 생성

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/b5c2521f-2a8d-44c8-a260-11c4d1df0743)

## 실습 환경

* 예제는 ubuntu 18.04, Yocto dunfell 버전을 기반으로 설명할 것이다. (Yocto, 쉽게 이해하고 깊게 다루기 -조운래 저- 참조)

## Yocto 시작 전에 설치해야 할 기본 패키지

* ubuntu에서 패키지 설치하기

```$ sudo apt install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 xterm make xsltproc docbook-utils fop dblatex xmlto source git tree python3.8```

* 실습용으로 Ubuntu와 위의 패키지가 모두 설치된 [sourceforce 사이트](https://sourceforge.net/projects/greatyocto/files/)에서 installed ubuntu great yocto.ova 가상 파일을 다운로드해서 사용해도 된다. (계정: great / PW: great) VirtualBox 7.0.8 버전 이상, Extension Pack Manager에서 확장 팩을 설치하는 것을 권장한다. 권장 사양은 CPU 8 Core, RAM 8192MB 이다.
  - VirtualBox 및 확장팩 다운로드 경로: [여기](https://www.virtualbox.org/wiki/Downloads)에서 VirtualBox x.x.x platform packages와 VirtualBox x.x.x Oracle VM VirtualBox Extension Pack을 받아서 설치하면 된다.

# bitbake

* bitbake: 파이썬, 셸 스크립트 혼합 코드를 분석하는 작업 스케줄러, 임베디드 리눅스의 크로스 컴파일을 위한 패키지 및 관련 파일을 빌드하는 데 사용되는 도구
  - GNU Make가 Makefile을 사용하는 것처럼, bitbake는 .bb 파일(메타데이터)을 사용하여 빌드한다.
  - Poky 안에 포함되어 있지만 단독으로도 여러 가지를 수행할 수 있음

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/f02a4438-64b0-4919-ac20-66f3744809af)

## bitbake 설치 및 실행

※ 이것은 연습입니다. Poky 챕터부터 보시면 됩니다.

* 소스 받기
  - dunfell 버전: `$ wget http://git.openembedded.org/bitbake/snapshot/bitbake-1.46.0.tar.gz`

* 압축 풀기: `$ tar -xzf bitbake-1.46.0.tar.gz`
  - bitbake-1.46.0/bin 디렉토리에 bitbake 실행 파일이 존재함

* 실행하기: `$ export PATH=/home/user/bitbake_src/bitbake-1.46.0/bin:$PATH`
  - bitbake-1.46.0/bin 디렉토리를 $PATH에 추가할 것
  - `$ bitbake --version` 버전이 제대로 출력되면 성공한 것

## 메타데이터

* 메타데이터는 소프트웨어를 어떻게 빌드할지, 그리고 빌드하려는 소프트웨어들 간에 어떤 의존성이 있는지 기술하고 있음 (환경 설정 파일은 변수만 있고, 나머지는 변수와 함수(task)가 있음)
  - 환경 설정 파일(.conf): 전역 변수 집합 (bitbake.conf는 메인 환경 설정 파일이며, 그 외에 레시피 별로 conf 디렉토리 안에 있음)
  - 클래스 파일(.bbclass): 여기에 선언된 변수, 함수는 해당 클래스 파일을 상속(inherit)한 레시피에서만 사용할 수 있음 (base.bbclass은 전역적임, classes 디렉토리 안에 있음)
  - 레시피 파일(.bb): 소프트웨어를 어디서 다운로드할지, 받은 소프트웨어를 어떻게 빌드할지, 빌드된 산출물을 어디에 위치할지 기술되어 있음
  - 레시피 확장 파일(.bbappend): 레시피 파일에서 선언된 변수, 함수를 재정의하거나 확장할 수 있게 해주며, 레이어 개념을 알아야 함
  - 인클루드 파일(.inc): 클래스 파일은 공식적인 기능을 제공하는 반면, 인클루드 파일은 비공식적인 내용을 공유할 때 사용함
 
* bitbake는 메타데이터를 활용하여 루트 파일 시스템 이미지, 커널 이미지, 부트로더 이미지를 생성함

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
~$ mkdir poky_src
~$ cd poky_src
~/poky_src$ git clone git://git.yoctoproject.org/poky
~/poky_src$ git checkout dunfell
```

* 디렉토리 구조는 다음과 같다.

`~$ tree -d -L 2 poky_src/`

```
poky_src/
|- poky
    |- bitbake
    |- contrib
    |- documentation
    |- meta            # 오픈임베디드 코어 레이어
    |- meta-poky       # Yocto 배포 참조 레이어
    |- meta-selftest   # oe-selftest 스크립트가 사용하는 bitbake 테스트 레이어
    |- meta-skeleton   # 소프트웨어 레이어를 생성하는 데 사용되는 템플릿 레이어
    |- meta-yocto-bsp  # Yocto 프로젝트의 BSP 레이어
    |- scripts
```
  
* Poky 소스 빌드하기
  - poky_src 디렉토리에서 실행한다: `~/poky_src$ source poky/oe-init-build-env`
  - 실행 후에는 현재 작업 디렉토리 위치가 build 디렉토리로 변경된다.
  - 빌드를 실행하여 Yocto에서 제공된 커스텀 리눅스 이미지를 만든다: `~/poky_src/build$ bitbake core-image-minimal -k` (-k 옵션은 오류가 발생하더라도 끝까지 빌드를 계속 하라는 뜻) [여기서, 레시피 core-image-minimal은 다른 것이 될 수 있음]
  - 레시피 파일에서 사용하는 모든 환경 변수를 확인하는 방법: `~/poky_src/build$ bitbake core-image-minimal -e > env.txt` (메타데이터 분석 절차를 수행한 결과로 얻어진 변수, 함수를 env.txt로 저장)
  - `~/poky_src/build$ bitbake-getvar -r core-image-minimal DL_DIR`: 위와 비슷함 (주로 이것을 사용함), 이렇게 하면 DL_DIR 변수의 할당 과정을 상세하게 볼 수 있음

* oe-init-build-env 스크립트
  - 기본 빌드 환경을 설정한다.
  - `source oe-init-build-env`를 실행해야 bitbake 등 빌드 명령어를 실행할 수 있음
  - `~/poky_src$ source poky/oe-init-build-env`를 실행하면 다음과 같은 conf 파일이 생성된다.

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/fc151e20-85c6-427f-b5f7-cbad387ba030)

```
poky_src/
|- build
    |- conf
        |- bblayers.conf        # 생성된 레이어들의 정보를 bitbake에게 알려줌, 레이어들의 경로는 BBLAYERS 변수에 할당됨
        |- local.conf           # 환경 설정 파일, bitbake.conf 파일에서 이 파일을 인클루드해 사용한다. (타깃 머신 지정, 크로스 툴체인 지정, 전역 변수 처리)
        |- templateconf.cfg     # 프로젝트를 생성하는 데 사용되는 템플릿 환경 설정을 포함하는 디렉토리를 포함하고 있음
```

* 커스텀 리눅스 예시
  - core-image-minimal: 타깃 머신이 부팅이 되도록 지원하며, 커널과 부트로더 테스트 및 개발에 유용한 작은 이미지
  - core-image-full-cmdline: 콘솔만 가능한 이미지로 리눅스 시스템의 기능 대부분을 제공함
  - core-image-weston: Wayland 프로토콜 라이브러리와 레퍼런스 Weston 컴포지터를 제공하는 이미지
  - core-image-x11: 터미널을 지원하는 기본적인 x11 이미지
  - core-image-sato: sato를 지원하고 모바일 디바이스를 위한 모바일 환경을 지원하는 X11 이미지. 터미널, 편집기, 파일 매니저, 미디어 플레이어와 같은 애플리케이션을 지원함

* 빌드 결과를 QEMU 에뮬레이터로 실행
  - 먼저 QEMU를 빌드한다: `~/poky_src/build$ bitbake runqemu`
  - 실행하는 방법은 다음과 같다. (비디오 콘솔을 따로 생성하지 않고 위에서 생성한 커스텀 리눅스 이미지를 동작시킴)
  - `~/poky_src$ cd poky/scripts`
  - `~/poky_src/poky/scripts$ runqemu core-image-minimal nographic`
  - 종료 시에는 `# poweroff`를 실행한다.
 
* QEMU(Quick EMUlator)란 무엇인가?
  - QEMU에 기반하는 머신은 실제 타깃 머신 없이 개발 및 테스트가 가능하다. (현재 ARM, MIPS, MIPS64, PowerPC, x86, x86-64 에뮬레이터를 지원함)
  - `$ runqemu <machine> <zimage> <filesystems>`
  - machine: qemuarm, qemumips, qemuppc, qemux86, qemux86-64 등 머신 타입을 지정할 수 있다.
  - zimage: 커널의 경로
  - filesystems: ext3 이미지나 NFS 폴더 경로

## 디렉토리 구조

```
poky_src/
|- poky
    |- bitbake         # bitbake 실행 파일이 들어 있음
    |- build           # source oe-init-build-env를 실행했을 때 생성됨 (${TOPDIR})
    |   |- buildhistory              # 빌드 히스토리가 기록됨
    |   |- conf
    |   |   |- local.conf            # 환경 설정 파일, bitbake.conf 파일에서 이 파일을 인클루드해 사용한다. (타깃 머신 지정, 크로스 툴체인 지정, 전역 변수 처리)
    |   |   |- bblayers.conf         # 생성된 레이어들의 정보를 bitbake에게 알려줌, 레이어들의 경로는 BBLAYERS 변수에 할당됨
    |   |   |- bitbake.conf          # 메인 환경 설정 파일
    |   |   |- templateconf.cfg      # 프로젝트를 생성하는 데 사용되는 템플릿 환경 설정을 포함하는 디렉토리를 포함하고 있음
    |   |- cache
    |   |   |- sanity_info           # 빌드 중에 생성되며 완전성 검사 상태를 의미함
    |   |- downloads                 # 다운로드된 소스 파일이 보관됨 (${DL_DIR})
    |   |- sstate-cache              # 공유 상태 캐시가 포함되어 있음 (${SSTATE_DIR})
    |   |- tmp                       # 모든 빌드 시스템의 출력물이 여기에 보관됨 (${TMPDIR})
    |   |   |- buildstats            # 빌드 통계가 저장됨
    |   |   |- cache                 # 메타데이터(레시피, 환경설정) 구문 분석할 때 결과를 캐시하여 향후 빌드 속도를 높임, 머신별로 결과가 저장됨
    |   |   |- deploy                # 빌드 프로세스의 최종 결과물이 저장됨 (${DEPLOY_DIR})
    |   |   |   |- deb
    |   |   |   |- rpm
    |   |   |   |- ipk
    |   |   |   |- licenses
    |   |   |   |- images
    |   |   |   |- sdk
    |   |   |- sstate-control        # 공유 상태 매니페스트 파일이 저장됨
    |   |   |- sysroots-components   # do_prepare_recipe_sysroot 태스크가 DEPENDS 레시피에 대해 특정 sysroot로 링크하거나 복사하는 sysroot 내용물의 위치  (${COMPONENTS_DIR})
    |   |   |- sysroots
    |   |   |- stamps                # bitbake 작업을 위해 기록하는 태스크별 타임 스탬프를 저장함
    |   |   |- log                   # ${WORKDIR}에 배치되지 않은 일반 로그가 포함되어 있음
    |   |   |- work                  # 빌드한 패키지에 대한 아키텍처별 작업 하위 디렉토리가 포함되어 있음
    |   |   |   |- tunearch          # (${PACKAGE_ARCH}-poky-${TARGET_OS} 또는 ${MACHINE}-poky-${TARGET_OS} 형태를 가짐)
    |   |   |       |- recipename    # 레시피 이름 (${PN})
    |   |   |           |- version   # 레시피 작업 디렉토리 (${WORKDIR}: ${PV}-${PR})
    |   |   |               |- temp                     # 이 레시피에 대해 실행된 각 태스크의 로그 파일, 실행 파일 등이 들어 있음
    |   |   |               |- image                    # do_install 태스크의 출력이 저장됨 (해당 태스크의 ${D})
    |   |   |               |- pseudo                   # 레시피에 대해 의사 실행된 모든 태스크에 대한 의사 데이터베이스 및 로그가 포함되어 있음
    |   |   |               |- sysroot-destdir          # do_populate_sysroot 태스크의 출력이 저장됨
    |   |   |               |- package                  # 개별 패키지로 분할되기 전 do_package 태스크의 출력이 저장됨
    |   |   |               |- packages-split           # 개별 패키지로 분할된 후의 태스크 출력이 저장됨
    |   |   |               |- recipe-sysroot           # 레시피의 대상 종속성이 저장된 디렉토리 (C 라이브러리 등)
    |   |   |               |- recipe-sysroot-native    # 레시피의 기본 종속성이 저장된 디렉토리 (컴파일러, autoconf, libtool 등)
    |   |   |               |- build                    # 소스가 빌드 아티팩트와 분리된 빌드를 지원하는 레시피에만 적용됨 (${B})
    |   |   |- work-shared           # 다른 레시피와 작업 디렉토리를 공유하는 레시피를 저장함 (libgcc, gcc-runtime 등)
    |- contrib
    |- documentation   # Yocto 프로젝트 문서의 소스와 PDF/HTML 버전 매뉴얼을 생성할 수 있는 템플릿 및 도구가 있음
    |- meta            # 오픈임베디드 코어 레이어
    |   |- classes
    |   |   |- base.bbclass    # 전역 클래스 파일
    |   |- conf
    |   |   |- machine
    |   |   |- distro
    |   |   |- machine-sdk
    |   |- files
    |   |- lib
    |   |- recipes-bsp
    |   |- recipes-connectivity
    |   |- recipes-core
    |   |- recipes-devtools
    |   |- recipes-extended
    |   |- recipes-gnome
    |   |- recipes-graphics
    |   |- recipes-kernel
    |   |- recipes-multimedia
    |   |- recipes-rt
    |   |- recipes-sato
    |   |- recipes-support
    |   |- recipes-site
    |- meta-poky       # Yocto 배포 참조 레이어
    |- meta-yocto-bsp  # Yocto 프로젝트의 BSP 레이어
    |- meta-selftest   # oe-selftest 스크립트가 사용하는 bitbake 테스트 레이어 (오픈임베디드 자체 테스트에서 빌드 시스템의 동작을 확인하는 데 사용되는 추가 레시피와 추가 파일이 있음)
    |- meta-skeleton   # BSP, 커널 개발을 위한 템플릿 레이어
    |- meta-custom             # 사용자가 추가한 소프트웨어 레이어
    |   |- conf
    |   |   |- layer.conf      # 레시피 파일의 위치를 알려줌
    |   |- custom.bb           # 레시피 파일
    |- scripts
```

# 빌드 속도 개선하기

* PREMIRRORS(로컬 미러 저장소), sstate-cache(공유 상태 캐시)를 구성하여 소스를 미리 다운로드함으로써 fetch 시간을 단축하여 빌드 속도를 개선하는 것임
  - 소스를 fetch, 즉 다운로드하는 데 시간을 가장 많이 소요한다. (빌드 속도는 로컬 머신의 성능에 달려 있음)
  - Yocto에서 자주 빌드할 경우 미리 소스를 받아두는 편이 시간을 단축하는 데 도움이 많이 된다.
  - 또 경우에 따라서는 원격 저장소에 문제가 생겨서 소스를 받지 못하는 경우도 있기 때문에 미리 받아두는 편이 도움이 될 수 있다.
  - 로컬 미러 저장소는 단순하게 git 소스를 저장해 두는 방식이고, 공유 상태 캐시는 빌드 단계 스냅샷을 저장해 두는 방식이다. (전자를 권장함)

## 소스 다운로드 절차

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/adfcc5d7-bc4b-49fe-81f5-7ca0454bc1c6)

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/11dfc064-224a-4bcb-8022-0c1b783cf3ae)

* 레시피 파일(.bb, .bbappend)의 SRC_URI 변수에 다운로드할 소스의 위치를 지정한다.
* DL_DIR 변수가 가리키는 경로에 소스가 저장된다.
  - 기본값: poky_src/build/downloads (bitbake.conf 파일의 780 라인에 '{TOPDIR}/downloads'라고 지정되어 있음)
  - tar.gz, tar.xz와 같은 압축 파일들과 확장자 .done으로 끝나는 파일들이 존재한다. (done은 소스를 전부 다운로드했다는 표시로 생성되는 파일이다)
  - .done 파일이 확인되면 레시피에서 지정한 S 변수가 가리키는 위치에 받은 소스가 복사된다.
  - (S 변수는 압축 해제, 패치, 컴파일이 진행되는 디렉토리이다)
* 소스 다운로드를 성공했다면 S 변수가 가리키는 디렉토리로 소스를 복사하고, 깃을 통해 소스를 받았다면 체크아웃(특정 브랜치, 태그, 해시, 커밋 등으로 이동)이 발생한다.

## 나만의 소스 저장소 PREMIRRORS 구성하기

* 먼저 소스를 다운로드해야 한다.
  - `~/poky_src_build$ bitbake core-image-minimal`: bitbake <레시피 이름>을 입력하면 소스를 다운로드하고 빌드도 같이 진행한다.
  - `~/poky_src_build$ bitbake core-image-minimal --runall=fetch`: 옵션 --runall=fetch를 붙이면 소스만 다운로드하고 빌드는 하지 않는다. (do_fetch 태스크까지만 수행한다는 뜻)
* 용량을 줄이기 위해 tarball을 생성한다.
  - poky_src/build/conf/local.conf 파일을 열어 `BB_GENERATE_MIRROR_TARBALLS = "1"`로 설정한다.
  - 위에서 이미 소스를 받았으므로 다시 받으려면 다음과 같이 해야 한다.
  - `~/poky_src/build$ rm -rf downloads`
  - `~/poky_src/build$ core-image-minimal --runall=fetch -f`  # 강제로 fetch 진행
  - 깃으로부터 받은 소스는 'downloads/git2'에 저장되지 않고 downloads 디렉토리에 '.tar.gz'로 압축되어 저장된다.
* 자체 소스 미러 구축을 위해 다음과 같이 따라한다.
  - 간혹 "xxx_bad-checksum.."과 같은 파일이 남아 있다면 다시 fetch  명령어를 시도하면 대부분 해결된다. 모두 다운로드 후에 checksum 파일도 삭제한다.
  - build/downloads 디렉토리에서 '.done' 파일을 모두 삭제한다. (`$ rm -rf *.done`)
  - downloads 디렉토리 내에서 git2 디렉토리를 삭제한다. (`$ rm -rf git2`)
* poky/src/source-mirrors 디렉토리를 생성한다. (자체 저장소인 PREMIRRORS 용도로 사용함)
  - downloads 디렉토리에 받은 소스들을 source-mirrors 디렉토리로 복사한다. (`~/poky_src/source-mirrors$ cp -r ../build/downloads/* .`)
  - poky_src/build/conf/local.conf 파일 하단에 다음 내용을 추가한다.
    ```
    ...
    # Specify own PREMIRRORS location
    INHERIT += "own-mirrors"  # poky/meta/classes/own-mirrors.bbclass에 있음
    SOURCE_MIRROR_URL = "file://${COREBASE}/../source-mirrors"  # own-mirrors.bbclass 클래스 파일에서 선언된 변수, COREBASE 변수는 meta 디렉토리의 부모 디렉토리를 가리킴
    ```
* 이제 자체 소스 미러인 PREMIRRORS가 준비되었으며 정상적으로 작동하는지 확인해 본다.
  - `~/poky_src/build$ rm -rf downloads`
  - `~/poky_src/build$ bitbake core-image-minimal -f --runall=fetch`
  - 로컬에 있는 PREMIRRORS를 사용하여 소스를 다운로드하므로 매우 빠르게 소스 fetch가 끝날 것이다.

## 나만의 공유 상태 캐시(Shared State Cache) 생성하기

### 시그니처란 무엇인가?

* 레시피의 각 태스크 수행시 시그니처 값이 생성된다.
  - signaure 값: 태스크 코드, 변수 등 입력 데타데이터로부터 생성됨 (checksum 값이라고도 함)
  - 빌드의 결과를 오브젝트 형태인 공유 상태 캐시를 만들어 특정 디렉토리에 저장함
  - 이미 수행된 작업을 다시 수행하려고 할 때 저장된 시그니처 값과 새로운 시그니처 값을 계산하고 동일할 경우 건너뛴다. (재작업 스킵)
  - 시그니처 값이 동일해 공유 상태 캐시에서 수행된 태스크의 결과를 그대로 가져오는 태스크를 setscene 태스크라고 함

* setscene 태스크 종류

태스크 / setscene 태스크 | 설명
------------------------ | ---
do_packagedata_setscene | 최종 패키지 생성을 위해 빌드 시스템에 의해 사용되는 패키지 메타데이터를 생성함
do_package_setscene | do_install 태스크에 의해 생성된 파일들을 이용할 수 있는 패키지들과 파일들에 근거해 나눈다.
do_package_write_rpm_setscene | RPM 패키지를 생성하고 패키지 피드에 패키지들을 배치시킴
do_populate_lic_setscene | 이미지가 생성될 때 모아 놓은 레시피를 위한 라이선스 정보를 생성한다.
do_populate_sysroot_setscene | 다른 레시피들에 의해 이용될 수 있도록 do_install 태스크에 의해 설치된 파일들을 sysroot로 복사함
do_package_qa_setscene | 패키지로 만들어진 파일들에 대해 QA 검증이 실시됨

* 소스 다운로드 및 빌드를 한 후에 sstate-cache 디렉토리를 확인할 수 있습니다. (`~/poky_src/build/sstate-cache`)
  - 이 디렉토리의 경로를 지정하는 변수는 SSTATE_DIR이다. (~/poky_src/poky/meta/conf/btbake.conf 에 정의되어 있음)
  - SSTATE_DIR: 레시피의 각 태스크 수행 시마다 시그니처와 빌드 결과에 대한 오브젝트가 만들어져서 이곳에 저장된다. (스냅샷 저장 개념)
  - SSTATE_MIRRORS: 공유 상태 캐시의 저장소로 사용되는 디렉토리의 경로를 갖고 있다. (빌드시 이미 수행된 태스크인지 식별되기 위해 이 저장소를 제일 먼저 확인함)

### 공유 상태 캐시(Shared State Cache) 생성하는 방법

* 프로젝트 최상위 디렉토리에 sstate-cache를 만든다. (`~/poky_src$ mkdir sstate-cache`)
* build 디렉토리에 있는 sstate-cache 디렉토리의 모든 파일들을 새로 만든 디렉토리로 복사한다. (`~/poky_src/sstate_cache$ cp -r ../build/sstate-cache/* .`)
* ~/poky_src/build/conf/local.conf 파일 내용 하단에 다음 내용을 추가한다.
  ```
  ...
  # make shared state cache mirror
  SSTATE_MIRRORS = "file://.* file://${COREBASE}/../sstate-cache/PATH"
  ```
* 만약 외부 서버에 구축할 경우 다음과 같이 하면 된다. (dunfell 버전은 라인 끝에 '\n'을 넣어야 하고, kirkstone 버전은 '\n'을 넣어야 함)
  ```
  SSTATE_MIRRORS ?= "\
  file://.* http://<server>/share/sstate/PATH;downloadfilename=PATH \n \
  file://.* file://<local directory>/local/dir/sstate/PATH"
  ```
* 주의할 점은 프로젝트를 진행하며 계속 빌드하다보면 공유 상태 캐시의 크기가 커지기 때문에 중복되거나 필요 없는 것은 삭제해 주어야 한다.

# Poky를 이용하여 레이어, 레시피 생성하기

## Poky 다운로드 및 빌드

* Poky 소스 다운로드

```
~$ mkdir poky_src
~$ cd poky_src
~/poky_src$ git clone git://git.yoctoproject.org/poky
~/poky_src$ git checkout dunfell
```

* Poky 소스 빌드하기
  - poky_src 디렉토리에서 실행한다: `~/poky_src$ source poky/oe-init-build-env`
  - 실행 후에는 현재 작업 디렉토리 위치가 build 디렉토리로 변경된다.
  - 빌드를 실행하여 Yocto에서 제공된 커스텀 리눅스 이미지를 만든다: `~/poky_src/build$ bitbake core-image-minimal -k`

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

* __레이어가 정상적으로 추가되었는지 확인하는 방법은 다음과 같다.__
  - 레이어 이름, 경로, 우선순위가 표시됨
  - `$ bitbake-layers show-layers`

## 예제 실행하기

* 실행하는 방법은 다음과 같다: `$ bitbake <recipe-name>`
  - `$ bitbake hello`

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

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/59f62750-ac57-4a7b-9e7f-25d20e467732)

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/73928095-be41-4af2-bc1c-77f9a8f8ec87)

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

* 기존 GitHub에서 받은 소스에서 다음과 같이 확인할 수 있다.
  - 기존 GitHub에서 받은 소스: `$ git checkout hello_second`
  - 새로 받는 방법: `$ git clone https://GitHub.com/greatYocto/poky_src.git -b hello_second`

* 기존 core-image-minimal.bb 파일은 다음 경로에 존재한다.
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

core-image-minimal.bbappend
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

* __다음 명령어는 여러 레이어에서 사용된 메타데이터들을 단일 계층 디렉토리로 만들어 준다.__
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

# 초기화 관리자

* 초기화 관리자: 리눅스 시스템의 부팅 후 가장 먼저 생성되고 다른 프로세스를 실행하는 init 데몬 (예시: System V Init, systemd 등)
  - 앞에서는 hello 실행 파일을 만들어 루트 파일 시스템에 추가하고 타깃 시스템을 부팅시켜 콘솔 창에서 "hello"를 실행해 보았다.
  - 초기화 관리자를 이용하면 자동으로 프로세스를 실행할 수 있다.
  - Yocto의 경우 기본 초기화 관리자로 System V Init을 사용한다.
  - 대부분의 현업 리눅스 시스템은 systemd를 사용하므로 이번 장에서는 systemd만을 다룰 것이다.
  - systemd는 백그라운드에서 실행되는 데몬(프로세스)이며 부모 프로세스를 갖지 않고 PID 1을 갖는다.
  - systemd는 System V Init와 달리 컴포넌트들을 유닛(unit)으로 다루며 최소한의 서비스들을 병렬로 실행한다.
  - 유닛의 종류는 다음과 같다: .service, .socket, .device, .mount, .automount, .swap, .target, .path, .timer, .snapshot, .slice, .scope
  - 이번 장에서는 .service 유닛에 대해서만 다룰 것이다. (/lib/systemd/system/에 있음)

systemd의 구조
![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/ca96d170-835a-4eae-a1e7-1da42dd584c3)

## .service 유닛 파일 내에서의 섹션 종류

* Unit 섹션: 최초로 동작하는 섹션, 다른 unit 간의 관계를 설정함
  - Description: unit에 대한 설명.
  - Documentation: 서비스에 대한 문서가 있는 URI 또는 main 페이지 제공.
  - After: 현재 unit보다 먼저 실행되어야 하는 unit들을 나열함.
  - Before: 현재 unit보다 나중에 실행되어야 하는 unit들을 나열함.
  - Requires: 현재 unit이 의존하는 모든 unit들을 나열함. (모두 현재 실행 중이어야 함)
  - Wants: Requires와 동일한 기능. (실행 중이지 않아도 현재 unit이 실행될 수 있음)
  - BindsTo: 여기에 나열된 unit들이 종료되면 현재 서비스도 같이 종료됨.
* Service 섹션
  - Type: 서비스가 어떤 형태로 동작되는지 설정함
    1. simple: 기본 방식. 이 경우 ExecStart가 기술되어야 함. 해당 unit이 시작되면 systemd는 unit의 시작이 완료됐다고 판단함.
    2. forking: 서비스가 자식 프로세스를 생성할 때 사용함. PIDFile 값에 PID 파일을 지정해야 함. (부모 프로세스를 추적하는데 필요함)
    3. oneshot: 서비스가 시작되면 상태를 activating으로 바꾸고 끝날 때까지 기다린다. (단기간 프로세스에 사용함)
    4. dbus: DBUS에 지정된 BusName이 준비될 때까지 대기하며, DBUS가 준비되면 서비스가 실행됨.
    5. notify: 서비스가 startup이 끝날 때 systemd에 시그널을 보냄. 이 시그널을 받은 systemd는 다음 unit을 실행함.
    6. idle: 모든 서비스가 실행된 이후에 실행됨.
    7. ExecStart: 서비스를 시작하기 위한 전체 경로. 서비스 시작 전에 이 명령어를 실행해야 함.
    8. ExecStartPre: 서비스가 시작되기 전에 실행할 명령.
    9. ExecStartPost: 서비스가 시작되고 나서 실행할 명령.
    10. Restart: 서비스가 종료됐을 때 자동으로 시작하게 함. (always, on-success, on-failure, on-abnormal, on-abort, on-watchdog 등)
  - TimeoutStartSec: 서비스 시작시 대기하는 시간.
* Install 섹션
  - WantedBy: `# systemctl enable` 명령어로 유닛을 등록할 때 등록에 필요한 유닛을 지정함.
  - Also: `# systemctl enable`, `# systemctl disable` 명령어로 unit을 등록/해제할 때 함께 enable, disable할 unit들을 지정함.
  - Alias: 유닛의 별칭.

## systemd를 통한 hello 애플리케이션 실행

* 다음과 같이 소스를 받는다.
  - 기존 GitHub에서 받은 소스: `$ git checkout systemD`
  - 새로 받는 방법: `$ git clone https://GitHub.com/greatYocto/poky_src.git -b systemD`

* 다음과 같은 파일이 있어야 한다.

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
                |- hello.service
```

hello.bb
```
DESCRIPTION = "Simple helloworld application example"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://COPYING;md5=80cade1587e04a9473701795d41a4f0c"

SRC_URI = "file://hello.c"
SRC_URI_append = " file://COPYING"
SRC_URI_append = " file://hello.service"  # systemd가 실행할 서비스 파일을 추가함
inherit system
S = "${WORKDIR}"
SYSTEMD_SERVICE_${PN} = "hello.service"  # .service 파일을 등록하는 데 필요함
SYSTEMD_AUTO_ENABLE = "enable"           # 부팅시 systemd에 추가한 서비스 파일이 자동적으로 실행되도록 등록함

do_compile(){
    ${CC} hello.c ${LDFLAGS} -o hello
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 hello ${D}${bindir}
    install -d ${D}${systemd_unitdir}/system                       # .service 파일을 '/lib/systemd/system'에 위치시킴
    install -m 0644 hello.service ${D}${systemd_unitdir}/system    # .service 파일을 '/lib/systemd/system'에 위치시킴
}
FILESEXTRAPATHS_prepend := "${THISDIR}/source:"
FILES_${PN} += "${bindir}/hello"
FILES_${PN} += "${systemd_unitdir}/system/hello.service"
```

hello.service
```
[Unit]
Description=Hello World startup script

[Service]
ExecStart=/usr/bin/hello

[Install]
WantedBy=multi-user.target
```

* Yocto는 기본 초기화 관리자가 System V Init이기 때문에 systemd로 변경해야 한다.

~/poky_src/build/conf/local.conf
```
...
DISTRO_FEATURES_append = " systemd"
DISTRO_FEATURES_remove = "sysvinit"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = "systemd-compat-units"
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"
VIRTUAL-RUNTIME_initscript = "systemd-compat_units"
```

hello.c 수정 (sleep 함수를 주석처리함)
```
#include <stdio.h>
#include <unistd.h>

int main(){
    int i = 0;
    while (i < 10) {
        printf ("Hello world!\n");
//      sleep(5);
        i++;
    }
    return 0;
}
```

* 변경된 코드 실행

```
$ bitbake hello -C fetch
$ bitbake core-image-minimal -C rootfs
$ runqemu core-image-minimal nographic
```

* QEMU에서 로그인한 후에 다음과 같이 실행해본다.
  - poky/meta/classes/systemd.bbclass 파일: do_install 태스크가 완료되면 98-hello.preset이라는 사전 설정 파일이 생성됨
  - 이 파일은 systemd_postinist() 함수에서 실행되는 구조로 되어 있음
  - 타깃이 처음 부팅될 때 hello.service를 자동으로 실행함

```
# journalctl -u hello -f &    # hello 서비스에서 출력되는 메시지를 실시간으로 출력하려면 백그라운드에서 동작하라는 명령어이다.
# systemctl stop hello
# systemctl start hello
```

# 로그 파일을 통한 디버깅

* 빌드가 실패했을 때 bitbake가 생성하는 로그 파일을 통해 실패의 원인을 찾는 것이 가장 유용하고 쉬운 방법이다.
  - 오픈임베디드 빌드 시스템에서 제공하는 로그 파일 생성 방법과 디버깅 스킬을 학습한다.

* bitbake를 실행할 때 백그라운드로 쿠커(cooker)라는 백엔드 프로세스가 실행된다.
  - 쿠커 프로세스: 실제 빌드와 더불어 모든 메타데이터 파일 처리를 수행하고 다수의 쓰레드를 실행한다.
  - bitbake 클라이언트(knotty)가 IPC(Inter Process Communication)를 통해 빌드 출력, 빌드 상태, 빌드 진행 상황을 로깅한다.
  - 쿠커의 로그는 `~/poky_src/build/tmp/log/cooker/qemux86-64`에 기록된다.
  - console-latest.log 파일을 열어보면 bitbake 버전, 호스트 빌드 플랫폼, 타깃 플랫폼 등을 알 수 있다.

* 태스크의 로그 파일, 스크립트 파일의 저장 위치는 T 변수에 할당되어 있다. (기본값: T = "${WORKDIR}/temp")
  - 로그 파일은 log.do_<taskname>.<pid> 형식으로 되어 있다. (오류 분석을 위해 이것을 보면 됨)
  - 스크립트 파일은 run.do_<taskname> 형식으로 되어 있다. (로깅 데이터를 추가하여 디버깅 수행하는 데 도움이 됨)

# 유용한 오픈임베디드 코어 클래스 기능을 사용한 빌드 최적화

## Autotools를 이용한 nano editor 빌드

* GNU Autotools: GNU 빌드 시스템으로 UNIX 기반 시스템에서 소스 코드를 빌드하는 데 도움을 주는 빌드 툴이다.
  - autoconf: configure.ac 파싱 --> configure 스크립트 생성 및 실행 --> 최종 Makefile 생성
  - automake: Makefile.am 파싱 --> Makefile.in 생성 --> configure 파일에서 Makefile 생성시 참고
  - libtool: 라이브러리 생성 처리

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout nano`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b nano`
  - git으로부터 nano editor tarball 파일 받는 방법: `~/tmp$ wget https://www.nano-editor.org/dist/v6/nano-6.0.tar.gz`

### 실습 순서

* 레시피 파일을 만들기 전에 라이선스 파일 checksum 값 계산
  - `~/tmp$ md5sum nano-6.0.tar.gz`  # 다운로드 받은 tarball 파일의 md5sum 값을 구함
* 다운로드 받은 파일의 압축 풀기
  - `~/tmp$ tar -xvf nano-6.0.tar.gz`
* COPYING, COPYING.DOC 파일의 라이선스 checksum 값 계산
  - `~/tmp/nano-6.0$ md5sum COPYING`
  - `~/tmp/nano-6.0$ md5sum COPYING.DOC`
* 새로운 메타 레이어 생성 (여기서부터 해도 된다. checksum 값을 기입하지 않으면 bitbake가 오류 메시지에서 알려준다)
  ```
  poky_src/
  |- build
  |   |- conf
  |       |- bblayers.conf
  |- poky
      |- meta-nano-editor
          |- conf
          |    |- layer.conf
          |- recipes-nano
               |- nano_6.0.bb
  ```
  * 파일 내용을 다음과 같이 작성한다.
    - layer.conf
      ```
      BBPATH =. "${LAYERDIR}:"
      BBFILES += "${LAYERDIR}/recipes*/*.bb"
      BBFILE_COLLECTIONS += "nano-editor"
      BBFILE_PATTERN_nano-editor = "^${LAYERDIR}/"
      BBFILE_PRIORITY_nano-editor = "10"
      LAYERSERIES_COMPAT_nano-editor = "${LAYERSERIES_COMPAT_core}"
      ```
    - nano_6.0.bb
      ```
      DESCRIPTION = "Nano editor example"
      LICENSE = "GPLv3"
      LIC_FILES_CHKSUM = "file://COPYING:;md5=f27defe1e96c2e1ecd4e0c9be8967949 \
                          file://COPYING.DOC;md5=ad1419ecc56e060eccf8184a87c4285f"
      SRC_URI = "https://www.nano-editor.org/dist/v6/nano-6.0.tar.gz"
      SRC_URI[md5sum] = "191152bb1d26cefba675eb0e37592c4e"
      DEPENDS = "ncurses"  # 의존성
      inherit gettext pkgconfig autotools
      ```
    - bblayers.conf
      ```
      POKY_BBLAYERS_CONF_VERSION = "2"
      BBPATH = "${TOPDIR}"
      BBFILES ?= ""
      BBLAYERS ?= " \
          /home/user/poky_src/poky/meta \
          /home/user/poky_src/poky/meta-poky \
          /home/user/poky_src/poky/meta-yocto-bsp \
          /home/user/poky_src/poky/meta-hello \
          /home/user/poky_src/poky/meta-nano-editor \
          "
      ```
* 빌드하기
  - `~/poky_src/build$ bitbake nano`
  - 빌드가 완료되면 다음 위치에 실행 파일이 생성됨 (`~/poky_src/build/tmp/work/core2-64-poky-linux/nano/6.0-r0/image/usr/bin`)
* 로그 파일 확인하기
  - `poky_src/build/tmp/work/core2-64-poky-linux/nano/6.0-r0/temp/log.task_order' 파일을 확인한다.
  - nano editor의 소스 위치를 bitbake에게 알려주기만 했지만 bitbake는 최종 바이너리까지 생성했다.
* 이미지를 생성하기 위해 .bbappend(레시피 확장 파일) 만들기
  - ~/poky_src/poky/meta-nano-editor/recipes-core-images/core-image-minimal.bbappend 생성
    ```
    IMAGE_INSTALL += "nano"
    ```
  - ~/poky_src/poky/meta-nano-editor/conf/layer.conf 수정
    ```
    BBPATH =. "${LAYERDIR}:"
    BBFILES += "${LAYERDIR}/recipes*/*.bb"
    BBFILES += "${LAYERDIR}/recipes*/*/*.bbappend"  # 새로 추가됨
    BBFILE_COLLECTIONS += "nano-editor"
    BBFILE_PATTERN_nano-editor = "^${LAYERDIR}/"
    BBFILE_PRIORITY_nano-editor = "10"
    LAYERSERIES_COMPAT_nano-editor = "${LAYERSERIES_COMPAT_core}"
    ```
  - 이미지 생성 및 QEMU 실행
    ```
    $ bitbake nano -c cleanall && bitbake nano
    $ bitbake core-image-minimal -C rootfs
    $ runqemu core-image-minimal nographic
    ```
  - QEMU 부팅 후 다음 커맨드를 입력하면 nano editor가 실행됨
    ```
    # nano
    ```

## 빌드 히스토리

* 빌드 히스토리: 이미지의 변경점을 추적하고 변경된 이미지가 수행한 빌드 절차를 이전과 비교할 수 있는 기능
  - ~/poky_src/build/conf/local.conf 파일에 다음 내용을 추가한다.
    ```
    ...
    INHERIT += "buildhistory"    # buildhistory.bbclass 클래스를 상속함 (INHERIT은 .conf(환경 설정)에서 사용하고, inherit은 .bb(레시피)에서 사용함)
    BUILDHISTORY_COMMIT = "1"    # git 리포지토리에 빌드 히스토리를 커밋하도록 함 (0은 마지막 빌드에 관련된 정보만 저장함)
    BUILDHISTORY_COMMIT_AUTHOR = "Soonbum Jeong <peacemaker84@gmail.com>"
    BUILDHISTORY_DIR = "${TOPDIR}/buildhistory"    # buildhistory.bbclass가 빌드 히스토리를 저장하는 디렉토리 경로
    BUILDHISTORY_IMAGE_FILES = "/etc/passwd /etc/group"    # 특정 파일의 내용을 추적할 수 있도록 함 (여기서는 사용자 및 그룹 항목의 변경을 모니터링함)
    ```
  - 이미지 생성 및 QEMU 실행
    ```
    $ bitbake nano -c cleanall && bitbake nano
    $ bitbake core-image-minimal -C rootfs
    $ runqemu core-image-minimal nographic
    ```
  - bitbake를 실행하면 `poky_src/build/buildhistory` 디렉토리가 생성된다. (각 디렉토리 내 텍스트 파일을 통해 빌드에 대한 정보를 확인할 수 있다. (자세한 내용은 공식 문서 참조)

## rm_work.bbclass를 통한 빌드 임시 파일 제거하기

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout rm_work`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b rm_work`

* rm_work.bbclass: 오픈임베디드 빌드 시스템이 빌드 작업을 끝내고 작업한 파일들을 삭제하는 기능을 수행하는 태스크
  - 빌드 과정에서 생성된 불필요한 파일들을 모두 저장할 필요가 없을 때 이 기능을 사용한다.
  - 이 기능을 사용하려면 레시피(.bb) 또는 레시피 확장 파일(.bbappend)에서 `inherit rm_work`를 추가한다.
  - 만약 모든 레시피들에서 rm_work 클래스를 상속 받으려면 local.conf 환경 설정 파일에 `INHERIT += "rm_work"`를 추가한다.
  - `du -sh <디렉토리>` 명령어를 통해 rm_work 적용 전/후의 빌드 작업 디렉토리의 크기를 비교해 본다.
    (효과는 어마어마하다! 수백, 수천배 차이... 대신 빌드를 여러 번 할 경우 속도 저하는 피할 수 없다.)

## externalsrc를 이용한 외부 소스로부터 소스 빌드

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout externalsrc`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b externalsrc`

* 실수로 clean, fetch, unpack, patch 등의 태스크를 수행하면 다시 소스를 새롭게 받아오므로 수정한 소스가 사라지는 문제가 있다.
  - 이런 문제를 피하기 위해 externalsrc.bbclass라는 클래스를 사용한다. (fetch, unpack, path 태스크 생략함)
  - 변경하고자 하는 소스만 따로 로컬에 저장했다가 편집하고 즉시 빌드할 수 있도록 해준다.
  - 이 방법은 내가 개발하는 나만의 소스를 로컬에서 따로 작업하고 있을 때 주로 사용한다.

* 참고로 소스 코드를 받는 방법에 따라 로컬에 위치는 장소가 다르다. (예: nano editor)
  소스 받는 방법 | SRC_URI 표현식 | ${S} 값
  -------------- | ------------- | ---------
  git에서 다운로드 | SRC_URI = "git://GitHub.com/greatYocto/\bbexample.git;protocol=https;branch=master" | S = "${WORKDIR}/git"
  tarball 다운로드 | SRC_URI = "https://www.nano-editor.org/\dist/v2.7/nano-${PV}.tar.xz" | S = "${WORKDIR}/${BPN}-${PV}"
  로컬 파일 사용 | SRC_URI = "file://ldconfig-native-2.12.1.tar.gz2 | S = "${WORKDIR}

* 외부 소스 빌드시 발생할 수 있는 문제점
  - 소스 디렉토리가 bitbake 작업 디렉토리 아래 있으므로 레시피가 업데이트 되어서 do_fetch 태스크를 수행하거나 `$ bitbake nano -C fetch' 명령을 강제로 수행할 경우 기존 변경된 코드는 모두 삭제된다.
  - 소스 코드가 바뀐 경우 do_unpack 태스크부터 다시 수행하므로, 수정된 코드를 보존하려면 빌드에 적용하려면 반드시 `$ bitbake -C compile`과 같이 명령을 입력해야 한다.
  - 코드 수정 후 빌드를 수행하면 불필요한 태스크들이 추가 수행되므로 빌드 시간이 오래 걸린다.
  - 의도적으로, 혹은 실수로 build 디렉토리가 삭제되면 변경된 코드가 사라지게 된다.

* externalsrc.bbclass 클래스의 기능
  - 빌드 대상인 소스를 build 디렉토리 아래의 작업 디렉토리인 WORKDIR에서 찾지 않고 내가 직접 지정한 로컬 디렉토리에서 찾는 방법을 제공한다. (외부 소스를 빌드 대상으로 삼을 수 있음)
  - 소스 코드를 로컬에 저장했기 때문에 do_fetch, do_unpack, do_patch 태스크 수행이 생략된다.

### 실습 순서

* ~/poky_src/source/nano 디렉토리로 nano editor 소스 코드를 복사한다.
  - `~/poky_src/source/nano$ cp -r ~poky_src/build/tmp/work/core2-64-poky-linux/nano/6.0-r0/nano-6.0/* .`
  - 특정 레시피에서 사용된 소스 코드의 위치는 S 변수에 저장된다. (`$ bitbake-getvar -r nano S`)
* 기존에 만든 레시피 확장 파일 nano_6.0.bbappend에 다음 내용을 추가한다. (rm_work 클래스는 꺼야 함)
  ```
  # inherit rm_work
  inherit externalsrc
  EXTERNALSRC = "${COREBASE}/../source/nano"
  ```
* 혹은 환경 설정 파일에서 externalsrc 클래스를 넣고 싶으면 local.conf 파일을 수정하면 된다.
  ```
  INHERIT += "externalsrc"
  EXTERNALSRC_pn-nano = "${COREBASE}/../source/nano"
  EXTERNALSRC_BUILD_pn-nano = "${COREBASE}/../source/nano"
  ```
* 재빌드를 수행한다. (`$ bitbake nano -c cleanall && bitbake nano`)
  - 이렇게 하면 기존에 저장된 소스는 사라져 있다. (`poky_src/build/tmp/work/core2-64-poky-linux/nano/6.0-r0/nano-6.0/src`)
  - 또 log.task_order 파일을 열어보면 fetch, unpack, patch 태스크 실행이 모두 생략되어 있는 것을 볼 수 있다. (`poky_src/build/tmp/work/core2-64-poky-linux/nano/6.0-r0/temp/log.task_order`)

# 의존성

* 의존성에는 2가지 종류가 있다.
  - 빌드 의존성: 빌드시에 필요한 라이브러리가 있을 경우 (DEPENDS)
  - 실행시간 의존성: 패키지가 동작하기 위해 미리 설치해야 하는 패키지가 있을 경우 (RDEPENDS)

## 레시피 파일에서의 빌드 의존성 표현
  
* `DEPENDS = "ncurses"` (예시: nano editor 레시피)
  - 위는 bitbake 내부적으로 `do_prepare_recipe_sysroot[deptask] = "do_populate_sysroot"`를 의미한다.
  - 이는 nano editor 레시피 파일의 do_prepare_recipe_sysroot 태스크가 ncurses 레시피의 do_populate_sysroot 태스크에 의존하고 있다는 뜻이다. (`meta/classes/staging.bbclass` 참조)
  - ncurse의 do_populate_sysroot 태스크가 실행되면 결과물 recipe-sysroot, recipe-sysroot-native 디렉토리가 `tmp/work/<arch>/<recipe name>/` 아래에 생성된다. (nano editor의 경우 `build/tmp/work/core2-64-poky-linux/nano/6.0-r0/')

 * sysroot 디렉토리: 헤더와 라이브러리를 찾기 위한 루트 디렉토리로 간주되는 디렉토리 (레시피당 하나씩 존재함)
   - recipe-sysroot: 타깃 시스템에서 사용하는 헤더, 라이브러리가 포함되어 있다. (이 디렉토리의 위치는 STAGING_DIR_TARGET 변수에 할당되어 있다)
   - recipe-sysroot-native: 크로스 컴파일에서 사용되는 컴파일러, 링커, 빌드 스크립트 등이 포함되어 있다. 호스트에서 사용하는 빌드 관련 도구가 여기에 배치된다. (이 디렉토리의 위치는 STAGING_DIR_NATIVE 변수에 할당되어 있다)
   - 다음과 같이 의존하는 레시피들의 태스크가 다 완료되어야 메인 레시피 태스크를 구성, 빌드할 수 있다.

    ```
    <nano 레시피>                       <ncurses 레시피>
    do_fetch                            do_fetch
       |                                   |
       v                                   v
    do_unpack                           do_unpack
       |                                   |
       v                                   v
    do_patch                            do_patch
       |                                   |
       v                                   v
    do_prepare_recipe_sysroot  <--      do_prepare_recipe_sysroot
       |                         |         |
       v                         |         v
    do_configure                 |      do_configure
       |                         |         |
       v                         |         v
    do_compile                   |      do_compile
       |                         |         |
       v                         |         v
    do_install                   |      do_install
                                 |          |
                                 |          v
                                 -----  do_populate_sysroot
    ```

## 레시피 파일에서의 실행시간 의존성 표현

* `RDEPENDS_${PN} = <package name>`
  - 빌드 의존성과 다르게 이것은 레시피 이름이 아니라 패키지 이름을 할당해야 한다.
  - 보통 'xxx_${PN}' 형식으로 되어 있는 변수는 패키징 단계에서 사용되는 이름을 뜻함 (레시피 빌드의 결과물)
  - 위는 bitbake 내부적으로 `do_build[rdeptask] = "do_package_write_xxx"`를 의미한다. (xxx는 ipk, deb, tar, rpm이 될 수 있음)
  - 이는 do_build 태스크가 RDEPENDS_${PN} 변수에 할당된 패키지를 생성하는 레시피의 do_package_write_xxx 태스크에 의존성을 갖고 있다는 뜻이다. (`meta/classes/package_xxx.bbclass` 참조)
  - do_build 태스크는 각각의 패키지들을 생성하는 레시피들의 do_package_write_xxx 태스크가 완료되어야 실행될 수 있다.
  - do_build 태스크는 모든 태스크들의 마지막에 위치하며 실제로는 실행되지 않고 태스크 체인 구성을 위해 존재하는 태스크에 불과하다.
    ```
    # poky/meta/classes/base.bbclass 내용
    
    ...
    addtask build after do_populate_sysroot
    do_build[noexec] = "1"
    do_build () {
        ...
    }
    ...
    ```

* RDEPENDS 변수는 패키징 단계에서 사용됨
  - 패키징을 수행하는 레시피는 루트 파일 시스템 이미지를 생성하는 core-image-minimal.bb와 같은 레시피를 뜻한다.
  - 소스 빌드를 위한 레시피와 달리 패키징 빌드를 위한 레시피는 do_rootfs, do_image, do_image_complete 등의 태스크를 갖고 있다.
  - 이 태스크들의 주요 목적은 루트 파일 시스템을 생성하는 것이며 이 태스크 체인 끝에 do_build 태스크가 존재한다.

* RRECOMMENDS 변수를 통한 의존성
  - RDEPENDS 변수를 통한 의존성은 할당한 패키지가 존재하지 않으면 빌드 오류가 발생한다.
  - RRECOMMENDS 변수를 통한 의존성은 해당 패키지가 존재하지 않아도 빌드 오류는 발생하지 않는다. (부드러운 실행시간 의존성)

* 예시: kdb 패키지 (리눅스 콘솔을 다루기 위한 도구들을 포함)
  - 과거에는 console-tools라는 패키지를 사용했으나 현재는 kdb 패키지가 대체 패키지가 되었다.
  - console-tools 패키지와 실행시간 의존성을 가진 다른 패키지의 레시피 파일은 수정되지 않아야 한다.
  - kdb 패키지가 이전 패키지인 console-tools와 호환되도록 바꾸기 위해 다음과 같이 레시피를 작성한다.
  - poky/meta/recipes-core/kdb/kdb_2.2.0.bb
    ```
    RREPLACES_${PN} = "console-tools"    # 대체되기 전의 패키지 이름 (패키지가 대체됐음을 알려줌)
    RPROVIDES_${PN} = "console-tools"    # 패키지 이름의 별칭 (호환되는 이전 패키지의 이름, 다른 레시피에서 console-tools라는 이름을 수정하지 않아도 됨)
    RCONFLICTS_${PN} = "console-tools"   # 현재 패키지가 설치될 때 충돌하는 것으로 알려진 패키지 이름 (대체되기 전의 패키지 설치를 방지함)
    ```

## 의존성을 제공하는 레시피의 PROVIDES 변수

* DEPENDS 변수에 제공할 이름을 PROVIDES 변수에 할당하면 된다. (2가지 방식으로 할당할 수 있음)
  - 레시피 파일 이름으로부터 PROVIDES 변수 값 할당 받기: PROVIDES 변수는 레시피 파일에서 정의하지 않아도 기본적으로 PN 변수의 값을 갖게 된다. (`PROVIDES = "${PN}"`)
    * 패키지 관련 변수 이름의 예시 (nano-6.0.bb 또는 nano_6.0_r0.bb)
      - PN (package name): nano
      - PV (package version): 6.0
      - PR (package revision): r0
  - 명시적으로 PROVIDES 변수 값 설정하기 (`PROVIDES =+ "nano_alias"`)
    * bitbake-getvar 명령어를 통해 PROVIDES 변수를 확인하면 다음과 같다. (`PROVIDES="nano nano_alias"`)
    * 이름이 충돌할 경우 이 방법을 사용하면 된다.

* 다중 패키지, 다중 버전을 위한 virtual PROVIDER
  - 예를 들어 현재 프로젝트에 적용된 커널이 Yocto에서 레퍼런스로 제공하는 커널인 linux-yocto.bb 레시피라고 하자.
  - 리누스 토발즈가 배포한 바닐라 커널 5.19를 사용해야 한다고 하자. (torvalds-kernel_5.19.bb)
  - 환경 설정 파일에서 다음과 같이 선호하는 패키지 이름을 지정할 수 있다.
    * linux-yocto.bb: `PROVIDES =+ "virtual/kernel"`
    * torvalds-kernel_5.19.bb: `PROVIDES =+ "virtual/kernel"`
    * qemu.inc: `PREFERRED_PROVIDER_virtual/kernel = "torvalds-kernel"`
  - 환경 설정 파일에서 다음과 같이 선호하는 패키지 버전을 지정할 수 있다.
    * torvalds-kernel_4.14.bb
    * torvalds-kernel_5.19.bb
    * poky.conf: `PREFERRED_VERSION_torvalds-kernel = "5.19"`  # 선호하는 PV 값을 할당하면 됨 (할당하지 않으면 최신 버전이 빌드된다)
    * 만약 `PREFERRED_VERSION_linux-yocto = "5.14%"`라고 하면 5.14 이후의 버전을 허용한다는 뜻이다.

* __다음 명령어는 의존성을 갖고 있는 레시피 파일들을 출력해 준다.__
  - `$ bitbake-layers show-cross-depends`
  - 만약 `$ bitbake-layers show-cross-depends | grep nano`라고 하면 nano 레시피와 의존성이 있는 모든 것을 출력한다.

# 패키지 그룹 및 빌드 환경 구축 (빌드 스크립트 작성)

## IMAGE_INSTALL, IMAGE_FEATURES 변수

* IMAGE_INSTALL 변수: 루트 파일 시스템에 설치할 **패키지 이름**을 나열한 변수
  - 예를 들어 `IMAGE_INSTALL += "hello nano"`를 하면 이미지에 hello, nano 패키지가 들어가게 됨

* IMAGE_FEATURES 변수: 이미지에 추가되는 기능들을 레시피에서 추가할 때 사용함
  - 루트 파일 시스템을 생성하는 대상 이미지 레시피에서 상속한 기반 이미지 클래스에 따라 할당할 수 있는 값들이 결정됨
  - 이미지 클래스는 **미리 정의된 기능 목록**을 갖고 있으며, 기능 목록들 중에서 필요한 기능들을 할당함
  - local.conf 환경 설정 파일에 추가할 때는 EXTRA_IMAGE_FEATURES 변수를 사용한다. (최종적으로 IMAGE_FEATURES에 추가됨)

## IMAGE_FEATURES

* 다음은 image.bbclass 클래스에서 미리 정의된 기능 목록의 일부이다.
  - debug-tweaks: 개발시에 주로 사용하는 기능으로 주된 특징은 비밀번호 없이 SSH 로그인이 가능하다.
  - package-management: 루트 파일 시스템을 만들 경우 PACKAGE_CLASSES 변수에 설정된 패키지 관리 유형에 따른 패키지 관리 시스템을 설치한다.
  - read-only-rootfs: 읽기 전용으로 루트 파일 시스템을 생성한다.
  - splash: 부팅시에 스플래시 스크린이 동작하도록 한다. (재빌드 후 QEMU를 실행하면 부팅시 스플래시 이미지를 볼 수 있음)
    ```
    $ bitbake core-image-minimal -C rootfs
    $ runqemu
    ```
  - 그 외 IMAGE_FEATURES 변수에서 제공하는 기능들이 있으니 공식 문서를 참고할 것

* IMAGE_FEATURES 변수는 FEATURES_PACKAGES 변수와 함께 사용된다.
  - `FEATURES_PACKAGE_feature1 = "package1 package2"`  # 기능 정의 (feature1 기능: package1, package2 포함)
  - image.bbclass에서 splash를 정의하는 예시는 다음과 같다.
    ```
    ...
    SPLASH ?= "psplash"    # psplash 패키지를 SPLASH 변수에 추가
    FEATURE_PACKAGES_splash = "${SPLASH}"    # splash 기능 정의
    ```
  - 만약 local.conf 파일에서 EXTRA_IMAGE_FEATURES 변수에 'tools-sdk' 기능을 추가할 경우 이미지에 gcc 컴파일러 등이 설치된다. (타깃에서 vim으로 c 파일을 작성하기 컴파일도 할 수 있음)

## 패키지 그룹

* 패키지 그룹: 이미지에 포함될 수 있는 패키지들의 집합
  - IMAGE_INSTALL 변수와의 차이점: 각각의 패키지를 추가하는 것이 아니라 그룹핑해 여러 개의 패키지를 한 번에 추가한다.
  - packagegroups.bbclass 클래스 파일을 상속하여 사용한다.
  - 이것은 어떤 것도 빌드하지 않고 어떤 결과물도 만들지 않는다.
  - 다만 여러 패키지들을 그룹핑해 의존성만을 부여한다.
  - 패키지 그룹의 레시피 파일 이름: packagegroup-<name>.bb (레시피 작업 디렉토리인 recipes-xxx 디렉토리 아래 packagegroups라는 디렉토리를 만들고 그 안에 넣으면 됨)

### 실습 순서 (패키지 생성)

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout packagegroup`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b packagegroup`

* poky 디렉토리 아래에 meta-great 디렉토리를 생성한다. 이 디렉토리는 최종적으로 다음과 같은 구조를 가질 것이다.
  ```
  meta-great
  |- conf
  |   |- layer.conf
  |- recipes-core
      |- image
      |   |- core-image-minimal.bbappend
      |- packagegroups
          |- packagegroup-great.bb
  ```

layer.conf
```
BBPATH =. "${LAYERDIR}:"
BBFILES += "${LAYERDIR}/recipes*/*/*.bb"
BBFILES += "${LAYERDIR}/recipes*/*/*.bbappend"
BBFILE_COLLECTIONS += "great"
BBFILE_PATTERN_great = "^${LAYERDIR}/"
BBFILE_PRIORITY_great = "11"    # 기존 레이어들보다 큰 우선순위 할당
LAYERSERIES_COMPAT_great = "${LAYERSERIES_COMPAT_core}"
```

packagegroup-great.bb
```
DESCRIPTION = "this package group is great's packages"
inherit packagegroup

PACKAGE_ARCH = "${MACHINE_ARCH}"    # 머신에 의존적인 패키지: 패키지가 빌드 대상인 특정 머신에 의존적인 경우
#Inherit allarch                    # 아키텍처 의존적 패키지: 빌드 대상인 머신과는 관계없이 모든 아키텍처에 적용되는 패키지의 경우
RDEPENDS_${PN} = "\
              hello \
              nano \
              "
```

* 새로 생성한 패키지 그룹 packagegroup-great를 이미지 생성 레시피인 core-image-minimal의 루트 파일 시스템에 추가한다.

core-image-minimal.bbappend
```
IMAGE_INSTALL += "packagegroup-great"
```

* ~/poky_src/build/conf/bblayers.conf 파일에 새로 생성한 meta-great 레이어를 추가한다.

bblayers.conf
```
POKY_BBLAYERS_CONF_VERSION = "2"
BBPATH = "${TOPDIR}"
BBFILES ?= ""
BBLAYERS ?= " \
  /home/user/poky_src/poky/meta \
  /home/user/poky_src/poky/meta-poky \
  /home/user/poky_src/poky/meta-yocto-bsp \
  /home/user/poky_src/poky/meta-hello \
  /home/user/poky_src/poky/meta-nano-editor \
  /home/user/poky_src/poky/meta-great \
  "
```

* 기존의 meta-hello, meta-nano-editor를 패키지 그룹으로 묶어서 새로 생성된 meta-great 레이어 아래 루트 파일 시스템을 생성하는 레시피 확장 파일 core-image-minimal.bbappend에 추가할 것이다.
  - 기존 레이어의 레시피 확장 파일에서 패키지 추가 내용을 삭제함

~/poky_src/poky/meta-hello/recipes-core/images/core-image-minimal.bbappend
```
# IMAGE_INSTALL_append =" hello"    # 주석 처리
```

~/poky_src/poky/meta-nano-editor/recipes-core/images/core-image-minimal.bbappend
```
# IMAGE_INSTALL_append =" nano"    # 주석 처리
```

* 이미지 빌드를 통해 core-image-minimal.bb 레시피로 루트 파일 시스템을 생성하고 QEMU를 실행한다.

```
$ bitbake core-image-minimal
$ runqemu core-image-minimal nographic
```

* 시스템 부팅, 로그인 후에 hello, nano 애플리케이션이 정상적으로 실행될 것이다.

* 패키지 그룹에 어떤 패키지들이 설치되어 있는지 확인해 본다.
  - `$ bitbake -g great-image`
  - '-g' 옵션은 지정한 레시피 파일에 대해 의존성을 가진 모든 패키지들을 pn-buildlist라는 파일로 출력함
  - pn-buildlist 파일의 내용을 확인하면 어떤 패키지가 설치되어 있는지 확인할 수 있다.

### 실습 순서 (대체 패키지 생성)

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout replace_package`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/local_conf.git -b replace_package`

* 기존 hello 패키지 예제를 확장할 것이다.
  - newhello 패키지: hello 대체 패키지 (기존의 코드를 수정하거나 삭제하지 않음)

* 기존 recipes-hello 디렉토리를 그대로 복사하고 recipes-newhello 디렉토리 아래 붙여 넣는다. 기존의 hello.bb 파일은 newhello.bb로 바꾸고 hello.c 파일은 newhello.c로 바꾼다.
  ```
  meta-hello/recipes-newhello
  |- newhello.bb
  |- source
      |- COPYING
      |- hello.service
      |- newhello.c
  ```

* newhello.bb 파일을 다음과 같이 수정한다.

newhello.bb
```
DESCRIPTION = "Simple helloworld application example"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://COPYING;md5=80cade1587e04a9473701795d41a4f0c"

SRC_URI = "file://newhello.c"    # 바뀐 부분
SRC_URI += "file://COPYING"
SRC_URI += "file://hello.service"

inherit systemd

S = "${WORKDIR}"
SYSTEMD_SERVICE_${PN} = "hello.service"
SYSTEMD_AUTO_ENABLE = "enable"

do_compile(){
    ${CC} newhello.c ${LDFLAGS} -o hello    # 바뀐 부분
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 hello ${D}${bindir}

    install -d ${D}${systemd_unitdir}/system
    install -m 0644 hello.service ${D}${systemd_unitdir}/system
}

# 대체되는 기존 패키지를 사용하지 않도록 하고, 다른 패키지와의 호환성도 보장함
RREPLACES_${PN} = "hello"    # 바뀐 부분
RPROVIDES_${PN} = "hello"    # 바뀐 부분
RCONFLICTS_${PN} = "hello"    # 바뀐 부분

FILESEXTRAPATHS_prepend := "${THISDIR}/source:"
FILES_${PN} += "${bindir}/hello"
FILES_${PN} += "${systemd_unitdir}/system/hello.service"
```

* newhello.c 파일을 작성한다. (그 외 COPYING, hello.service 파일은 수정하지 않음)

newhello.c
```
#include <stdio.h>
#include <unistd.h>

int main(){
    int i = 0;
    while (i < 10) {
        printf ("New hello world!\n");
        i++;
    }
    return 0;
}
```

* 기존 hello 패키지가 중복으로 생성되지 않도록 BBMASK 변수를 사용해야 한다.
  - BBMASK 변수는 bitbake가 특정 디렉토리 내에 있는 레시피 파일들(.bb, .bbappend)을 처리하지 못하게 함

~/poky_src/build/conf/local.conf
```
...
BBMASK = "meta-hello/recipes-hello/"
```

* 빌드 후 대체 패키지 실행 테스트
  - QEMU 실행, 로그인 후 hello를 실행해 본다.

```
$ bitbake newhello
$ bitbake core-image-minimal -C rootfs
$ runqemu nographic
```

## 미리 정의된 패키지 그룹

* 오픈임베디드 코어에는 이미지들에서 사용할 수 있도록 사전에 만들어진 공용 패키지 그룹이 존재한다.
  - core-image-minimal 이미지의 경우 패키지 그룹 packagegroup-core-boot를 사용한다.
  - 보통 이미지 생성을 위한 레시피 파일을 만들 때 packagegroup-core-boot.bb 패키지 그룹 레시피 파일을 넣는다. (부팅이 가능한 이미지를 얻을 수 있으므로)
  - 패키지를 제공하거나 설치하지 않는다.
  - 다만 실행 시간 의존성(RDEPENDS)만을 줘 각각의 패키지가 루트 파일 시스템이 만들어질 때 이미지에 함께 설치되도록 한다.
  - 정의된 패키지 그룹에서 제공하는 기능은 IMAGE_FEATURES 변수를 사용해서도 제공할 수 있다.

* 자주 사용하는 중요한 패키지 그룹 레시피 파일은 다음과 같다.
    ```
    poky/meta/recipes-core/packagegroups/
    |- nativesdk-packagegroup-sdk-host.bb
    |- packagegroup-base.bb
    |- packagegroup-core-boot.bb
    |- packagegroup-core-buildessential.bb
    |- packagegroup-core-eclipse-debug.bb
    |- packagegroup-core-nfs.bb
    |- packagegroup-core-sdk.bb
    |- packagegroup-core-ssh-dropbear.bb
    |- packagegroup-core-ssh-openssh.bb
    |- packagegroup-core-standalone-sdk-target.bb
    |- packagegroup-core-tools-debug.bb
    |- packagegroup-core-tools-profile.bb
    |- packagegroup-core-tools-testapps.bb
    |- packagegroup-cross-canadian.bb
    |- packagegroup-go-cross-canadian.bb
    |- packagegroup-go-sdk-target.bb
    |- packagegroup-self-hosted.bb
    ```

## 커스텀 빌드 스크립트를 통한 빌드 환경 구축

* 기존 빌드 환경을 초기화할 때에는 빌드 전에 oe-init-build-env 스크립트를 실행했다.
  - `source poky/oe-init-build-env`
  - 이 스크립트를 실행하면 기본적으로 build 디렉토리가 생성되고 내부에 몇 가지 환경 설정 파일과 디렉토리가 만들어진다.
  - build 디렉토리는 bitbake가 빌드 작업을 수행할 때 산출물들이 만들어지는 곳이다.
  - 실습을 진행하면서 머신 레이어, 배포 레이어를 만들 것이며 빌드의 산출물을 구분해 따로 저장해야 하므로 환경 설정 파일(local.conf, bblayers.conf)도 그때마다 바뀌어야 한다.

* TEMPLATECONF 변수: 빌드에 필요한 환경 설정 파일을 어디서 복사해 올지 지정해 주는 변수
  - 이 변수의 기본값은 'poky/meta-poky/conf' 디렉토리를 가리킨다.
  - 여기에서 .sample 파일을 build/conf 디렉토리에 복사하고 확장자 .sample을 삭제한다.
  - bblayers.conf.sample 파일의 placeholder인 OEROOT 변수는 오픈임베디드 빌드 시스템의 특정 파일의 절대 경로로 대체된다.
  - 만약 build 디렉토리를 포함한 소스 전체를 배포할 경우 절대 경로값이 문제가 될 수 있다. (PC 환경마다 레이어 경로가 다르므로)
  - oe_init_build_env 스크립트에서 인클루드하고 있는 poky/scripts/oe-setup-builddir 파일에서 bblayers.conf, local.conf, conf-notes.txt 파일의 위치를 지정한다.

* conf-notes 파일: oe_init_build-env 스크립트를 실행했을 때 화면에 출력되는 안내 문구들이 들어 있으며 자신이 원하는 문구가 있으면 이 파일을 수정하면 된다.
  - 이 파일의 위치는 다음과 같다: ~/poky_src/poky/meta-poky/conf-notes.txt

### 실습 순서

* 커스텀 빌드 스크립트를 작성해 보자.

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout buildscript`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b buildscript`

* 앞에서 만든 meta-great 레이어에 template이라는 디렉토리를 만든다.
  - 이 디렉토리 안에 poky_src/poky/meta-poky/conf 디렉토리에 있는 bblayers.conf.sample, local.conf.sample, conf-notes.txt 파일을 복사한다.

* 복사한 파일들을 수정한다.

~/poky_src/poky/meta-great/template/bblayers.conf.sample (기존 내용 수정)
```
POKY_BBLAYERS_CONF_VERSION = "2"
BBPATH = "${TOPDIR}"
BBFILES ?= ""
BBLAYERS ?= " \
  ##OEROOT##/meta \
  ##OEROOT##/meta-poky \
  ##OEROOT##/meta-yocto-bsp \
  ##OEROOT##/meta-hello \
  ##OEROOT##/meta-nano-editor \
  ##OEROOT##/meta-great \
  "
```

~/poky_src/poky/meta-great/template/local.conf.sample (아래 내용을 추가함)
```
...

CONF_VERSION = "1"

# PREMIRRORS 위치를 지정함
INHERIT += "own-mirrors"
SOURCE_MIRROR_URL = "file://${COREBASE}/../source-mirrors"

# 미러를 위한 tarball 파일을 압축함
BB_GENERATE_MIRROR_TARBALLS = "1"

# 공유 상태 캐시 미러를 생성함
SSTATE_MIRRORS = "file://.* file://${COREBASE}/../sstate-cache/PATH"
SSTATE_DIR = "${TOPDIR}/sstate-cache"

DISTRO_FEATURES_append = " systemd"
DISTRO_FEATURES_remove = "sysvinit"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = "systemd_compat-units"
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"
VIRTUAL-RUNTIME_initscript = "systemd-compat-units"
```

~/poky_src/poky/meta-great/template/conf-notes.txt (기존 내용 수정)
```
### Shell environment set up for builds. ###
Welcome! This is my yocto example.
You can now run 'bitbake <target>'

Common targets are:
    core-image-minimal

You can also run generated qemu images with a command like 'runqemu qemux86'
```

* 빌드 스크립트 buildenv.sh를 작성한다.

~/poky_src/buildenv.sh
```
#!/bin/bash
# find_top_dir(): poky 디렉토리의 절대 경로를 찾아내는 함수
function find_top_dir()
{
    local TOPDIR=poky
# move into script file path
    cd $(dirname ${BASH_SOURCE[0]})
    if [ -d $TOPDIR ]; then
        echo $(pwd)
    else
        while [ ! -d $TOPDIR ] && [ $(pwd) != "/" ];
        do
            cd ..
        done
        if [ -d $TOPDIR ]; then
            echo $(pwd)
        else
            echo "/dev/null"
        fi
    fi
}

ROOT+$(find_top_dir)
export TEMPLATECONF=${ROOT}/poky/meta-great/template/    # template 디렉토리의 절대 경로 지정
source poky/oe_init-build-env build2    # 차후 bitbake가 빌드 작업을 하며 모든 작업의 결과물들을 저장할 디렉토리를 지정함
```

* buildenv.sh 스크립트에게 실행 권한을 부여하고 실행한다.

```
$ chmod 777 buildenv.sh
$ source buildenv.sh
```

* build2/conf/templateconf.cfg 파일에는 현재 만들어진 환경 설정 파일을 어디서 복사했는지 기술되어 있다.

templateconf.cfg
```
/home/user/poky_src/poky/meta-great/template
```

* 이제 core-image-minimal에 대한 이미지를 생성하고 싶다면 다음을 실행하면 된다.

```
$ bitbake core-image-minimal
```

# BSP 레이어 작성

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/ddbcc186-567f-493f-a8b5-ce394dc01683)

* 이번 장에서는 great라는 이름을 가진 가상의 타깃 시스템을 만들 예정이다.
  - 바닐라 커널 version 5.4, u-boot
  - 머신 이름: great
  - 배포명: great-distro
  - 이미지 지원 기능: splash, great 계정 및 group 계정 추가, password 지원, 기존에 작성했던 nano, hello 패키지 추가 등

* great 시스템 전체 구조는 다음과 같다.

레이어 구조 | 설명
----------- | -----
소프트웨어 레이어 (./meta-myproject) | 만들어야 할 레이어
배포 레이어 (./meta-great) | 만들어야 할 레이어
BSP 레이어 (./meta-great-bsp) | 만들어야 할 레이어
포키 참조 배포 레이어 (./meta-poky) | 기본 제공 레이어
oe-core (./meta) | 기본 제공 레이어

## 커스텀 이미지 레시피 생성

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout custom_image`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b custom_image`

* 커스텀 이미지 레시피를 포함하는 메타 레이어는 기존에 추가한 meta-great를 그대로 사용하고, 커스텀 이미지를 생성하는 이미지 레시피 파일로 great.bb를 만들 것이다.
  - meta-great 레이어의 디렉토리 전체 구조는 다음과 같다.

```
~/poky_src/poky/meta-great
|- classes
|   |- great-base-image.bbclass    # 패키지 그룹 레시피들이 추가되어 있음 (오픈임베디드 코어에서 정의된 것)
|- conf
|   |- layer.conf
|- recipes-core
|   |- images
|   |   |- core-image-minimal.bbappend
|   |   |- great-image.bb    # great-base-image.bbclass 클래스 파일을 상속함
|   |- packagegroups
|       |- packagegroup-great.bb
|- template
    |- bblayers.conf.sample
    |- conf-notes.txt
    |- local.conf.sample
```

~/poky_src/poky/meta-great/conf/layer.conf 파일 수정
```
BBPATH =. "${LAYERDIR}:"
BBFILES += "${LAYERDIR}/recipes*/*/*.bb"
BBFILES += "${LAYERDIR}/recipes*/*/*.bbappend"
BBFILE_COLLECTIONS += "great"
BBFILE_PATTERN_great = "^${LAYERDIR}/"
BBFILE_PRIORITY_great = "11"
LAYERDEPENDS_great = "core"    # great 레이어가 core 레이어를 의존함 (core-image.bbclass를 사용하므로 core 레이어가 필요함)
LAYERSERIES_COMPAT_great = "${LAYERSERIES_COMPAT_core}"
```

* core 레이어는 이렇게 찾을 수 있다: `~/poky_src/poky$ grep -rni "BBFILE_COLLECTIONS" ./ | grep core`

~/poky_src/poky/meta-great/classes/great-base-image.bbclass 생성
```
inherit core-image
IMAGE_FSTYPES = " tar.gz2 ext4"    # 2개의 루트 파일 시스템을 생성할 것을 의미함
IMAGE_ROOTFS_SIZE = "10240"    # 생성될 루트 파일 시스템 이미지를 킬로바이트 단위로 지정 (최종 루트 파일 시스템이 이 크기보다 크면 이 값은 더 커지게 됨)
IMAGE_ROOTFS_EXTRA_SPACE = "10240"    # 루트 파일 시스템에 추가적인 빈 공간을 만듦
IMAGE_ROOTFS_ALIGNMENT = "1024"    # 루트 파일 시스템 이미지 크기를 이 값의 배수로 맞춤
CORE_IMAGE_BASE_INSTALL = "\
    packagegroup-core-boot \
    packagegroup-base-extended \
    ${CORE_IMAGE_EXTRA_INSTALL} \
"
```

~/poky_src/poky/meta-great/recipes-core/images/great-image.bb 생성
```
SUMMARY = "A very small image for yocto test"
inherit great-base-image
LINGUAS_KO_KR = "ko-kr"    # 한국어 추가
LINGUAS_EN_US = "en-us"    # 영어 추가
IMAGE_LINGUAS = "${LINGUAS_KO_KR} ${LINGUAS_EN_US}"
IMAGE_INSTALL += "packagegroup-great"    # 앞의 예제에서 만든 패키지 그룹 레시피 (hello, nano 패키지 포함)
IMAGE_OVERHEAD_FACTOR = "1.3"    # 실제 필요한 루트 파일 시스템 크기에 30%의 여유 공간을 추가함
```

~/poky_src/poky/meta-great/template/conf_notes.txt 수정
```
### Shell environment set up for builds. ###
Welcome! This is my yocto example.
You can now run 'bitbake <target>'

Common targets are:
   great-image

You can also run generated qemu images with a command like 'runqemu qemux86'
```

~/poky_src/poky/meta-great/recipes-core/images/great-image.bb 수정
```
SUMMARY = "A very small image for yocto test"
inherit great-base-image

LINGUAS_KO_KR = "ko-kr"
LINGUAS_EN_US = "en-us"
IMAGE_LINGUAS = "${LINGUAS_KO_KR} ${LINGUAS_EN_US}"
IMAGE_INSTALL += "packagegroup-great"
IMAGE_OVERHEAD_FACTOR = "1.3"

inherit extrausers

EXTRA_USERS_PARAMS = "\
   groupadd greatgroup; \    # 그룹 생성
   useradd -p `openssl passwd 9876` great; \    # 사용자 생성, 암호 9876
   useradd -g greatgroup great; \    # 사용자 great의 그룹을 greatgroup으로 지정
"
```

~/poky_src/poky/meta-great/template/local.conf.sample
```
...
EXTRA_IMAGE_FEATURES += "splash"
```

* 이미지 생성을 위해 빌드를 진행한다.

```
$ source buildenv.sh
$ bitbake great-image
$ runqemu great-image nographic
```

* 이제 QEMU에서 계정 great, 암호 9876을 입력하여 로그인할 수 있다.

* QEMU를 종료하려면 다음과 같이 하면 된다.

```
# su
# poweroff
```

## BSP 레이어

* BSP(Board Support Package): 주어진 타깃이 되는 장치의 하드웨어를 구동하는 데 필요한 OS와 장치 드라이버 등을 말한다.
  - OS와 장치 드라이버를 메모리에 배치하는 부트로더도 포함한다.

| 레이어 구조 |
| ----------- |
| 소프트웨어 레이어 (./meta-myproject) |
| 배포 레이어 (./meta-great) |
| BSP 레이어 (./meta-great-bsp) |
| 포키 참조 배포 레이어 (./meta-poky) |
| oe-core (./meta) |

* 이 실습에서는 실제 물리적인 보드가 없고 QEMU라는 가상의 장치를 사용하므로 물리적인 디바이스 설정, 구동을 해볼 수는 없다.
  - BSP 레이어가 무엇이고, 어떻게 만드는지만 알아보자.

* QEMU 에뮬레이터에 대한 머신 환경 설정 파일 poky/meta/conf/machine/qemux86-64.conf 파일에 대해 알아보자.

~/poky_src/poky/meta/conf/machine/qemux86-64.conf
```
PREFERRED_PROVIDER_virtual/xserver ?= "xserver-xorg"    # 다중 패키지 중 어떤 패키지를 사용하는지 정함
PREFERRED_PROVIDER_virtual/libgl ?= "mesa"
PREFERRED_PROVIDER_virtual/libgles1 ?= "mesa"
PREFERRED_PROVIDER_virtual/libgles2 ?= "mesa"

require conf/machine/include/qemu.inc
DEFAULTTUNE ?= "core2-64"                                # 빌드 시스템에서 사용할 CPU 아키텍처와 ABI(Application Binary Interface)를 선택함
require conf/machine/include/tune-core2.inc
require conf/machine/include/qemuboot-x86.inc
UBOOT_MACHINE ?= "qemu-x86_64_defconfig"
KERNEL_IMAGETYPE = "bzImage"                            # 커널 이미지 파일의 명명 체계 결정

SERIAL_CONSOLES ?= 115200;ttyS0 115200;ttyS1"            # getty를 사용할 수 있도록 serial console(tty)을 정의함 (전송속도와 tty 장치)

# Install swrast and glx if opengl is in DISTRO_FEATURES and x32 is not in use.
# This is because gallium swrast driver was found to crash X server on startup in qemu x32
XSERVER = "xserver-xorg \
          ${@bb.utils.contains('DISTRO_FEATURES', 'opengl', \
          bb.utils.contains('TUNE_FEATURES', 'mx32', '',
          'mesa-driver-swrast xserver-xorg-extension-glx', d), '', d)} \
          xf86-video-cirrus \
          xf86-video-fbdev \
          xf86-video-vmware \
          xf86-video-modesetting \
          xf86-video-vesa \
          xserver-xorg-module-libint10 \
          "

MACHINE_FEATURES += "x86 pci"                # 장치에서 지원되는 하드웨어 관련 기능들을 기술함 (alsa, bluetooth, usbgadget, screen, vfat 등도 있음)
MACHINE_ESSENTIAL_EXTRA_RDEPENDS+= "v86d"    # 이미지 빌드의 일부, 특정한 머신에 설치해야 하는 필수적인 패키지들의 목록 (기기가 부팅하는 데 필수적임)
MACHINE_EXTRA_RRECOMMENDS = "kernel-module-snd-ens1370 kernel-module-snd-rawmidi"    # 특정 머신에 설치해야 하는 패키지들의 목록 (기기가 부팅하는 데 필수적이지는 않음)

WKS_FILE ?= "qemux86-directdisk.wks"
do_image_wic[depends] += "syslinux:do_populate_sysroot syslinux-native:do_populate_sysroot mtools-native:do_populate_sysroot dosfstools-native:do_populate_sysroot"

# For runqemu
QB_SYSTEM_NAME = "qemu-system-x86_64"
```

qemu.inc
```
PREFERRED_PROVIDER_virtual/xserver ?= "xserver-xorg"
PREFERRED_PROVIDER_virtual/egl ?= "mesa"
PREFERRED_PROVIDER_virtual/libgl ?= "mesa"
PREFERRED_PROVIDER_virtual/libgles1 ?= "mesa"
PREFERRED_PROVIDER_virtual/libgles2 ?= "mesa"

XSERVER ?= "xerver-gz \
           ${@bb.utils.contains('DISTRO_FEATURES', 'opengl',
           'mesa-driver-swrast xserver-xorg-extension-glx', '', d)} \
           xf86-video-fbdev \
           "

MACHINE_FEATURES = "alsa bluetooth usbgadget screen vfat"
MACHINEOVERRIDES =. "qemuall:"
IMAGE_FSTYPES += "tar.bz2 ext4"    # 최종적으로 생성될 이미지의 타입을 알려줌 (bzip2를 사용한 tar 압축 파일, ext4 이미지 타입의 파일)
# Don't include kernels in standard images
RDEPENDS${KERNEL_PACKAGE_NAME}-base = ""
# Use a common kernel recipe for all QEMU machines
PREFERRED_PROVIDER_virtual/kernel ??= "linux-yocto"    # 다중 PROVIDES가 존재할 때 어떤 PROVIDES를 선택할 것인가 결정함 (여기서는 linux-yocto.bb 레시피 파일에서 만들어진 커널을 사용함)
EXTRA_IMAGEDEPENDS += "qemu-system-native qemu-helper-native"    # 네이티브 플랫폼, 즉 호스트 PC(x86)를 위한 패키지를 생성하는 레시피
# Provide the nfs server kernel module for all qemu images
KERNEL_FEATURES_append_pn-linux-yocto = " features/nfsd/nfsd-enable.scc"    # 진보된 메타데이터로부터 제공된 커널 환경 설정 정보나 패치 등이 포함됨
KERNEL_FEATURES_append_pn-linux-yocto-rt = " features/nfsd/nfsd-enable.scc"
IMAGE_CLASSES += "qemuboot"
```

* `$ bitbake-getvar -r core-image-minimal OVERRIDES`  # 여기서 qemu.inc 파일에서 사용한 MACHINEOVERRIDES 변수를 보십시오
  ```
  ...
  #   "${TARGET_OS}:${TRANSLATED_TARGET_ARCH}:pn-${PN}:${MACHINEOVERRIDES}:${DISTROOVERRIDES}:${CLASSOVERRIDE}${LIBCOVERRIDE}:forcevariable"
  OVERRIDES="linux:x86-64:pn-core-image-minimal:qemuall:qemux86-64:poky:class-target:libc-glibc:forcevariable"
  ```
  - local.conf (`MACHINE = "qemux86-64"`) --> bitbake.conf (`MACHINEOVERRIDES = ${MACHINE}`) --> bitbake.conf (`OVERRIDES = ${MACHINEOVERRIDES})

## 커스텀 BSP 레이어 만들기

* 고유의 타깃 머신 환경 설정을 제공하고 이름을 great라고 명명한다.
  - 가상의 QEMU 머신을 사용하기 때문에 기존에 Poky에서 제공한 머신 관련 설정을 그대로 사용하고, BSP 레이어에 포함된 커널 레시피는 수정할 것이다.

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout custom_bsp`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b custom_bsp`

* BSP 레이어 생성하기
  ```
  ~/poky_src/poky/meta-great-bsp
  |- conf
      |- layer.conf
      |- machine
          |- great.conf
  ```

* 레이어를 인식할 수 있도록 bblayers.conf.sample 파일을 수정한다.

~/poky_src/poky/meta-great/template/bblayers.conf.sample
```
POKY_BBLAYERS_CONF_VERSION = "2"
BBPATH = "${TOPDIR}"
BBFILES ?= ""
BBLAYERS ?= " \
  ##OEROOT##/meta \
  ##OEROOT##/meta-poky \
  ##OEROOT##/meta-yocto-bsp \
  ##OEROOT##/meta-hello \
  ##OEROOT##/meta-nano-editor \
  ##OEROOT##/meta-great \
  ##OEROOT##/meta-great-bsp \ "
```

* 수정한 bblayers.conf.sample 파일을 빌드 환경에 반영하려면 build/conf 디렉토리를 삭제해야 한다.
  - bitbake는 한 번 생성된 conf 디렉토리 내에 파일들의 내용을 빌드시 업데이트하지 않는다.
  - conf 디렉토리를 삭제해야 bblayers.conf 파일에 수정된 내용이 반영된다.

```
~/poky_src$ rm -rf build/conf
~/poky_src$ source buildenv.sh
```

* 새로 생성된 레이어의 conf 디렉토리에 layer.conf 파일을 생성한다.
  - 이 파일은 bitbake가 layer.conf 파일을 처리하면서 특정 레이어에 포함된 레시피, 클래스, 환경 설정 파일들을 인지하게 해준다.
  - BBFILES 변수에 레시피, 레시피 확장 파일들의 경로를 추가해 줌으로써 레시피, 레시피 확장 파일을 인지할 수 있도록 해준다.

~/poky_src/poky/meta-great-bsp/conf/layer.conf
```
BBPATH =. "${LAYERDIR}:"
BBFILES += "${LAYERDIR}/recipes*/*/*.bb"
BBFILES += "${LAYERDIR}/recipes*/*/*.bbappend"
BBFILE_COLLECTIONS += "greatbsp"
BBFILE_PATTERN_greatbsp = "^${LAYERDIR}/"
BBFILE_PRIORITY_greatbsp = "6"
LAYERSERIES_COMPAT_greatbsp = "${LAYERSERIES_COMPAT_core}"
```

* 새로 생성된 레이어의 conf 디렉토리에 machine 디렉토리를 만들고 머신 환경 설정 파일인 great.conf 파일을 생성한다.
  - 전체 파일 내용은 qemux86-64.conf 파일과 동일하나, MACHINEOVERRIDES 변수에 great 머신 이름을 넣은 것만 다르다.

~/poky_src/poky/meta-great-bsp/conf/machine/great.conf
```
PREFERRED_PROVIDER_virtual/xserver ?= "xserver-xorg"
PREFERRED_PROVIDER_virtual/libgl ?= "mesa"
PREFERRED_PROVIDER_virtual/libgles1 ?= "mesa"
PREFERRED_PROVIDER_virtual/libgles2 ?= "mesa"
require conf/machine/include/qemu.inc    # 이 파일이 존재하지 않으므로 모든 레이어 상의 conf 디렉토리에서 찾아본다. (poky/meta/conf/machine/includeqemu.inc 파일을 찾음)
                                         # require, include는 역할이 같지만 전자의 경우 파일이 존재하지 않으면 오류를 발생시키고, 후자의 경우 오류를 발생시키지 않음

DEFAULTTUNE ?= "core2-64"
require conf/machine/include/tune-core2.inc
require conf/machine/include/qemuboot-x86.inc
UBOOT_MACHINE ?= "qemu-x86_64_defconfig"
KERNEL_IMAGETYPE = "bzImage"
SERIAL_CONSOLES ?= "115200;ttyS0 115200;ttyS1"

XSERVER = "xserver-xorg \
          ${@bb.utils.contains('DISTRO_FEATURES', 'opengl', \
          bb.utils.contains('TUNE_FEATURES', 'mx32', '',
          'mesa-driver-swrast xserver-xorg-extension-glx', d), '', d)} \
          xf86-video-cirrus \
          xf86-video-fbdev \
          xf86-video-vmware \
          xf86-video-modesetting \
          xf86-video-vesa \
          xserver-xorg-module-libint10 \
          "
MACHINEOVERRIDES =. ":great:"    # MACHINEOVERRIDES 변수는 OVERRIDES 변수에 포함됨
MACHINE_FEATURES += "x86 pci"
MACHINE_ESSENTIAL_EXTRA_RDEPENDS += "v86d"
MACHINE_EXTRA_RRECOMMENDS = "kernel-module-snd-ens1370 kernel-module-snd-rawmidi"
WKS_FILE ?= "qemux86-directdisk.wks"
do_image_wic[depends] += "syslinux:do_populate_sysroot syslinux-native:do_populate_sysroot mtools-native:do_populate_sysroot dosfstools-native:do_populate_sysroot"
#For runqemu
QB_SYSTEM_NAME = "qemu-system-x86_64"
```

* 전체 레이어의 구성은 다음과 같다.
  ```
  ~/poky_src/poky/meta-great-bsp
  |- conf
  |   |- layer.conf
  |   |- machine
  |       |- great.conf
  |- recipes-kernel
      |- linux
          |- file
          |- linux-yocto_5.4.bbappend
  ```

* 새로운 머신에서 사용할 커널에 대해 정의한다.
  - Yocto에서 제공한 리눅스 커널을 새용한다.

~/poky_src/poky/meta-great-bsp/recipes-kernel/linux/linux-yocto_5.4.bbappend
```
KBRANCH_great = "v5.4/standard/base"
KMACHINE_great = "qemux86-64"    # 커널 메타데이터
SRCREV_machine_great = "35826e154ee014b64ccfa0d1f12d36b8f8a75939"    # 커널 소스의 리비전 (Git의 커밋 해시 값)
COMPATIBLE_MACHINE_great = "great"    # 호환되는 머신 이름
LINUX_VERSION_great = "5.4.219"    # 현재 사용하려는 커널 버전
```

* 새로 생성한 머신 환경 설정 파일 great.conf 파일이 bitbake 파싱 대상이 되려면 머신 이름을 나타내는 MACHINE 변수 값을 "great"로 설정해야 한다.

~/poky_src/buildenv.sh
```
# !/bin/bash
function find_top_dir()
{
    local TOPDIR=poky
# move into script file path
    cd $(dirname ${BASH_SOURCE[0]})

    if [ -d $TOPDIR ]; then
        echo $(pwd)
    else
        while [ ! -d $TOPDIR ] && [ $(pwd) != "/" ];
        do
            cd ..
        done

        if [ -d $TOPDIR ]; then
            echo $(pwd)
        else
            echo "/dev/null"
        fi
    fi
}

ROOT=$(find_top_dir)
export TEMPLATECONF=${ROOT}/poky/meta-great/template/
export MACHINE="great"

function build_target() {
    source poky/oe-init-build-env build2
}

build_target
```

* 빌드 진행하기

```
~/poky_src/build2$ rm -rf conf/
~/poky_src$ source buildenv.sh
~/poky_src/build2$ bitbake great-image
```

* QEMU 실행하기

```
$ runqemu great-image nographic
```

# 커널 레시피

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/0fc17250-3fbb-4d1b-b2b2-829eb1ee23fb)

* 커널 레시피: kernel 클래스를 상속한 레시피
  - kernel-yocto-bbclass 클래스 파일을 상속하여 kernel 클래스의 기능을 확장할 수 있다.
  - 커널의 패치, 커널 환경 설정 옵션들을 따로 제공할 수 있는 커널 메타데이터 또는 진보된 메타데이터라고 하는 Yocto 커널 리포지터리에 메타 브랜치를 갖고 있다.
  - 이 브랜치는 커널 브랜치와 분리되어 데이터를 제공한다.
  - 이 메타 브랜치에서 제공하는 데이터를 사용하려면 오픈임베디드 코어에서 제공하는 linux-yocto.inc 파일을 인클루드해야 한다.

## 커널 환경 설정

* Kconfig: 리눅스 커널용으로 개발된 선택 기반의 환경 설정 시스템
  - 대부분의 사용자는 `$ make menuconfig` 명령어를 실행하여 메뉴 편집기 GUI를 통해 Kconfig과 상호 작용한다.
  - 여기서 사용자가 원하는 옵션과 기능을 선택하고 커널 환경 설정 파일을 저장한다.
  - 메뉴 편집기를 저장하고 종료하면 커널 소스 최상위 디렉토리에 .config 파일이 만들어진다.
  - 커널 빌드시 .config 파일을 기반으로 autoconfig.h 파일이 생성되는데 선택된 환경 설정 옵션들은 autoconfig.h 파일에 매크로 상수로 표현되어 커널 소스 전체에 영향을 끼친다.

Kconfig 파일의 예
```
config ACPI_I2C_OPREGION
  bool "ACPI I2C Operation region support"
  depends on I2C=y && ACPI
  default y
  help
    Say Y here if you want to enable ACPI I2C operation region support
    Operation Regions allow firmware (BIOS) code to access I2C slave devices,
    such as smart batteries through an I2C host controller driver.
```

make menuconfig 명령어 실행 화면
```
    *** Compiler: x86_64-poky-linux-cc (GCC) 9.5.0 ***
    General setup  --->
[*] 64-bit kernel
    Processor type and features  --->
    Power management and ACPI options  --->
    Bus options (PCI etc.)  --->
    Binary Emulations  --->
    Firmware Drivers  --->
[*] Virtualization  --->
    General architecture-dependent options  --->
[*] Enable loadable module support  --->
-*- Enable the block layer  --->
    IO Schedulers  --->
    Executable file formats  --->
    Memory Management options  --->
[*] Networking support  --->
    Device Drivers  --->
    File systems  --->
    Security options  --->
-*- Cryptographic API  --->
    Library routines  --->
    Kernel hacking  --->
```

.config 파일의 일부
```
CONFIG_CC_IS_GCC=y
CONFIG_GCC_VERSION=90500
CONFIG_CLANG_VERSION=0
CONFIG_CC_HAS_ASM_GOTO=y
CONFIG_CC_HAS_ASM_INLINE=y
CONFIG_IRQ_WORK=y
CONFIG-BUILDTIME_EXTABLE_SORT=y
CONFIG_THREAD_INFO_IN_TASK=y
```

생성된 autoconfig.h
```
#define CONFIG_NLS_CODEPAGE_861_MODULE 1
#define CONFIG_RING_BUFFER 1
#define CONFIG_NF_CONNTRACK_H323_MODULE 1
#define CONFIG_HAVE_ARCH_SECCOMP_FILTER 1
#define CONFIG_SND_PROC_FS 1
#define CONFIG_SCSI_DMA 1
```

* 메뉴 편집기를 사용하지 않고 사전에 타깃 머신에 맞게 커널 환경 옵션들을 만들어 놓을 수 있다. (defconfig 파일)
  - 미리 필요한 설정이 저장되어 있어 사용하기 편하다.
  - 주로 칩 벤더에서 초기 보드 bring-up을 위해 defconfig 파일을 배포하는 경우가 많다.
  - 보통 이 파일들은 커널 소스 내의 'arch/<아키텍처>/configs/' 디렉토리에 존재한다.

x86_64_defconfig 파일
```
CONFIG_POSIX_MQUEUE=y
CONFIG_BSD_PROCESS_ACCT=y
CONFIG_TASKSTATS=y
CONFIG_TASK_DELAY_ACCT=y
CONFIG_TASK_XACCT=y
```

* 커널 소스에서 직접 .config, defconfig 파일 생성하기
  - 다음 명령어를 입력하면 커널 소스 트리의 최상위 디렉토리에 defconfig이라는 파일이 생성된다.

```
$ make ARCH=<아키텍처 이름, arm, x86 등> menuconfig
$ make ARCH=<아키텍처 이름, arm, x86 등> savedefconfig
```

* 혹은 Yocto 명령어를 통해 커널 환경 설정 파일들을 생성할 수도 있다.
  - 1-1번 방법: make menuconfig 명령 실행을 통한 메뉴 편집기 실행
    `$ bitbake -c menuconfig <커널 레시피 이름>` 또는 `$ bitbake -c menuconfig virtual/kernel`
  - 1-2번 방법: .config 파일 생성
    `$ bitbake -c kernel-configme <커널 레시피 이름>` 또는 `$ bitbake -c kernel-configme virtual/kernel`
  - 2번 방법: defconfig 파일 생성
    `$ bitbake -c savedefconfig <커널 레시피 이름>` 또는 `$ bitbake -c savedefconfig virtual/kernel`

* .config, defconfig 파일을 생성한 후에 커널을 컴파일한다.

```
$ bitbake -C compile virtual/kernel
```

## 변경 또는 추가된 커널 환경 옵션들을 패치로 생성

* 리눅스 커널에서 새로운 드라이버를 작성할 때 커널 환경 옵션(CONFIG_XXX)을 만들어야 한다.
  - 그러나 .config 파일이 빌드 디렉토리 내에 생성되기 때문에 빌드 환경이 삭제될 수 있으므로 변경된 커널 환경 옵션을 따로 저장하는 것이 좋다.
  - 동일한 머신이라 하더라도 하드웨어/소프트웨어 변경에 따른 파생 머신들이 만들어질 수 있다.
  - 기본 환경 설정 파일 defconfig에는 공통적인 환경 설정 옵션들만 모아 놓는다.
  - 파생된 머신에 따라 추가/변경되는 환경 설정 옵션들만 따로 분리해 관련된 머신에만 반영할 수 있다. (Configuration Fragment 파일)
  - 이 파일은 새로 추가되는 환경 설정 옵션만 따로 저장하고 커널 레시피에 환경 설정 단편 파일을 추가한다.
  - 최종적으로 커널 레시피가 빌드될 때 .config 파일에 추가된 환경 설정 옵션들이 반영된다.

* 환경 설정 단편 (Configuration Fragment) 파일
  - 예를 들어, 새로운 드라이버를 만들게 되었다고 가정하자. (옵션 CONFIG_NEW_DRIVER)
  - 이 환경 설정 옵션은 Kconfig 파일에 추가된다.
  - 드라이버 작업이 완료되면 새로 만든 환경 설정 옵션의 활성화를 위해 `$ bitbake -c menuconfig` 명령을 실행하여 추가된 환경 설정 옵션을 선택하고 저장하면 .config 파일이 생성된다.
  - 이때 새로 생성된 .config 파일은 추가한 환경 설정 옵션이 추가된 상태이다.
  - `$ bitbake -c diffconfig virtual/kernel` 명령을 통해 현재 상태에서 추가된 환경 설정 옵션만 추출한다.
  - 커널의 빌드 결과가 저장되는 WORKDIR 변수의 경로에 fragment.cfg라는 환경 설정 단편 파일이 생성된다.

* 환경 설정 단편 (Configuration Fragment) 파일 생성 예시
  - `$ bitbake virtual/kernel -c menuconfig`을 실행하면 다음과 같은 메뉴 편집기가 실행된다.
    ```
        *** Compiler: x86_64-poky-linux-cc (GCC) 9.5.0 ***
        General setup  --->
    [*] 64-bit kernel
        Processor type and features  --->
        Power management and ACPI options  --->
        Bus options (PCI etc.)  --->
        Binary Emulations  --->
        Firmware Drivers  --->
    [*] Virtualization  --->
        General architecture-dependent options  --->
    [*] Enable loadable module support  --->
    -*- Enable the block layer  --->
        IO Schedulers  --->
        Executable file formats  --->
        Memory Management options  --->
    [*] Networking support  --->
        Device Drivers  --->
        File systems  --->
        Security options  --->
    -*- Cryptographic API  --->
        Library routines  --->
        Kernel hacking  --->
    ```
  - 메뉴 편집기 실행 화면에서 '/'를 누르면 검색 화면이 나타난다. 검색 화면에서 EXT3_FS를 입력하고 OK를 클릭한다.
  - '1'을 눌러 환경 설정 옵션 EXT3_FS를 선택한다.
  - 스페이스바를 2번 눌러 '*'로 선택이 되도록 한다.
  - 화면 하단에서 SAVE를 클릭하고 엔터를 눌러 .config 파일로 저장한 다음 EXIT를 클릭해 메뉴 편집기를 종료한다.
  - 갱신된 .config 파일을 토대로 `$ bitbake -c diffconfig virtual/kernel` 명령을 입력하여 환경 설정 단편 파일 .cfg를 만든다.
  - fragment.cfg 파일이 생성되었다는 문구가 출력될 것이다.
  - fragment.cfg 파일을 열어보면 변경된 EXT3_FS 환경 설정 옵션이 추가된 것을 확인할 수 있다.
    ```
    CONFIG_EXT3_FS=y
    # CONFIG_EXT3_FS_POSIX_ACL is not set
    # CONFIG_EXT3_FS_SECURITY is not set
    ```

## 변경 또는 추가된 커널 소스를 패치로 생성

* 리눅스 커널 소스에 공식적으로 배포되는 커널 패치를 반영하는 방법은 2가지가 있다.
  - SCM 도구인 깃 리포지터리에 저장된 커널 소스를 받아 커널 패치를 반영하고 다시 리포지터리에 저장하는 방법 (소스 관리가 용이함)
  - 커널 레시피 파일에 패치를 추가해 빌드시 패치가 커널 소스에 반영되도록 하는 방법 (파생 커널을 생성할 때 용이함) --> 이 방법을 설명할 것이다.

* 먼저 실제 커널 소스가 위치한 것을 찾는다. (커널 소스 위치: STAGING_KERNEL_DIR 변수)
  - `~/poky_src/build2$ bitbake-getvar -r virtual/kernel STAGING_KERNEL_DIR`
  - 이것을 실행하면 실제 커널 소스의 경로가 출력된다.

* 커널 소스 상에서 간단한 디바이스 드라이버를 추가하고, 환경 설정 단편 파일을 만들어 커널 레시피에 추가해 본다.

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout kernel_patch`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b kernel_patch`

* ~poky_src/build2/tmp/work-shared/great/kernel-source/drivers/misc/new_test_driver.c 파일 생성
  ```
  #include <linux/module.h>
  static int __init new_test_driver_init(void)
  {
      pr_warn("This is new test driver! \n");
      return 0;
  }

  static void __exit new_test_driver_exit(void)
  {
      pr_warn("Exit new test driver! \n");
  }

  module_init(new_test_driver_init);
  module_init(new_test_driver_exit);
  MODULE_AUTHOR("Dennis Cho");
  MODULE_DESCRIPTION("New test driver");
  MODULE_LICENSE("GPL");
  ```

* 드라이버를 위한 커널 설정 옵션을 Kconfig 파일 제일 하단에 추가한다.

~/poky_src/build2/tmp/work-shared/great/kernel-source/drivers/misc/Kconfig
```
...
config NEW_TEST_DRIVER
    tristate "Kernel test driver"
    help
        This driver is kernel test driver

source "drivers/misc/c2port/Kconfig"
source "drivers/misc/eeprom/Kconfig"
source "drivers/misc/cb710/Kconfig"
source "drivers/misc/ti-st/Kconfig"
source "drivers/misc/lis3lv02d/Kconfig"
source "drivers/misc/altera-stapl/Kconfig"
```

* Makefile 제일 하단에 다음 내용을 추가한다.

~/poky_src/build2/tmp/work-shared/great/kernel-source/drivers/misc/Makefile
```
...
obj-$(CONFIG_NEW_TEST_DRIVER) += new_test_driver.o
```

* 현재 수정하는 커널 소스는 깃으로부터 받았기 때문에 수정/추가한 파일에 대해서는 `$ git status` 명령을 통해 확인할 수 있다.

* 패치 파일을 생성하는 방법은 크게 3가지가 있다.
  - `$ git diff` 명령어를 사용한 패치 파일 생성
    * 패치 생성
      ```
      ~/poky_src/build2/tmp/work-shared/great/kernel-source/drivers/misc$ git diff > new_test_driver.patch
      ```
    * 패치 반영: 해당 소스 파일로 이동해 다음 명령어를 실행한다.
      ```
      $ patch -p1 < xxx.patch
      또는
      $ git apply xxx.patch
      ```
    * 패치 적용 후 변경 내용을 수작업으로 깃에 커밋해 주어야 한다. (패치 적용 실패 시에는 롤백시킬 수 있음)
  - `$ git format-patch -n` 명령어를 사용한 패치 파일 생성
    * 변경/추가된 파일을 깃의 스테이징 영역으로 옮기고 `$ git commit` 명령을 입력한다.
      ```
      ~/poky_src/build2/tmp/work-shared/great/kernel-source/drivers/misc$ git add Kconfig Makefile new_test_driver.c
      ~/poky_src/build2/tmp/work-shared/great/kernel-source/drivers/misc$ git commit
      ```
    * 커밋 메시지를 입력한다. ("[Learning yocto] add new kernel driver")
    * 패치 파일을 생성한다.
      ```
      ~/poky_src/build2/tmp/work-shared/great/kernel-source/drivers/misc$ git format-patch -1
      ```
    * 현재 디렉토리에 '0001-xxx.patch' 파일이 생성된다.
    * `$ git format-patch -n` 명령어는 패치 파일을 만들 때 현재 커밋을 기준으로 n번 직전 커밋과 비교해 패치 파일을 만들어낸다.
    * 만들어진 패치 파일을 다른 동일한 커널 소스에 반영하려면 패치를 반영하려는 커널 소스로 이동해 다음을 실행한다.
      ```
      $ git am xxx.patch    # 패치를 적용하고 추가로 커밋을 자동 생성함
      ```
  - quilt 툴을 사용한 패치 파일 생성
    * 퀼트: 패키지 및 사용자 정의 애플리케이션을 패치로 만들어 작업할 때 사용할 수 있는 범용 패치 메커니즘이다. (각 패치의 변경 사항을 추적함)
    * 퀼트 패치는 스택으로 관리되며 이전의 모든 패치와 함께 기본 가상 폴더 트리 상단에 점진적으로 적용됨
    * quilt 사용 방법은 다음과 같다
      - quilt 툴 설치: `$ sudo apt install quilt`
      - 패치 생성하기: `$ quilt new <패치 이름>`
      - 현재 패치에 파일 추가하기: `$ quilt add <파일 이름>`
      - 패치 파일 수정
      - `$ quilt files` 명령으로 현재 패치에 들어 있는 파일들을 볼 수 있다.
      - `$ quilt remove <파일 이름>` 명령어를 통해 패치 파일에 추가한 파일을 삭제할 수 있다.
      - 현재 패치의 내용 보기: `$ quilt diff`
      - 만들어진 패치 저장하기: `$ quilt refresh` (patches 디렉토리에 저장됨)
      - 최근 refresh 이후의 수정 내용 보기: `$ quilt diff`
    * 다음은 퀼트에 대한 예제이다.
      - 임의의 디렉토리(예: test)를 생성하기 디렉토리 내에 test.txt 파일을 생성한다.
        ```
        Hello
        I'm yocto.
        bye
        ```
      - 새로운 패치 파일을 생성한다. (`$ quilt new mychange.patch`)
      - 패치에 파일을 추가한다. (`~/test$ quilt add test.txt`)
      - test.txt 파일을 수정한다.
        ```
        Hello
        I'm yocto.
        I supports linux build
        bye
        ```
      - `$ quilt diff` 명령으로 변경 사항을 비교할 수 있다.
      - `$ quilt refresh` 명령으로 기존 생성된 패치 파일을 갱신한다.

## 생성된 패치 및 환경 설정 단편 파일 커널 레시피에 추가

* 앞에서 만든 환경 설정 단편 파일, 패치 파일을 복사해 커널 레시피를 위한 작업 디렉토리(recipes-kernel/linux)에 이동한다.

* 앞에서 `$ git format-patch -1`을 통해 만든 '0001-xxx.patch' 파일을 'recipes-kernel/linux/file' 디렉토리에 복사하고 커널 환경 설정 단편 파일을 만든다.

* `$ make -c menuconfig virtual/kernel` 명령을 통해 메뉴 편집기에서 새로 추가한 커널 환경 설정을 선택하고 저장한 후 `$ make -c diffconfig virtual/kernel`을 통해 커널 환경 설정 단편 파일을 만든다.
  - 간단한 명령어로 만들 수도 있다: `$ echo "CONFIG_NEW_TEST_DRIVER=y" >> new-kernel-driver.cfg

* 생성된 커널 환경 설정 단편 파일을 'recipes-kernel/linux/file' 디렉토리에 복사한다.

```
~/poky_src/poky/meta-great-bsp
|- conf
|   |- layer.conf
|   |- machine
|       |- great.conf
|- recipes-kernel
    |- linux
        |- file
        |   |- 0001-Learning-yocto-add-new-kernel-driver.patch
        |   |- new-kernel-driver.cfg
        |- linux-yocto_5.4.bbappend
```

* 커널 레시피 확장 파일(linux-yocto_5.4.bbappend) 내용을 다음과 같이 수정한다.

~/poky_src/poky/meta-great-bsp/recipes-kernel/linux/linux-yocto_5.4.bbappend
```
SRC_URI += "file://new-kernel-driver.cfg \
          file://0001-Learning-yocto-add-new-kernel-driver.patch \
          "

KBRANCH_great = "v5.4/standard/base"
KMACHINE_great = "qemux86-64"
SRCREV_machine_great = "35826e154ee014b64ccfa0d1f12d36b8f8a75939"
COMPATIBLE_MACHINE_great = "great"
LINUX_VERSION_great = "5.4.219"

FILESEXTRAPATHS_prepend := "${THISDIR}/file:"
```

* 새로 만든 디바이스 드라이버가 정상적으로 동작하는지 확인한다. (fetch 태스크부터 실행해야 함)

```
$ bitbake virtual/kernel -C fetch
$ bitbake great-image
$ runqemu great-image nographic
```

* QEMU가 실행되면 로그인 후 신규 디바이스 드라이버가 정상적으로 실행됐는지 확인한다.

```
root@great:~# dmesg | grep "This is new"
[커널 메시지 출력...]
```

* 참고: 다음은 추가된 환경 설정 단편 파일의 유효성 검사를 위한 명령어이다. (추가된 환경 설정 옵션이 최종 .config 파일에 생성되었는지 여부)

```
$ bitbake -c kernel_configcheck -f virtual/kernel
```

## devshell을 이용한 코드 수정

* devshell: bitbake와 동일한 컨텍스트에서 실행되는 터미널 셸
  - 소스 수정/빌드를 위한 도구
  - devshell 실행 명령어: `$ bitbake <recipe name> -c devshell`
  - 이처럼 devshell을 이용하면 특정 레시피 소스로 쉽게 이동할 수 있다.
  - 경우에 따라서는 태스크 스크립트 '../temp/do.run_compile'을 실행하여 컴파일을 실행할 수도 있다. (가령 u-boot 패키지) 이는 devshell을 실행하면 빌드에 필요한 기본 환경 설정이 초기화되어 있기 때문이다.

* 만약 커널 소스로 이동하고 싶으면 `$ bitbake virtual/kernel -c devshell`이라고 하면 된다.
  - 레시피에 포함된 모든 패치가 적용된 커널 소스가 존재하는 디렉토리에서 터미널이 열린다.
  - 열린 터미널에서 커널 코드를 수정하고 패치, 환경 설정 단편 파일을 생성한다.
  - 작업을 마치고 devshell을 종료하고 커널 레시피에 방금 생성한 패치 및 환경 설정 단편 파일을 추가한다.
  - bitbake 빌드를 진행하면 된다.

## 커널 메타데이터

* 커널 메타데이터 (= 진보된 메타데이터)
  - 사전에 만들어 놓은 커널 패치와 환경 설정 옵션을 리포지터리를 통해 제공하는 데이터
  - 커널 환경 설정의 복잡성을 줄이고, 다양한 BSP를 지원하는 데 사용되는 소스 패치들을 제공하고, 다양한 리눅스 커널 유형들을 다루기 위함
  - Yocto에서는 커널 개발자들이 승인한 리눅스 커널의 개별 버전을 위한 각각의 리포지터리를 갖고 있으며, 각 리포지터리는 커널 소스를 위한 다수의 브랜치와 커널 메타데이터를 위한 하나의 브랜치(고아 브랜치)를 갖고 있음

* 커널 메타데이터를 포함하려면 커널 레시피에서 linux-yocto.inc 파일을 인클루드 해야 한다.
  - linux-yocto.inc 파일은 내부적으로 kernel.bbclass, kernel-yocto.bbclass 클래스 파일을 상속한다.
  - kernel-yocto.bbclass 내부에는 커널 메타데이터를 처리하는 부분이 존재함 (linux-yocto 스타일 레시피)

* linux-yocto 스타일 레시피에는 2개의 필수 변수와 2개의 선택적인 변수가 있음
  - KMACHINE (필수): 보통 MACHINE 변수와 같은 값을 갖지만 이것은 커널 혹은 커널 메타데이터와 연관성이 있는 반면, MACHINE 변수는 BSP 레이어에서 머신을 식별하는 데 사용한다.
  - KBRANCH (필수): 리눅스 커널을 빌드하는 데 사용하는 리눅스 커널 소스 리포지터리 브랜치를 가리키고 있음
  - KERNEL_FEATURES (선택): 기본적으로 BSP와 연관된 커널 메타데이터는 KMACHINE, LINUX_KERNEL_TYPE 변수를 통해 결정됨. 추가적인 커널 메타데이터가 필요할 때는 KERNEL_FEATURES 변수에 해당 커널 메타데이터를 할당한다.
  - LINUX_KERNEL_TYPE (선택): 베이스 커널 브랜치에 따른 커널 유형을 정의한다. (standard, tiny, preempt-rt 타입이 존재함)

* 커널 메타데이터는 3가지 기본 유형의 파일들로 구성되어 있다.
  - scc description 파일 (.scc): Serial Configuration Control. 일련의 환경 설정을 제어하는 파일.
    * define: 변수 정의
    * kconf: 환경 설정 단편 파일(.cfg)을 적용하는 지시어
    * patch: 패치 파일을 적용하는 지시어
    * include: 또 다른 scc 파일을 인클루드하는 지시어
  - 환경 설정 단편 파일 (.cfg)
  - 패치 파일 (.patch)

* 자세한 내용은 생략한다.
  - 아직까지 주요 칩 벤더들은 Yocto를 사용한 리눅스 커널을 배포할 때 커널 레시피에서 kernel.bbclass 클래스 파일만을 상속해 배포한다.
  - 다양한 보드와 아키텍처에 적합한 커스텀 리눅스 커널을 적용할 때 상당히 효율적으로 대체가 가능하다는 점에서 충분히 검토할 만한 내용이다.

## non linux-yocto 스타일 커널 레시피 구성

* 이번 장에서는 www.kernel.org 사이트에서 배포된 바닐라 커널 소스를 포함하는 새로운 커널 레시피를 만든다.
  - linux-yocto 스타일을 따르지 않는 방식이므로 non linux-yocto 스타일이라고 함
  - linux-yocto.inc 파일을 커널 레시피 파일에서 인클루드하지 않음
  - 즉 커널 메타데이터를 사용하지 않는다.

* 커널 환경 설정 단편 파일과 커널 소스 내에 defconfig 파일을 사용하려면 kernel-yocto.bbclass 클래스 파일이 필요하다.
  - 현재 주요 칩 벤더에서 배포되는 리눅스 커널은 non linux-yocto 스타일인 경우가 대부분이다.

### 실습 순서

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout non_linux_yocto`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b non_linux_yocto`

* 우리가 작성하고자 하는 커스텀 커널 레시피는 BSP 레이어에 포함되어 있어야 한다. BSP 레이어의 디렉토리 구성은 다음과 같다.
  - defconfig과 linux-mykernel.bb 파일이 새로 추가됨

```
meta-great-bsp/
|- conf
|   |- layer.conf
|   |- machine
|       |- great.conf
|- recipes-kernel
    |- linux
        |- file
        |   |- 0001-Learning-yocto-add-new-kernel-driver.patch
        |   |- defconfig
        |   |- myscc.scc
        |   |- new-kernel-driver.cfg
        |- linux-mykernel.bb
        |- linux-yocto_5.4.bbappend
```

* defconfig 파일 생성
  - .config 파일 추출하기: `$ bitbake -c kernel_configme linux_yocto`
  - 추출된 .config 파일의 위치는 다음과 같다: `~/poky_src/build2/tmp/work/great-poky-linux/linux-mykernel/5.4-rc8+gitAUTONIC+af42d3466b-r0/build/.config`
  - 이 파일을 defconfig으로 rename하고 recipes-kernel/linux/file 디렉토리로 복사한다.

* 새로운 커널 레시피 파일 생성
  - 새로 작성되는 커널 레시피인 linux-mykernel.bb 파일은 다음과 같다.

~/poky_src/poky/meta-great-bsp/recipes-kernel/linux/linux-mykernel.bb
```
DESCRIPTION = "Linux kernel from kernel.org git repository"
SECTION = "kernel"
LICENSE = "GPLv2"

inherit kernel
inherit kernel-yocto

SRC_URI = "git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git;protocol=git;nocheckout=1"
SRC_URI += "file://defconfig"

SRCREV = "af42d3466bdc8f39806b26f593604fdc54140bcb"

LIC_FILES_CHKSUM="file://COPYING;md5=bbea815ee2795b2f4230826c0c6b8814"

LINUX_VERSION ?= "5.4-rc8"
LINUX_VERSION_EXTENSION = "-mylinux"    # 커널 이미지 파일에 붙이는 접미어

PROVIDES += "virtual/kernel"
PV = "${LINUX_VERSION}+git${SRCPV}"    # 패키지 버전 넘버로 여기서는 리눅스 버전과 리비전 값이 할당됨
COMPATIBLE_MACHINE = "great"

FILESEXTRAPATHS_prepend := "${THISDIR}/file:"
```

* 새로 작성한 커널 레시피 파일을 설명하기 전에 사용하고자 하는 커널 소스를 미리 받아보도록 하자.
  - 임시 디렉토리 내에서 다음과 같이 소스를 받아온다.
  - `$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git`
  - `$ git show v5.4-rc8` 명령어를 통해 특정 커밋의 리비전 hash 값을 얻을 수 있다. (SRCREV 변수 값)
  - 만약 `SRCREV = "${AUTOREV}"`라고 하면 최신 리비전을 반영한다.

* SRC_URI 변수에서 커널 소스를 가져올 수 있다. (fetch) 다음은 fetcher가 주로 사용하는 파라미터이다.
  - branch: 체크아웃할 브랜치 이름. 지정하지 않으면 기본값으로 master를 설정한다. (Yocto kirkstone 버전에서는 필수로 입력해야 함)
  - name: 브랜치를 가리키는 가명. SRC_URI에서 기술된 파일들의 checksum을 제공하는 데 주로 사용됨.
  - tag: 특정 태그로 체크아웃하려고 할 때 사용됨. 별도로 추가하지 않으면 기본값으로 HEAD를 설정한다.
  - nocheckout: 1로 설정하면 깃을 통해 가져온 소스를 unpacking할 때 체크아웃하지 않도록 한다. 기본값은 0이다.

* 머신 환경 설정 파일 수정
  - 새로 생성한 커널 레시피가 현재 사용 중인 머신에서 사용되도록 설정하려면 머신 환경 설정 파일에서 현재 다중으로 정의된 커널 레시피 중 특정 레시피를 선택하기 위해 PREFERRED_PROVIDER_virtual/kernel 변수에 사용하고자 하는 레시피 파일 이름을 넣으면 된다.

~/poky_src/poky/meta-great-bsp/conf/machine/great.conf
```
...
WKS_FILE ?= "qemux86-directdisk.wks"

do_image_wic[depends] += "syslinux:do_populate_sysroot syslinux-native:do_populate_sysroot mtools-native:do_populate_sysroot dosfstools-native:do_populate_sysroot"

PREFERRED_PROVIDER_virtual/kernel = "linux-mykernel"

# For runqemu
QB_SYSTEM_NAME = "qemu-system-x86_64"
```

* 새로 만들어진 커널 레시피를 빌드하고 이미지를 다시 생성해 QEMU를 실행해본다.

```
$ bitbake linux-mykernel
$ bitbake great-image -C rootfs
$ runqemu great-image nographic
```

* QEMU 실행 후 로그인하고 커널이 반영되었는지 확인해 본다.

```
# uname -r
5.4.0-rc8-mylinux
```

# 커널 레시피 확장

* 깃을 이용해 외부로부터 커널 소스를 받아 빌드를 진행하는 방식은 커널 소스 수정이 번거롭고 수정된 소스 코드가 삭제될 우려가 있음
  - 커널 소스를 로컬에 위치시키고 커널 빌드를 진행하면 이런 문제들을 해결할 수 있다.
  - externalsrc 클래스 상속에 의해 구축된 로컬 커널 소스에서는 소스 내에서 defconfig 파일을 사용하는 것이 용이하다.
  - externalsrc 클래스를 상속 받으면 빌드시 fetch부터 patch까지의 태스크를 건너뛰게 됨

## externalsrc 클래스를 통한 로컬 커널 소스 사용

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout kernel_externalsrc`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b kernel_externalsrc`

* 커널 소스 위치 찾기
  - `~/poky_src$ bitbake-getvar -r great-image STAGING_KERNEL_DIR`
  - 커널 소스의 위치는 STAGING_KERNEL_DIR 변수에 저장된다. ("/home/poky_src/build2/tmp/work-shared/great/kernel-source")

* 커널 소스를 mykernel 디렉토리로 복사
  - `~/poky_src/source/mykernel$ cp -a ~/poky_src/build2/tmp/work-shared/great/kernel-source/* .`

* externalsrc 클래스를 사용해 커널 소스를 로컬로 지정하려면 레시피 파일을 수정해야 한다. 따라서 레시피 확장 파일을 새로 생성한다.

예제의 전체적인 디렉토리 구조는 다음과 같다.
```
meta-great-bsp
|- append
|   |- linux-mykernel.bbappend
|- conf
|   |- layer.conf
|   |- machine
|       |- great.conf
|- recipes-kernel
    |- linux
        |- file
        |   |- 0001-Learning-yocto-add-new-kernel-driver.patch
        |   |- defconfig
        |   |- myscc.scc
        |   |- new-kernel-driver.cfg
        |- linux-mykernel.bb
        |- linux-yocto_5.4.bbappend
```

~/poky_src/poky/meta-great-bsp/append/linux-mykernel.bbappend
```
inherit externalsrc
EXTERNALSRC = "${COREBASE}/../source/mykernel/kernel-source"
```

* bitbake가 새로운 레시피 확장 파일을 인식할 수 있도록 layer.conf 파일의 BBFILES 변수에 추가된 레시피 확장 파일의 경로를 추가한다.

~/poky_src/poky/meta-great-bsp/conf/layer.conf
```
BBPATH =. "${LAYERDIR}:"
BBFILES += "${LAYERDIR}/recipes*/*/*.bb"
BBFILES += "${LAYERDIR}/recipes*/*/*.bbappend"
BBFILES += "${LAYERDIR}/append/*.bbappend"
BBFILE_COLLECTIONS += "greatbsp"
BBFILE_PATEERN_greatbsp = "^${LAYERDIR}/"
BBFILE_PRIORITY_greatbsp = "6"
LAYERSERIES_COMPAT_greatbsp = "${LAYERSERIES_COMPAT_core}"
```

* 커널을 빌드한다.

```
$ bitbake linux-mykernel -c cleanall && bitbake linux-mykernel
$ bitbake great-image -C rootfs
```

* 빌드 완료 후 기존에 externalsrc 클래스를 사용하기 전에 커널 소스가 존재했던 위치에 가면 소스 디렉토리가 심볼릭 링크로 되어 있는 것을 볼 수 있다.
  - 심볼릭 링크는 현재 커널 소스가 위치한 로컬의 경로를 가리킴

```
~/poky_src/build2/tmp/work-shared/great$ ls -al
kernel-source -> /home/poky_src/poky/../source/mykernel/kernel_source
```

## 커널 소스 내의 defconfig 파일 사용

* 보통은 커널 소스 내에서 defconfig 파일을 제공하는 방식을 더 많이 사용한다. (트리 내 defconfig)
  - 커널 소스 내에 위치하는 defconfig 파일을 사용하기 위해서는 kernel-yocto.bbclass 클래스 파일을 커널 레시피에서 상속해야 한다.
  - 보통 defconfig 파일은 리눅스 커널 소스를 배포하는 쪽에서 함께 배포함
  - 그러므로 이 방법이 커널 소스를 배포하는 배포자의 입장에서 관리하기 편함
  - defconfig 파일은 커널 소스 내 'arch/<아키텍처>/configs/' 디렉토리에 위치함
  - KBUILD_DEFCONFIG 변수에 사용하고자 하는 defconfig 파일 이름을 할당한다.

### 실습 순서

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout internal_defconfig`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b internal_defconfig`

* 기존에 사용하던 defconfig 파일을 커널 소스 내로 옮긴다.
  - 기존 defconfig 파일 위치: `~/poky_src/poky/meta-great-bsp/recipes-kernel/linux/file/defconfig`
  - x86 기반 QEMU 머신의 경우 defconfig이 저장되는 위치: `kernel-source/arch/x86/configs/my_defconfig` (파일명을 defconfig에서 my_defconfig으로 변경)

* 마지막으로 ~/poky_src/poky/meta-great-bsp/recipes-kernel/linux/linux-mykernel.bb 파일을 수정하여 KBUILD_DEFCONFIG 변수에 my_defconfig을 할당한다.
  ```
  DESCRIPTION = "Linux kernel from kernel.org git repository"
  SECTION = "kernel"
  LICENSE = "GPLv2"

  inherit kernel
  inherit kernel-yocto

  SRC_URI = "git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git;protocol=git;tag=v5.4-rc8"
  # SRC_URI += "file://defconfig"
  # SRC_URI += "file://0001-Learning-yocto-add-new-kernel-driver.patch"
  # SRC_URI += "file://new-kernel-driver.cfg"

  # SRCREV = "af42d3466bdc8f39806b26f593604fdc54140bcb"
  KBUILD_DEFCONFIG = "my_defconfig"

  LIC_FILES_CHKSUM = "file://COPYING;md5=bbea815ee2795b2f4230826c0c6b8814"

  LINUX_VERSION ?= "5.4-rc8"
  LINUX_VERSION_EXTENSION = "-mylinux"

  PROVIDES += "virtual/kernel"

  PV = "${LINUX_VERSION}+git${SRCPV}"
  COMPATIBLE_MACHINE = "great"
  FILESEXTRAPATHS_prepend := "${THISDIR}/file:"
  ```

* 앞에서 추가했던 커널 드라이버에 대한 커널 패치 및 커널 환경 설정 단편 파일은 이 예제에서는 추가할 수 없다.
  - 이 예제는 externalsrc 클래스를 상속하므로 do_patch 태스크가 실행되지 않기 때문이다.
  - '.scc' 파일을 사용하게 되면 패치 파일 반영에 do_patch 태스크가 필수이다.

* 커널 빌드를 수행하고 이미지를 생성해 QEMU를 실행해 본다.

* 빌드를 진행하고 QEMU를 실행해 보면 정상적으로 실행되는 것을 볼 수 있다.
  ```
  $ bitbake virtual/kernel -C fetch
  $ bitbake great-image -C rootfs
  $ runqemu great-image nographic
  ```

## 커널 소스 밖에서 커널 모듈 생성

* 본래부터 커널 소스 코드에서 제공하지 않는 디바이스 드라이버의 경우 커널 소스 밖에서 모듈을 빌드하도록 배포할 수 있다.

* 다음과 같은 이유로 많은 디바이스 제공 업체가 이와 같은 방식으로 디바이스 드라이버 코드를 배포한다.
  - 라이선스로 인한 비공개 모듈이 있는 경우
  - 비필수 드라이버의 로딩을 연기해 부팅 시간을 줄이려는 경우
  - 로드해야 하는 드라이버가 많아 정적으로 링크할 때 너무 많은 메모리를 사용할 경우

* 커널 모듈을 커널 소스 밖에서 빌드하려면 module.bbclass 클래스 파일을 상속해야 한다.

### 실습 순서

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout external_kernelmod`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b external_kernelmod`

```
~/poky_src/poky/meta-great-bsp/
|- append
|   |- linux-mykernel.bbappend
|- conf
|   |- layer.conf
|   |- machine
|       |- great.conf
|- recipes-kernel
|   |- linux
|       |- file
|       |   |- 0001-Learning-yocto-add-new-kernel-driver.patch
|       |   |- defconfig
|       |   |- myscc.scc
|       |   |- new-kernel-driver.cfg
|       |- linux-mykernel.bb
|       |- linux-yocto_5.4.bbappend
|- recipes-mykernelmod
    |- file
    |   |- COPYING
    |   |- Makefile
    |   |- mykernelmod.c
    |- mykernelmod.bb
```

* layer.conf 수정
  - 새로 생성된 레시피 파일을 bitbake가 인식할 수 있도록 함

~/poky_src/poky/meta-great-bsp/conf/layer.conf
```
BBPATH =. "${LAYERDIR}:"
BBFILES += "${LAYERDIR}/recipes*/*/*.bb"
BBFILES += "${LAYERDIR}/recipes*/*/*.bbappend"
BBFILES += "${LAYERDIR}/append/*.bbappend"
BBFILES += "${LAYERDIR}/recipes*/*.bb"    # 추가됨
BBFILE_COLLECTIONS += "greatbsp"
BBFILE_PATTERN_greatbsp = "^${LAYERDIR}/"
BBFILE_PRIORITY_greatbsp = "6"
LAYERSERIES_COMPAT_greatbsp = "${LAYERSERIES_COMPAT_core}"
```

* COPYING 파일 생성
  - 모든 레시피 파일들을 레시피가 빌드하는 소프트웨어 패키지에 적용되는 소스 라이선스 목록을 LICENSE 변수에 할당해야 함
  - 라이선스 유효성 검증을 위해 LIC_FILES_CHKSUM 변수도 설정할 것

~/poky_src/poky/meta-great-bsp/recipes-mykernelmod/file/COPYING
```
This is my kernel module example file.
This file is for checksum.
Bye!
```

* mykernelmod.c 파일 생성

~/poky_src/poky/meta-great-bsp/recipes-mykernelmod/file/mykernelmod.c
```
#include <linux/module.h>
int init_module(void)
{
    printk("hello kernel module! \n");
    printk("hello kernel module! \n");
    printk("hello kernel module! \n");
    printk("hello kernel module! \n");
    printk("hello kernel module! \n");
    return 0;
}

void cleanup_module(void)
{
    printk("Goodbye kernel module!\n");
}

MODULE_LICENSE("MIT");
```

* Makefile 생성
  - 파라미터 M: 커널 소스 밖에서 모듈이 빌드된다는 것을 의미함
  - 코드 내에 탭 대신 스케이스가 들어가면 오류가 발생함 ('missing separator.  Stop.')

~/poky_src/poky/meta-great-bsp/recipes-mykernelmod/file/Makefile
```
obj-m := mykernelmod.o
SRC := $(shell pwd)
all:
  $(MAKE) -C $(KERNEL_SRC) M=$(SRC) modules
modules_install:
  $(MAKE) -C $(KERNEL_SRC) M=$(SRC) modules_install
clean:
  rm -rf *.o
  rm -f Module.markers Module.symvers modules.order
  rm -rf .tmp_versions Modules.symbers
```

* 커널 모듈을 추가하는 레시피 파일 mykernelmod.bb

~/poky_src/poky/meta-great-bsp/recipes-mykernelmod/mykernelmod.bb
```
SUMMARY = "example of how to build an external linux kernel module"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://COPYING;md5=ade99a31b7125f81bb82ffc454b3e6ac"

inherit module    # 커널 모듈 사용을 위해 상속 받아야 하는 클래스 파일이며 모듈을 컴파일하고 자동 실행하게 하는 등의 작업을 손쉽게 할 수 있음
SRC_URI = "file://Makefile \
          file://mykernelmod.c \
          file://COPYING \
          "
KERNEL_MODULE_AUTOLOAD += "mykernelmod"    # 부팅시 커널 모듈이 자동으로 로드되도록 함
S = "${WORKDIR}"
ALLOW_EMPTY_${PN} = "1"    # 따로 패키지를 만들지 않아 오류가 발생할 수 있으므로 이와 같이 처리함
FILESEXTRAPATHS_prepend := "${THISDIR}/file:"
```

* 만든 커널 모듈 파일을 루트 파일 시스템에 저장
  - 빌드를 통해 만들어진 커널 모듈(.ko)을 루트 파일 시스템에 저장하고, 저장된 커널 모듈이 부팅시 systemd에 의해 자동으로 실행되도록 KERNEL_MODULE_AUTOLOAD 변수에 커널 모듈의 이름을 할당함
  - 루트 파일 시스템 이미지를 생성하는 레시피 파일은 great-image.bb이므로 생성된 커널 모듈 패키지를 이미지 레시피의 IMAGE_INSTALL 변수에 추가함

~/poky_src/poky/meta-great/recipes-core/images/great-image.bb
```
SUMMARY = "A very small image for yocto test"

inherit great-base-image

LINGUAS_KO_KR = "ko-kr"
LINGUAS_EN_US = "en-us"

IMAGE_LINGUAS = "${LINGUAS_KO_KR} ${LINGUAS_EN_US}"
IMAGE_INSTALL += "packagegroup-great"
IMAGE_INSTALL += "mykernelmod"

IMAGE_OVERHEAD_FACTOR = "1.3"

inherit extrausers
EXTRA_USERS_PARAMS = "\
  groupadd greatgroup; \
  useradd -p `openssl passwd 9876` great; \
  useradd -g greatgroup great; \
  "
```

* 새로 생성된 커널 모듈 레시피를 빌드하고, 최종 이미지를 만들어 QEMU에서 실행한다.

```
$ bitbake mykernelmod
$ bitbake great-image -C rootfs
$ runqemu great-image nographic
```

## MACHINE_EXTRA_RDEPENDS, MACHINE_ESSENTIAL_EXTRA_RDEPENDS 변수를 이용한 커널 모듈 설치

* IMAGE_INSTALL 변수에 커널 모듈을 생성하는 패키지 이름을 추가하는 방법보다는 MACHINE_EXTRA_RDEPENDS, MACHINE_ESSENTIAL_EXTRA_RDEPENDS 변수에 커널 모듈을 추가하는 방법을 더 선호한다.
  - 커널 모듈 대부분이 하드웨어에 의존적이며, BSP 레이어에서 추가한 설치 파일이 이미지 레시피의 IMAGE_INSTALL에 들어가는 것이 모양상 좋지 않기 때문이다.
  - MACHINE_EXTRA_RDEPENDS: packagegroup-base.bb 패키지 그룹 레시피 파일 기반으로 만들어진 레시피 파일에서 사용됨, 부팅시 필수적으로 사용되는 패키지가 아닐 때 사용됨
  - MACHINE_ESSENTIAL_EXTRA_RDEPENDS: packagegroup-core-boot.bb 패키지 그룹 레시피 파일 기반으로 만들어진 이미지 레시피 파일에서 사용됨, 부팅시 필수적으로 사용되는 패키지일 때 사용됨

### 실습 순서

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout external_kernelmod2`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b external_kernelmod2`

* ~/poky_src/poky/meta-great/classes/great-base-image.bbclass 파일 수정
  ```
  inherit core-image

  IMAGE_FSTYPES = " tar.bz2 ext4"
  IMAGE_ROOTFS_SIZE = "10240"
  IMAGE_ROOTFS_EXTRA_SPACE = "10240"
  IMAGE_ROOTFS_ALIGNMENT = "1024"

  CORE_IMAGE_BASE_INSTALL = "\
     packagegroup-core-boot \    # 추가됨
     packagegroup-base-extended \
     ${CORE_IMAGE_EXTRA_INSTALL} \
  "
  ```

* 기존 great-image.bb 레시피 파일에서 IMAGE_INSTALL 변수를 통해 추가한 커널 모듈을 주석 처리한다.

~/poky_src/poky/meta-great/recipes-core/images/great-image.bb
```
SUMMARY = "A very small image for yocto test"

inherit great-base-image
LINGUAS_KO_KR = "ko-kr"
LINGUAS_EN_US = "en-us"
IMAGE_LINGUAS = "${LINGUAS_KO_KR} ${LINGUAS_EN_US}"
IMAGE_INSTALL += "packagegroup-great"
# IMAGE_INSTALL += "mykernelmod"
IMAGE_OVERHEAD_FACTOR = "1.3"

inherit extrausers
EXTRA_USERS_PARAMS = "\
  groupadd greatgroup; \
  useradd -p `openssl passwd 9876` great; \
  useradd -g greatgroup great; \
  "
```

* 머신 환경 설정 파일 great.conf에서 제일 하단의 MACHINE_ESSENTIAL_EXTRA_RDEPENDS 변수에 새로 생성된 커널 모듈을 추가 할당한다.
  ```
  ...
  PREFERRED_PROVIDER_virtual/kernel = "linux-mykernel"

  #For runqemu
  QB_SYSTEM_NAME = "qemu-system-x86_64"

  MACHINE_ESSENTIAL_EXTRA_RDEPENDS += "kernel-module-mykernelmod"
  ```

* 커널 모듈을 만드는 레시피 파일에서 RPROVIDES 변수를 추가한다.

~/poky_src/poky/meta-great-bsp/recipes-mykernelmod/mykernelmod.bb
```
SUMMARY = "Example of how to build an wxternal linux kernel module"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://COPYING;md5=ade99a31b7125f81bb82ffc454b3e6ac"

inherit module
SRC_URI = "file://Makefile \
          file://mykernelmod.c \
          file://COPYING \
          "

RPROVIDES_${PN} += "kernel-module-mykernelmod"    # 추가됨

KERNEL_MODULE_AUTOLOAD += "mykernelmod"
S = "${WORKDIR}"
ALLOW_EMPTY_${PN} = "1"
FILESEXTRAPATHS_prepend := "${THISDIR}/file:"
```

* 새롭게 커널 모듈을 빌드하고, 이미지를 생성해 QEMU로 실행해 본다.

```
$ bitbake mykernelmod -c cleanall && bitbake mykernelmod
$ bitbake virtual/kernel && bitbake great-image -C rootfs
$ runqemu great-image nographic
```

## 커널 모듈이 자동으로 실행되는 원리

* 부팅시 커널 모듈을 로딩하려면 '/etc/module-load.d/' 디렉토리에 '<module name>.conf' 파일을 만들어야 한다.
  - 파일 내용에 모듈 이름을 기록한다: `mykernelmod`
 
* mykernelmod.conf의 위치는 다음과 같다: `~/poky_src/build6/tmp/work/great-great-linux/great-image/1.0-r0/rootfs/etc/modules_load.d`

* 이는 systemd_modules-load.service를 통해 자동으로 로드되는 방법으로 systemd_modules-load.service가 활성화되어 있어야 한다.
  - QEMU를 실행하고 `# systemctl status systemd-modules-load`를 실행하면 활성화 여부를 알 수 있다.
  - 참고로 부팅시 로드될 커널 모듈의 리스트는 다음에 저장될 수 있다.
      * /etc/modules-load.d/*.conf
      * /run/modules-load.d/*.conf
      * /usr/lib/modules-load.d/*.conf

# 배포 레이어 작성

| 레이어 구조 |
| ----------- |
| 소프트웨어 레이어 (./meta-myproject) |
| 배포 레이어 (./meta-great) |
| BSP 레이어 (./meta-great-bsp) |
| 포키 참조 배포 레이어 (./meta-poky) |
| oe-core (./meta) |

* 배포 레이어: 배포 전반에 대한 정책을 제공한다.
  - BSP 레이어와 달리 배포 레이어는 배포 전반에 걸친 빌드에 대한 환경 설정을 갖고 있다.
  - 툴체인, 패키지 형식, C 라이브러리, systemd와 같은 초기화 관리자 선택
  - wifi, bluetooth와 같은 기능이 배포에 포함될지를 결정하기도 함
  - <distribution layer>/distro/<distro name>.conf 레이어 파일이 존재한다. (배포 환경 설정 파일)

* Yocto에서 참조로 만들어 놓은 배포 레이어: poky/meta-poky
  - 배포 환경 설정 파일: poky/meta-poky/conf/distro/poky.conf (이 파일의 이름은 local.conf 파일의 DISTRO 변수에 정의되어 있음: `DISTRO ?= "poky"`)

* 어떤 툴체인을 사용할 것인지 여부는 배포 레이어가 결정하고 TCMODE 변수에 의해 설정된다.
  - `~/poky_src/build$ bitbake-getvar -r great-image TCMODE`를 실행하면 `TCMODE="default"`임을 확인할 수 있다.
  - TCMODE는 ToolChain Mode 혹은 Target Compiler Mode의 약어로, 타깃 장치에 따라 빌드 시스템이 사용하는 컴파일러의 종류와 동작 방식을 지정하는 변수이다.
  - 툴체인은 tcmode-${TCMODE}.inc 파일에 포함되어 있으며, 전체 경로는 'poky/meta/conf/distro/include/tcmode-${TCMODE}.inc'이다.
  - inc 파일 내부에는 PREFERRED_PROVIDER에 의해 선택된 레시피들을 볼 수 있는데, 툴체인을 컴파일하고 설치하는 방법에 대해 정의하고 있다.

## 자신만의 배포 레이어 생성하기

### 실습 순서

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout custom_distro`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b custom_distro`

* 만들고자 하는 배포 레이어의 전체 구조는 다음과 같다.

```
~/poky_src/poky/meta-great
|- classes
|   |- great-base-image.bbclass
|- conf
|   |- distro
|   |   |- great-distro.conf
|   |- layer.conf
|- recipes-core
|   |- images
|   |   |- core-image-minimal.bbappend
|   |   |- great-image.bb
|   |- packagegroups
|       |- packagegroup-great.bb
|- template
    |- bblayers.conf.sample
    |- conf-notes.txt
    |- local.conf.sample
```

* 빌드 스크립트 buildenv.sh에서 DISTRO 변수 설정

buildenv.sh 빌드 스크립트
```
# !/bin/bash
function find_top_dir()
{
    local TOPDIR=poky
# move into script file path
    cd $(dirname ${BASH_SOURCE[0]})
    if [ -d $TOPDIR ]; then
        echo $(pwd)
    else
        while [ ! -d $TOPDIR ] && [ $(pwd) != "/" ];
        do
            cd ..
        done
        if [ -d $TOPDIR ]; then
            echo $(pwd)
        else
            echo "/dev/null"
        fi
    fi
}

ROOT=$(find_top_dir)
export TEMPLATECONF=${ROOT}/poky/meta-great/template/
export MACHINE="great"
export DISTRO="great-distro"    # 추가됨
source poky/oe-init-build-env build2
```

* 배포 환경 설정 파일인 great-distro.conf 생성 (배포 설정 파일 poky.conf 파일을 이용함)

great-distro.conf
```
DISTRO = "great-distro"    # 짦은 배포 이름
DISTRO_NAME = "Great distro"    # 긴 배포 이름 (부팅시 콘솔 부트 프롬프트에 보여짐)
DISTRO_VERSION = "3.1.21"    # 배포 버전 문자열
DISTRO_CODENAME = "dunfell"    # Yocto 릴리즈 버전 코드네임
SDK_VENDOR = "-greatsdk"    # SDK를 제공한 벤더 이름
SDK_VERSION = "${@d.getVar('DISTRO_VERSION').replace('snapshot-${DATE}', 'snapshot')}"    # SDK 버전

MAINTAINER = "Great <great@great.org>"    # 유지 보수 담당자 이름과 이메일
TARGET_VENDOR = "-great"    # 타깃 공급 업체명
LOCALCONF_VERSION = "1"

DISTRO_VERSION[vardepsexclude] = "DATE"
SDK_VERSION[vardepsexclude] = "DATE"
# Override these in poky based distros

GREAT_DEFAULT_DISTRO_FEATURES = "largefile opengl ptest multiarch wayland vulkan"
GREAT_DEFAULT_EXTRA_RDEPENDS = "packagegroup-core-boot"
GREAT_DEFAULT_EXTRA_RRECOMMENDS = "kernel-module-af-packet"
DISTRO_FEATURES ?= "${DISTRO_FEATURES_DEFAULT} ${GREAT_DEFAULT_DISTRO_FEATURES}"    # 원하는 기능을 추가함 (아래 외에도 여러 가지가 있음)
    # nfs: 클라이언트 지원 포함
    # Opengl: Open Graphics Library 포함
    # pci: PCI 버스 지원 포함
    # pcmcia: PCMCIA/CompactFlash 지원 포함
    # ppp: PPP 다이얼업 지원 포함
    # ptest: 개발 레시피에서 지원되는 패키지 테스트를 빌드하도록 설정함
    # smbfs: SMB 네트워크 클라이언트 지원 포함
    # systemd: systemd init 관리자에 대한 지원 포함
    # usbgadget: USB Gadget Device 지원 포함 (USB 네트워킹/시리얼/스토리지)
    # usbhost: USB 호스트 지원 포함 (외부 키보드, 마우스, 스토리지, 네트워크 등)
    # wayland: Wayland 디스플레이 서버 프로토콜과 해당 프로토콜을 지원하는 라이브러리 포함
    # wifi: WiFi 지원 포함
    # x11: X 서버와 라이브러리 포함

PREFERRED_VERSION_linux-yocto ?= "5.4%"
SDK_NAME = "${DISTRO}-${TCLIBC}-${SDKMACHINE}-${IMAGE_BASENAME}-${TUNE_PKGARCH}-${MACHINE}"
SDKPATHINSTALL = "/opt/${DISTRO}/${SDK_VERSION}"
DISTRO_EXTRA_RDEPENDS += " ${GREAT_DEFAULT_EXTRA_RDEPENDS}"    # 배포를 위한 실행 시간 의존성을 설정함
DISTRO_EXTRA_RRECOMMENDS += " ${GREAT_DEFAULT_EXTRA_RRECOMMENDS}"    # 추가적인 기능 제공을 위해 추천되는 패키지를 할당함 (해당 패키지가 존재하지 않아도 빌드는 실패하지 않음)
GREATQEMUDEPS = "${@bb.utils.contains("INCOMPATIBLE_LICENSE", "GPL-3.0", "", "packagegroup-core-device-devel",d)}"

DISTRO_EXTRA_RDEPENDS_append_great = " ${GREATQEMUDEPS}"
TCLIBCAPPEND = ""

# QA check settings - a little stricter than the OE-Core defaults
WARN_TO_ERROR_QA = "already-stripped compile-host-path install-host-path \
                   installed-vs-shipped ldflags pn-overrides rpaths staticdev \
                   unknown-configure-option useless-rpaths"

WARN_QA_remove = ${WARN_TO_ERROR_QA}"
ERROR_QA_append = ${WARN_TO_ERROR_QA}"

require conf/distro/include/no-static-libs.inc
require conf/distro/include/yocto-uninative.inc
require conf/distro/include/security_flags.inc

INHERIT += "uninative"
INHERIT += "reproducible_build"
BB_SIGNATURE_HANDLER ?= "OEEquivHash"    # bitbake가 사용하는 시그니처 핸들러 이름, 공유 상태 캐시에서 사용되는 시그니처와 스탬프 파일이 생성되고 처리되는 방식을 위해 사용됨
BB_HASHSERVE ??= "auto"
```

* 새로 생성된 배포 레이어 반영을 위한 빌드 진행

```
$ bitbake great-image
$ runqemu great-image nographic
```

* QEMU를 실행하면 다음과 같은 로그인 화면을 볼 수 있다.
  - DISTRO_NAME 변수 값: Great distro
  - 현재 사용 중인 욕토 버전: 3.1.21
  - 머신 이름: great

```
Great distro 3.1.21 great ttyS0
great login: _
```

## DISTRO_FEATURES, IMAGE_FEATURES, MACHINE_FEATURES의 차이점

DISTRO_FEATURES | IMAGE_FEATURES | MACHINE_FEATURES
---- | ---- | ----
시스템 전체적인 정책 결정 | 배포되는 이미지에 포함되기를 원하는 소프트웨어 기능을 지정 | 머신이 지원할 수 있는 하드웨어 기능을 지정

* 만약 MACHINE_FEATURES에만 wifi를 할당하고, DISTRO_FEATURES에 wifi를 할당하지 않으면?
  - wifi에 해당하는 커널의 디바이스 드라이버는 준비되지만, wifi를 위해 필요한 미들웨어나 애플리케이션은 준비되지 않을 것이다.

* 만약 DISTRO_FEATURES에만 wifi를 할당하고, MACHINE_FEATURES에 wifi를 할당하지 않으면?
  - 커널의 디바이스 드라이버가 준비되지 않은 상태이기 때문에 wifi에 필요한 미들웨어나 애플리케이션은 설치되지 않는다.


# 소프트웨어 레이어 작성

| 레이어 구조 |
| ----------- |
| 소프트웨어 레이어 (./meta-myproject) |
| 배포 레이어 (./meta-great) |
| BSP 레이어 (./meta-great-bsp) |
| 포키 참조 배포 레이어 (./meta-poky) |
| oe-core (./meta) |

* BSP, 배포 레이어는 칩을 제공하는 벤더에서 배포하므로 처음부터 만들 필요는 거의 없다.

* 실제로 추가/수정해야 하는 레이어는 소프트웨어 레이어가 될 것이다.
  - 생성한 애플리케이션이나 필요에 따라 커널이나 부트로더를 수정할 수 있고, 새로 추가되는 기능을 위해 배포 정책을 수정할 수도 있다.
  - Yocto는 기존의 메타데이터들을 수정하기보다는 새롭게 레이어를 만들어 추가/수정하도록 권고한다.
  - 레시피 확장 파일(.bbappend)을 사용해 추가/수정해야 한다.

### 실습 순서

* meta-myproject 레이어를 만들 것이다.
  - 애플리케이션을 위해 만든 기존 meta-hello, meta-nano-editor 레이어를 삭제하고 meta-myproject 레이어에 합칠 것이다.
  - 다음과 같이 레이어를 생성할 것이다.

```
~/poky_src/poky/meta-myproject
|- appends
|   |- nano_6.0.bbappend
|- conf
|   |- layer.conf
|- recipes-core
|   |- images
|   |   |- great-image.bbappend
|   |- packagegroups
|       |- packagegroup-great.bb
|- recipes-hello
|   |- hello.bb
|   |- source
|       |- COPYING
|       |- hello.c
|       |- hello.service
|- recipes-nano
    |- nano_6.0.bb
```

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout customer_layer`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b customer_layer`

* poky 디렉토리에 meta-myproject 디렉토리를 새로 생성한다.

* meta-myproject 디렉토리에 conf 디렉토리를 만들고, conf 디렉토리에 layer.conf 파일을 생성한다.

layer.conf
```
BBPATH =. "${LAYERDIR}:"    # LAYERDIR 변수가 갖는 값은 레이어 최상위 디렉토리인 meta-myproject이다.
BBFILES += "${LAYERDIR}/recipes*/*.bb \    # reecipes-hello/hello.bb와 recipes-nano/nano6.0.bb 파일을 bitbake가 인식할 수 있도록 추가한 경로이다.
           ${LAYERDIR}/recipes*/*/*.bb \    # recipes-core/packagegroups/packagegroup-great.bb 파일을 bitbake가 인식할 수 있도록 추가한 경로이다.
           ${LAYERDIR}/recipes*/*/*.bbappend \    # recipes-core/images/great-image.bbappend 파일을 bitbake가 인식할 수 있도록 추가한 경로이다.
           ${LAYERDIR}/appends*/*.bbappend \    # appends/nano_6.0.bbappend 파일을 bitbake가 인식할 수 있도록 추가한 경로이다.
           "

BBFILE_COLLECTIONS += "myproject"
BBFILE_PATTERN_myproject = "^${LAYERDIR}/"
BBFILE_PRIORITY_myproject = "12"    # meta-myproject 레이어가 최상위 레이어가 되도록 가장 큰 우선순위인 12로 설정한다.
LAYERSERIES_COMPAT_myproject = "${LAYERSERIES_COMPAT_core}"
```

* 기존의 meta-hello 레이어에서 recipes-hello 디렉토리를 meta-myproject 디렉토리 아래로 이동시키고 meta-hello 디렉토리를 삭제한다.

* 기존의 meta-nano-editor 레이어에서 recipes-nano, appends 디렉토리를 meta-myproject 디렉토리 아래로 이동시키고 meta-nano-editor 디렉토리를 삭제한다.

* meta-myproject 디렉토리에 recipes-core 디렉토리를 만들고 meta-great/recipes-core 디렉토리에 있는 packagegroups 디렉토리를 meta-myprject/recipes-core 디렉토리로 이동시킨다. 그리고 기존의 meta-great 디렉토리에 존재하는 recipes-core/packagegroups 디렉토리를 삭제한다.

* meta-myproject/recipes-core 디렉토리에 images 디렉토리를 만들고, 레시피 확장 파일인 great-image.bbappend 파일을 생성한 후 다음 내용을 입력한다.
  - `IMAGE_INSTALL += "packagegroup-great"`

* 위에 내용이 추가된 이유로 meta-great/recipes-core/images/great-image.bb 파일을 다음과 같이 수정한다.
  ```
  SUMMARY = "A very small image for yocto test"
  inherit great-base-image
  LINGUAS_KO_KR = "ko-kr"
  LINGUAS_EN_US = "en-us"
  IMAGE_LINGUAS = "${LINGUAS_KO_KR} ${LINGUAS_EN_US}"
  # IMAGE_INSTALL += "packagegroup-great"
  IMAGE_OVERHEAD_FACTOR = "1.3"
  ...
  ```

* 기존에 존재하던 meta-hello, meta-nano-editor 레이어가 삭제되었으므로 bblayers.conf.sample 파일에서 추가된 이 레이어들도 삭제되어야 한다.

~/poky_src/poky/meta-great/template/bblayers.conf.sample
```
POKY_BBLAYERS_CONF_VERSION = "2"
BBPATH = "${TOPDIR}"
BBFILES ?= ""
BBLAYERS ?= " \
  ##OEROOT##/meta \
  ##OEROOT##/meta-poky \
  ##OEROOT##/meta-yocto-bsp \
  ##OEROOT##/meta-great \
  ##OEROOT##/meta-great-bsp \
"
```

* 이제 meta-myproject 레이어를 추가해야 한다. 레이어 파일을 생성하는 방법도 있지만 다음과 같이 빌드 스크립트를 통해 자동으로 추가할 수도 있다.
  - `bitbake-layers add-layer` 명령어: bblayers.conf파일에 새롭게 추가된 레이어를 자동으로 추가해 주는 명령어

~/poky_src/buildenv.sh
```
# !/bin/bash

function find_top_dir()
{
    local TOPDIR=poky
# move into script file path
    cd $(dirname ${BASH_SOURCE[0]})
    if [ -d $TOPDIR ]; then
        echo $(pwd)
    else
        while [ ! -d $TOPDIR ] && [ $(pwd) != "/" ];
        do
            cd ..
        done

        if [ -d $TOPDIR ]; then
            echo $(pwd)
        else
            echo "/dev/null"
        fi
    fi
}

ROOT=$(find_top_dir)
export TEMPLATECONF=${ROOT}/poky/meta-great/template/
export MACHINE="great"
export DISTRO="great-distro"
function build_target() {
    source poky/oe-init-build-env build2
    bitbake-layers add-layer ../poky/meta-myproject
}

build_target
```

* bblayers.conf.sample 파일을 수정했으므로 build/conf/bblayers.conf 파일을 업데이트해야 한다.
  - build2/conf 디렉토리를 삭제하고 빌드 초기화 스크립트 buildenv.sh 스크립트를 실행한다.

```
~/poky_src$ rm -rf build2/conf
~/poky_src$ source buildenv.sh
```

* 이제 build2/conf/bblayers.conf 파일을 열어보면 meta-myproject 디렉토리가 BBLAYERS에 추가된 것을 확인할 수 있다.

* 빌드를 다시 진행하고 QEMU를 실행해 본다.

```
$ bitbake great-image
$ runqemu great-image nographic
```

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/ee9288c6-4d18-4157-ba95-3500f2708b45)


# 패키지

* 패키지: 타깃 시스템에 배포/설치를 위해 소프트웨어 바이너리, 라이브러리, 헤더 등을 하나의 묶음 파일로 만드는 절차
  - 대부분의 소프트웨어 패키지는 인스톨러, 패키지 관리 시스템에 맞게 묶는다.
  - 보통 리눅스 시스템은 단일 설치 패키지를 사용하기보다는 배포판에 포함된 패키지 관리 시스템을 이용한다.
  - 패키지 관리 시스템의 예: RPM(Redhat Package Manager), dpkg(Debian Package), opkg(Open Package)

## 빌드 과정에서의 패키지 태스크들

* 패키지 챕터에서는 do_install 태스크 이후의 태스크들에 대해 학습할 것이다.

* 빌드의 기본적인 설정들을 갖고 있는 local.conf 환경 설정 파일을 보면 기본적인 패키지 관리 시스템은 RPM으로 되어 있음을 확인할 수 있다.

local.conf
```
PACKAGE_CLASSES ?= "package_rpm"
```

* 다음과 같이 bitbake 빌드 과정에서 각 태스크들은 태스크의 실행이 완료될 때마다 그 결과물을 특정 디렉토리에 저장한다.

```
                do_fetch
                     |
                     |  ----> ${DL_DIR}에 소스 코드 저장
                     v
                do_unpack
                     |
                     |  ----> ${WORKDIR}/${S}에 압축 해제된 소스 코드 저장 (${S} = ${BPN}-${PV})
                     v
                do_patch
                     |
                     |  ----> ${WORKDIR}/${S}에 패치(patch) 적용 (${S} = ${BPN}-${PV})
                     v
                do_prepare_recipe_sysroot
                     |
                     |  ----> ${WORKDIR}에 특정 sysroots로 파일 설치
                     v
                do_configure
                     |
                     |  ----> ${WORKDIR}/${B}에 소프트웨어 구성 (${B}는 보통 build를 가리킴)
                     v
                do_compile
                     |
                     |  ----> ${WORKDIR}/${B}에 소프트웨어 컴파일
                     v
                do_install
                     |
                     |  ----> ${WORKDIR}/${D}에 소프트웨어 설치 (${D}는 보통 image를 가리킴)
         ------------------------
         |                      |
         |                      |
         v                      v
do_populate_sysroot       do_package
                                |
                                |  ----> ${WORKDIR}/package, ${WORKDIR}/packages-split에 파일 패키지화, ${WORKDIR}/pkgdata에 패키지 메타데이터 저장
                                v
                          do_packagedata
                                |
                                |  ----> ${TMPDIR}/pkgdata/${MACHINE}에 데이터 생성됨
                                v
                          do_package_write_rpm
                                |
                                |  ----> ${DEPLOY_DIR}/rpm (이 디렉토리를 Package Feed라고 함)에 패키지 생성됨
                                v
                          do_package_qa
```

* poky/meta/conf/bitbake.conf의 변수 PACKAGES, FILES 변수에 대해 이해할 필요가 있다.
  - PACKAGES: 레시피가 생성하는 패키지 리스트들의 목록. 공백으로 분류되며 기본값은 다음과 같다. 왼쪽부터 먼저 처리된다.
    `PACKAGES = "${PN}-src ${PN}-dbg ${PN}-staticdev ${PN}-dev ${PN}-doc ${PN}-locale ${PACKAGE_BEFORE_PN} ${PN}"`
  - FILES: 특정 패키지에 들어가는 디렉토리와 파일의 목록을 정의한다. 기본값은 다음과 같다.
    ```
    FILES_${PN} = "${bindir}/* ${sbindir}/* ${libexecdir}/* ${libdir}/lib*S{SOLIBS} \
                ${sysconfdir} ${sharedstatedir} ${localstatedir} \
                ${base_bindir}/* ${base_sbindir}/* \
                ${base_libdir}/*${SOLIBS} \
                ${base_prefix}/lib/udev ${prefix}/lib/udev \
                ${base_libdir}/udev ${libdir}/udev \
                ${datadir}/${BPN} ${libdir}/${BPN}/* \
                ${datadir}/idl ${datadir}/omf ${datadir}/sounds \
                ${libdir}/bonobo/servers"

    FILES_${PN}-bin = "${bindir}/* ${sbindir}/*"

    FILES_${PN}-doc = "${docdir} ${mandir} ${infodir} ${datadir}/gtk-doc \
                ${datadir}/gnome/help"
    SECTION_${PN}-doc = "doc"

    FILES_SOLIBSDEV ?= "${base_libdir}/lib*${SOLIBSDEV} ${libdir}/lib*${SOLIBSDEV}"
    FILES_${PN}-dev = "${includedir} ${FILES_SOLIBSDEV} ${libdir}/*.la \
                    ${libdir}/*.o ${libdir}/pkgconfig ${datadir}/pkgconfig \
                    ${datadir}/aclocal ${base_libdir}/*.o \
                    ${libdir}/${BPN}/*.la ${base_libdir}/*.la \
                    ${libdir}/cmake ${datadir}/cmake"
    SECTION_${PN}-dev = "devel"
    ALLOW_EMPTY_${PN}-dev = "1"
    RDEPENDS_${PN}-dev = "${PN} (= ${EXTENDPKGV})"

    FILES_${PN}-staticdev = "${libdir}/*.a ${base_libdir}/*.a ${libdir}/${BPN/*.a"
    SECTION_${PN}-staticdev = "devel"
    RDEPENDS_${PN}-staticdev = "${PN}-dev (= ${EXTENDPKGV})"

    FILES_${PN}-dbg = "/usr/lib/debug /usr/lib/debug-static /usr/src/debug"
    ...
    ```

### do_package 태스크

* do_package 태스크: 이미 빌드된 소프트웨어나 라이브러리를 패키지로 묶어서 설치 가능한 형태로 만드는 작업을 수행한다.
  - do_install 태스크에 의해 생성된 산출물 디렉토리 ${D}를 PKGD 변수가 가리키는 디렉토리로 복사한다. PKGD 변수는 ${WORKDIR}/package 디렉토리를 가리킨다.
  - ${WORKDIR}/package 디렉토리의 파일들은 PKGDEST 변수에서 지정한 디렉토리로 복사된다. PKGDEST 변수는 ${WORKDIR}/packages-split 디렉토리를 가리킨다. ($FILES_{PN}-<name> 디렉토리에 $FILES_{PN}-<name> 변수에 할당된 파일들이 복사됨)
  - PKGDESTWORK 변수에서 지정한 디렉토리인 ${WORKDIR}/pkgdata에 패키지 메타데이터를 저장한다.

* 예시) FILES 변수와 PACKAGES 변수는 다음과 같이 매칭된다.
  FILES 변수 | PACKAGES 변수 | packages_split 디렉토리
  ---------- | ------------ | -------------------------
  FILES_${PN} | ${PN} | hello
  FILES_${PN}-dbg | ${PN}-dbg | hello-dbg
  FILES_${PN}-staticdev | ${PN}-staticdev | hello-staticdev
  FILES_${PN}-dev | ${PN}-dev | hello-dev
  ... | ... | ...

* 빌드 결과물은 각자의 선택에 따라 서로 다른 패키지로 만들어질 수 있다. (Package Splitting)
  - 이렇게 하면 하나의 레시피 파일에서 다수의 패키지를 만들 수 있다.
  - 만들어진 패키지들 중 일부만 타깃 장치의 루트 파일 시스템에 설치한다.
  - 필요한 결과물만 설치해 최종 이미지의 크기를 줄이고, 장치 보안에 위해를 줄 수 있는 바이너리, 라이브러리, 디버그 정보 등을 설치하지 않도록 할 수 있다.
  - package-split 디렉토리 내 파일들은 패키지를 만들 수 있는 패키지 메타데이터로 변경된다.

### do_packagedata 태스크

* do_packagedata 태스크: do_package 태스크가 생성한 패키지와 관련된 데이터를 생성하고 저장하는 작업을 수행한다.
  - 패키지 메타데이터를 사용해 패키지가 설치될 경로, 라이브러리 및 실행 파일의 의존성 정보, 설정 파일을 정의한다.
  - do_package 태스크가 생성한 패키지 메타데이터를 전역적으로 사용할 수 있도록 ${TMPDIR}/pkgdata/${MACHINE} 디렉토리로 복사한다.
  - 이전 태스크들까지는 태스크 산출물들이 ${WORKDIR}에 있었지만, do_packagedata 태스크부터는 태스크의 산출물이 전역적으로 사용될 수 있도록 상위 디렉토리에 저장된다. (예: build/tmp/pkgdata/great/)

### do_package_write_rpm 태스크

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/a6769961-2f70-417b-973b-3ebbef680e09)

* do_package_write_rpm 태스크: RPM 패키지를 생성하고, 생성된 패키지를 ${TMPDIR}/deploy/rpm 디렉토리(Package Feed)에 위치시킨다.
  - 만약 커스텀 패키지 생성을 위해 PACKAGES 변수에 포함되지 않은 패키지가 필요할 경우, PACKAGE_BEFORE_PN 변수를 사용한다. (${PN} 패키지 생성 직전에 삽입됨)

### do_package_qa 태스크

* do_package_qa 태스크: RPM 패키지들에 대해 품질 보증 확인을 한다. 이 태스크는 패키지가 빌드되고 생성된 후에 실행되며, 패키지에 대한 다양한 품질 검증 도구를 사용해 문제를 찾고 보고한다.
  - 패키지의 파일 누락, 라이브러리 의존성 오류, 중복 파일, 빈 디렉토리 등을 확인하고 경고/오류 메시지를 출력한다.
  - insane.bbclass 클래스: 패키지 생성 프로세스에서 생성된 패키지들에 대한 품질 보증 검사를 실시한다.

* 만약 레시피 파일에서 하나 이상의 검사를 건너뛰고 싶다면 INSANE_SKIP 변수를 사용하면 된다. (예: `INSANE_SKIP_${PN} += "dev-so"`의 경우 심볼릭 링크 .so 파일에 대한 검사를 건너뜀)

* 다음은 QA 검사 목록의 일부이며 자세한 것은 [여기](https://docs.yoctoproject.org/ref-manual/classes.html#insane-bbclass)를 확인한다.

  변수명 | 테스트 설명
  ----------------- | -------------------------
  already-stripped | 생성된 실행 파일이 빌드 시스템에 의해 디버그 심볼이 추출되기 전에 이미 스트립됐는지 확인한다.
  arch | 실행 가능하고 링크 가능한 형식(ELF)의 종류, 비트 크기 및 엔디언을 확인해 이를 대상 아키텍처와 일치하는지 확인한다. (부트로더의 경우 우회해야 할 수도 있음)
  buildpaths | 출력 파일 내에서 빌드 호스트의 경로를 확인한다. (노출되면 보안 문제 발생 가능)
  build-deps | DEPENDS, 명시적인 RDEPENDS, 또는 태스크 수준의 의존성을 통해 지정된 빌드 시간 의존성이 실행 시간 의존성과 일치하는지를 결정한다.
  compile-host-path | do_compile 로그를 확인해 빌드 호스트의 경로가 사용됐는지를 확인한다. (노출되면 보안 문제 발생 가능)
  debug-deps | -dbg 패키지를 제외한 모든 패키지가 -dbg 패키지에 의존하는지 확인한다. (의존하면 패키징 버그 발생 가능)
  debug-files | -dbg 패키지 이외의 패키지에 .debug 디렉토리가 있는지를 확인한다. (있으면 패키징 오류)
  ... | ...

* 빈 패키지를 생성할 경우 QA 오류가 발생하는데, 빈 패키지를 허용하려면 다음과 같은 지시어를 추가해야 한다.
  ```
  ALLOW_EMPTY_${PN} = "1"
  ALLOW_EMPTY_${PN}-dev = "1"
  ALLOW_EMPTY_${PN}-staticdev = "1"
  ```

## 패키지 관리 도구

* RPM(RedHat Package Manager): 레드햇 계열의 리눅스 배포판에서 사용하는 패키지 관리 도구.
  - 초기에는 모든 패키지를 tar, gzip으로 묶은 소스 파일을 직접 컴파일하고 수동으로 설치했다.
  - 수동 설치의 문제점: 번거롭고, 의존성이 있는 모듈을 따로 설치해 주어야 하므로 많은 시간과 노력이 소요된다.
  - RPM은 자동 설치가 되지만 여전히 의존 패키지까지 자동으로 설치되지 않았기 때문에 yum이 등장하게 되었다.

* yum(Yellowdog Update Modified): 레드햇 계열의 리눅스 배포판에서 사용하는 패키지 관리 도구. RPM 명령어가 해결하지 못했던 패키지 의존성 문제를 해결했다.
  - yum 명령어를 사용하면 패키지 의존성 문제를 자동으로 처리하면서 설치/업데이트/삭제를 진행할 수 있다.

* bitbake dunfell에서는 yum 개선 버전인 dnf(Dandified YUM)를 패키지 관리 도구로 사용하고 있다.
  - 느린 속도, 과다한 메모리 사용, 의존성 결정이 느린 단점을 개선한 새로운 패키지 관리 도구이다.

### 라이브러리 생성을 통한 패키지 실습

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout makelib`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b makelib`

* 공유 라이브러리(동적 라이브러리)를 만들어 본다.
  - 공유 라이브러리를 만드는 레시피와 소스는 기존에 만들었던 meta-myproject 레이어 아래 레시피를 위한 작업 디렉토리 recipes-makelib를 생성하고 그 아래 넣어준다.

```
recipes-makelib/
|- files
|   |- cat.c
|   |- dog.c
|   |- func.h
|- makelib.bb
```

* files 디렉토리에 들어가는 소스 파일과 헤더 파일을 생성한다.

~/poky_src/poky/meta-myproject/recipes-makelib/files/cat.c
```cpp
#include <stdio.h>

void makevoicefromcat(void){
    printf ("Meow! Meow! Meow! \n");
}
```

~/poky_src/poky/meta-myproject/recipes-makelib/files/dog.c
```cpp
#include <stdio.h>

void makevoicefromdog(void){
    print ("Bark! Bark! Bark! \n");
}
```

~/poky_src/poky/meta-myproject/recipes-makelib/files/func.h
```cpp
#ifndef __FUNC_H__
#define __FUNC_H__

void makevoicefromdog(void);
void makevoicefromcat(void);

#endif
```

* 공유 라이브러리를 만드는 레시피 파일을 만들어본다.

~/poky_src/poky/meta-myproject/recipes-makelib/makelib.bb
```
DESCRIPTION = "This recipe makes shared library"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"
# 소스 트리 내에서 라이선스 파일을 참조할 수 없으면 오픈임베디드 코어에서 제공하는 라이선스인 ${COMMON_LICENSE_DIR}/MIT를 참조한다.

SRC_URI = "file://dog.c \
        file://cat.c \
        file://func.h \
        "

do_compile() {
    ${CC} -fPIC -c dog.c    # -fPIC 옵션: 위치 독립적인 코드(Position Independent Code)를 뜻함
    ${CC} -fPIC -c cat.c
    ${CC} ${LDFLAGS} -shared -Wl,-soname=libtest.so.1 -o libtest.so.1.0 *.o
      # LDFLAGS(Linker Flags): 링커에 전달되는 플래그로서, 링크 단계에서 사용되는 뒤따라오는 옵션들을 지정하는 변수이다.
      # shared: 컴파일러에게 실행이 가능한 오브젝트에 링크를 걸어줄 수 있는 공유 라이브러리를 만든다고 알려줌
      # Wl: Wl은 링커에게 전해주는 옵션으로 여기서는 -soname=libtest.so.1을 전달한다.
      # soname: API 호환성을 명시적으로 링커에게 알려줌 (libtest.so -> libtest.so.1 -> libtest.so.1.0) = (컴파일 시 사용되는 라이브러리 -> 런타임에 사용되는 라이브러리 -> 실제로 생성된 라이브러리)
      # `$ objectdump -p libtest.so.1.0` 명령을 통해 NEEDED (실행 시간에 여기에 표시된 라이브러리가 로드되어야 한다는 뜻), SONAME 섹션 값을 확인할 수 있다.
}

RPROVIDES_${PN} = "makelib"    # makelib 레시피에 의해 생성된 라이브러리에 실행 시간 의존성을 가진 패키지가 있다면 여기에 makelib을 넣어야 함

do_install() {
    install -d ${D}${libdir}
    install -m 0755 libtest.so.1.0 ${D}${libdir}
    ln -s libtest.so.1.0 ${D}${libdir}/libtest.so.1
    ln -s libtest.so.1 ${D}${libdir}/libtest.so
    install -d ${D}${includedir}
    install -m 0644 func.h ${D}${includedir}
}

FILES_${PN} = "${libdir}/libtest.so.1.0 ${libdir}/libtest.so.1"    # 런타임에 필요함
FILES_${PN}-dev = "${libdir}/libtest.so ${includedir}/func.h"    # 컴파일에 필요함
S = "${WORKDIR}"
FILESEXTRAPATHS_prepend := "${THISDIR}/files:":
```

* 라이브러리를 수동으로 컴파일하는 방법은 다음과 같다.

```
~/poky_src/poky/meta-myproject/recipes-makelib/files$ ls
cat.c    dog.c    func.h

# 수동 컴파일
~/poky_src/poky/meta-myproject/recipes-makelib/files$ gcc -fPIC -c cat.c
~/poky_src/poky/meta-myproject/recipes-makelib/files$ gcc -fPIC -c dog.c
~/poky_src/poky/meta-myproject/recipes-makelib/files$ gcc -shared -Wl,-soname=libtest.so.1 -o libtest.so.1.0 *.o

~/poky_src/poky/meta-myproject/recipes-makelib/files$ ls
cat.c    cat.o    dog.c    dog.o    func.h    libtest.so.1.0

# 심볼릭 링크 생성
~/poky_src/poky/meta-myproject/recipes-makelib/files$ ln -s libtest.so.1.0 libtest.so.1
~/poky_src/poky/meta-myproject/recipes-makelib/files$ ln -s libtest.so.1 libtest.so
```

* 이제 `$ bitbake makelib` 명령어로 빌드해 본다.
  - 특정 레시피를 빌드하면서 생성되는 작업 산출물 디렉토리는 WORKDIR 변수에 지정되어 있다.
  - image 디렉토리는 do_install 태스크가 수행되어 나오는 산출물들이 저장되는 디렉토리이다. (${WORKDIR}/{D} 디렉토리)

```
~/poky_src/build5/tmp/work/core2-64-great-linux/makelib/1.0-r0/image
|- usr
    |- include
    |   |- func.h
    |- lib
        |- libtest.so -> libtest.so.1
        |- libtest.so.1 -> libtest.so.1.0
        |- libtest.so.1.0
```

* do_package 태스크는 do_install 태스크의 결과물을 갖고 ${WORKDIR}/package 디렉토리에 저장한다.

```
~/poky_src/build5/tmp/work/core2-64-great-linux/makelib/1.0-r0/package
|- usr
    |- include
    |   |- func.h
    |- lib
        |- libtest.so -> libtest.so.1
        |- libtest.so.1 -> libtest.so.1.0
        |- libtest.so.1.0
```

* do_package 태스크는 패키지를 분류해 ${WORKDIR}/packages-split 디렉토리에 배치한다. 분리된 패키지들은 각각 rpm 파일로 만들어진다.

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/f1370049-6a1f-422a-8320-fe9c861ffdc7)

```
~/poky_src/build5/tmp/work/core2-64-great-linux/makelib/1.0-r0/packages-split
|- makelib
|   |- usr
|       |- lib
|           |- libtest.so.1 -> libtest.so.1.0
|           |- libtest.so.1.0
|- makelib-dbg
|   |- usr
|       |- lib
|- makelib-dev
|   |- usr
|       |- include
|       |   |- func.h
|       |- lib
|           |- libtest.so -> libtest.so.1
|- makelib-doc
|- makelib-locale
|- makelib-shlibdeps
|- makelib-src
|- makelib-staticdev
```

* do_package_write_rpm 태스크가 샐행되면 산출물들은 최종적으로 Package Feed인 tmp/deploy-rpms에 들어가게 된다.

```
~/poky_src/build5/tmp/work/core2-64-great-linux/makelib/1.0-r0/deploy-rpms
|- core2_64
    |- libtest1-1.0-r0.core2_64.rpm
    |- libtest-dbg-1.0-r0.core2_64.rpm
    |- libtest-dev-1.0-r0.core2_64.rpm
```

* 이제 libtest.so.1.0 파일로 런타임에 사용하는 애플리케이션을 만들어 보자.
  - meta-myproject 디렉토리 아래에 recipes-uselib 디렉토리를 생성한다.

```
recipes-uselib
|- files
|   |- func.h
|   |- libtest.so (생성된 libtest.so.1.0 파일을 복사해 와서 이름을 변경할 것)
|   |- makevoicemain.c
|- uselib.bb (애플리케이션 생성을 위한 레시피)
```

~/poky_src/poky/meta-myproject/recipes-uselib/files/func.h
```cpp
#ifndef __FUNC_H__
#define __FUNC_H__

void makevoicefromdog(void);
void makevoicefromcat(void);

#endif
```

~/poky_src/poky/meta-myproject/recipes-uselib/files/makevoicemain.c
```cpp
#include <stdio.h>
#include "func.h"

int main(void) {
    printf ("Hello makevoicemain! \n");
    makevoicefromdog();
    makevoicefromcat();

    return 0;
}
```

~/poky_src/poky/meta-myproject/recipes-uselib/uselib.bb
```
DESCRIPTION = "This recipes makes execution file which uses libtest.so"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://makevoicemain.c \
          file://libtest.so \
          file://func.h \
          "

do_compile() {
    ${CC} ${LDFLAGS} -I -wl,-rpath=${libdir} -L. makevoicemain.c -ltest -o makevoicemain
}
# rpath: 빌드 결과로 만들어진 애플리케이션이 실행될 때 해당 라이브러리의 위치
# L: 만들려는 애플리케이션을 위해 필요한 라이브러리의 위치
# I: 라이브러리의 이름으로 lib 접두어를 생략한 이름 (libtest.so인 경우 -ltest)

do_install() {
    install -d ${D}${bindir}
    install -m 0755 makevoicemain ${D}${bindir}
}

RDEPENDS_${PN} = "makelib"    # 현재 만들려는 애플리케이션의 실행 시간 의존성이 걸려 있는 패키지를 기술함 (makelib.bb 레시피에서 RPROVIDES 변수에 넣어준 값과 동일해야 하며 패키지의 이름이어야 함)
S = "${WORKDIR}"

FILESEXTRAPATHS_prepend := "${THISDIR}/files:"
FILES_${PN} += "${bindir}/makevoicemain"
```

* 이제 2개의 레시피에 의해 만들어진 패키지들이 루트 파일 시스템에 포함되도록 한다.
  - 이미지 생성 레시피인 great-image.bbappend 레시피 확장 파일에 IMAGE_INSTALL 변수를 통해 추가한다.

~/poky_src/poky/meta-myproject/recipes-core/images/great-image.bbappend
```
IMAGE_INSTALL += "packagegroup-great"
IMAGE_INSTALL += "uselib makelib"    # 추가됨
```

* 새로 만들어진 레시피를 빌드한다.

```
$ bitbake makelib
$ bitbake uselib
$ bitbake great-image -C rootfs
$ runqemu great-image nographic
```

### 개선된 라이브러리 생성 패키지 실습

* 앞에서 "라이브러리 생성을 통한 패키지 실습"을 할 때 라이브러리 파일(libtest.so)과 헤더 파일(func.h)을 수작업으로 복사했었다.
  - Yocto에서는 빌드 시간 의존성 방법을 이용해 수작업으로 라이브러리를 복사할 필요가 없다.
  - RDEPENDS, RPROVIDES 변수는 실행 시간 의존성을 나타낸다.
  - makelib 레시피의 라이브러리가 생성되기 전에 uselib 레시피의 do_compile 태스크를 실행한다고 할 때, libtest.so 파일이 존재하지 않으므로 실패한다.
  - 즉, uselib 레시피 빌드를 위해 makelib 레시피 빌드를 먼저 마쳐야 하므로 DEPENDS, PROVIDES 변수를 사용해야 한다.

* bitbake는 각각의 레시피를 빌드하면서 빌드 작업 디렉토리 아래 recipe-sysroot 디렉토리를 만들고, 컴파일/링크시 필요한 헤더, 라이브러리 파일을 여기서 찾는다.
  - recipe-sysroot: bitbake가 레시피를 빌드하는 중에 do_prepare_recipe_sysroot 태스크에서 만들어진다. 레시피에서 DEPENDS 변수에 의해 지정된 다른 레시피의 결과물을 recipe-sysroot로 복사해온다.

    ```
    <uselib 레시피>                       <makelib 레시피>
    do_fetch                            do_fetch
       |                                   |
       v                                   v
    do_unpack                           do_unpack
       |                                   |
       v                                   v
    do_patch                            do_patch
       |                                   |
       v                                   v
    do_prepare_recipe_sysroot  <--      do_prepare_recipe_sysroot
       |                         |         |
       v                         |         v
    do_configure                 |      do_configure
       |                         |         |
       v                         |         v
    do_compile                   |      do_compile
       |                         |         |
       v                         |         v
    do_install                   |      do_install
                                 |          |
                                 |          v
                                 -----  do_populate_sysroot
    ```

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout improved_makelib`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b improved_makelib`

* 앞의 "라이브러리 생성을 통한 패키지 실습" 소스에서 바뀌는 부분이 2개 있다.
  - SRC_URI에서 기존에 존재했던 libtest.so, func.h 파일을 삭제한다.
  - RDEPENDS를 DEPENDS로 수정해 실행 시간 의존성을 빌드 시간 의존성으로 바꾼다.

~/poky_src/poky/meta-myproject/recipes-uselib/uselib.bb
```
DESCRIPTION = "This recipes makes execution file which uses libtest.so"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://makevoicemain.c \
          "

do_compile() {
    ${CC} ${LDFLAGS} -I -wl,-rpath=${libdir} -L. makevoicemain.c -ltest -o makevoicemain
}
# rpath: 빌드 결과로 만들어진 애플리케이션이 실행될 때 해당 라이브러리의 위치
# L: 만들려는 애플리케이션을 위해 필요한 라이브러리의 위치
# I: 라이브러리의 이름으로 lib 접두어를 생략한 이름 (libtest.so인 경우 -ltest)

do_install() {
    install -d ${D}${bindir}
    install -m 0755 makevoicemain ${D}${bindir}
}

DEPENDS_${PN} = "makelib"
S = "${WORKDIR}"

FILESEXTRAPATHS_prepend := "${THISDIR}/files:"
FILES_${PN} += "${bindir}/makevoicemain"
```

* 이제 makelib.bb 파일을 수정한다.
  - RPROVIDES 변수만 주석 처리해주면 된다.
  - PROVIDES 변수에 makelib 레시피 이름을 할당해야 하지만, 생략하면 레시피 이름을 기본값으로 갖게 되므로 PROVIDES 값이 makelib이 된다.

~/poky_src/poky/meta-myproject/recipes-makelib/makelib.bb
```
DESCRIPTION = "This recipe makes shared library"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"
# 소스 트리 내에서 라이선스 파일을 참조할 수 없으면 오픈임베디드 코어에서 제공하는 라이선스인 ${COMMON_LICENSE_DIR}/MIT를 참조한다.

SRC_URI = "file://dog.c \
        file://cat.c \
        file://func.h \
        "

do_compile() {
    ${CC} -fPIC -c dog.c    # -fPIC 옵션: 위치 독립적인 코드(Position Independent Code)를 뜻함
    ${CC} -fPIC -c cat.c
    ${CC} ${LDFLAGS} -shared -Wl,-soname=libtest.so.1 -o libtest.so.1.0 *.o
      # LDFLAGS(Linker Flags): 링커에 전달되는 플래그로서, 링크 단계에서 사용되는 뒤따라오는 옵션들을 지정하는 변수이다.
      # shared: 컴파일러에게 실행이 가능한 오브젝트에 링크를 걸어줄 수 있는 공유 라이브러리를 만든다고 알려줌
      # Wl: Wl은 링커에게 전해주는 옵션으로 여기서는 -soname=libtest.so.1을 전달한다.
      # soname: API 호환성을 명시적으로 링커에게 알려줌 (libtest.so -> libtest.so.1 -> libtest.so.1.0) = (컴파일 시 사용되는 라이브러리 -> 런타임에 사용되는 라이브러리 -> 실제로 생성된 라이브러리)
      # `$ objectdump -p libtest.so.1.0` 명령을 통해 NEEDED (실행 시간에 여기에 표시된 라이브러리가 로드되어야 한다는 뜻), SONAME 섹션 값을 확인할 수 있다.
}

# RPROVIDES_${PN} = "makelib"    # makelib 레시피에 의해 생성된 라이브러리에 실행 시간 의존성을 가진 패키지가 있다면 여기에 makelib을 넣어야 함

do_install() {
    install -d ${D}${libdir}
    install -m 0755 libtest.so.1.0 ${D}${libdir}
    ln -s libtest.so.1.0 ${D}${libdir}/libtest.so.1
    ln -s libtest.so.1 ${D}${libdir}/libtest.so
    install -d ${D}${includedir}
    install -m 0644 func.h ${D}${includedir}
}

FILES_${PN} = "${libdir}/libtest.so.1.0 ${libdir}/libtest.so.1"    # 런타임에 필요함
FILES_${PN}-dev = "${libdir}/libtest.so ${includedir}/func.h"    # 컴파일에 필요함
S = "${WORKDIR}"
FILESEXTRAPATHS_prepend := "${THISDIR}/files:":
```

* 기존의 makelib.bb 레시피 파일과 uselib.bb 레시피 파일의 빌드 결과물을 삭제하고 다시 빌드한다.
  - 이전 예제에서는 makelib.bb 레시피를 먼저 빌드하고 uselib.bb 레시피를 빌드해야만 했다.
  - 그러나 이번에는 uselib.bb 레시피를 빌드해도 문제가 없다. (makelib.bb 레시피를 미리 빌드하여 recipe-sysroot 디렉토리에 가져온 것을 볼 수 있다)

```
$ bitbake makelib -c cleanall
$ bitbake uselib -c cleanall
$ bitbake uselib
```

* 다시 루트 파일 시스템을 만들고 QEMU를 실행시켜 실행 파일인 amkevoicemain을 실행해본다.

```
$ bitbake great-image -C rootfs
$ runqemu great-image nographic
```

# do_rootfs 태스크

```
                do_fetch
                     |
                     |  ----> ${DL_DIR}에 소스 코드 저장
                     v
                do_unpack
                     |
                     |  ----> ${WORKDIR}/${S}에 압축 해제된 소스 코드 저장 (${S} = ${BPN}-${PV})
                     v
                do_patch
                     |
                     |  ----> ${WORKDIR}/${S}에 패치(patch) 적용 (${S} = ${BPN}-${PV})
                     v
                do_prepare_recipe_sysroot
                     |
                     |  ----> ${WORKDIR}에 특정 sysroots로 파일 설치
                     v
                do_configure
                     |
                     |  ----> ${WORKDIR}/${B}에 소프트웨어 구성 (${B}는 보통 build를 가리킴)
                     v
                do_compile
                     |
                     |  ----> ${WORKDIR}/${B}에 소프트웨어 컴파일
                     v
                do_install
                     |
                     |  ----> ${WORKDIR}/${D}에 소프트웨어 설치 (${D}는 보통 image를 가리킴)
         ------------------------
         |                      |
         |                      |
         v                      v
do_populate_sysroot       do_package
                                |
                                |  ----> ${WORKDIR}/package, ${WORKDIR}/packages-split에 파일 패키지화, ${WORKDIR}/pkgdata에 패키지 메타데이터 저장
                                v
                          do_packagedata
                                |
                                |  ----> ${TMPDIR}/pkgdata/${MACHINE}에 데이터 생성됨
                                v
                          do_package_write_rpm
                                |
                                |  ----> ${DEPLOY_DIR}/rpm (이 디렉토리를 Package Feed라고 함)에 패키지 생성됨
                                v
                          do_package_qa
                                |
                                |
                                v
                          do_rootfs
                                |
                                |
                                v
                          do_image
```

* Package Feed에 배치된 rpm 파일들을 루트 파일 시스템에 설치하는 과정을 배울 것이다.
  - do_rootfs 태스크: 루트 파일 시스템에 rpm 파일(패키지)들을 설치함
  - Pakcage Feed의 위치: build/tmp/deploy/rpm

* do_rootfs 태스크 이후에 루트 파일 시스템을 커스터마이즈하려면 ROOTFS_POSTPROCESS_COMMAND 변수를 이용하면 된다.
  - 빌드 시스템이 루트 파일 시스템을 생성한 후에 처리할 작업들이 있을 때 사용함
  - 이 변수에 셸 함수를 할당하면 do_rootfs가 끝나고 바로 셸 함수를 실행한다.

* do_rootfs 태스크 이후에 do_image 태스크가 실행된다.
  - do_image: 루트 파일 시스템 이미지를 생성함

* do_rootfs, do_image 태스크들은 일반 바이너리를 생성하는 레시피에서 처리하는 과정이 아니라 루트 파일 시스템 이미지를 생성하는 레시피에 의해서만 실행된다.
  - great-image.bb 레시피가 루트 파일 시스템 이미지를 생성하는 레시피이므로 여기서만 do_rootfs, do_image 태스크들이 실행된다.
  - 최종 루트 파일 시스템이 만들어질 파일들과 디렉토리가 모여 있는 곳은 IMAGE_ROOTFS 변수에 지정되어 있다.

* 패키지 관리자 활성화 여부
  - 루트 파일 시스템에 패키지 설치시 패키지 관리자의 활성 여부와 관계없이 패키지 관리자는 수행된다.
  - 만약 패키지 관리자를 활성화하지 않으면 이미지에 패키지 관리자와 관련된 데이터가 포함되지 않지만, 패지키 관리자는 여전히 루트 파일 시스템을 구성하는 데 사용된다. (태스크 수행이 끝날 때 루트 파일 시스템에서 패키지 관리자의 데이터 파일이 삭제됨)
  - IMAGE_FEATURES += "package-management": 패키지 관리자를 활성화함 (이렇게 하면 이미지에 기본적인 패키지 데이터베이스와 런타임 패키지 관리를 위해 필요한 도구들이 포함됨)
  - 패키지 관리자를 이미지에 포함시켜야 새로운 이미지(루트 파일 시스템)를 생성하지 않고 런타임에 타깃 디바이스의 패키지를 업데이트하거나 추가/제거할 수 있다.

* 관련 변수
  - 패키지 설치
    * IMAGE_INSTALL: 패키지 피드 영역에서 설치할 기본 패키지들을 이 변수에 할당한다.
    * PACKAGE_EXCLUDE: 이미지에 설치하면 안 되는 패키지를 이 변수에 지정한다.
    * IMAGE_FEATURES: 이미지에 포함할 기능을 지정한다. 설치될 기능에 따라 설치해야 할 피키지들이 정해진다.
    * PACKAGE_CLASSES: 사용할 패키지들의 종류(rpm, deb, ipk)를 선택한다.
    * IMAGE_LINGUAS: 지원되는 언어 지원 패키지를 선택한다.
    * PACKAGE_INSTALL: 이미지에 설치하려고 패키지 관리자에게 전달된 패키지의 최종 목록이다.
  - 파일 시스템 생성
    * IMAGE_ROOTFS: 루트 파일 시스템의 위치를 가리킨다.
  - 후처리
    * ROOTFS_POSTPROCESS_COMMAND: do_rootfs 태스크가 실행된 후에 실행하는 함수를 지정한다.

* do_rootfs 태스크의 루트 파일 시스템을 만드는 과정은 다음과 같다.
  - createrepo_c 툴은 RPM 패키지들로부터 XML에 기반한 메타데이터를 만들어낸다.
  - 이 툴을 사용해 만들어 낸 결과 루트 파일 시스템을 생성하는 레시피 파일인 great-image.bb의 빌드 작업 디렉토리 내에 oe-rootfs-repo 디렉토리가 생성된다.
  - 패키지를 설치할 루트 파일 시스템을 지정하고, 루트 파일 시스템에 설치할 패키지들을 입력한다.
  - 의존성 패키지를 찾아내고, 원래 설치하려는 패키지들과 함께 설치 목록이 도출된다. 그리고 패키지 설치 과정이 시작된다.
  - 실제 설치된 패키지들에 대해 알아보려면 'tmp/deploy/images/great/xxx.rootfs.manifest' 파일을 참고하거나, `bitbake -g great-image` 명령어를 이용하면 된다. (의존성 패키지 리스트를 보거나 생성함)

## 루트 파일 시스템 커스터마이즈하기

* 루트 파일 시스템이 생성되기 전에 루트 파일 시스템을 커스터마이즈하는 방법이 있다.
  - do_rootfs 태스크가 실행된 후 실행될 함수, 태스크를 만든다고 생각하면 된다.

* ROOTFS_POSTPROCESS_COMMAND 변수는 다음과 같이 세미콜론으로 분리된 셸 함수의 목록을 갖는다.
  - `ROOTFS_POSTPROCESS_COMMAND += "func1;func2;...funcN"`
  - 이 변수는 루트 파일 시스템을 만들어 내는 레시피 파일이나 local.conf 파일에서만 추가될 수 있다. (그렇지 않으면 빌드 오류 발생)

### 실습 순서

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout rootfs_postprocess`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b rootfs_postprocess`

* 루트 파일 시스템 최상위 디렉토리에 yocto.txt 파일과 dummy 디렉토리를 생성한다.
  - yocto.txt 파일에는 ROOTFS_POSTPROCESS_COMMAND 변수의 내용을 넣을 것이다.
  - great-image.bbappend 레시피 확장 파일을 변경해야 한다.

~/poky_src/poky/meta-myproject/recipes-core/images/great-image.bbappend
```
IMAGE_INSTALL += "packagegroup-great"
IMAGE_INSTALL += "uselib makelib"

test_postprocess_func(){
    echo "${ROOTFS_POSTPROCESS_COMMAND}" > ${IMAGE_ROOTFS}/yocto.txt
}

ROOTFS_POSTPROCESS_COMMAND += "test_postprocess_func;"

create_dummy_dir() {
    mkdir ${IMAGE_ROOTFS}/dummy
}

ROOTFS_POSTPROCESS_COMMAND += "create_dummy_dir;"
```

* `$ bitbake great-image -C rootfs` 명령으로 재빌드를 진행한다.
  - 빌드가 완료되면 루트 파일 시스템 최상위 디렉토리에 yocto.txt 파일과 dummy 디렉토리가 생성된 것을 볼 수 있다.

* ROOTFS_POSTPROCESS_COMMAND 변수에 대해 좀 더 상세한 예제가 필요하다면 오픈임베디드 코어에서 사용한 예제를 살펴보도록 한다.
  - poky/meta/classes/rootfs-postcommands.bbclass 클래스 파일을 살펴보면 많은 도움이 될 것이다.

## 설치 후 스크립트

* 설치 후 스크립트: 루트 파일 시스템 이미지 생성 중에 실행되거나 타깃이 처음 부팅되고 난 후에 실행되는 스크립트
  - 주의할 점은 패키지를 생성하는 레시피에서 사용하는 스크립트이지, 루트 파일 시스템을 생성하는 이미지 레시피에서 사용하는 스크립트가 아니라는 것이다.
  - 이 스크립트는 주로 디렉토리 및 임시 파일 생성, 접근 권한 설정 등에 사용된다.
  - 설치 후 스크립트의 예제는 다음과 같다.

* 루트 파일 시스템 생성 중 사용하는 설치 후 스크립트
  ```
  pkg_postinst_${PN} () {
      # 수행할 셸 스크립트
  }
  ```

* 타깃이 처음 부팅되고 난 후에 실행되는 설치 후 스크립트
  ```
  pkg_postinst_ontarget_${PN} () {
      # 수행할 셸 스크립트
  }
  ```

* 참고로 타깃이 처음 부팅되고 난 후 pkg_posting을 실행하는 스크립트는 meta/recipes-devtools/run-postinsts/ 디렉토리에서 확인할 수 있으며 관련 레시피 파일도 존재한다.

* 패키지 관리 시스템은 다음과 같이 패키지가 설치 전후, 삭제 전후로 실행되는 스크립트를 제공한다.
  스크립트 | 설명
  -------- | ----
  pkg_postinst_<package name> | 패키지가 설치되고 난 후 수행되는 스크립트
  pkg_preinst_<package name> | 패키지가 설치되기 전에 수행되는 스크립트
  pkg_prerm_<package name> | 패키지가 삭제되기 전에 수행되는 스크립트
  pkg_postrm_<package name> | 패키지가 삭제되고 난 후 수행되는 스크립트

### 실습 순서

* 관련 소스 다운로드 방법
  - 기존 GitHub에서 받은 소스: `$ git checkout pkg_postinst`
  - 미리 완성된 실습 소스를 받는 방법: `~$ git clone https://GitHub.com/greatYocto/poky_src.git -b pkg_postinst`

* 설치 후 스크립트가 반영된 uselib.bb 파일은 다음과 같다.
  - 루트 파일 시스템에 uselib 패키지를 설치할 때 /usr/bin 디렉토리에 test.txt 파일을 생성한다.
  - 타깃이 처음 부팅할 때도 타깃의 /usr/bin 디렉토리에 test2.txt 파일을 생성한다.
  - if [ "x$D" = "x" ] 표현식은 do_install 태스크 실행이 완료됐을 때 결과물이 생성되는 디렉토리인 $D가 존재하는지 여부를 비교하는 식이다.

~/poky_src/poky/meta-myproject/recipes-uselib/uselib.bb
```
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"
SRC_URI = "file://makevoicemain.c \
         "

do_compile() {
    ${CC} ${LDFLAGS} -I -wl,rpath=${libdir} -L. makevoicemain.c -ltest -o makevoicemain
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 makevoicemain ${D}${bindir}
}

# RDEPENDS_${PN} = "makelib"
DEPENDS = "makelib"

S = "${WORKDIR}"
FILESEXTRAPATHS_prepend := "${THISDIR}/files:"

FILES_${PN} += "${bindir}/makevoicemain"

pkg_postinst_${PN} () {
    if [ "x$D" = "x" ]; then
        printf "It shouldn't be executed"    # 타깃에서 처음 부팅하고 난 후 실행됨
    else
        file=$D${bindir}/test.txt            # 호스트 시스템에서 루트 파일 시스템 생성 시간 동안 실행됨
        printf "This is postinst test.\n" > $file
    fi
}

# 첫 부팅 이후에 타깃에서 실행되는 셸 스크립트 (명시적)
pkg_postinst_ontarget_${PN} () {
    echo "This is postinst test on target" > ${bindir}/test2.txt
}
```

* uselib 레시피를 빌드한다.
  - `$ bitbake uselib -c cleanall && bitbake uselib && bitbake great-image`
  - 빌드 완료 후에 루트 파일 시스템이 만들어져 있는 tmp/work/great-great-linux/great-image/1.0-r0/rootfs/ 디렉토리에 가보면 /usr/bin 디렉토리에 test.txt 파일이 생성되어 있다.

* 생성된 패키지 내에 설치 후 스크립트가 들어가 있는지 확인해 본다.
  - 먼저 2가지 패키지를 ubuntu에 설치해야 한다. `$ sudo apt install rpm2cpio rpm`
  - uselib 레시피 패키지 결과물인 uselib-1.0-r0.core2_64.rpm 파일이 존재하는 ~/poky_src/build2/tmp/work/core2-64-great-linux/uselib/1.0-r0/deploy-rpms/core2_64 디렉토리로 이동한다.
  - .rpm 파일을 extract하여 설치 후 스크립트를 확인한다.
    ```
    ... deploy-rpms/core2_64$ mkdir extract
    ... deploy-rpms/core2_64$ cd extract/
    ... deploy-rpms/core2_64/extract$ rpm -qp --scripts ../usblib-1.0-r0.core2_64.rpm
    output
    ```
  - output 파일을 열어보면 생성된 파일을 확인할 수 있다.

# do_image 태스크

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/921ac41f-b8ac-4506-9521-4f095db8e81c)

```
                do_fetch
                     |
                     |  ----> ${DL_DIR}에 소스 코드 저장
                     v
                do_unpack
                     |
                     |  ----> ${WORKDIR}/${S}에 압축 해제된 소스 코드 저장 (${S} = ${BPN}-${PV})
                     v
                do_patch
                     |
                     |  ----> ${WORKDIR}/${S}에 패치(patch) 적용 (${S} = ${BPN}-${PV})
                     v
                do_prepare_recipe_sysroot
                     |
                     |  ----> ${WORKDIR}에 특정 sysroots로 파일 설치
                     v
                do_configure
                     |
                     |  ----> ${WORKDIR}/${B}에 소프트웨어 구성 (${B}는 보통 build를 가리킴)
                     v
                do_compile
                     |
                     |  ----> ${WORKDIR}/${B}에 소프트웨어 컴파일
                     v
                do_install
                     |
                     |  ----> ${WORKDIR}/${D}에 소프트웨어 설치 (${D}는 보통 image를 가리킴)
         ------------------------
         |                      |
         |                      |
         v                      v
do_populate_sysroot       do_package
                                |
                                |  ----> ${WORKDIR}/package, ${WORKDIR}/packages-split에 파일 패키지화, ${WORKDIR}/pkgdata에 패키지 메타데이터 저장
                                v
                          do_packagedata
                                |
                                |  ----> ${TMPDIR}/pkgdata/${MACHINE}에 데이터 생성됨
                                v
                          do_package_write_rpm
                                |
                                |  ----> ${DEPLOY_DIR}/rpm (이 디렉토리를 Package Feed라고 함)에 패키지 생성됨
                                v
                          do_package_qa
                                |
                                |
                                v
                          do_rootfs
                                |
                                |
                                v
                          do_image
```

* do_image 태스크: 이미지 생성을 시작하는 태스크
  - do_image 태스크는 오픈임베디드 빌드 시스템이 do_rootfs 태스크 실행이 완료되고 실행된다.
  - do_image 태스크가 실행되는 동안 이미지에 설치되는 패키지들이 식별되며, 최종적으로 루트 파일 시스템이 생성된다.

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/95d4804f-6ed5-4bef-a1ec-57ac38aace32)

* 루트 파일 시스템: 계층적 파일 트리의 최상단에 있으며 여기에는 시스템 부팅을 위한 장치 디렉토리 및 프로그램을 포함해 시스템 동작에 중요한 파일/디렉토리가 포함된다.
  - 오픈임베디드 코어는 image_type.bbclass 클래스를 제공해 다양한 파일 시스템의 루트 파일 시스템을 생성한다.
  - 파일 시스템의 종류는 IMAGE_FSTYPES 변수를 통해 설정할 수 있다. (tar.bz2, ext4 등)

# 공유 상태 캐시와 시그니처

## 공유 상태 캐시

* 공유 상태 캐시(Shared State Cache, sstate 캐시라고도 함)
  - bitbake는 레피시를 성공적으로 빌드하면 출력 결과를 공유 상태 캐시에 저장함
  - 변경되지 않은 레시피를 재빌드할 경우 이전에 저장한 사전 빌드된 오브젝트를 사용한다.
  - checksum(시그니처)들은 불필요하게 재빌드되는 것을 최소화하기 위해 각각의 태스크에 대해 계산한다.
  - 태스크의 해시값이 변경되면 태스크를 재실행해야 한다.
  - checksum(시그니처) 계산시 환경 설정 파일(local.conf, sitro.conf 등)과 레시피 파일들(.bb, .bbappend) 그리고 의존성을 갖고 있는 레시피 파일들과 함수들, SRC_URI에 추가된 파일들을 고려한다.
  - checksum(시그니처) 계산값이 동일하다면 빌드 결과물들은 sstate-cache에 그대로 복사해서 이용한다.

* setscene 태스크: 공유 상태 캐시를 통해 현재 실행해야 하는 태스크가 건너뙬 수 있는지 결정한다.
  - 건너뛸 수 있으면 해당 태스크의 사전 빌드된 오브젝트 결과물을 가져와 사용한다.
  - setscene 태스크가 정의된 태스크는 do_install 태스크 이후 바이너리나 패키지 결과물을 생성하는 특정 태스크들에서만 존재한다.
  - 다음은 do_package 태스크에서 setscene 태스크를 추가하는 예시이다.
    ```
    ...
    SSTATETASKS += "do_package"
    do_package[cleandirs] = "${PKGDEST} ${PKGDESTWORK}"
    do_package[sstate-plaindirs] = "${PKGD} ${PKGDEST} ${PKGDESTWORK}"
    do_package_setscene[dirs] = "${STAGING_DIR}"
    python do_package_setscene () {
        sstate_setscene(d)
    }
    addtask do_package_setscene
    ...
    ```

* bitbake가 setscene 태스크를 실행하는 과정은 다음과 같다.
  - bitbake는 빌드를 진행하기 전에 공유 상태 캐시를 확인한다. (BB_HASHCHECK_FUNCTION 변수가 지정한 함수(sstate_checkhashes)를 사용함. 이 함수는 재사용 가능한 빌드된 오브젝트들의 리스트를 리턴한다. 태스크의 해시 값을 확인함.)
  - bitbake는 앞에서 얻은 재사용 가능한 빌드된 오브젝트를 가진 태스크들의 setscene 태스크를 실행한다.
  - 이후 재상요이 가능한 빌드된 오브젝트를 갖지 못한 태스크들이 순차적으로 실행된다.

* setscene 태스크 종류

태스크 / setscene 태스크 | 설명
------------------------ | ---
do_packagedata_setscene | 최종 패키지 생성을 위해 빌드 시스템에 의해 사용되는 패키지 메타데이터를 생성함
do_package_setscene | do_install 태스크에 의해 생성된 파일들을 이용할 수 있는 패키지들과 파일들에 근거해 나눈다.
do_package_write_rpm_setscene | RPM 패키지를 생성하고 패키지 피드에 패키지들을 배치시킴
do_populate_lic_setscene | 이미지가 생성될 때 모아 놓은 레시피를 위한 라이선스 정보를 생성한다.
do_populate_sysroot_setscene | 다른 레시피들에 의해 이용될 수 있도록 do_install 태스크에 의해 설치된 파일들을 sysroot로 복사함
do_package_qa_setscene | 패키지로 만들어진 파일들에 대해 QA 검증이 실시됨
do_image_qa_setscene | 결과 이미지의 유효성을 검사함
do_image_complete_setscene | 이미지 생성에 연관돼 최종적으로 실행되는 태스크

* 특정 레시피의 태스크가 공유 상태 캐시에 존재하는지 여부를 알려면 다음 커맨드를 입력한다.
  - `$ oe-check-sstate <image recipe name> | grep <recipe name>`
  - 다음과 같이 특정 레시피의 공유 상태 캐시 존재를 확인할 수 있다.
    ```
    $ oe-check-sstate great-image | grep uselib
    uselib:do_package_qa
    uselib:do_package_write_rpm
    uselib:do_populate_lic
    uselib:do_populate_sysroot
    uselib:do_packagedata
    ```

## 시그니처

* bitbake는 태스크 실행이 필요한지 결정할 때 setscene 태스크와 함께 시그니처를 사용한다.
  - 시그니처 = checksum
  - 시그니처는 STAMP_DIR 변수가 지정하는 디렉토리에 저장된다.
  - bitbake는 각각의 태스크가 수행 완료되면 스탬프 파일을 만든다.
  - 스탬프 파일은 일부 태스크의 실행이 완료됐다는 표시만 하며, 태스크의 출력을 기록하지는 않는다.
  - 가령 do_compile 태스크가 실행 완료되면, 스탬프 저장 디렉토리에 "<package version>_do_compile_<signature>" 스탬프 파일이 생성된다.
  - 계산된 시그니처 값과 동일한 시그니처 값이 붙은 파일이 스탬프 디렉토리에 존재한다면 do_compile 태스크를 건너뛰게 된다.

* 시그니처에 대한 예제는 건너뛰도록 한다.

* `$ bitbake -f -c <task name> <recipe name>` 명령어에서 -f 옵션은 강제로 지정한 태스크를 실행하는 역할을 한다.
  - 정확히는 지정한 태스크의 스탬프 파일을 삭제하는 역할을 한다.
  - 그러면 bitbake는 해당 태스클르 빌드한 적이 없다고 생각하고 다시 태스크를 실행하는 것이다.

* 다음은 유용한 bitbake 옵션들을 일부 보여주고 있다.

bitbake 옵션 | 설명
------------ | ----------------
-c <task> | 주어진 태스크를 실행한다.
-s | 내부적으로 사용할 수 있는 모든 레시피들의 리스트들과 레시피들의 버전을 출력한다.
-f | 주어진 태스크의 스탬프 파일을 삭제함으로써 강제로 주어진 태스크를 실행한다.
world | 단순하게 모든 레시피들에 속한 모든 태스크들을 실행한다.
-b <recipe file name> | 주어진 레시피 파일을 실행한다. 단, 의존성에 대한 고려를 하지 않는다.

## 이미 생성된 공유 상태 캐시 최적화

* 프로젝트를 계속 빌드하면서 공유 상태 캐시의 크기는 점점 커진다.
  - 시간이 오래 지나면 중복된 데이터를 공유 상태 캐시에서 삭제해 주어야 한다.
  - sstate-cache-management.sh 명령을 사용하면 중복된 데이터가 있을 때 이전의 캐시를 삭제하여 불필요한 데이터를 정리할 수 있다.
    `$ poky/scripts/sstate-cache-management.sh --remove-duplicated -d --cachedir=<SSTATE_IDR>`

* 만약 build2 디렉토리 내에 sstate-cache 디렉토리를 정리할 경우 다음과 같이 하면 된다.
  - 먼저 sstate-cache 디렉토리의 크기를 확인한다. (`~/poky_src/build2$ du -s sstate-cache/`)
  - 공유 상태 캐시 최적화 스크립트를 실행한다. (`~/poky_src/poky/scripts$ ./sstate-cache-management.sh --remove-duplicated -d --cache-dir=/home/poky_src/build2/sstate-cache`)

* 한편, 빌드 결과물을 삭제하는 clean, cleansstate, cleanall 태스크의 용도는 다음과 같다.
  - do_clean 태스크: do_unpack, do_configure, do_compile, do_install, do_package로부터 생성된 모든 출력 파일들을 삭제한다. 또한 스탬프 파일도 함께 삭제한다. 그러나 다운로드 받은 파일과 공유 상태 캐시는 삭제되지 않는다. (`$ bitbake -c clean <recipe>`)
  - do_cleanall 태스크: 공유 상태 캐시와 스탬프 파일, 모든 생성된 출력 파일들 그리고 다운로드 받은 파일도 함께 삭제된다. (`$ bitbake -c cleanall <recipe>`)
  - do_cleansstate 태스크: 모든 생성된 출력 파일들, 공유 상태 캐시, 스탬프 파일이 삭제된다. (`$ bitbake -c cleansstate <recipe>`)

# kirkstone

## kirkstone의 특징

* 대략 93개의 보안 관련 패치(Common Vulnerability and Exposures)가 반영됨
* CVE 패치가 반영된 패키지들은 다음과 같다: binutils, curl, epiphany, expat, ffmpeg, gcc, glibc, gmp, go, grub2, gzip, libarchive, libxml2, libxslt, lighttpd, linux-yocto, amdgpu, lua, openssl, qemu, rpm, seatd, speex, squashfs-tools, systemd, tiff, unzip, vim, virglrenderer, webkitgtk, xz, zlib
* 대략 318개의 오픈 소스 패키지 버전이 업그레이드됨
* 기본적으로 패키지 간 의존성의 숫자를 줄였기 때문에 빌드에 걸리는 시간이 감소함 (Rust 컴파일러가 적용됨)
* gzip 대신 zstd로 압축 해제 표준을 바꿔 공유 상태 성능을 향상시킴 (zstd은 gzip과 압축률이 비슷하지만 압축 해제 속도가 더 빠름)
* 기존 대비 훨씬 더 정확한 라이선스 준수를 통해 라이선스 관리 툴의 개선을 가져옴
* kirkstone은 최소 4.x 버전 이상의 커널을 지원함
* linux_kernel_header는 더 이상 의무사항이 아님 (linux_kernel_header: 리눅스 커널 개발과 관련된 작업을 수행하는 데 필요한 헤더 파일들을 제공하는 패키지)
* __append, prepend, remove 연산자는 이제 '=' 또는 ':=' 연산자와 결합해서만 사용할 수 있음__
* __BB_ENV_EXTRAWHITE 변수는 셸 환경 변수를 bitbake 전역 환경 변수로 만들 수 있는 방법을 제공함. 이 변수는 kirkstone 버전에서 BB_ENV_PASSTHROUGH_ADDITIONS로 바뀌었음.__
* __append, prepend, remove 연산자 사용에 있어서 '변수_<append, prepend, remove>' 형식이 '변수:<append, prepend, remove>' 형식으로 바뀌었음__
* __변수와 마찬가지로 조건부 변수에서 '_'가 ':'로 바뀌었음__

## kirkstone 설치

* ubuntu에서 패키지 설치하기 (ubuntu 18.04 환경 기준)
  - 의존성 패키지 설치에 대한 정보는 [여기](https://docs.yoctoproject.org/4.0.9/brief-yoctoprojectqs/index.html)를 참조하십시오.

```
$ sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio

$ sudo apt install python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2

$ sudo apt install libegl1-mesa libsdl1.2-dev pylint3 xterm python3-subunit mesa-common-dev zstd liblz4-tool
```

* kirkstone 소스 가져오기

```
$ git clone https://GitHub.com/yoctoproject/poky.git .
$ git checkout kirkstone
```

* 테스트를 위해 Poky에서 제공하는 이미지 레시피 core-image-minimal.bb를 빌드해본다.

```
$ source poky/oe-init-build-env
$ bitbake core-image-minimal
```

## dunfell 버전을 kirkstone으로 마이그레이션하기

* 오픈임베디드 빌드 시스템은 dunfell에서 kirkstone으로 마이그레이션 할 때 문법, 라이선스 사용법과 같은 바뀐 부분들을 알아서 수정해주는 스크립트를 제공한다. (~/kirkstone/poky/scripts/contrib 디렉토리 참조)
  - convert-srcuri.py 스크립트
    * kirkstone에서는 SRC_URI에 git을 사용할 때 브랜치를 꼭 기입해야 한다. (master 브랜치라도 예외없음)
    * 만약 현재 해시 값이 tag를 사용하기 때문에 브랜치를 기입할 수 없으면 'nobranch=1' 옵션을 설정해야 한다.
    * GitHub에서 git protocol은 안 되고 http protocol만 허용함
    * `convert-srcuri.py <recipe name>`
  - convert-overrides.py 스크립트
    * kirkstone에서는 append, prepend, remove 변수와 조건부 변수를 사용할 때 '_'를 ':'로 바꿔야 하는데 이 스크립트가 해당 문법을 자동으로 수정해준다.
    * `convert-overrides.py <recipe name>`
  - convert-spdx-licenses.py 스크립트
    * kirkstone에서는 레시피에 기술된 라이선스 이름이 기존보다 엄격한 기준을 적용해 검증된다.
    * `convert-spdx-licenses.py <recipe name>`
    * 다음은 변경된 라이선스 이름이다.

dunfell 라이선스 이름 | kirkstone 라이선스 이름
--------------------- | -------------------------
BSD | BSD-3-Clause
GPLv3 | GPL-3.0-only
GPLv3+ | GPL-3.0-or-later
GPLv2 | GPL-2.0-only
GPL-2.0 | GPL-2.0-only
GPLv2+ | GPL-2.0-or-later

* image-mklibs.bbclass 클래스 파일과 image-prelink.bbclass 클래스 파일이 삭제되었으므로 local.conf 파일을 수정해야 할 수도 있다.
  - template 디렉토리에 존재하는 local.conf.sample 파일을 수정해야 함

* BB_DISKMON_DIRS 변수에서 HALT 값이 ABORT 값으로 바뀌었음
  - template 디렉토리에 존재하는 local.conf.sample 파일을 수정해야 함

* 커널 레시피 파일을 수정할 것
  - SRCREV 변수에 commit id에 해당하는 해시(revision)값을 정해줄 것
  - COMPATIBLE_MACHINE 변수에 할당하는 형식이 바뀜

* 커널 코드를 수정할 것
  - kirkstone으로 업데이트되면서 gcc 버전이 11 버전으로 업데이트되었다. ('-fno-common' 옵션을 기본값으로 사용하게 됨)
  - 따라서 컴파일러가 초기화되지 않은 전역 변수를 객체 파일의 BSS 섹션에 배치하도록 지정된다. (동일한 변수가 둘 이상의 컴파일 단위에 정의된 경우 다중 정의 오류가 발생함)
  - _force_order 변수가 pgtable_64.c와 kaslr_64.c에 중복으로 정의되어 있으므로 한쪽 파일에서 _force_order 변수를 extern 변수로 수정해야 함

* machine 환경 설정 파일을 수정할 것
  - QEMU 머신을 사용할 경우 machine의 인클루드 파일 경로를 변경해야 함 (`conf/machine/include/qemuboot-x86.inc` -> `conf/machine/include/x86/qemuboot-x86.inc`로 변경)

* reproducible_build.bbclass 클래스 파일이 삭제됨
  - 배포 환경 설정 파일에서 이 클래스를 상속 받는 부분을 주석 처리할 것
  - kirkstone에서 이 클래스 파일이 base.bbclass 클래스 파일로 합쳐졌으므로 더 이상 reproducible_build.bbclass 클래스 파일이 존재하지 않음

* Yocto 리눅스 레시피 확장 파일인 linux-yocto_5.4.bbappend 파일을 삭제할 것
  - kirkstone으로 업데이트되면서 지원되는 Yocto 리눅스는 5.10, 5.15이므로 5.4에 대한 레시피 파일이 존재하지 않음

* EXTRA_USERS_PARAMS 변수에서 'useradd -p `openssl passwd 9876` great;` 부분을 삭제해야 함
  - 보안상의 문제로 이 방법은 더 이상 사용할 수 없음
  - 패스워드를 외부 툴(mkpasswd)을 이용해 생성한 후 생성된 해시값을 다음과 같이 사용자 계정에 넣어줄 것
  - `mkpasswd -m sha-512 <패스워드> -s "시드값: 8~16글자 문자열"`

~/kirkstone/poky/meta-great/recipes-core/image/great-image.bb
```
SUMMARY = "A very small image for yocto test"

inherit great-base-image

LINGUAS_KO_KR = "ko-kr"
LINGUAS_EN_US = "en-us"

IMAGE_LINGUAS = "${LINGUAS_KO_KR} ${LINGUAS_EN_US}"

# IMAGE_INSTALL += "packagegroup-great"
# IMAGE_INSTALL += "mykernelmod"

IMAGE_OVERHEAD_FACTOR = "1.3"

inherit extrausers

EXTRA_USERS_PARAMS = " \
    usermod -p '\$6\$12345678\$zYTErUfpqzN5dmxG3gGKbhqFHaN9SxsePAd7oKlc5M1Qf.c.vhkThAe0Wyx8jRf37/HtlKAJvFfluXwQn/FWi1' root;\
    useradd -p '' great \
"
```

`$ mkpasswd -m sha-512 greatyocto -s "12345678"`을 실행하면 다음과 같은 결과가 나온다.
  - `$6$12345678$zYTErUfpqzN5dmxG3gGKbhqFHaN9SxsePAd7oKlc5M1Qf.c.vhkThAe0Wyx8jRf37/HtlKAJvFfluXwQn/FWi1`

* 기존에 생성된 build/conf 디렉토리를 삭제하고 환경 설정 초기화 스크립트인 oe-init-build-env 대신 buildenv.sh을 실행하고 빌드를 진행한다.

```
~/kirkstone$ rm -rf build/conf
~/kirkstone$ source buildenv.sh
~/kirkstone/build$ bitbake great-image
~/kirkstone/build$ runqemu nographic
```

# SDK(Software Development Kit)

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/84a9f0ad-ca32-4f5c-93bf-33457a5e4028)

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/de05fe93-de73-445e-8756-c3f7fb9f8fdd)

![image](https://github.com/Soonbum/What_is_Yocto_Project/assets/16474083/6e02b6de-0bb8-48f1-a623-a32c36437842)

* Yocto의 주요 목적은 리눅스 배포판을 생성하는 것이다.
  - 그러나 Yocto를 이용해 새롭게 패키지를 개발하고자 한다면 따로 개발 환경을 구축해야 한다.
  - 개발 환경 구축을 위한 방법으로 다음 2가지를 제공한다: SDK, meta-toolchain
  - 툴체인 구축: 컴파일러, 링커, 디버거, 외부 라이브러리 헤더 등을 준비하는 것
  - 개발 환경을 구축하는 것은 툴체일 구축한다는 것과 동일한 뜻이다.
  - 오픈임베디드 빌드 시스템은 SDK 패키지를 제공하는데 이것은 개발, 디버깅을 하기 위한 툴체인을 포함한다.

* bitbake는 do_populate_sdk, do_populate_sdk_ext 태스크를 실행해 표준 및 확장이 가능한 SDK를 위한 SDK 설치 스크립트를 만든다.
  - do_populate_sdk 태스크: 표준 SDK 생성을 돕는다. 타깃(타깃 하드웨어, 라이브러리, 헤더 포함)/호스트(SDK가 설치된 머신이며 SDKMACHINE이라고도 함) 부분 모두를 처리함.
  - do_populate_sdk_ext 태스크: 확장이 가능한 SDK 생성을 돕는다. 표준 SDK와 다른 방식으로 타깃/호스트 부분을 처리한다. 호스트/타깃을 포함하는 빌드 시스템을 캡슐화한다. devtool을 포함한다.
  - 쉽게 말해 이 태스크들은 개발자가 외부에서 개발할 수 있도록 개발 환경을 제공하는 SDK를 쉽게 구성하도록 도와주는 태스크이다.
  - SDK를 구축해주는 실행 스크립트를 최종적으로 만들어낸다.
  - 실행 스크립트 위치: /build/tmp/deploy/sdk/*.sh
  - 이 스크립트를 실행하면 크로스 컴파일 환경이 구축된 디렉토리가 만들어진다. (나중에 변경 가능)

* 리눅스 배포판 생성과 달리 do_rootfs, do_image 태스크 대신 do_populate_sdk 태스크를 실행한다. (패키지 개발을 위한 태스크 절차)

```
                do_fetch
                     |
                     |  ----> ${DL_DIR}에 소스 코드 저장
                     v
                do_unpack
                     |
                     |  ----> ${WORKDIR}/${S}에 압축 해제된 소스 코드 저장 (${S} = ${BPN}-${PV})
                     v
                do_patch
                     |
                     |  ----> ${WORKDIR}/${S}에 패치(patch) 적용 (${S} = ${BPN}-${PV})
                     v
                do_prepare_recipe_sysroot
                     |
                     |  ----> ${WORKDIR}에 특정 sysroots로 파일 설치
                     v
                do_configure
                     |
                     |  ----> ${WORKDIR}/${B}에 소프트웨어 구성 (${B}는 보통 build를 가리킴)
                     v
                do_compile
                     |
                     |  ----> ${WORKDIR}/${B}에 소프트웨어 컴파일
                     v
                do_install
                     |
                     |  ----> ${WORKDIR}/${D}에 소프트웨어 설치 (${D}는 보통 image를 가리킴)
         ------------------------
         |                      |
         |                      |
         v                      v
do_populate_sysroot       do_package
                                |
                                |  ----> ${WORKDIR}/package, ${WORKDIR}/packages-split에 파일 패키지화, ${WORKDIR}/pkgdata에 패키지 메타데이터 저장
                                v
                          do_packagedata
                                |
                                |  ----> ${TMPDIR}/pkgdata/${MACHINE}에 데이터 생성됨
                                v
                          do_package_write_rpm
                                |
                                |  ----> ${DEPLOY_DIR}/rpm (이 디렉토리를 Package Feed라고 함)에 패키지 생성됨
                                v
                          do_package_qa
                                |
                                |
                                v
                          do_populate_sdk
```

... (내용 생략)

# oe-pkgdata-util 툴

* oe-pkgdata-util 툴은 다양한 패키지 관련 정보를 표시한다.
  - 단, 이미 빌드된 패키지에 대한 정보만 볼 수 있다.
  - PKGDATA_DIR 디렉토리에 저장된 정보를 바탕으로 정보를 출력한다. (tmp/pkgdata 디렉토리 아래 있음)
 
* `oe-pkgdata-util find-path <query>`: query로 주어진 경로를 제공하는 모든 패키지들을 출력한다.
  - `oe-pkgdata-util find-path */libtest.so*`: libtest.so 라이브러리 및 심볼릭 링크의 경로를 찾아줌

* `oe-pkgdata-util list-pkg-files -p <recipe name>`: 주어진 레시피에 대해 레시피가 무슨 파일들을 생성해 내는지 출력한다.
  - `oe-pkgdata-util list-pkg-files -p makelib`: makelib 레시피가 만들어내는 파일들을 모두 보여줌
 
* `oe-pkgdata-util list-pkgs -p <recipe name>`: 주어진 레시피가 제공하는 각각의 패키지들을 보여준다.
  - `oe-pkgdata-util list-pkgs -p makelib`: makelib 레시피가 만들어내는 패키지들을 모두 보여줌
 
* `oe-pkgdata-util package-info <package name>`: 주어진 패키지의 정보를 출력한다.

* `oe-pkgdata-util lookup-recipe <package name>`: 주어진 패키지를 만들어 내는 레시피 파일을 찾아낸다.

# PACKAGECONFIG 변수

* PACKAGECONFIG 변수는 특정 레시피에서 지원하는 기능에 대해 활성화/비활성화하는 데 사용하며, 의존성을 설정하는 데에도 사용한다.

* 컴파일러에게 옵션 추가 전달
  - EXTRA_OECONF 변수: 해당 레시피를 빌드할 때 컴파일러에게 추가로 전달하는 옵션으로 Yocto 프로젝트의 Autotools 클래스를 상속 받은 경우에 사용할 수 있다.
  - EXTRA_OECMAKE: CMake를 사용할 경우 EXTRA_OECONF 변수 대신 사용할 수 있다.
  - EXTRA_OEMESON: Meson을 사용할 경우 EXTRA_OECONF 변수 대신 사용할 수 있다.

* PACKAGECONFIG 사용법
  - 1번째 인자: 기능이 활성화되어 있다면 1번째 인자는 EXTRA_OECONF 변수에 추가되어 추가적인 환경 설정 스크립트 옵션에 반영된다.
  - 2번째 인자: 기능이 비활성화되어 있다면 2번째 인자는 EXTRA_OECONF 변수에 추가되어 추가적인 환경 설정 스크립트 옵션에 반영된다.
  - 3번째 인자: 기능이 활성화되어 있다면 3번째 인자는 추가적인 빌드 의존성을 설정할 수 있다. DEPENDS 변수에 빌드 의존성이 추가된다.
  - 4번째 인자: 기능이 활성화되어 있다면 4번째 인자는 추가적인 실행 시간 의존성을 설정할 수 있다. RDEPENDS 변수에 실행 시간 의존성이 추가된다.
  - 5번째 인자: 기능이 활성화되어 있다면 5번째 인자는 약한 실행 시간 의존성을 설정할 수 있다. RRECOMMENDS 변수에 약한 실행 시간 의존성이 추가된다.
  - 6번째 인자: 설정된 기능에 대해 충돌이 일어날 수 있는 PACKAGECONFIG 설정을 기능한다.
  - 아래는 wifi, wayland 기능을 정의하고 있다. 인자는 , 기호로 구분된다.

  ```
  PACKAGECONFIG ??= "wifi wayland"
  PACKAGECONFIG[wifi] = "--enable-wifi, --disable-wifi, wpa-supplicant, wpa-supplicant"    # wpa-supplicant에 빌드 의존성, 실행 시간 의존성을 갖고 있음 (와이파이 기반 무선 랜 환경을 구축할 때 사용하는 프로그램)
  PACKAGECONFIG[wayland] = "-Dbackend-wayland=true,-Dbackend-wayland=false,virtual/egl,virtual/libgles2"    # 다중 PROVIDES 중 PREFERRED_PROVIDER 변수를 통해 정해진 레시피를 먼저 빌드함
  ```

* 다음은 레시피 확장 파일과 환겅 설정 파일에서 PACKAGECONFIG 변수를 설정하는 방법이다.
  - 레시피 확장 파일(.bbappend): `PACKAGECONFIG:append = " <feature>"
  - 환경 설정 파일(.conf): "PACKAGECONFIG:append:pn-<recipe file name> = " <feature>"
 
* 각각의 레시피에서 사용할 수 있는 PACKAGECONFIG 변수 플래그는 list-packageconfig-flag.py 스크립트로 확인할 수 있다. (-a 옵션을 넣으면 상세한 내용을 볼 수 있음)
  ```
  ~/kirkstone/poky/scripts/contrib$ ./list-packageconfig-flags.py

  RECIPE NAME                PACKAGECONFIG FLAGS
  ==================================================
  alsa-utils                bat manpages udev
  alsa-utils-scripts        bat manpages udev
  apr                       ipv6 timed-tests xsi-strerror
  apr-native                ipv6 timed-tests xsi-strerror
  apr-util                  crypto gdbm ldap sqlite3
  apr-util-native           crypto gdbm ldap sqlite3
  aspell                    curses
  at                        selinux
  binutils-native           debuginfod
  ...
  ```

# 소스 코드 배포

* 오픈임베디드 코어에서는 프로젝트에서 사용하는 레시피의 소스 코드나 메타데이털르 묶어 배포할 수 있는 기능을 제공한다.
  - archiver.bbclass 클래스를 사용하면 이미지 생성의 제일 마지막에 tarball 형태의 압축된 소스와 각각의 패치들이 tmp/deploy/sources 디렉토리에 생성된다.
  - COPYLEFT_LICENSE_EXCLUDE 변수: 라이선스를 가진 레시피를 배제해 소스 코드 배포가 생성되지 않도록 할 수 있음
  - 환경 설정 파일 local.conf 파일에서 INHERIT += "archiver"를 추가하고 ARCHIVER_MODE 변수를 설정해야 한다.
  - 단, externalsrc 클래스를 사용한 레시피의 경우 따로 tmp/deploy/sources 디렉토리에 소스가 생성되지 않는다. (fetch 태스크가 생략되므로)

ARCHIVER_MODE 변수 플래그 | 설명
----- | -----
ARCHIVER_MODE[src] = "original" | 원본 소스 파일들을 사용한다. (단, 패치가 적용되지 않은 소스)
ARCHIVER_MODE[src] = "patched" | 패치가 적용된 소스 파일을 사용한다. (기본값)
ARCHIVER_MODE[src] = "configured" | 설정된 소스 파일들을 사용한다.
ARCHIVER_MODE[diff] = "1" | do_unpack 태스크와 do_patch 태스크 사이의 패치들을 사용한다.
ARCHIVER_MODE[diff-exclude] ?= "file file ..." | diff로부터 제외하기를 원하는 파일들과 디렉토리들을 기술함
ARCHIVER_MODE[dumpdata] = "1" | 환경 설정 데이터를 사용한다.
ARCHIVER_MODE[recipe] = "1" | 레시피 파일과 인클루드 파일을 사용한다.
ARCHIVER_MODE[srpm] = "1" | RPM 패키지 파일을 사용한다.

# 이미 만들어져 있는 레이어 포팅하기

* 새로 레이어를 개발하기보다는 다른 사람이 개발한 레이어가 있다면 이를 사용하는 것이 더 현명한 방법이다.
  - 필요한 부분만 레시피 확장 파일(.bbappend)로 만들어 수정/추가해 주면 된다.
  - 예를 들어 현재 구축한 시스템에 security 기능, 특히 selinux 기능을 추가한다고 가정하자.
  - https://layers.openembedded.org 사이트에서 meta-selinux를 검색하고 검색된 리포지터리에서 소스를 받아오자. (`git clone git://git.yoctoproject.org/meta-selinux`)
  - 다운로드 받은 meta-selinux 레이어의 브랜치를 현재 Poky 버전과 동일하게 맞춘다. (`git checkout kirkstone`)
  - 새로 다운로드 받은 레이어 아래의 README 파일에서 의존성 관련 정보를 볼 수 있다. 그 외에도 변수 설정 방법이 나와 있으니 그것들을 참조하면 된다.

# devtool

* devtool 명령줄 도구는 소프트웨어 빌드, 테스트, 패키징하는 데 도움이 되는 여러 기능을 제공한다.
  - 앞에서 했던 모든 수동 과정을 자동화해주기 때문에 작업이 용이하다.
  - devtool은 workspace라는 기존의 작업 영역과 분리된 레이어를 사용한다.
  - 기존의 build/tmp/work 디렉토리에서 소스를 수정하지 않아도 되며, devtool 역시 externalsrc를 사용하므로 작업하던 소스가 삭제될 염려가 없다.
  - `$ devtool create-workspace [레이어 절대 경로]` (만약 경로 이름을 생략하면 현재 위치에 workspace를 생성한다)
    * 생성된 workspace 레이어는 conf/layer.conf 파일이 포함되어 있다.
    * bblayers.conf 파일에 생성된 workspace 레이어의 경로가 자동으로 추가된다.
    * build/conf 디렉토리에 devtool.conf 파일이 생성된다. (작업 공간인 workspace 레이어의 절대 경로가 기록됨)

### 실습 순서

* kirkstone 소스 받기
  ```
  $ git clone https://GitHub.com/yoctoproject/poky.git .
  $ git checkout kirkstone
  ```

* devtool을 통해 생성된 바이너리를 QEMU에 전송하려면 이미지에 ssh를 설치해야 한다.

~/kirkstone/poky/meta-poky/conf/local.conf.sample
```
...
EXTRA_IMAGE_FEATURES += "ssh-server-openssh"    # ssh를 설치해야 함
```

* 빌드 환경 초기화 스크립트를 실행하고 빌드를 진행한다.

```
~/kirkstone$ source poky/oe-init-build-env

### Shell environment set up for builds. ###
You can now run 'bitbake <target>'
Common targets are:
    core-image-minimal
    core-image-full-cmdline
    core-image-sato
    core-image-weston
    meta-toolchain
    meta-ide-support

You can also run generated qemu images with a command like 'runqemu qemux86'
Other commonly useful commands are:
 - 'devtool' and 'recipetool' handle common recipe tasks
 - 'bitbake-layers' handles common layer tasks
 - 'oe-pkgdata-util' handles common target package tasks

~/kirkstone/build$ bitbake core-image-minimal
```

* build 디렉토리에서 `$ bitbake-layers create-layer` 명령어를 이용해 새로운 레이어 meta-greatyocto를 생성한다.

```
~/kirkstone/build$ bitbake-layers create-layer ../poky/meta-greatyocto

NOTE: Starting bitbake server...
Add your new layer with 'bitbake-layers add-layer poky/meta-greatyocto'
```

* 다음과 같이 poky 디렉토리에 meta-greatyocto 레이어가 생성된다.

```
poky/meta-greatyocto/
|- conf
|   |- layer.conf
|- COPYING.MIT
|- README
|- recipes-example
    |- example
        |- example_0.1.bb
```

* bitbake가 새로 생성된 레이어를 인식할 수 있돌고 `$ bitbake-layers add-layer` 명령어로 bblayers.conf 파일에 새로 생성된 레이어를 추가한다.

```
~/kirkstone/build$ bitbake-layers add-layer ../poky/meta-greatyocto/

NOTE: Starting bitbake server...
```

* 다음과 같이 bblayers.conf 파일에 새로 생성된 meta-greatyocto 레이어가 추가된 것을 볼 수 있다.

```
~/kirkstone/build$ cat conf/bblayers.conf

# POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
# changes incompatibly

POKY_BBLAYERS_CONF_VERSION = "2"

BBPATH = "${TOPDIR}"
BBFILES ?= ""

BBLAYERS ?= " \
  /home/user/kirkstone/poky/meta \
  /home/user/kirkstone/poky/meta-poky \
  /home/user/kirkstone/poky/meta-yocto-bsp \
  /home/user/kirkstone/poky/meta-greatyocto \
  "
```

* devtool을 위한 작업 공간 workspace 레이어를 생성한다.

```
~/kirkstone/build$ devtool create-workspace workspace

NOTE: Starting bitbake server...
INFO: Enabling workspace layer in bblayers.conf
```

* 다음과 같이 bblayers.conf 파일에 workspace 레이어가 새로 추가된 것을 확인할 수 있다.

```
~/kirkstone/build$ cat conf/bblayers.conf

# POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
# changes incompatibly

POKY_BBLAYERS_CONF_VERSION = "2"

BBPATH = "${TOPDIR}"
BBFILES ?= ""

BBLAYERS ?= " \
  /home/user/kirkstone/poky/meta \
  /home/user/kirkstone/poky/meta-poky \
  /home/user/kirkstone/poky/meta-yocto-bsp \
  /home/user/kirkstone/poky/meta-greatyocto \
  /home/user/kirkstone/build/workspace \
  "
```

* worksapce 레이어에 새로운 레시피를 추가한다.
  - `$ devtool add <recipes name> <source name>`
  - 레시피 이름을 지정하지 않고 url을 지정하여 소스를 fetch 할 수도 있다.
  - `~/kirkstone/build$ devtool add https://GitHub.com/greatYocto/bbexample.git`
  - 레시피 이름이 greatyocto인데 이것은 Makefile.am 파일에서 "bin_PROGRAMS = greatyocto"라고 되어 있기 때문이다.

```
kirkstone/build/workspace/
|- appends
|   |- greatyocto_git.bbappend
|- conf
|   |- layer.conf
|- README
|- recipes
|   |- greatyocto
|       |- greatyocto_git.bb
|- sources
    |- greatyocto
        |- autogen.sh
        |- configure.ac
        |- LICENSE
        |- main.c
        |- Makefile.am
        |- README
```

* build/conf/devtool.conf 파일이 생성되고 이 내용은 다음과 같다.

```
[General]
workspace_pth = /home/user/kirkstone/build/workspace
```

* 자동으로 생성된 레시피 파일 greatyocto_git.bb 파일의 내용은 다음과 같다. (기본적으로 recipetool이 생성한 레시피라고 주석에 써있음)

~/kirkstone/build/workspace/recipes/greatyocto/greatyocto_git.bb
```
# Recipe created by recipetool
# This is the basis of a recipe and may need further editing in order to be fully functional.
# (Feel free to remove these comments when editing.)

# WARNING: the following LICENSE and LIC_FILES_CHKSUM values are best guesses - it is
# your responsibility to verify that the values are complete and correct.

LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://LICENSE;md5=1b4446e69313dfe99b262b1ed1dfddc4"

SRC_URI = "git://git@GitHub.com/greatYocto/bbexample.git;protocol=ssh;branch=master"

# Modify these as desired
PV = "1+git${SRCPV}
SRCREV = "2d8588ac1cf8d9dd4705e2c8e6757871da791ec2"

S = "${WORKDIR}/git"

# NOTE: if this software is not capable of being built in a separate build directory
# from the source, you should replace autotools with autotools-brokensep in the
# inherit line

inherit autotools

# Specify any options you want to pass to the configure script using EXTRA_OECONF:
EXTRA_OECONF = ""
```

* 이 경우 소스를 직접 작업 공간인 workspace 레이어로 가져오기 때문에 레이어 확장 파일 greatyocto_git.bbappend에서 externalsrc를 사용해 작업할 수 있게끔 도와준다.

~/kirkstone/build/workspace/appends/greatyocto_git.bbappend
```
inherit externalsrc
EXTERNALSRC = "/home/user/kirkstone/build/workspace/sources/greatyocto"
```

* 예제가 정상적으로 설치되었는지 확인하기 위해 `$ devtool status` 명령어를 사용한다.

```
~/kirkstone/build$ devtool status

NOTE: Starting bitbake server...
greatyocto: / home/user/kirstone/build/workspace/sources/greatyocto
(/home/user/kirkstone/build/workspace/recipes/greatyocto/greatyocto_git.b)
```

* 추가된 레시피를 다음과 같이 devtool을 사용해 빌드한다. (`$ devtool build <recipe name>`)
  - 빌드를 실행하면 bitbake 작업 디렉토리 build/tmp 디렉토리에 패키지 피드 배치 전까지의 태스크가 실행된다. (즉 do_package_write_rpm 태스크 전까지의 태스크들이 실행됨)

```
~/kirkstone/build$ devtool build greatyocto
```

* 새로운 창에서 QEMU를 실행시킨다.
  - 같은 호스트에서 또 다른 터미널을 띄우고 다음 명령어를 실행한다.
    ```
    $ source poky/oe-init-build-env
    $ runqemu nographic
    ```

* QEMU를 통해 새로 생성된 바이너리를 테스트한다.
  - 같은 호스트에 QEMU가 동작하고 있는 터미널은 그대로 두고, 작업하던 터미널 창에서 다음과 같은 명령을 입력해 새로 빌드된 패키지를 QEMU로 배포한다. (타깃 시스템에 SSH 서버(ssh-server-openssh)가 실행 중이어야 함)
  - `$ devtool deploy-target <recipe name> <타깃의 url>`
  - `$ devtool deploy-target -s greatyocto` (명령어에 의해 생성된 파일을 타깃으로 전송)
  - 만약 'WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!' 오류가 발생하면 `$ ssh-keygen -R 192.168.7.2` 명령어를 입력하고 다시 시도해 본다.

* QEMU에서 greatyocto 바이너리를 실행한다.

```
root@qemux86-64:~# greatyocto

Hello great yocto!
```

* 타깃에 설치된 바이너리를 제거한다.
  - `$ devtool undeploy-target <recipe name> <타깃의 url>`
  - `$ devtool undeploy-target -a <타깃의 url>` (이렇게 하면 모든 바이너리를 제거함)
  - `$ devtool undeploy-target greatyocto root@192.168.7.2`

* 소스를 수정하고 재빌드한다.
  - 소스의 위치는 '~/kirkstone/build/workspace/sources/greatyocto/main.c'이다.
  - 소스에서 printf 내용을 바꿔보고 다시 빌드한다.
    ```
    $ git add main.c
    $ git commit -m "source is changed"
    $ devtool build greatyocto
    ```

* 기존의 작업 공간에 새로 생성된 greatyocto 레시피를 추가한다.
  - 작업이 완료됐으므로 수정된 소스 파일과 레시피를 원래의 작업 공간으로 이동시킨다.
  - `$ devtool finish <recipe name> <이동되는 경로>`
  - `$ devtool finish greatyocto ../poky/meta-greatyocto/`
  - 위 명령어를 실행하면 이전에 수정한 소스의 패치가 자동으로 생성되어 원래의 작업 공간으로 이동하게 된다.
  - 다음과 같이 예제 레시피와 패치 파일이 원래의 작업 공간으로 이동된다.

```
meta-greatyocto/
|- conf
|   |- layer.conf
|- COPYING.MIT
|- README
|- recipes-example
|   |- example
|       |- example_0.1.bb
|- recipes-greatyocto
    |- greatyocto
        |- greatyocto
        |   |- 0001-source-is-changed.patch
        |- greatyocto_git.bb
```

* 하지만 소스 파일이 여전히 존재하기 때문에 다음과 같이 소스 파일을 수작업으로 삭제해 줘야 한다.
  - `~/kirkstone/build/workspace/sources$ rm -rf greatyocto/`

* 이미지 빌드 및 QEMU 실행을 통해 확인한다.
  - 이제 추가된 레시피의 패키지가 루트 파일 시스템에 추가되도록 build/conf 디렉토리에 local.conf 파일 제일 하단에 다음 내용을 추가한다.
  - `CORE_IMAGE_EXTRA_INSTALL += "greatyocto"`

* 원래 작업 공간에서 `$ bitbake core-image-minimal -C rootfs`를 통해 루트 파일 시스템을 다시 생성한다.

* `$ runqemu nographic` 명령을 입력하고 QEMU를 통해 변경된 바이너리 결과를 확인한다.

```
root@qemux86-64:~# greatyocto
It's changed!
root@qemux86-64:~# 
```

## devtool을 이용한 커널 모듈 생성

* 이미 정의된 레시피의 소스를 수정하려면 먼저 패키지의 소스 코드를 받아 로컬에 위치시켜야 하고, 받은 소스 코드에 레시피에서 정의된 patch 파일이 반영되어 있어야 한다. 이 상태에서 소스를 수정하고, 레시피에 추가할 패치를 만들게 된다. 최종적으로는 수정된 코드에 대한 패치 파일이 레시피 확장 파일 형태로 원래 작업 공간으로 추가/업데이트된다.

* 기존에 존재하는 커널에 새로운 드라이버를 추가하는 예제를 수행한다.
  - 기존에 존재하는 커널 레시피와 커널 소스 받아오기: `$ devtool modify <recipe name>`
    * `~/kirkstone/build$ devtool modify -x linux-yocto`: 여기에서 -x 옵션은 레시피에서 사용이 가능한 규칙에 따라 소스 코드를 추출하고 패치하도록 요청함
      ```
      workspace/
      |- appends
      |   |- linux-yocto_5.15.bbappend
      |- conf
      |   |- layer.conf
      |- README
      |- recipes
      |- sources
          |- linux-yocto
      ```
  - 커널 모듈 소스 작성
    * 간단한 커널 모듈인 devtool_test.c 파일을 커널 최상위 디렉토리 drivers/misc 디렉토리에 넣는다.
      ~/kirkstone/build/workspace/sources/linux-yocto/drivers/misc/devtool_test.c
      ```cpp
      #include <linux/module.h>
      #include <linux/kernel.h>
      #include <linux/init.h>

      MODULE_LICENSE("GPL");
      MODULE_AUTHOR("user");
      MODULE_DESCRIPTION("devtool test kernel module");
      static int __init devtool_test_init(void)
      {
          printk(KERN_INFO "Hello devtool test module!\n");
          return 0;
      }

      static void __exit devtool_test_cleanup(void)
      {
          printk(KERN_INFO "Cleaning up devtool test module.\n");
      }

      module_init(devtool_test_init);
      module_exit(devtool_test_cleanup);
      ```
    * 기존의 Makefile 파일 제일 하단에 다음 내용을 추가한다.
      `obj-y += devtool_test.o`
  - 깃을 이용해 변경된 소스 리포지터리에 커밋
    * 수정된 소스와 레시피 확인을 위해 `$ git status` 명령어를 입력한다.
    * 변경된 소스에 문제가 없다면 깃의 리포지터리에 커밋하고 devtool을 이용해 빌드를 진행한다. (주의: 빌드시 시간이 많이 소요됨)
      ```
      $ ~/kirkstone/build/workspace/sources/linux-yocto$ git add .
      $ ~/kirkstone/build/workspace/sources/linux-yocto$ git commit -m "Add new kernel module"
      ...
      $ ~/kirkstone/build/workspace/sources/linux-yocto$ devtool build linux-yocto
      ```
  - 원래 작업 공간에 변경 또는 추가된 내용 반영
    * 커널 소스의 변경/추가가 마무리됐으므로 이 내용을 원래 작업 공간으로 업데이트한다. (다음 2가지 방법 중 택일)
      - `$ devtool update-recipe <recipe name>` (기존 레시피에 내용을 추가함)
      - `$ devtool update-recipe -a <layer path> <recipe name>` (레시피 확장 파일을 만듦, 이 방법을 권장함)
    * `$ ~/kirkstone/build/workspace/sources/linux-yocto$ devtool update-recipe -a ../poky/meta-greatyocto/ linux-yocto`
    * 명령어를 실행하고 원래 작업 공간인 poky 디렉토리의 meta-greatyocto 디렉토리에 가보면 다음과 같은 디렉토리 구조를 볼 수 있다.
      ```
      poky/meta-greatyocto/recipes-kernel/
      |- linux
          |- linux-yocto
          |   |- 0001-Add-new-kernel-module.patch
          |- linux-yocto_5.15.bbappend
      ```
    * linux-yocto_5.15.bbappend 파일은 다음과 같다.
      ```
      FILESEXTRAPATHS:prepend := "${THISDIR}/${PN}:"
      SRC_URI += "file://0001-Add-new-kernel-module.patch"
      ```
  - workspace 레이어에 존재하는 커널 레시피 파일과 소스 파일 삭제
    * 소스 변경 작업이 완료되었으므로 workspace 레이어에서 커널 레시피와 소스를 삭제한다.
    * 커널 레시피는 `$ devtool reset`으로 삭제가 가능하나, 소스는 수작업으로 삭제해야 한다.
      ```
      ~/kirkstone/build$ devtool reset linux-yocto
      ~/kirkstone/build$ rm -rf worspace/sources/linux-yocto/
      ```
  - QEMU를 통해 변경된 커널 레시피와 패치의 결과 확인
    * 다시 루트 파일 시스템 생성을 위해 `$ bitbake core-image-minimal -C rootfs`와 같이 빌드를 수행하고, `$ runqemu nographic`을 통해 QEMU를 실행한다.
    * 다음과 같이 QEMU에서 앞에서 만든 커널 모듈이 실행됐는지 확인해본다.
      ```
      root@qemux86-64:~# dmesg | grep "Hello devtool test module"
      [    1.297494] Hello devtool test module!
      ```
  - 생성된 workspace 레이어 삭제
    * 원하는 작업을 모두 마무리했으므로 devtool의 작업 공간 workspace 레이어를 삭제한다: `$ bitbake-layers remove-layer workspace`
    * build 디렉토리의 workspace 디렉토리도 함께 삭제한다: `$ ~/kirkstone/build$ rm -rf workspace/`

## devtool 명령어 모음

devtool 명령어 | 설명
------------- | --------------
`$ devtool add <recipe name> <source name>` | 빌드될 새로운 소프트웨어를 추가한다.
`$ devtool modify <recipe name>` | 존재하는 레시피의 소스 파일을 수정하는 데 사용된다.
`$ devtool update-recipe <recipe name> -a <path to custom layer>` | workspace 레이어에서 변경한 내용을 원래 작업 공간에 반영한다.
`$ devtool build <recipe name>` | 레시피를 빌드한다.
`$ devtool finish <recipe name> <이동되는 경로>` | 원래 작업 공간으로 생성된 레시피와 패치를 이동시킨다. workspace 레이어에 생성됐던 레시피와 패치는 사라진다.
`$ devtool deploy-target <recipe name> <타깃의 url>` | 빌드를 통해 생성된 파일들을 타깃으로 전송한다.
`$ devtool undeploy-target -a <타깃의 url>` | 타깃에서 레시피의 출력 파일들을 제거한다.
`$ devtool status` | workspace 레이어의 상태를 보여준다.


# 부록: 문법 설명

## bitbake 문법

* 타입
  - 변수 타입 없음, 모든 값을 문자열로 인식함 ("value" 식으로 표현함)

* 변수명 규칙
  - 변수 이름에 제약을 두지 않음. 다만 관행적으로 변수 이름을 대문자로 시작하도록 함

* 문자열 표현 방식
  - 예) BITBAKE_VAR = "value" 또는 BITBAKE_VAR = 'value'

* 주석
  - #로 시작하면 주석으로 간주함

* 의존성 지정
  - `DEPENDS = "hello"` : hello 레시피 파일에 의존함 (그것이 먼저 빌드되어야 함)

* 변수값 할당
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

* 조건부 변수값 할당
  - `OVERRIDES = "sun:rain:snow"` : 콜론으로 구분된 값 목록을 갖는다. (오른쪽이 우선순위가 높음)
  - 예시
    ```
    OVERRIDES = "korean:american:vietnamese"
    FOOD_korean = "rice"    # 값이 할당됨
    FOOD_american = "bread"    # 값이 새로 할당됨 (우선순위가 american이 높으므로)
    FOOD_british = "sandwitch"    # 이것은 실행되지 않음, 최종적으로 FOOD 변수에는 "bread"가 할당됨
    CLOTHES = "kilt"
    CLOTHES_korean = "hanbok"    # CLOTHES 값이 "hanbok"으로 오버라이드됨
    ```

* 변수 플래그
  - 변수에 대한 속성의 구현
  - 예시
    ```
    VARIABLE_FLAGS[a] = "123"
    VARIABLE_FLAGS[b] = "456"
    VARIABLE_FLAGS[a} += "789"
    ```

* 변수 플래그의 종류
  - postfuncs: 태스크 완료 후 호출될 함수들의 리스트를 지정함 (`do_test1[postfuncs] += "posttest"`: do_test1 태스크 완료 후 posttest 함수 실행)
  - prefuncs: 태스크 실행 전에 호출될 함수들의 리스트를 지정함
  - nostamp: 특정 태스크 실행시 스탬프 파일을 생성하지 못하게 함
  - stamp-extra-info: 태스크 스탬프에 추가할 추가 스탬프의 정보 (오픈임베디드는 이 플래그를 사용해 머신별 작업을 허용함)
  - deptask: 빌드 의존성
    * `do_prepare_recipe_sysroot[deptask] = "do_populate_sysroot"`
    * do_prepare_recipe_sysroot 태스크 실행 전에 DEPENDS 변수에 포함된 레시피의 do_populate_sysroot 레시피를 먼저 실행해야 한다는 뜻이다.
  - rdeptask: 실행시간 의존성
    * `do_build[rdeptask] = "do_package_write_rpm"`
    * do_build 태스크 실행 전에 RDEPENDS 변수에 포함된 패키지의 do_package_write_rpm 태스크를 먼저 실행해야 한다는 뜻이다.
  - recrdeptask: Recursive 의존성
    * 어떤 태스크가 실행되기 전에 완료돼야 하는 의존성을 나타냄
    * 레시피 빌드, 실행 시간 의존성, addtask를 사용한 의존성 모두를 확인한 후 나열된 작업에 대한 의존성을 추가함
  - recideptask: 추가 의존성을 검사해야 하는 작업을 지정함
  - depends: 태스크 간 빌드 의존성
    * `do_image_wic[depends] += "syslinux:do_populate_sysroot"`
    * syslinux_xx.bb 파일에 정의된 do_populate_sysroot 패키지가 먼저 실행되고 do_image_wic 태스크가 실행된다는 뜻이다.
  - rdepends: 태스크 간 실행시간 의존성
  - umask: 파일/디렉토리 생성시 초기 접근 권한을 설정할 때 사용함
    * umask 값이 '0002'일 경우, 파일의 경우 666-002=664 (-rw-rw-r--)이고 디렉토리의 경우 777-002=775 (drwxrwxr-x)
    * `do_rootfs[umask] = "022"`: do_rootfs 태스크가 생성한 파일들의 권한이 644로 설정됨
  - fakeroot: 태스크가 루트 전용으로 작업을 수행할 수 있도록 pseudo 루트 환경을 제공함 (가령 권한 설정 등)
    * `do_taskname[fakeroot] = "1"`
    * 변수 플래그를 사용하지 않고 태스크의 이름 앞에 fakeroot 지시자를 넣어서 사용할 수도 있다.
  - vardeps: 시그니처 계산을 위해 변수의 의존성에 추가할 추가 변수의 공백으로 구분된 목록을 지정함
  - vardepsexclude: 시그니처 계산을 위해 변수의 의존성에서 제외돼야 하는 공백으로 구분된 변수 목록을 지정함
  - vardepvalue: 이 플래그가 설정된 경우 bitbake가 변수의 실제 값을 무시하고 대신 변수 시그니처를 계산할 때 지정된 값을 사용하도록 지시함
  - vardepvalueexclude: 변수 시그니처를 계산할 때 변수값에서 제외할 파이프로 구분된 문자열 목록을 지정함
  - dirs: 태스크가 실행되기 전에 생성되어야만 하는 디렉토리를 지정함

## 로그 출력 함수

Log Level | 파이썬 함수 | 셸 함수
--------- | ---------- | --------
plain | bb.plain(message) | bbplain message
debug | bb.debug(message) | bbdebug level message
note | bb.note(message) | bbnote message
warn | bb.warn(message) | bbwarn message
error | bb.error(message) | bberror message
fatal | bb.fatal(message) | bbfatal message

## bitbake/bin 실행파일 종류 (자세한 것은 --help 옵션으로 확인 가능함)
  - `bitbake [options] [recipename/target recipe:do_task ...]` : 대상 레시피(.bb 파일)의 특정 태스크를 실행한다. (기본 값은 'build')
  - `bitbake-getvar [-h] [-r RECIPE] [-u] [-f FLAG] [--value] variable` : BitBake 변수에 대해 질의한다. (**중요**)
  - `bitbake-layers [-d] [-q] [-F] [--color COLOR] [-h] <subcommand> ...` : BitBake 레이어 유틸리티. (bblayers.conf에 레이어 추가/삭제, 레이어 평탄화, 레이어 보기, 오버레이된 레이어 보기, 레시피/레시피 확장 보기, 레이어 생성 등)
  - `bitbake-diffsigs [-h] [-D] [-c color] [-d] [-t recipename taskname] [-s fromsig tosig] [sigdatafield1] [sigdatafile2]` : BitBake가 기록한 siginfo/sigdata 파일을 비교한다. (시그네처 파일 비교)
  - `bitbake-dumpsig [-h] [-D] [-t recipename taskname] [sigdatafile]` : BitBake가 기록한 siginfo/sigdata 파일을 덤프한다.
  - `bitbake-hashclient`
  - `bitbake-hashserv`
  - `bitbake-prserv`
  - `bitbake-selftest`

## 파이썬 함수 및 변수 확장

* bitbake는 파이썬, 셸 스크립트로 구성되어 있다.
  - 파이썬으로 구성된 태스크의 경우, 태스크 이름 앞에 python 지시어를 추가해야 함
    ```
    python do_mypythontask() {
        import time
        print time.strftime('%Y%m%d', time.gmtime())
    }
    ```
  - 파이썬 함수를 구현하기 위해서는 기존 파이썬 문법 그대로 함수 이름 앞에 def 지시어를 붙이면 된다.
    ```
    def get_depends(bb.d);
        if bb.data.getVar('SOMECONDITION', d, 1):
            return "dependencywithcond"
        else:
            return "dependency"
    SOMECONDITION = "1"
    DEPENDS = "${@get_depends(bb,d)}"
    ```
  - @ 연산자는 bitbake에게 현재 코드는 파이썬 코드로 이루어진 표현식이라고 알려준다. (inline 방식)

* bitbake는 파이썬 함수에서 변수를 직접 읽고 쓸 수 없고 데이터 사전(data dictionary)을 통해 변수를 읽고 쓸 수 있다. (d 변수: bitbake의 변수들이 저장되어 있는 장소)
  - pythontest.bb 파일 예시는 다음과 같다.
    ```
    LICENSE = "CLOSED"
    TESTVAR = "This var is read by python function"

    python do_testA () {
        pythonvar = d.getVar('TESTVAR', True)
        bb.warn(pythonvar)
    }
    addtask do_testA before do_build
    
    python do_testB () {
        d.setVar('TESTVAR', "This var is set by python function")
        pythonvar2 = d.getVar('TESTVAR', True)
        bb.warn(pythonvar2)
    }
    addtask do_testB before do_testA
    ```
  - 다음과 같은 결과가 나온다.
    ```
    WARNING: pythontest-1.0-r0 do_testB: This var is set by python function
    WARNING: pythontest-1.0-r0 do_testA: This var is read by python function    # 이렇게 나오는 이유? do_testA를 실행할 때는 do_testB 함수를 실행하지 않기 때문이다.
    ```
  - 파이썬 코드로 작성하는 것은 디버깅이 불편하기 때문에 대화형 파이썬 개발 셸을 이용한 디버깅을 이용할 수 있다.

* devshell과 비슷한 대화형 파이썬 개발 셸 pydevshell이 있다.
  - do_pydevshell 태스크를 실행하면 된다.
  - `$ bitbake pythontest -c do_devpyshell`: pythontest.bb 레시피를 타깃으로 디버깅하는 방법
  - pydevshell 터미널을 종료하려면 Ctrl+D를 입력하면 된다.
 
* 함수나 태스크를 만들 때 가장 많이 사용하는 파이썬 함수는 다음과 같다.

함수 | 설명
---- | -----
d.getVar("X", expand=False) | 변수 X의 값을 리턴한다.
d.setVar("X", "value") | 변수 X에 "value" 값을 할당한다.
d.appendVar("X", "value") | 변수 X에 값을 append 해준다.
d.prependVar("X", "value") | 변수 X에 값을 prepend 해준다.
d.expand(expression) | expression으로 변수들을 확장해준다.

* 익명 파이썬 함수
  - 레시피가 파싱될 때마다 실행되는 파이썬 함수로, 레시피에 대한 후처리를 돕는다. (__anonymous 키워드를 생략해도 됨)
    ```
    LICENSE = "CLOSED"
    TESTVAR = "This var is read by python function"

    python do_testA () {
        pythonvar = d.getVar('TESTVAR', True)
        bb.warn(pythonvar)
    }
    addtask do_testA before do_build
    
    python do_testB () {
        d.setVar('TESTVAR', "This var is set by python function")
        pythonvar2 = d.getVar('TESTVAR', True)
        bb.warn(pythonvar2)
    }
    addtask do_testB before do_testA

    python __anonymous() {
        pythonvar3 = d.getVar('TESTVAR', True)
        bb.warn(pythonvar3)
    }
    ```
  - 다음과 같은 순서로 태스크가 실행된다: do_testB 태스크 파싱 -> anonymous 함수 실행 -> do_testB 태스크 실행 -> do_testA 태스크 파싱 -> anonymous 함수 실행 -> do_testA 태스크 실행
