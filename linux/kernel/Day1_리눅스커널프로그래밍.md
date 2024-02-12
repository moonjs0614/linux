# 커널 일자별 수업 내용

> day1 (리눅스) -> 디바이스 드라이버, 간단히 체험
> day2 -> user space와 kernel space 간 데이터 교환
> day3 -> 본격적인 H/W 제어 (양이 많음)
> day4 -> 부트로더

# SoC, BSP 업체 (임베디드 리눅스 개발자가 주로 가는 기업들)

> 리눅스 임베디드 개발자는 리눅스를 사용하는 업체에서 필요로 한다.
> 리눅스가 필요한 회사람 ? -> AP SoC 업체
> AP : Application Processor (앱이 동작하는 고성능 CPU)
> SoC : System on Chip (내부에 다양한 기능이 있는 칩) ex. 삼성 LSI
>
> SoC 업체에 취업을 한다 -> 임베디드 리눅스 개발을 한다.
> SoC(칩)안에 많은 전자제품이 들어간다. 부품들이 원활하게 동작할 수 있도록 디바이스 드라이버를 개발하면 된다.
> SoC 업체는 자체적으로 하드웨어도 만들고, 하드웨어가 동작하는 소프트웨어도 만드는 업체이다.
> 할일 : 리눅스 코드, 드라이버 코드, 샘플 코드를 제작하게 된다.
>
> ---
>
> SoC 업체에서 개발한 칩을 갖고 개발을 시작하는 업체:
> BSP 업체 (Board Support Package 개발 회사) -> 임베디드 계의 SI 업체라고도 불린다.
>
> SI 업체? : 시스템 통합업무를 담당하는 업체
> SoC 부품회사들의 부품을 가져와 보드에 장착해서 제대로 동작할 수 있는 시스템을 구축하는 업무를 한다.
> -> BSP 업체는 SoC가 포함된 보드를 제작하는 업체
>
> SI 업체 (일이 많다.) 시스템이 굴러가게 해야 한다.
> -> 야근이 많다
> 중소 / 중견기업이 많다. -> 신입 개발자가 입문하기 좋은 회사
> 일은 많고 능력은 부족한 신입 개발자가 많이 있는 회사 -> 야근이 잦은 회사
> 회사 평가가 안 좋을 수 밖에 없는 회사
>
> BSP 업체는 일이 많고 다양한 부품을 다룰 수 밖에 없는 환경이기 때문에 꼭 한두명씩 괴물이 있다.
> (군대식 문화도 많다. 하라면 해야한다.)
>
> SI 업체중에서는 대기업도 있다. (대기업이지만 정체성은 SI)
>
> - 삼성 SDS
> - 현대오토에버
> - LG CNS
> - 한화시스템 ICT부 등등
>
> BSP 업체:
> 보드 회로도 제작 -> 여러개 SoC 붙이고, 중국에 회로도를 보내면 2주 뒤에 테스트 샘플 보드가 온다.
> 테스트를 하고 주문을 한다.
>
> SoC 업체에서 제작한 "리눅스 소스코드", "드라이버 코드", "샘플 코드" 로
> BSP 업체에서 제작한 결과가 "H/W", "S/W" 샘플 ( + document 사용설명서)
> -> 라즈베리파이 보드 Rpi4 model B+
>
> 여러분(고객)은 이 보드를 갖고 놀거나, 여기에 내가 새로운 디바이스 드라이버를 추가해서 나만의 업그레이드 된 리눅스 제작이 가능하다. -> 커널 수업에서 진행하는 것 (== 임베디드 시장)

---

> 라즈베리 파이에서 커널 빌드를 할 것이다.
> 6.1 ~ 버전이 설치 된다.
> 라즈베리파이 커널 소스코드는 리눅스 커널 소스코드를 수정하여 개발이 되었다.
> Q. 맨 처음 어떤 업체에서 이 작업을 했을까? SoC 업체 (BroadCom)
>
> 라즈베리파이가 만들어지는 과정 :
> BroadCom 에서 리눅스 커널 소스코드를 보고 칩을 제작한다. (수정된 리눅스 소스코드, 드라이버 코드, 샘플코드 가 만들어짐)
> BSP 업체에서 BroadCom이 제작한 `삼신기` 로 보드를 제작한다. (Rpi4 보드, S/W 샘플, (최종 커널 코드, document 사용설명서) 가 만들어짐)

