Netfilx는 영화, 제작일, 직원, 촬영지 등 클라이언트가 필요한 데이터는 여러 프로토콜들로 흩어져 있었습니다.
(gRPC, JSON API, GraphQL 등)
> [!tip]
> Protocol이란? 데이터베이스나 서비스 간 통신에 사용되는 표준이나 규격을 말합니다. 즉 필요한 데이터가 다른 시스템 간 다양한 방식(프로토콜)으로 분산되어 저장되어 있었다는 뜻입니다.

문제 : Netfilx는 기존 테이블이 300개가 넘게 존재하였는데,
이러한 점 이 계속 성장하면서 서비스의 영역이 넒어지는 것이 아닌 전문성만 깊어지는 것으로 발전했다
이는 점점 비즈니스 모델 자체가 확장하는 것이 아닌 해당 도메인이 전문적으로 점점 집중되어지게 개발되어지는 것을 느껴 Netfilx는 특정 서비스와 도메인 단위로 분해하기로 결정 하엿다.

해결 : 이러한 결정을 통해 특정 도메인 서비스를 담당한 팀에서 서비스를 독립적으로 전문성있게 개발할 수 있게 되었다.


Netfilx가 Hexagonal Architecture 를 사용한 이유 :
	제 생각에는 
	1. Netfilx는 MSA로 변경하면서 테이블 구조 등 관심사 부분이 바뀌어야 하는데 관심사 같은 경우는 클라이언트에 받는 입출력 과 단순 마이크로 서비스 에 사용하는 API, CSV 요청 등을 어디에서 받아오는지 관심사를 분리하기 위해서 이를 중요한 코드를 수정하지 않고 데이터베이스 세부 정보를 쉽게 변경할 수 있도록 해주기 때문에 사용한 거 같습니다.
	2. 또한 테스트에 대한 이점이 존재하여 프로토콜에 의존하지 않고 비즈니스 로직을 검증할 수 있다는 장점이 있어 사용한 거 같습니다.

육각형 설계를 활용하여 비즈니스 로직을 정의하는 세 가지 주요 개념 : 
Entity, Repository, Interactors

* **Entity**는 도메인 객체이므로 Java에 JPA와 달리 Entity들은 자신들이 어디에 저장되는지 전혀 모릅니다. ( 동영상 촬영 위치)
* **Repository**는 Entity를 만들고 변경하는 인터페이스입니다. 데이터베이스와 상호작용하는 메소드 들을 가지고 있으며 단일 엔티티 또는 엔티티 목록을 반환 
* **Interactors**는 도메인 행동을 조정하고 수행하는 클래스 서비스 , 유스케이스 와 비슷합니다.

헥사고날은 모든 의존성은 내부를 가리킵니다. 핵심 비즈니스 로직은 전송 계층이나 데이터베이스에 대한 관심사가 분리되어져 있습니다.

Netfilx는 갑자기 모놀리스로 읽기 제약 조건에 부딪혀 한 Entity에 대한 특정 읽기는 GraphQL의 집합 계층에 노출된 새로운 마이크로 서비스로 전환해야 했다.
이를 Hexagonal Architecture를 도입하여 2시간 만에 JSON API -> GraphQL을 이용한 데이터베이스 읽기로 변경 할 수 있었다
이렇게 이루어 낼 수 있었던 이유는 지속성 계층의 세부사항이 비즈니스 로직에 누출되지 않도록 하여 Repository 인터페이스를 구현한 GraphQL 데이터베이스를 만들었다 다른 데이터베이스에서 읽기 시작하는데 필요한 것은 간단한 코드 한줄 변경 뿐이었다.

> [!tip]
> 진짜 중요한 것은 코드 한 줄 변경으로 릴리스에 대한 위험을 완화가 가능하다는 것이다.
> 코드 배포 후 마이크로 서비스가 fail 할 경우 롤백하기도 쉬워지고 버그를 찾기도 쉬워진다.


