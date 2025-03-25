## Grafana Beyla
메트릭 및 트레이스를 얻기 위해서는 APP 내부에 SDK 또는 에이전트를 추가해야 한다. 애플리케이션 옵저버빌리티를 수행하기 위한 eBPF 기반의 애플리케이션 자동 계측 도구. eBPF를 사용하여 APP 또는 OS 네트워크 계층을 자동으로 검사하고 HTTP 및 gRPC 서비스에 대한 웹 트랜잭션, RED 메트릭을 추적한다.

> 대표적인 예로는 CNI(cilium)

## Belya 배포 모드
- 백엔드 저장소에 직접 전송하는 방식
- Alloy를 거쳐 전송하는 방식
  
![](https://grafana.com/media/docs/grafana-cloud/beyla/alloy-vs-direct.png)

alloy를 거치게 되면 전처리 및 배치 작업을 수행한다.  
![](https://grafana.com/media/docs/grafana-cloud/beyla/nodes-2.png)

## eBPF란?
Linux 커널 단계에서 커널 소스 코드 변경 또는 모듈을 로드하지 않더라도 기존의 커널 단계를 안정적이고 효율적으로 확장시키기 위한 기술이다. 기존의 문제점은 커널을 변경하기 어려웠다. 하지만 eBPF를 통해 기존 OS에 유연하게 추가적인 기능을 제공할 수 있다. OS는 JIT 컴파일러와 kernal helper API를 통해서 추가된 프로그램의 안정성과 효율성을 보장한다.

eBPF 이점
- high performance networking
- observability
- load balancing
- security(include container runtime)

## eBPF 동작 방식 
eBPF는 이벤트 기반으로 동작한다. 커널 또는 APP이 특정 훅 지점을 지나갈 떄 실행된다. 만일 특정 훅이 정의되어 있지 않다면 새로운 커널 프로브를 생성하여 훅을 실행시킬 수 있다.

![](https://ebpf.io/static/b4f7d64d4d04806a1de60126926d5f3a/691bc/syscall-hook.webp)

사전 정의 훅  
- system calls
- function entry/exit  
- kernel tracepoints
- network events
- others

## eBPF 아키텍처
![](https://ebpf.io/static/1a1bb6f1e64b1ad5597f57dc17cf1350/691bc/go.webp)


1. verifier  
- eBPF 프로그램을 로드하는 프로세스는 특별한 권한이 필요하다. 특권이 필요없는 eBPF를 허용하지 않는 한, 특별한 권한을 가진 프로세스들만 eBPF 프로그램을 로드할 수 있습니다.
- eBPF 프로그램은 크래시가 나거나 시스템에 악영향을 끼치지 한다.
- eBPF 프로그램은 항상 종료해야 한다. (무한 루프 또는 추가 처리를 위한 대기)

![](https://ebpf.io/static/7eec5ccd8f6fbaf055256da4910acd5a/691bc/loader.webp)

2. JIT Compiler  
JIT 컴파일는 프로그램의 실행 시간을 최적화하기 위해 프로그램의 일반화된 바이트 코드를 특정 머신에서 동작하는 명령어 집합으로 변환한다. 이를 통해 eBPF 프로그램은 사전에 컴파일 된 커널 코드 또는 로드 된 커널 모듈과 비슷하게 효율적으로 동작할 수 있다.

3. maps  
eBPF는 수집된 정보를 공유하기 위해 상태를 저장한다. 이를 위해 map 개념을 사용하여 다양한 자료 구조를 지원하는 데이터를 저장한다. eBPF map은 eBPF 프로그램 뿐만 아니라 시스템 콜을 사용하면 유저 공간에서의 애플리케이션에서도 접근할 수 있다.

![](https://ebpf.io/static/e7909dc59d2b139b77f901fce04f60a1/691bc/map-architecture.webp)

4. helper Calls  
임의의 커널 함수를 호출할 수 없다. 특정 커널 버전을 사용하게 된다면 프로그램의 호환성을 복잡하게 만들 수 있기 때문이다. 대신 eBPF는 커널에 의해 제공되는 잘 알려지고 안정적인 API 인 커널 함수를 호출할 수 있다.

![](https://ebpf.io/static/6e18b76323d8520107fab90c033edaf4/691bc/helper.webp)

5. Tail & Function Calls    
꼬리 재귀(tail call)의 개념과 함수 호출을 사용할 수 있습니다. 함수 호출은 eBPF 안에서 함수를 정의하거나 호출하는 기능을 의미한다. 꼬리 재귀 호출은 다른 eBPF 프로그램을 실행하거나, 현재 실행 컨텍스트를 변경할 수 있도록 하는 기능을 제공한다. 이 방식은 execve() 시스템 콜이 일반적인 프로세스에서 동작하는 방식과 유사합니다.

![](https://ebpf.io/static/106a9d37e6b2b88e24b923d96e852dd5/691bc/tailcall.webp)

## eBPF가 리눅스 커널에 끼친 영향
![리눅스 커널의 고수준 아키텍처](https://ebpf.io/static/560d57883f7df9beafb47eee1d790247/691bc/kernel-arch.webp)
> 리눅스 커널의 주요 목적  
> 하드웨어의 추상화를 제공하여 일관성 있는 시스템 콜(API)를 제공함으로써 다양한 APP이 실행을 지원하고 자원을 공유하는 것에 있다.

목적을 달성하기 위해 커널의 여러 서브 시스템과 계층은 이러한 커널의 책임을 분산하도록 유지되고 있다. 각 서브 시스템은 일반적으로 사용자의 다양한 요구를 고려할 수 있도록 어느정도 설정이 변경 가능토록 설계되어있다. 커널의 변경을 하기 위해서는 2가지 방식이 있다.

1. 커널 소스코드 변경  
2. 커널 모듈 수정  

## 이해가 안가는 점
alloy + beyla vs otel  
alloy는 otel의 배포판으로 즉 grafana 환경에 맞게끔 커스텀한 버전. 메트릭과 로그는 수집하지만 APP에 대한 계측을 자동으로 지원하지 않는다. 이를 대체하기 위해 beyla를 사용하여 APP이 아닌 네트워크로 트레이스를 제공하는 것으로 보인다.

[공식 문서]    
https://grafana.com/docs/beyla/latest/  
https://ebpf.io/ko-kr/what-is-ebpf/  
