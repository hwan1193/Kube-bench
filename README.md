# Kube-bench

![image](https://github.com/user-attachments/assets/586203e9-acb7-4f3c-8cde-205889a209c1)

일단 처음으로 Kube-bench를 받아서 실행을 돌려본 사진이다.

Docker 명령으로 `aquasec/kube-bench:latest` 이미지를 성공적으로 **다운로드(pull)** 하였지만,

Docker 컨테이너 실행 직후 에러가 발생

exec ./entrypoint.sh: no such file or directory

최근 `aquasec/kube-bench:latest` 이미지에서 **엔트리포인트 스크립트 (`entrypoint.sh`)가 누락**

또는, **기본 실행 커맨드가 실패 한 상태이다.**

게속 실행하고, 돌려본 결과 동일한 에러문이 나왔고,

Docker 컨테이너가 기본적으로 실행하려는 /entrypoint.sh 파일이 아예 존재하지 않아서 생긴 문제

라는 결론에 도출했다.

![image](https://github.com/user-attachments/assets/a586569f-4a98-4080-ae1d-4a37f1c1b6ea)

혹시나 해서 쿠버네티스 파일의 누락이 있는지 검사해 보았지만,

문제는 없었다

## 해결 방법

Kube-bench는 Go 언어로 작성되었기 떄문에, 직접 빌드가 가능하다

그 뜻은, 직접 파일 값을 수정해서 작동하도록 만들 수 있다는 뜻이다

![image](https://github.com/user-attachments/assets/4b6ffb6a-e873-4dea-a250-742367d6b2b2)

그래서 go를 설치하고, 큐브 벤치를 컴파일을 다시 돌렸더니 버전이 맞지 않아서 또 오류가 발생했다

![image](https://github.com/user-attachments/assets/061edf02-1707-4490-8351-28a0475da78e)

일단 Go 버전을 제일 최신의 버전으로 설정해 주고,

# 최신 버전 다운로드 (예: 1.21.6)
wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz

# 기존 Go 제거 후 새로 설치
sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz

# 환경변수 설정
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

# 확인
go version

go version go1.21.6 linux/amd64
이 버전이 나오면 완료

nano go.mod
go 1.24.2 -> go 1.21

를 통해 Go 버전을 고정해 준다

디렉토리 위치는 kube-bench가 설치된 곳이다.

![image](https://github.com/user-attachments/assets/85e3348e-65d5-4e8e-9d27-093c72a5624b)

해당 스크린샷은 kube-bench를 돌리고 나서의 nano go.mod 사진인데,

버전이 1.23.0으로 나와있는 부분은 나중에 설명하겠다

수정을 거치고 저장을 한 다음,

go mod tidy
go build -o kube-bench.bin

해당 명령어를 통해 사용해야 할 kube-bench의 파일을 새로 빌드 해준다

그리고

./kube-bench.bin --benchmark cis-1.23

해당 명령어를 통해

cis-1.23 버전으로 벤치마크를 돌리게 된다

아래의 사진들을 벤치마크의 결과값이다

![image](https://github.com/user-attachments/assets/f0fc74c6-cb11-4929-b35f-4d7ab54c7700)

![image](https://github.com/user-attachments/assets/fc5fe114-eb3e-4a8e-87b3-f82e24bb3308)

![image](https://github.com/user-attachments/assets/93f2b0ab-fe6b-4b99-93bb-267e0592b80a)

![image](https://github.com/user-attachments/assets/3c50133d-5ff6-4563-809e-7f6da9fc89fa)

![image](https://github.com/user-attachments/assets/0dd9993b-e78e-42d9-b36d-1c29600a8574)

![image](https://github.com/user-attachments/assets/c4c215ff-f907-4363-8a6f-95ccc744a7cf)

![image](https://github.com/user-attachments/assets/dd7c0268-827c-446a-979e-3f83a31144bb)

![image](https://github.com/user-attachments/assets/d5f435c7-af36-4750-a7e9-af7366a719dd)

![image](https://github.com/user-attachments/assets/23a2a458-6387-481e-9865-b1d2e10a9479)

![image](https://github.com/user-attachments/assets/e43bbe5a-9824-4b36-a0fa-87d2d57dd615)

![image](https://github.com/user-attachments/assets/ce026180-113b-49a9-9a27-b58065a15e9c)

이런 식으로 결과값들이 전부 생성되는 것을 확인했다

./kube-bench.bin --benchmark cis-1.23 --json > result.json

이런 추가 명령어를 통해 json 파일로 벤치마크 결과를 뽑아낼 수 있다

추가로

--benchmark string        # 사용할 벤치마크 버전 (예: cis-1.23)
--config string           # 사용자 정의 config 파일 지정
--config-dir string       # config 디렉토리 직접 지정 (기본값: ./cfg)
--json                    # 출력 결과를 JSON 형식으로 표시
--no-summary              # 결과 요약 생략
--noremediations          # remediation(조치 방법) 출력 생략
--outputfile string       # 출력 파일 지정
--version                 # kube-bench 버전 정보 출력
--log-level string        # 로그 수준 설정 (debug, info, warn 등)
--target string           # 점검할 구성 지정 (etcd, controlplane, policies 등)

이런 식으로 옵션을 추가해서 값을 뽑아 낼 수 있다

뽑아낸 json 파일을 gpt에게 던져주고 해석해달라고 해봤다

![image](https://github.com/user-attachments/assets/80a4ccfc-fd1e-4256-924d-724c5d08033d)

![image](https://github.com/user-attachments/assets/175df859-414e-4bcd-a076-d19ad349b2a7)

![image](https://github.com/user-attachments/assets/edaf2bb5-be07-4c37-bbab-951067e66768)

제대로 된 프롬프트를 갖춘다면 정보를 보기 쉽게 정제해서 보고서를 작성하면 될 것 같다

# 설치 순서도

시스템에 쿠버네티스가 설치되어있는 것을 가정

1. 도커 설치
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker

2. Go 설치
wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz

2.1 환경변수 설정
echo 'export PATH=/usr/local/go/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

go version
했을때
go version go1.21.6 linux/amd64
가 나와야 함

3. 버전 맞춰주기
#kube-bench 위치에서
nano go.mod

go 1.24.2
이 부분을 찾아서
go 1.21
로 수정

4. 실행 파일 컴파일
cd ~/kube-bench
go mod tidy
go build -o kube-bench.bin

5. 파일 실행
./kube-bench.bin --benchmark cis-1.23
또는
./kube-bench.bin --benchmark cis-1.23 --json > result.json
로 로그 생성

해당 링크 : https://seungjuitmemo.tistory.com/371?category=1074586
