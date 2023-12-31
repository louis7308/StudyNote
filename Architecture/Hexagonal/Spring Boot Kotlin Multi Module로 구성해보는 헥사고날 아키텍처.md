일단 흔히 개발자들은 프로젝트를 처음 구성할 때 어디서 주워들은걸로 한번 해볼까? 이거 흥하던데 같은 말로 토론 을 하고 이게 윗선의 결정을 받기까지 시간적 비용이 발생하지만 이러한 과정이 개발자들은 일상적으로 이루어집니다.

하지만 이 과정을 통해 선정한 기술이 프로젝트 중후반에 와서는 이OO 기술을 사용하면 안되었구나.. 라는 이러한 멘트를 남기면서 고쳐야 하지만 이 OO 기술이 비즈니스 로직 전반적으로 영향을 뻗치고 있다면 앞이 어두 캄캄할 것입니다.

그러하여 제가 지금 약을 팔 아키텍처는 비즈니스 요구사항을 빠르게 개발할 때 기술 선택에 대한 고민으로 소모되는 비용을 아낄 수 있는 대표적인 어플리케이션 아키텍처 입니다.

헥사고날 아키텍처는 기술 선택에 대한 고민으로 소모되는 비용을 아낄 수 있고 독립적이어야 하는데 왜 Spring 으로 하냐 라는 말이 나올 수 있지만 가장 대중적으로 많이 사용되고 있으며 100% 이론을 실제화 하기 보다 상황에 맞는 방법을 택하여 생산성을 높이는 선택이 맞을 것 같아 조금 배타적이지만 부득이하게도 Spring에 종속적인 모듈 구성을 하게 되었습니다.

## 구성
헥사고날 아키텍처를 토입하기로 한 프로젝트인 naru는 사람들이 근처에서 울리면 좋아하는지 알 수 있는 프로그램의 백엔드 기능을 담당하기 위해 총 4개의 핵사곤 (layer ) 으로 정의하였습니다.

* Domain Hexagon
* Application Hexagon
* Framework Hexagon
* BootStrap Hexagon
![[Pasted image 20231117145742.png]]

### Domain Hexagon
* Domain Layer 영역
* DDD(도메인 주도 개발)의 그 Domain Layer로 기술에 독립적인 POJO로 개발
* Utility, Extension, Constants, Entity, VO(Enum 포함), Aggregate를 포함하며 프로젝트 도메인의 비즈니스 룰을 정의
* POJO로 구현하기 때문에 Spring의 Component Service annotation 등 비사용
* 비즈니스 규칙을 위한 Service 개념이 존재할 수 있어야 하지만 static class의 메서드로만 정의
* 실제로 gradle에 어떤 dependency도 포함하지 않음
```kotlin
import org.springframework.boot.gradle.tasks.bundling.BootJar

val jar: Jar by tasks
val bootJar: BootJar by tasks

bootJar.enabled = false
jar.enabled = true

dependencies {

}
```
> [!tip]
> POJO란 Java로 생성한 순수한 객체를 뜻 한다.
> 객체지향적인 원리에 충실하면서 환경과 기술에 종속되지 않고, 필요에 따라 재 활용될 수 있는 방식으로
> 설계된 오브젝트를 의미한다.

깔끔

![[Pasted image 20231117150226.png]]

### Application Hexagon
* Application Layer영역
* Domain의 구성요소를 사용하여 시스템이 가지는 기능/사례(useCase)를 정의한 집합
* logging, exception, usecase, outputPort(interface)등을 포함
* DB가 어떤 것인지, 외부에서 시스템을 가동하기 위한 기술은 무엇인지 아무것도 알 필요가 없다. 알아서도 안된다.
* 팀에 신규 인력이 추가되거나 더 나아가 PO, PM같이 비 개발 인력도 읽을 수 있을 만큼 쉬운 코드로 작성
* POJO로 개발하고 다른 영역에서 DI를 제어해야 하지만 Spring Framework을 버리고 다른 DI Framework를 사용할 여지가 있는가? 라는 물음에 그렇지 않다 라는 결론을 내렸고 각 기능(usecase) 들에 대하여 Service Annotation을 사용한 Service로 정의
* 의존성을 Domain Hexagon에 대해서만 가짐

```kotlin
import org.springframework.boot.gradle.tasks.bundling.BootJar

val jar: Jar by tasks
val bootJar: BootJar by tasks

bootJar.enabled = false
jar.enabled = true

dependencies {
    implementation(project(":ceo-united-domain")) // domain Hexagon 의존

    implementation("org.springframework.data:spring-data-commons")
}
```
AppConfig.kt  
(component scan의 lazyInit option에 대한 내용은 하단에 기재합니다)
```kotlin
@Configuration
@ComponentScan(basePackages = ["com.naru.application.usecase"], lazyInit = true)
class AppConfig
```
![[Pasted image 20231117151014.png]]

### Framework Hexagon
* Secondary/Driven Adapters (Infrastructure) 영역
* Application Hexagon이소유한 OutputPort( interface ) 구현체들의 집합
* Framework Hexagon은 여러개의 module로 분리, 관리
* Application Hexagon과 마찬가지로 Spring에 대한 의존성은 가짐
* 각 Framework Hexagon은 각 기술의 이름을 딴 config class를 포함하며, config class는 그 기술이 사용할 component들을 bean으로 등록할 수 있게 scan 영역을 격리