---

> 나는 나만의 (리눅스 커널 기반의) OS 개발을 하고 싶다면?
>
> SoC 부터 만들려면
>
> 1. 회로 제작 (FPGA, arm 라이선스(개인이 못 얻음), 오픈소스인 RISC V 회로 (라즈베리파이의 회로를 가져옴, 개인 O) -> 수정)
> 2. 중국에 보낸다.
> 3. 주문형 반도체를 받는다.
> 4. "리눅스 커널 코드"를 다운 받아 내가 만든 SoC에 맞게 코드를 수정해서 시작한다.
>    -> 초고수가 하는 방식
>
> 만들어진 보드에서부터 시작 하려면 (Rpi4)
> 만들어진 리눅스 소스코드를 수정해서 개발하면 된다.
> 이게 우리가 수업 때 할 일

**개요 끝, 대 서막의 시작**

# 1장. 리눅스에서 임베디드 개발

## 리눅스 임베디드 개발

이 문서 맨 앞 내용

## Device Driver 란?

### 디바이스 드라이버는 주변에 항상 있다.

USB 스틱을 PC에 연결할 때 뜨는 메세지

- 모니터, 프린터, 공유기 등 구매 할 때 항상 들어있는 CD

디바이스 드라이버란

- PC와 외부 H/W 장치가 서로 신호를 주고 받을 수 있도록 통신을 하는 것을 도와주는 소프트웨어

H/W 개발하는 업체들은 장치를 개발하고 나서 장치가 보내는 신호를 PC가 받아서 처리할 수 있게 해주는 App 과 App으로 전달해주는 디바이스 드라이버 개발을 해야 한다.

디바이스 드라이버의 사전적 정의

- Program 이 H/W를 제어하기 위한 SW를 뜻함
- Software Interface를 통해 Application 이 H/W 스펙을 이해하지 않아도 되는 장점이 있다.

> 개념적인 한 문장으로 Device Driver를 이해하기는 힘들다. 이것이 필요했던 스토리를 알아야 디바이스 드라이버의 정의를 이해할 수 있다.

### Firmware 에서 임베디드 개발

HW 메모리 맵 Address에 직접 값 Access 가능

- Application이 HW를 직접 제어 한다.(중간 Layer가 없음)

```c
int main(void)
{
    (*(volatile unsigned*)0x40021018) |= 0x8;
    (*(volatile unsigned*)0x40010C04) |= 0x10;
}
```

### 디바이스 드라이버의 필요성

HW를 사용하는 여러개의 Firmware 들을 개발했는데, 만약 HW 장치를 교체해야 된다면?

- 모든 Firmware 의 HW 관련 코드들을 수정하여 모든 하드웨어에 다시 Firmware 를 다운로드 후 실행해야 한다.

이를 위해 커널(중간 Layer)가 나타났다.

- Kernel 은 공통적으로 쓰는 API를 제공
- Kernel 소스코드만 새로운 HW가 동작되도록 수정하여 다시 Build 하면, 다른 Firmware 들 수정은 필요 없다.

> `Application` <==> `Kernel` <==> `HW`

새로운 HW 추가를 위해 커널 소스코드 수정시 재 빌드가 필요하다

- Kernel 만 재 빌드하면 되는데, Build 시간이 너무 오래 걸린다.

커널 빌드에 걸리는 시간 (개발 PC기준)

- 2010년도: 한국에서 미국까지 비행기 타고 걸리는 시간
- 2020년도: 3분, 라면 익는 시간
- AMD의 64코어: 20초

> Rpi4 기준 1 ~ 2시간

커널 빌드에 오래걸려서 커널 안에서도 2개의 Layer로 나눴다.

- 커널을 다시 Build 하지 않도록 모듈 방식 사용
- Device Files 에게 API 를 던진다
- Device Driver만 재 Build를 하여 커널에 넣고 뺀다

> `커널 모듈` 이 등장
> 커널 모듈: 커널에 들어가는 코드 덩어리
>
> 디바이스 드라이버를 커널 모듈 형식으로 제작하여, 커널 모듈말 동작 중인 커널에 적재/해제 하는 방식으로 테스트 하는 개념을 착안

