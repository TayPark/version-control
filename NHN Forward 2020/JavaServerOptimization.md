# Java 실시간 게임 서버 성능을 10배 올린 3가지 썰

[영상](https://www.youtube.com/watch?v=MZmZGwFRM2U&t=5s&ab_channel=TOAST)

## 결함 1 Twisted Fibers

> 이 프로젝트에서는 Quasar를 사용하고있다. 이는 스레드 내에 Fiber를 사용하는 것이고, 이는 차후 OpenJDK에서 프로젝트 Loom이란 이름으로 추가될 수 있다.

하나의 스레드는 여러개의 fiber(일종의 경량 스레드)가 구동될 수 있다. Fiber 스케줄러에 의해 Fiber들은 균등하게 스레드를 선점하게된다. 여기서 싱글 스레드 기반의 Fiber 구동 모델을 `Node`라고 하자. 

하나의 노드에서 Fiber는 스레드를 선점하면서 게임 로직을 처리한다. 여기서 Fiber는 다른 Fiber와 하는 일이 각자 다르므로, 상호 관계를 형성한다. 그러므로 스레드를 선점하며 적절하게 context switching이 일어나야한다.

정상적인 경우에는 다음의 순서를 따른다.

1. User Fiber가 Room Fiber에게 접속 요청을 시도한다.
2. 요청을 받은 Room Fiber는 Room pool을 체크하거나, 생성, validation을 하기 위한 로직을 수행하는데, 이는 Blocking code 이다.
3. 그러므로 User Fiber는 suspend 상태가 된다. 
4. 해당 로직이 모두 끝나면 User Fiber로 스레드를 넘겨준다. 

하지만 의도하지 않은 상태가 발생한다. 방 Fiber가 처리를 하다 User Fiber의 Suspend 상태가 풀릴 수 있다. 만약 Room Fiber가 정상적으로 로직을 수행했다 하더라도 두 상태를 신뢰할 수 없다.

또 다른 경우 Room Fiber는 모두 처리하였음에도 불구하고 User Fiber가 깨어나지 않았을 경우이다.

이는 코드에서 문제가 있었는데, 유저의 Join 요청이 Room 에서 `Queue`로 처리되고 있었다. 일반적인 Queue로는 Fiber 사이의 명확한 흐름을 만들 수 없다. 또한 Room 데이터를 직접 참조를 통해(public) 데이터를 바꾸고 있었다. 이 경우 Room Fiber가 데이터를 바꾼 후에 DB 쿼리를 통해 데이터 변경을 알릴 때 Suspend가 되고, User가 이를 변조할 경우 문제가 생긴다.

1번의 문제는 Queue를 Fiber Channel로 바꿨다. Channel을 통해 상태 Fiber가 처리할 코드를 전송한다. 

2번의 문제는 Restricter 메소드를 통해 해당 Fiber가 자기자신이 아니거나 허용된 Fiber가 아니면 예외를 발생시키도록 하였다.

## 결함 2 Bottle-neck

클라이언트와 게임 서버는 GateWay Nodes를 통해 통신한다. 클라이언트는 Netty, 서버사이드는 ZeroMQ를 사용한다.

하나의 코드가 있다. 이 코드는 다수의 스레드가 접근하며, Netty와 ZeroMQ 모두 접근한다.

`ConcurrentHashMap`을 사용하는 코드가 있다. 이는 여러 Segment로 이루어져있고, 각 Segment는 Read-Write Lock의 단위가 된다. 즉, READ가 발생하였을 때, 다른 스레드의 WRITE Lock까지 같이 발생하여 작업을 진행할 수 없는 특징을 가진다.

또한 코드 구조상 하나의 패킷에 대해 4회의 READ가 발생한다. 새로운 접속시에 put(), 접속 종료 시에 remove()를 하고, 패킷 하나당 2회씩 호출하여 4회가 발생하는 것이다.

해결하기 위해 PacketParser를 static화 하여 매 연결마다 새로운 객체를 만들어서 Map에 넣을 필요가 없어진다. 그리고 getHandler() 메소드를 추가하여 4회 READ를 1회 READ로 변경하였다. 

## 결함 3 잘못된 Lock-free 구조

각 스레드마다 NodeInfo를 두어 각 노드 정보를 공유하도록 하였다. 이렇게 하면 Lock을 쓸 필요가 없을 것이라 생각했기 때문이다. 

하지만 이 경우 스레드가 늘어날 수록 CPU 사용량이 급격히 증가했고, 200대 서버 환경에서 프로세스당 8개의 노드로 구성했을 때 가만히 놔두기만 해도 CPU 사용량이 20%까지 올라갔다. 1600개의 노드가 서로 정보를 교환하며 취합하고 정렬을 했기 때문이다. 이 때문에 메모리 사용량과 NIO 비용도 심각했다.

해결하기 위해 프로세스마다 하나의 NodeInfo 관리자를 두었다. 전체 정보를 교환하는 것이 아니라 변경된 값만 주고받도록 하였다. 

## 결함 4. VM의 함정

CPU Stil Time: VM의 가상 CPU가 다음 처리를 위한 물리 CPU를 대기하는 시간값에 대한 백분율. 0에 가까울 수록 좋음.

동일한 물리 서버상에서 다른 VM이 과도한 작업을 하고 있다면 나머지 VM의 성능이 떨어질 수 있다. 

다른 요소로는 서로 다른 물리 스펙으로 구성할 수도 있다. C/T뿐만 아니라 클럭 수도 있다.

VM 구성 차이도 있을것이다. 같은 물리 서버를 다른 VM 셋업으로 구성할 수도 있다. 32코어 물리 서버를 2코어 VM 16대 구성과 4코어 VM 4대 구성으로 나눌 수도 있다.

해결 방법으로는 로드 밸런싱의 기준값을 변경했다. 아까 언급했더 NodeInfo 의 데이터가 로드밸런싱의 주체인데, 각 VM의 처리 속도를 반영하여 로드밸런싱을 실행하였다. 이럴 위해 NodeInfo 관리자를 고도화하였다.

## 다이어트

### 패킷과 프로토콜 다이어트

사용하는 패킷에 데이터를 줄였다. 또한 주기적으로 주고받는 NodeInfo 또한 줄이고, 바뀐 값만 주고받았다.

### 문자열을 정수로

문자열로 이루어진 NodeId 50바이트, RoomID 80바이트, UserId 32바이트이다. 이는 서버 엔진에서 저장, 비교, 전송을 사용하므로 CPU, RAM, NIO 비용에 많은 부담이 된다.

그러므로 ID를 정수화한다. 외부로 나갈 필요가 없기 때문에 Global할 필요가 없다. NodeId를 정수 8바이트, RoomId를 4바이트, UserId를 4바이트로 줄였다. 서버군 내에서만 고유한 값이면 상관 없었기 때문이다.

### 네트워크에서의 정수

구글 프로토콜 버퍼를 사용했다. 지그재그 인코딩을 사용하며, 이는 정수를 사용했을 때 값이 작으면 작을수록 데이터를 작게 사용한다는 장점을 갖는다. 이 때 unsigned 값을 사용한다면 절대 음수여서는 안된다. Overflow로 인해 데이터가 커질 수 있다.

## 최적화

### HashMap vs. ArrayList

HashMap은 말 그대로 Map에 Hashing을 하는 자료구조이다. 또한 boxing/unboxing 연산이 들어가는데, 적잖게 연산이 들어간다.

굳이 그럴 필요가 없는 곳에서는 ArrayList를 사용하는게 더 낫다.

다만 단점으로는, 65536의 크기로 잡은 ArrayList(static)는 약 256KB의 데이터를 미리 선점하게 된다는 점이다. 하지만 성능을 택했다.

자료구조 성능 비교를 했을 때도, worst case에서는 극명하게 갈린다. ~~그리고 hashing 비용과  boxing/unboxing을 생각한다면...~~

### 루프 최적화

while 루프 안의 메소드를 까보니 2중 for loop에 CAS(Compare and Swap) 메소드를 실행하고 있었다.

이를 Loop를 돌아야 할 ArrayList.size()를 가져와서 loop를 돌도록 변경했다.

### 과유불급, TLS(스레드 로컬 스토리지)

각 Node에서 자기 자신의 객체에 접근할 수 있는 가장 간단한 방법은 TLS를 사용하는 방법이다. 

이런 방식을 멤버 변수 혹은 파라미터로 필요한 객체를 넘겨 받기로 변경했다.