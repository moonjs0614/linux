# POSIX 등장 배경

## 수 많은 OS의 등장

App 개발자들은 OS가 제공하는 API를 알아야 하는데, 임베디드 제품마다 사용되는 OS가 다를 수 있다.
Application 개발자들을 위해 OS마다 제공되는 API들을 하나로 통일하였다.
이렇게 통일된 API의 이름이 `POSIX` 다.

# POSIX

## POSIX 란?

- OS들이 지원하는 API들의 표준 규격
- IEEE 에서 제정

> POSIX API 만 배워두면 여러 임베디드 OS에서도 편리하게 App 개발이 가능하다.

## `System Call` 과 `POSIX` 차이

- `POSIX` : OS가 App에게 제공하는 API들의 표준
- `System Call` : 리눅스가 App 개발자들을 위해 제공하는 API

> System Call 에는 POSIX API 호환도 있고, 아닌것도 있다.

## 결론

OS를 쓰는 임베디드 시스템의 Application 개발자들은 POSIX API를 쓰면서 App을 만들어낸다.

## 용어 정리

1. API : API는 정의 및 프로토콜 집합을 사용하여 두 소프트웨어 구성 요소가 서로 통신할 수 있게 하는 메커니즘이다. 
2. System Call
3. POSIX
4. RTOS