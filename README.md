# Yocto 프로젝트란 무엇인가?

* 임베디드 장치용 맞춤형 배포판 빌드를 위한 오픈 소스 프로젝트
  - 커스텀 리눅스를 만드는 도구는 Yocto 프로젝트 외에도 Linux Live Kit, Linux From Scracth(LFS), Live Magic, SUSE Studio Express 등이 있다.
* 내가 개발하고자 하는 임베디드 환경(ARM, x86 등)에 맞게 커스텀 리눅스를 빌드해주는 도구 (셋팅의 번거로움을 줄여주는 역할)
  - Yocto: 오픈 임베디드 빌드 시스템이 리눅스 소프트웨어 스택을 빌드하는 데 필요한 모든 정보를 제공함 (크로스 컴파일러, 라이브러리 등)
  - bitbake: Yocto에서 제공하는 정보를 기반으로 빌드를 수행하는 빌드 도구
  - Poky: 빌드를 위해 필요한 소스 코드를 git에서 가져오거나, 빌드 환경 설정, 컴파일, 생성된 이미지를 설치하는 방법을 기술하는 .bb 파일(메타데이터)이 들어 있음

* 전체적인 맵은 다음과 같다.
  - 사용자는 조리법에 해당하는 것만 작성하면 된다. [conf (환경 설정 파일), bb (레시피 파일), bbclass (클래스 파일), bbappend (레시피 확장 파일), inc (인클루드 파일)]
  - 재료 역할을 하는 meta 안에는 HW, SW에 대한 모든 정보가 들어 있다. (링크만 있음, 실질적인 정보는 다운로드해야 함)
  - bitbake가 알아서 조리법을 기반으로 결과물을 만들어준다.
  - 결과물로 root file system image, kernel image, boot loader image가 나온다.

* Yocto 프로젝트 작동 절차는 다음과 같다.
  1. Poky reference system 준비 (download, 환경 설정)
  2. target board에 맞는 BSP layer 생성
  3. meta-layer 생성
  4. recipe 토대로 download, patch, configure, build(compile, link)
  5. install
  6. package 생성
  7. boot 이미지 생성

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
    |- meta-skeleton   # 커스텀 레이어를 생성하는 데 사용되는 템플릿 레이어
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

# 빌드 속도 개선하기

* PREMIRRORS(로컬 미러 저장소), sstate-cache(공유 상태 캐시)를 구성하여 소스를 미리 다운로드함으로써 fetch 시간을 단축하여 빌드 속도를 개선하는 것임
  - 소스를 fetch, 즉 다운로드하는 데 시간을 가장 많이 소요한다. (빌드 속도는 로컬 머신의 성능에 달려 있음)
  - Yocto에서 자주 빌드할 경우 미리 소스를 받아두는 편이 시간을 단축하는 데 도움이 많이 된다.
  - 또 경우에 따라서는 원격 저장소에 문제가 생겨서 소스를 받지 못하는 경우도 있기 때문에 미리 받아두는 편이 도움이 될 수 있다.
  - 로컬 미러 저장소는 단순하게 git 소스를 저장해 두는 방식이고, 공유 상태 캐시는 빌드 단계 스냅샷을 저장해 두는 방식이다. (전자를 권장함)

## 소스 다운로드 절차

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

### 실습 순서

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
  - '-g' 옵션은 지정한 레시피 파일에 대해 의존성을 가진 모든 패키지들을 pn-buildlist라는 파일로 출력함
  - pn-buildlist 파일의 내용을 확인하면 어떤 패키지가 설치되어 있는지 확인할 수 있다.

```
$ bitbake -g great-image
...
NOTE: PN build list saved to 'pn-buildlist'
NOTE: Task dependencies saved to 'task-depends.dot
```

...

# BSP 레이어 작성

...

# 커널 레시피

...

# 커널 레시피 확장

...

# 배포 레이어 작성

...

# 커스텀 레이어 작성

...

# 패키지

* 배포 및 설치를 위한 패키지는 대표적으로 다음 종류가 있다.
  - RPM, DEB, IPK, TAR 등

...

# 패키지 설치를 위한 태스크

...

# 공유 상태 캐시와 시그니처

...

# kirkstone

...

# 개발 환경 구축

...

# 기타

...

# devtool

...

# 부록: 문법 설명

## bitbake 문법 (1)
  - 변수 타입 없음, 모든 값을 문자열로 인식함 ("value" 식으로 표현함)
  - 변수 이름에 제약을 두지 않음. 다만 관행적으로 변수 이름을 대문자로 시작하도록 함
  - 예) BITBAKE_VAR = "value" 또는 BITBAKE_VAR = 'value'
  - #로 시작하면 주석으로 간주함

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
  - `bitbake-getvar [-h] [-r RECIPE] [-u] [-f FLAG] [--value] variable` : BitBake 변수에 대해 질의한다.
  - `bitbake-layers [-d] [-q] [-F] [--color COLOR] [-h] <subcommand> ...` : BitBake 레이어 유틸리티. (bblayers.conf에 레이어 추가/삭제, 레이어 평탄화, 레이어 보기, 오버레이된 레이어 보기, 레시피/레시피 확장 보기, 레이어 생성 등)
  - `bitbake-diffsigs [-h] [-D] [-c color] [-d] [-t recipename taskname] [-s fromsig tosig] [sigdatafield1] [sigdatafile2]` : BitBake가 기록한 siginfo/sigdata 파일을 비교한다. (시그네처 파일 비교)
  - `bitbake-dumpsig [-h] [-D] [-t recipename taskname] [sigdatafile]` : BitBake가 기록한 siginfo/sigdata 파일을 덤프한다.
  - `bitbake-hashclient`
  - `bitbake-hashserv`
  - `bitbake-prserv`
  - `bitbake-selftest`
