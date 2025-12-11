1. 아바게로 해야함. 슈퍼카인 firedancer는 네트워크 어댑터부터 장비차원이 다름
공식 리드미
1.. Install rustc, cargo and rustfmt.
$ sudo apt-get update
$ sudo apt-get install libssl-dev libudev-dev pkg-config zlib1g-dev llvm clang cmake make libprotobuf-dev protobuf-compiler libclang-dev

2. Download the source code.
$ git clone https://github.com/anza-xyz/agave.git
$ cd agave

3. Build.
$ ./cargo build


2. 안깔린거 깔고 난 여기서 해결됨
sudo apt-get update
sudo apt-get install --reinstall libclang-dev clang llvm

3. 안되면
LIBCLANG_PATH 환경 변수 설정 (가장 확실한 방법)
시스템에 libclang.so 파일이 어디에 있는지 찾아서 직접 경로를 지정해 주는 방법입니다
find /usr/lib -name "libclang.so*"

4. 
검색 결과 확인 결과가 다음과 비슷하게 나올 것입니다 (버전 숫자는 다를 수 있습니다, 예: llvm-14, llvm-15):
/usr/lib/llvm-14/lib/libclang.so.1 /usr/lib/llvm-14/lib/libclang.so
여기서 libclang.so 파일이 있는 디렉토리 경로를 복사합니다. (예: /usr/lib/llvm-14/lib
# 경로 예시가 /usr/lib/llvm-14/lib 인 경우
export LIBCLANG_PATH=/usr/lib/llvm-14/lib
# 다시 빌드 시도
./cargo build
# Release 모드로 빌드 (시간이 좀 더 걸리지만 최적화됨)
./cargo build --release

5. UDP 의 버퍼를 키우라함 일단 디폴트 확인
# 한 번에 5가지 항목의 현재 값 확인
sysctl net.core.rmem_max net.core.rmem_default net.core.wmem_max net.core.wmem_default net.ipv4.udp_mem

sysctl net.core.rmem_max net.core.rmem_default net.core.wmem_max net.core.wmem_default net.ipv4.udp_mem > original_sysctl_settings.txt

원복 명령어
# 1. 수신/송신 버퍼 원상복구 (212,992 bytes)
sysctl -w net.core.rmem_max=212992
sysctl -w net.core.rmem_default=212992
sysctl -w net.core.wmem_max=212992
sysctl -w net.core.wmem_default=212992
# 2. UDP 메모리 설정 원상복구 (본인의 서버 디폴트 값)
sysctl -w net.ipv4.udp_mem="1533762 2045016 3067524"

6. sysctl -w 는 원래 재부팅하면 초기화 되긴함 일단 적용
# 수신 버퍼(rmem)와 송신 버퍼(wmem) 크기를 134MB로 늘립니다.
sysctl -w net.core.rmem_max=134217728
sysctl -w net.core.rmem_default=134217728
sysctl -w net.core.wmem_max=134217728
sysctl -w net.core.wmem_default=134217728

# UDP 관련 설정 최적화
sysctl -w net.ipv4.udp_mem="65536 131072 262144"

7. 메모리 맵을 늘리라함
sysctl -w vm.max_map_count=1000000

원복
sysctl -w vm.max_map_count=65530

8. 그다음 문제는 방화벽임
# 1. Gossip 및 제어 포트 개방 (TCP/UDP)
sudo ufw allow 8000/udp
sudo ufw allow 8000/tcp
sudo ufw allow 8001/udp
sudo ufw allow 8001/tcp
sudo ufw allow 8899/tcp
sudo ufw allow 8899/udp

# 2. 동적 포트 범위 개방 (아까 설정한 8000-8025)
sudo ufw allow 8000:8025/udp
sudo ufw allow 8000:8025/tcp

# 3. 방화벽 끄고 켜서 적용 (이미 켜져 있다면 reload)
sudo ufw reload

9. 그다음 문제는 solana-perf-libs
# agave 루트 디렉토리에서 실행
./fetch-perf-libs.sh
# 먼저 위 1번 스크립트를 실행한 후,
# target/release 폴더 안에 perf-libs라는 이름으로 링크를 생성
ln -s /root/agave/solana-perf-libs /root/agave/target/release/perf-libs
# 만약 solana-perf-libs 폴더가 안 보이면, 1번 스크립트 실행 후 생성된 폴더 이름을 확인해서 연결해주세요.


