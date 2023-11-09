흔히 JPA에서는 영속성 컨텍스트가 존재하는데 이는 EntityManager 에서 트랜잭션 범위 안에서만 동작하며
트랜잭션이 종료되면 1차 캐시 안에 있는 데이터는 사라진다.

## 동일성

만약 같은 기본키 값을 가진 Entity를 두번 조회한다면
```kotlin

	val a = em.find(Member::class.java, "member1")
	val b = em.find(Member::class.java, "member1")

	println(a == b) // true
```
이는 같은 1차 캐시 안에서 같은 참조값을 가진 entity를 반환해주기 때문에 동일성이 지켜진다.

## 쓰기 지연
쓰기 지연은 persist에서 db와 싱크(SQL문 보내기)를 맞추는 것이 아니라 flush 시점에 쿼리가 날아가는 것이다.
BatchSize에 해당 크기 만큼 모아서 네트워크 보낼 수 있다.
```kotlin
	val memberA = Member(200L, 'A')
	val memberB = Member(201L, 'A')

	em.persist(memberA)
	em.persist(memberB)

	println("******")
	tx.commit()

```

## 변경감지
```kotlin
	val member = em.find(Member::class.java, 201L)
	member.name = "전승원"

	tx.commit()
```
이 상황에서 어 persist가 실행이 안되었는데 어떻게 동작하는지 궁금할 수 있는데
em.find()를 통해 1차 캐시에 저장되는데 이때 snapshot 도 같이 저장되어진다.
그리고 commit 시
변경 감지의 과정에 대해서 정리하자면

1. 1차 캐시에 저장될 때 1차 캐시에는 최초 저장 시점의 SnapShot을 저장해둔다.
2. 만약 변경이 감지되면 commit 되는 시점에 flush가 호출되면서 JPA가 일일이 각각의 SnapShot을 비교해서 변경을 감지한다.
3. 변경을 감지하였다면 UPDATE 쿼리를 쓰기 지연 SQL 저장소에 생성을 하고 데이터베이스에 반영하고 커밋을 진행한다.
## 지연로딩
![[Pasted image 20231109102045.png]]