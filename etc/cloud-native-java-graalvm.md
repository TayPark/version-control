# 클라우드 네이티브 자바, GraalVM

https://www.youtube.com/watch?v=DIshNpDES78&ab_channel=OracleKorea

고성능 자바, 언어 상호 운용, 네이티브 이미지에 초점을 맞춘 새로운 자바

## GraalVM

GraalVM은 OpenJDK, OracleJDK 를 확장한 Java이다. 전자는 CE, 후자는 EE로 배포된다. 전자는 무료로 사용할 수 있다. 후자는 유료이며 Oracle의 지원을 받을 수 있고 OCI(오라클 클라우드)$$에서 무료로 사용할 수 있다.

## 고성능 자바

이전까지의 자바
- 기존의 자바는 단위 시간당 최대 처리량이었음. 
- 자바의 성능 개선은 GC 알고리즘 변경, 런타임 메모리 구조 개선, 자바 라이브러리 효율성 향상, CPU 최적화, JIT 컴파일러 등
- JIT 컴파일러: 런타임에서 자바 바이트코드를 기계어로 바꾸는 컴파일러
  - 최적화의 핵심
- C1과 C2 컴파일러
  - C1 컴파일러: 빠른 실행을 위한 컴파일러
  - C2 컴파일러: 프로파일링을 통한 코드 최적화
  - GraalVM의 핵심은 **C2 컴파일러를 GraalVM 으로 바꾸는 것** 

퍼포먼스 메트릭
- 빠른 시작 시간
- 단위 시간당 최대 처리량
- 지연시간 최소화
- 낮은 메모리 사용량
- 패키지 파일 용량

## 언어 상호 운용

Polygrot 컨셉 - `Truffle framework` 을 이용한 통합 언어 플랫폼으로의 JVM

## 빠른 시작

Java bytecode 를 읽고 native 실행 파일을 만들어 `native-image` 명령어로 class 파일 실행

## 후기

JDK를 바꾸는 것만으로도 성능 향상이 있고 빅데이터 플랫폼에서 JVM은 필수적이니, 실험적으로 도입해보는 것도 괜찮다고 생각