ceo-united-framework-aws
build.gradle.kts
```kotlin
import org.springframework.boot.gradle.tasks.bundling.BootJar

val jar: Jar by tasks
val bootJar: BootJar by tasks

bootJar.enabled = false
jar.enabled = true

dependencies {
    implementation(project(":ceo-united-domain"))
    implementation(project(":ceo-united-application"))

    implementation("org.springframework.boot:spring-boot-starter-data-mongodb")
    implementation("software.amazon.awssdk:s3")
    implementation("software.amazon.awssdk:netty-nio-client")
}
```
MongoConfig.kt
```kotlin
@Configuration
@EnableMongoRepositories
@ComponentScan(basePackages = ["com.naru.framework.aws.adapter.mongo"])
class MongoConfig {}
```

S3Config.kt
```kotlin
@Configuration
@ComponentScan(basePackages = ["com.naru.framework.aws.adapter.s3"])
class S3Config {}
```
![[Pasted image 20231117152837.png]]
ceo-unifed-framework-retrofit
build.gradle.kts

```kotlin
import org.springframework.boot.gradle.tasks.bundling.BootJar

val jar: Jar by tasks
val bootJar: BootJar by tasks

bootJar.enabled = false
jar.enabled = true

dependencies {
    implementation(project(":ceo-united-domain"))
    implementation(project(":ceo-united-application"))
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-jackson:2.9.0")
    implementation("com.squareup.okhttp3:okhttp:3.14.9")
    implementation("com.squareup.okhttp3:logging-interceptor:3.14.9") 
```
RetrofitConfig.kt

```kotlin
@Configuration
@ComponentScan(
    basePackages = [
        "com.baemin.ceo.united.framework.retrofit.adapter",
        "com.baemin.ceo.united.framework.retrofit.config"
    ]
)
class RetrofitConfig {
```
![[Pasted image 20231120104553.png]]
## BootStrap Hexagon
* Primary/Driving Adapters ( User Interface 영역 )
* 프로그램의 기능을 사용하기 위한 시작점
* 우리는 RestAPI를 제공하는 WebService, Apache, Kafka, AWS의 SQS(Simple Queue Service)를 구독하는 Consumer가 존재 (Controller 라고 보는게 맞겠네) API 호출 및 security
* Domain, Applicaiton, Framework Hexagon에 대하여 의존적, 애플리케이션 구동에 필요한 모듈의 config만 선별적으로 import

ceo-united-bootstrap-api
build.gradle.kts
```kotlin
apply(plugin = "kotlin-kapt")

dependencies {

	implementation(project(":ceo-united-domain"))
    implementation(project(":ceo-united-application"))
    implementation(project(":ceo-united-framework-aws"))
    implementation(project(":ceo-united-framework-retrofit"))

    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    implementation("com.fasterxml.jackson.datatype:jackson-datatype-jdk8")
    implementation("com.fasterxml.jackson.datatype:jackson-datatype-jsr310")
    implementation("org.springframework.boot:spring-boot-starter-webflux")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    implementation("org.springframework.boot:spring-boot-starter-aop")
}
```

CeoUnitedApiApplication.kt
```kotlin
@SpringBootApplication
@Import(
	value = [AppConfig::class, MongoConfig::class, RetrofitConfig::class]
)

@ConfigurationPropertiesScan
class CeoUnitedApiApplication

fun main(args: Array<String>) {
	runApplication<CeoUnitedApiApplication>(*args)
}
```
![[Pasted image 20231120105218.png]]
#### ceo-united-bootstrap-worker

build.gradle.kts

```kotlin
apply(plugin = "kotlin-kapt")

dependencies {
    implementation(project(":ceo-united-domain"))
    implementation(project(":ceo-united-application"))
    implementation(project(":ceo-united-framework-aws"))
    implementation(project(":ceo-united-framework-cache"))
    implementation(project(":ceo-united-framework-retrofit"))

    kapt("org.springframework.boot:spring-boot-configuration-processor")

    implementation("org.springframework.cloud:spring-cloud-starter-aws-messaging:2.2.6.RELEASE")
    implementation("org.springframework.kafka:spring-kafka")
```

CeoUnitedWorkerApplication.kt

```kotlin
@SpringBootApplication(
    scanBasePackageClasses = [
        CeoUnitedWorkerApplication::class
    ]
)
@Import(value = [AppConfig::class, MongoConfig::class, RetrofitConfig::class, CacheConfig::class, S3Config::class])
@ConfigurationPropertiesScan
class CeoUnitedWorkerApplication
fun main(args: Array<String>) {
    runApplication<CeoUnitedWorkerApplication>(*args)
}
```
## 개발사례

> 사장님이 사용하는 포스(POS)가 시작될 때 클라이언트 환경설정을 조회하되, 그 설정이 존재하지 않으면 초기화된 설정으로 제공하고 싶다.

위와 같은 요구사항이 발생하였을 때 우리가 구성한 아키텍처에서는 다음과 같이 개발됩니다.

(code snippet은 콘셉트를 알기 위함으로 일부 발췌된 코드입니다.)
![[Pasted image 20231120105538.png]]
