## 혼용 사례
DTO(VO), DAO 혼용해서 쓰는 이유

이는 사실 J2EE Patterns 1판 초판에서는 데이터 전달용 객체를 VO로 정의 했는데
2판에서는 데이터 전달용 객체를 TO로 정의 했습니다.
VO- > TO 
현재는 D를 붙여 DTO라고 부릅니다.

DTO 데이터 전달용
VO 값 표현용 

DTO란 DAta Transfer Object 데이터를 전달하기 위해 사용하는객체 데이터를 담아서 전달하는 바구니 역활
계층간 데이터를 전달하기 위한 객체

흔히 Controller 레이어에서 Service레이어로 데이터를 전달할 때 Dto에 데이터를 실어 전달하는 역활을 하는게 DTO
오직 Getter/Setter 메서드 만을 갖는다 
다른 로직을 갖지 않는다.
순수하게 데이터 전달용 객체 이기 때문이다.
하지만 DTO에서 setter를 사용하게 되면 불변객체가 아니게 되므로
setter를 사용하면 가변
최대한 setter를 배제하며 불변객체로 만들어보자 setter를 사용하지 않는 필드는 final로 지정하고
getter/ setter 외에 로직을 사용하지 않음
DTO는 속성값이 모두 같다고 해서 같은 객체가 아니다.

그리고 우리가 오해하면 안되는 것이 Entity class가 있고 dto class가 있는데 entity class는 절대로 요청이나 응답 값을 전달하는 클래스로 사용되면 안됩니다.
왜냐하면 Entity class를 기준으로 스키마가 생성되고 테이블이 생성됩니다.
view는 비즈니스 로직에서 자주 변경되는 상황이 많은데 예를 들어 요청만 봐도
Entity클래스로 요청을 받는다고 하면 view에서는 레이아웃이 바뀌는 경우가 많은데 그러면 이럴 때 마다 entity를 구조가 바뀌는 문제가 발생한다.
이러한 문제 때문에
view에 변경에 따라 다른 클래스에 영향을 주지 않고 사용할 수 있는 Dto를 사용해야한다.
꼭 분리 해서 사용

VO
Value Object
값 그자체를 표현하는 객체
흔히 돈에서 만원 짜리 지페에 고유번호가 존재합니다.
하지마 저희는 이것은 다른 만원이야 라고 말하지 않습니다.
같은 만원이라고 표현하죠 
Value Object 또한 불변객체여야 한다.
setter 과 같은 메서드를 사용하면 안됌
무조건 생성자로 초기화 해야함
속성값이 모두 같으면 같은 객체이다.

Dto는 getter/ setter 로직을 가지고 있을 수 있지만 VO는 getter 외에 로직을 가질 수 있다. 뭐 금액을 더하는 함수라던지 


### DAO란
데이터 사용기능 담당 클래스이다. DB 데이터 조회나, 수정 , 입력, 삭제 와 같은 로직을 처리하기 위해 사용된다.
crud의 기능을 한다.
DAOInterface/ DAOImplement로 구분지어 구현 분리하며 개발한다.
만약 Mybatis 연동 때처럼 interface만 필요한 경우 그냥 DAO 락조 명시할 수 있다.
말그대로 DB에 쉽게 접근하기 위해 만들어둔 객체이다.
Customer 2단계에 나오는 CrudREpository 확장하는 CustomerRepository를 의미한다.
개인적으로는 데이터 접근에서 가장 핵심적인 부분이라고 생각하지만 그것을 대시해주는 것이 ORM 이기 떄문에 우리는 손쉽게사용 하면 된다.
- DAO란 Data Access Object 의 약자이며 이름 그대로

  **DB의 data에 접근하는 객체 이다.** 비즈니스 로직과, DB Access 로직을 분리하기 위해 사용된다.

- Java Spring을 기준으로 DAO는 여러 가지의 모양새를 취하고 있는걸 알 수 있을것이다. 

  여러가지 모양새라 함은 DAO에 DB Connection이 설정되어 있는 경우, 아닌 경우로 나눌 수 있다. 

- 많이 사용하는 **MyBatis**의 경우 DB Connection 정보를 root-context.xml이라는 파일에 정의한다. 

- JPA 같은 경우는 application.yml(properties)에 설정하여 사용하는 경우가 있다. 

**1. DAO 클래스 예제 (Connection)**

- 코드가 상당히 복잡해 보이지만. 사실상 간단한 Create 코드이다.

- DB Connection 정보만 메서드로 따로 정의해서 사용한다면, SQL 관련만 정의해주면 된다.

```
package connectToDB;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

import javax.swing.JOptionPane;

public class TravelReservationTableDAO {

	// 예약정보 입력 & 예약 확정
	public int createCard(String reserveID, String reserveAdult, String reserveChild, String day) throws Exception {
																					//,String totalCoast
		// -------------- DB Connect --------------
        Class.forName("com.mysql.jdbc.Driver");
		
		String url = "jdbc:mysql://localhost:3306/TravelDB?useSSL=false";
		String username = "root";
		String password = "1234";
		Connection con = DriverManager.getConnection(url, username, password);

		// --------------  SQL 문 생성  --------------
		
		
		String sql = "insert into TravelReservationTable values (?, ?, ?, ?, ?)";    

		int totalCoast1 = Integer.parseInt(reserveAdult);
		int totalCoast2 = Integer.parseInt(reserveChild);
		int totalCoast3 = (totalCoast1 * 27900) + (totalCoast2 * 10900);		
		
		PreparedStatement ps = con.prepareStatement(sql);
		ps.setString(1, reserveID);
		ps.setString(2, reserveAdult);
		ps.setString(3, reserveChild);
		ps.setString(4, day);
		ps.setInt(5, totalCoast3);

		// -------------- DB 로 전송 --------------
		
 
	int result = ps.executeUpdate();
		return result;
		
		
	}//create end
```

**2. DAO 클래스 예제 (MyBatis)**

- MyBatis를 사용한다면 DB 연결 정보, SQL 쿼리문 등등 따로 정의되어있고 

   DAO에서는 어떤 정의되어있는 SQL을 사용할 건지 파라미터는 무엇인지 정의해주면 끝이다. 

```
@Repository
@RequiredArgsConstructor
public class CustMgmtDAO {

	private final SqlSessionTemplate db;
	
	/**
	 * 
	 * <pre>
	 * </pre>
	 * @Name    : 고객 로그인
	 * @Method  : logIn
	 * @Return  : void
	 * @Version : V1
	 */
	public CustMgmtDTO logIn(CustMgmtDTO inDTO) {
		
		CustMgmtDTO outDTO = db.selectOne("O_custMgmt.retrieveCustLogin", inDTO);
		return outDTO;
		
	}
```


### DTO란?
데이터 저장 담당 클래스이다. Controller, Service, View처럼 계증 간의 데이터 교환을 위해 쓰인다. 로직을 갖고 있지 않으며 순수한 데이터 객체이며 Getter, Setter 메소드만을 갖고 있다.

### VO란?
DTO 와 마찬가지로 데이터 저장 담당 클래스이다. DTO와는 다르게 VO는 값을 위해 쓰이는 객체로 Read-Only 속성을 갖고 있다.
그렇기 때문에 Getter 기능만을 포함하고 있다.

### BO(Business Object)
VO와 마찬가지로 데이터 저장 담당 클래스이다. 근데 비즈니스 로직을 포함하고 있다. 즉 비즈니스 관련 내용을 담은 VO라고 생각하면 될 듯 하다.