> == 디바이스 드라이버를 빌드하면, 결과물이 커널 모듈 형식으로 만들어지고, 이 결과물을 커널 안에 적재/해제 하는 방식으로 테스트 할 수 있다.

> 이렇게 테스트가 끝나면 커널을 빌드하고, 배포하면 된다. (이렇게 만들어진 결과물이 `.ko` 파일)

Application과 HW 사이에 계층을 두었다.

- App은 커널의 API를 통해 H/W 접근이 가능하다.
- HW에 대한 지식이 없어도, 커널 API를 통해 H/W를 제어하는 Application 제작 가능

Device Driver Module 사용하기

- 커널에 Device Driver Module 을 넣는다. (`insmod` 명령어)
  - 넣으면 Kernel이 해당 모듈을 관리하기 시작한다.
- 필요 없으면 Module 을 제거한다. (`rmmod` 명령어)
  - 빼는 것이 아니라, 메모리 상 모두 Remove 하는 것이다.

> `Application` <==> Ker(Device `Files` <==> Device `Driver`)nel <==> `HW`

## Device Driver 실습

### 실습 part

**skip**

### 디바이스 드라이버의 장점

App / Driver의 역할 분담

Application 개발자
- HW 를 제어할 수 있는 API 사용 방법 익히기
- API가 건내 준 Data 처리
- API를 이용하여 HW 제어
- Log 남기기
- Network / UI 작업

Device Driver 개발자
- HW 제어 API 제작
- HW 불량 분석
- Latency 측정

### [참고] 타 OS에서 Device Driver 개발

- Windows : Windows Driver Kit (WDK) 으로 개발
  - C++
  - WDM (Windows Driver Model) : MS에서 Driver 개발 표준화 시킨 모델
  - 개발자 센터 : https://learn.microsoft.com/ko-kr/windows-hardware/drivers/



## 모듈 자세히 보기

# 2장. Device Driver 기초

## Device Node

## System Call

## 구조체 지정 초기화

## app과 driver 개발

# 오프라인 강의 노트

>  51페이지 Major Number, Minor Number 시험에 낼 것임 (주번호는 약속된 번호가 있고, 부번호는 개발자 마음대로 정해도 됨. 안지킨다고 빌드가 안되는건 아님)

디바이스 드라이버는 그냥 룰 따르는 것이라고 생각하면 된다.

커널 빌드할 때 컴퓨터 키면 할일: (디바이스 파일은 컴퓨터를 껐다 켜면 사라진다.)

```bash
$ sudo mknod /dev/nobrand c 100 0 # su 권한으로 nod(파일)를 만든다, /dev 경로에 nobrand라는 이름으로, 캐릭터에 메인 100 마이너 0
$ sudo chmod 666 /dev/nobrand # 모두가 읽고 쓸 수 있도록 권한을 바꾼다
$ dmesg -w # 커널 로그 보기
$ make
$ sudo insmod nobrand.ko # 커널에 모듈을 삽입함
$ ./app
$ sudo rmmod nobrand # 커널에서 모듈을 제거함
$ make clean
```

**파일 종류**
`app.c` : 애플리케이션 파일 (사용자에게 제공하는 파일. 커널 코드가 포함되어 있지 않다.)
`nobrand.c` : 디바이스 드라이버 (커널 모듈)
`Makefile` : 빌드


`Makefile` 내용

```make
# uname -r 하면 현재 버전의 커널 정보를 불러온다
KERNEL_HEADERS=/lib/modules/$(shell uname -r)/build # 여기에서 빌드 하고 와라
CC = gcc # 앱을 빌드할 컴파일러 (앱 레벨이여서 커널과는 상관 없음, gcc는 커널 빌드를 못한다)

TARGET := app
obj-m := nobrand.o

all : driver app # make만 입력하면 driver와 app을 실행한다

driver:
    make -C $(KERNEL_HEADERS) M=$(PWD) modules

app:
    $(CC) -o $@ $@.c

clean:
    make -C $(KERNEL_HEADERS) M=$(PWD) clean # 커널 디렉토리에서 빌드 한 결과를 다 가져와서 풀어준 것들을 클린한다
    rm -f *.o $(TARGET) # 모든 obj 파일과 app 바이너리 파일을 지움

```