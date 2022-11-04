---
layout: post
title: '운영체제 기술 면접 대비'
subtitle: '운영체제, Operating System'
date: 2022-11-04 12:00:00 +0900
categories: 'ETC'
background: '/img/posts/etc/git.png'
---

#### 프로세스와 스레드의 차이를 설명해보세요.
- 프로세스는 운영체제로부터 자원을 할당 받는 작업의 단위
- 스레드는 프로세스가 할당 받은 자원을 이용하는 실행의 단위
- 운영 체제 `->` 프로세스 `->` 스레드
- 운영체제는 시스템 자원을 효율적으로 관리하기 위해 스레드를 사용합니다.
> - 멀티 프로세스로 실행되는 작업을 멀티 스레드로 실행할 경우, 프로세스를 생성하여 자원을 할당하는 시스템 콜이 줄어들어 자원을 효율적으로 관리할 수 있습니다. 프로세스 간의 통신보다 스레드 간의 통신의 비용이 적으므로 작업들 간의 통신 부담이 줄어듭니다.
- 스레드를 활용하면 자원의 효율성이 증가하지만 스레드 간의 자원 공유는 전역 변수를 공유하므로 동기화 문제에 신경 써야 합니다. 
> - 그래서 멀티 스레드 프로그래밍은 주의가 필요합니다.

<br>

#### 컨텍스트 스위칭에 대해 설명해보세요.
- 멀티 프로세스 환경에서 CPU(프로세서)가 어떤 하나의 프로세스를 실행하고 있는 상태에서 인터럽트 요청에 의해 다음 우선 순위의 프로세스가 실행되어야 할 때 기존 프로세스의 상태(or 레지스터값)를 저장하고 다음 프로세스를 수행하도록 교체하는 작업입니다. 

- 컨텍스트 스위칭이 일어나는 상황
> - I/O interrupt
> - CPU 사용시간 만료
> - 자식  프로세스 fork
> - 인터럽트 처리를 기다릴 때

<br>

#### 동기와 비동기의 차이(블로킹, 넌블로킹) / 장단점에 대해 설명해보세요.
- 동기는 순차적, 직렬적으로 태스크를 처리하고, 비동기는 병렬적으로 태스크를 처리합니다.
- 동기의 상황에서 태스크를 처리할 때 뒤의 태스크는 블로킹(작업 중단)되고 대기하게 된다.
> - 제어권을 태스크를 처리하는 대상이 가지고 있다.
- 비동기의 상황은 태스크를 병렬적으로 처리하기 때문에 논블로킹(대기하지 않고) 계속 수행하고 처리한다.
> - 작업 완료 여부와 상관없이 계속 새로운 작업을 수행한다. 

<br>

#### 멀티스레드 프로그래밍에 대해 설명해보세요.
- 하나의 프로세스에서 여러 개의 스레드를 만들어 자원의 생성과 관리의 중복을 최소화하는 것을 의미합니다.
- 장점
> - 멀티 프로세스에 비해 메모리 자원 소모가 줄어든다.
> - 힙 영역을 통해 스레드끼리 통신이 가능하다.
> - 프로세스 컨텍스트 스위칭보다 스레드 컨텍스트 스위칭이 빠르다.
- 단점
> - 힙 영역의 자원을 이용할 때는 동기화가 필요하다.
> - 동기화를 위해 락을 과도하게 사용하면 성능이 저하된다.
> - 하나의 스레드가 비정상적으로 동작하면 다른 스레드도 종료될 수 있다. 


<br>

#### Thread-safe 하다는 의미와 설계하는 법을 설명해보세요.

- 두 개 이상의 스레드가 race condition(경쟁 상태: 여러 프로세스나 스레드가 동기화 메커니즘 없이 자원에 접근하려는 상황)에 들어가거나 같은  객체에 동시에 접근해도 연산결과의 정합성이 보장될 수 있게끔 메모리 가시성이 확보된 상태를 의미한다. 

- 설계하는 법
> - java.util.concurrent 패키지 하위의 클래스를 사용
> - 인스턴스 변수를 두지 않는다,
> - Singleton 패턴을 사용
> - 동기화(syncronized) 블럭에서 연산을 수행
> > - 그래서 Spring을 사용하는 것, Spring은 Singleton Registry인 Application Context를 가지고 있다.
> > - Spring이 singleton을 사용하는 이유는 대규모 트래픽을 처리하기 위함
> > - [[Spring] 애플리케이션 컨텍스트(Application Context)와 스프링의 싱글톤(Singleton)_망나니개발자](https://mangkyu.tistory.com/151)

<br>

#### 세마포어와 뮤텍스의 차이에 대해 설명해보세요.
- 뮤텍스는 Lock을 사용해 하나의 프로세스나 스레드를 단독으로 실행하게 한다.
> - 0, 1의 값만 가지는 세마포어, 이진 세마포어 = 뮤텍스
- 세마포어는 공유자원에 세마포어 변수(공유자원의 개수를 나타내는 변수)만큼 프로세스(또는 스레드)가 접근할 수 있다.
- 세마포어는 다른 프로세스가 해제할 수 있지만, 뮤텍스는 락을 획득한 프로세스만 락을 해체할 수 있습니다.

<br>

#### 프로세스 동기화에 대해 설명해보세요.
- 다중 프로세스 환경에서 자원에 한 프로레스만 접근 가능하게 하는 것을 의미한다.
- 프로세스 동기화를 하지 않으면 데이터의 일관성이 깨져 연산 결과의 에러가 뜰 수 있으므로 주의해야 한다. 

<br>

#### 추가 부분

프로세스 동기화에 대해 설명해보세요.

교착상태와 기아상태의 해결방법에 대해 설명해보세요.

가상 메모리에 대해 설명해보세요.

캐시의 지역성에 대해 설명해보세요.

데드락이란?

페이지 교체 알고리즘이란?

<br>

Reference:
- [Backend-Interview-Question_ ksundong](https://github.com/ksundong/backend-interview-question)
- [프로세스와 스레드의 차이_강관우](https://brunch.co.kr/@kd4/3)
- [신입 개발자 기술면접 질문 정리 - 운영체제)_슬기로운 개발 생활](https://dev-coco.tistory.com/162)