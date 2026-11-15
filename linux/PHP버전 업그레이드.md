## PHP 메이저 버전 업그레이드 작업 정리

**서버:** omnidnc (Rocky Linux 8, php-fpm)
**작업일:** 2026-10-22
**변경:** PHP 7.2.24 (Rocky AppStream `7.2.24-1.module+el8.4.0+413+c9202dda`) → **8.4.27** (Remi `php:remi-8.4` 모듈)

### 1. 작업 내용
1. 사전 백업
   - 모듈 목록: `~/php72_modules.txt`
   - 설치 패키지 목록: `~/php72_rpms.txt`
   - 설정: `/etc/php.ini` → `/etc/php.ini.bak72`, `/etc/php.d` → `/etc/php.d.bak72`, `/etc/php-fpm.d` → `/etc/php-fpm.d.bak72`
2. EPEL 및 Remi 저장소 추가
```
   dnf install -y epel-release
   dnf install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```
3. PHP 모듈 스트림 전환. AppStream의 7.2 모듈을 초기화하고 Remi 8.4 모듈을 활성화했습니다.
```
   dnf module reset php -y
   dnf module enable php:remi-8.4 -y
   dnf update -y php\* # 기존 php 패키지가 8.4로 교체됨
```
4. 패키지 설치. 기존 7.2 패키지 구성을 그대로 따르되, `php-json`은 8.0부터 코어에 내장되어 제외하고 `php-opcache`를 추가했습니다.
```
   dnf install -y php php-cli php-common php-fpm php-xml php-mbstring \
     php-gd php-mysqlnd php-pdo php-pecl-zip php-devel php-opcache
```
5. 백업한 `php.ini`의 커스텀 설정값을 새 `php.ini`에 반영
6. php-fpm 및 웹서버 재시작 후 `php -v`로 버전 확인

### 2. 결과
- `php -m` 비교 결과, 7.2 대비 모듈 변경은 아래와 같으며 모두 예상된 차이입니다.
  - 제거: `wddx` (PHP 7.4에서 제거, 대체 없음)
  - 추가: `random` (8.2부터 코어), `sodium` (Remi 8.x 기본 포함), `Zend OPcache` (신규 설치)

### 3. 이후 관리 시 유의사항
- **모듈 스트림 유지:** `php:remi-8.4`가 활성화되어 있어 `dnf update` 시 8.4.x 패치만 적용되고 8.5로 올라가지 않습니다.
- **전체 업데이트 주의:** `dnf update`를 전체로 실행하면 DB 등 다른 패키지도 함께 올라갈 수 있습니다. PHP만 올릴 때는 `dnf update php\*`를 사용합니다.
- **PHP 확장 추가 시:** Remi 저장소의 `php-xxx` 패키지로 설치합니다.
- **코드 호환성:** 7.2 → 8.4는 메이저 두 단계를 건너뛰므로 영카트5 소스의 8.x 호환 여부를 계속 점검해야 합니다. (null 전달 Deprecated, 문자열/숫자 비교 변경, 동적 프로퍼티 Deprecated 등)
- **롤백 방법:** `dnf module reset php -y && dnf module enable php:7.2 -y` 후 `dnf distro-sync php\*`로 7.2를 재설치하고 설정 백업을 복원합니다.

### 4. 점검 중 발견한 기존 이슈 (업그레이드와 무관)
| 우선순위 | 위치                        | 내용                                            |
| ---- | ------------------------- | --------------------------------------------- |
| 상    | [lib/common.lib.php:1595] | 방문자 로그 기록시 같은IP, 날짜가 중복 기록시도하면 오류 발생 (유니크 키). |



## PHP 마이너 버전 업그레이드 작업 정리

**서버:** web01 (Ubuntu 24.04 Noble, Apache 2.4.58 + mod_php)  
**작업일:** 2026-10-22  
**변경:** PHP 8.3.6 (Ubuntu 공식 `8.3.6-0ubuntu0.24.04.13`) → **8.3.36** (`8.3.36-1+ubuntu24.04.1+deb.sury.org+1`)

### 1. 작업 내용

1. 사전 백업
    - 설치 패키지 목록: `~/php-pkgs.txt`
    - 설정: `/etc/php/8.3` → `~/php83-etc-backup`
2. Ondřej Surý PPA 추가: `ppa:ondrej/php`
3. 메타패키지 고정. 이 패키지들이 PHP 8.4/8.5를 끌어오는 것을 막기 위한 조치입니다.
    
    ```
    sudo apt-mark hold php php-cli php-curl php-gd php-mysql php-xml php-zip php-bz2 libapache2-mod-php
    ```
    
4. `php-mcrypt` 메타패키지 제거. 신규 `php8.3-mcrypt`와 충돌했기 때문이며, 실제 확장인 `php8.3-mcrypt`는 유지했습니다.
5. 설치된 8.3 패키지 14개만 업그레이드
    
    ```
    sudo apt install --only-upgrade $(dpkg-query -W -f='${db:Status-Abbrev} ${Package}\n' 'php8.3*' 'libapache2-mod-php8.3' | awk '$1=="ii"{print $2}')
    ```
    
    - 대상: libapache2-mod-php8.3, php8.3, -bz2, -cli, -common, -curl, -gd, -mbstring, -mcrypt, -mysql, -opcache, -readline, -xml, -zip
6. Apache 재시작 후 `php_module` 로드 확인

### 2. 결과

- error.log에서 업그레이드로 인한 신규 에러는 확인되지 않았습니다.

### 3. 이후 관리 시 유의사항

- **메타패키지 hold 유지:** `apt upgrade`를 해도 8.4/8.5가 설치되지 않습니다. 해제 명령은 `sudo apt-mark unhold <패키지>`입니다.
- **PHP 확장 추가 시:** 반드시 `php8.3-xxx`처럼 버전을 지정해 설치합니다.
- **향후 업데이트:** 일반 `apt upgrade`로 8.3.x 패치를 받게 됩니다.
- **PPA 이전 예정:** 이 PPA는 `packages.sury.org`로 이전이 예정되어 있습니다. 이전 안내가 나오면 저장소를 바꿔야 합니다.
- **롤백 방법:** `sudo ppa-purge ppa:ondrej/php`를 실행해 공식 8.3.6으로 되돌린 뒤 설정 백업을 복원합니다.

### 4. 점검 중 발견한 기존 이슈 (업그레이드와 무관)

| 우선순위  | 위치                                    | 내용                                         |
| ----- | ------------------------------------- | ------------------------------------------ |
| 🔴 높음 | `sun_yuseong/sub2_11_2.html` 68행      | SQL 인젝션 시도가 확인되었습니다. 입력값 처리가 필요합니다.        |
| 🟠 중간 | `sun_yuseong/proc/tel_api.php` 3행     | `implode()` TypeError로 전화 관련 기능이 장애 상태입니다. |
| 🟡 낮음 | `sun_mediscreen/sub69.html`           | 존재하지 않는 `sub69.php`를 호출하고 있습니다.            |
| 🟡 낮음 | `sun_dental/board/free/down_list.php` | 직접 접근 시 `mem_auth()` undefined 에러가 납니다.    |
