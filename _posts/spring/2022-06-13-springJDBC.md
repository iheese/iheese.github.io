---
layout: post
title: '[JAVA, SPRING] SPRING JDBC, TRANSACTION-MANAGER'
subtitle: 'JdbcTemplate, DataSource, DAO(Data Access Object), Transaction(트랜잭션)'
date: 2022-06-14 12:00:00 +0900
categories: [spring]
background: '/img/posts/etc/spring.jpg'
---

- 기존의 JDBC 프로그램(DAO)은 반복적이고 긴 코드로 인해 유지 보수에 어려움을 주었다. Spring은 Spring JDBC를 제공하여 문제 해결에 도움을 준다.

### Spring JDBC

![springjdbc](/img/posts/spring/springjdbc.png)


- 기존의 Plain JDBC의 기능을 Spring JDBC가 모두 제공해준다. 

#### Spring JDBC 사용 방법

dataSource.properties

```python
#DataSource Properties 
jdbc.driveClassName=org.h2.Driver
jdbc.url=jdbc:h2:tcp://localhost/~/test
jdbc.username=username
jdbc.password=password
```

- DB Server에 대한 property 설정
- dataSource.properties 을 통해 .xml 파일 변경 없이 DB 변경이 가능해진다.

.xml 파일

```xml
<beans    ...생략... >

	<!-- DataSource 등록 -->
	<context:property-placeholder location="classpath:datasource.properties"/>
	
	<!-- DataSource 설정 정보 입력 -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
	destroy-method="close">
		<property name="driverClassName" value="${jdbc.driveClassName}"/>
		<property name="url" value="${jdbc.url}"/>
		<property name="username" value="${jdbc.username}"/>
		<property name="password" value="${jdbc.password}"/>
	</bean>

	<!-- JdbcTemplate bean 등록-->
	<bean class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dataSource"/>
	</bean>
```

- Spring contatiner가 DataSource를 사용할 수 있게 등록해준다.
- datasource.properties 파일의 연결 정보를 `id="dataSource"` 인 bean에 연결해준다.
> - destory-method를 이용하여 bean 소멸 전 close 메소드를 실행해준다. 
- Spring JDBC 을 사용하기 위해 JdbcTemplate를 bean 등록해야 한다.
> - setter 주입을 통해 dataSource 객체를 주입해준다.  
> - DAO 클래스에서 @Autowired를 이용해 타입을 주입해줄 것이다. 

<br>

StudentDAOSpring.java

```java
@Repository //DAO 객체 등록
public class StudentDAOSpring implements StudentDAO{
	
	@Autowired //타입 주입
	private JdbcTemplate spring;
	
	// SQL 명령어
	private final String STUDENT_INSERT = "insert into student(num, sex, name, role) values((select nvl(max(seq), 0) + 1 from student), ?, ?, ?)";
	private final String STUDENT_UPDATE = "update student set name = ?, role = ? where num = ?";
	private final String STUDENT_DELETE = "delete student where num = ?";
	private final String STUDENT_LIST_N = "select * from student where name like '%'||?||'%' order by num desc";
	private final String STUDENT_LIST_R = "select * from student where role like '%'||?||'%' order by num desc";
	private final String STUDENT_GET  = "select * from student where num = ?";
	
	// CRUD 기능의 메소드
	// 학생 정보 등록
	public void insertStudent(StudentVO vo) {
		System.out.println("===> SPRING 기반으로 insertStudent() 기능 처리");
		spring.update(STUDENT_INSERT, vo.getSex(), vo.getName(), vo.getRole());
		
	}
	
	// 학생 정보 수정
	public void updateStudent(StudentVO vo) {
		System.out.println("===> SPRING 기반으로 updateStudent() 기능 처리");
		spring.update(STUDENT_UPDATE,vo.getName(),vo.getRole(),vo.getNum());
	}
	
	// 학생 정보 삭제
	public void deleteStudent(StudentVO vo) {
		System.out.println("===> SPRING 기반으로 deleteStudent() 기능 처리");
		spring.update(STUDENT_DELETE,vo.getNum());
	}
    
    // 학생 상세 조회
	public StudentVO getStudent(StudentVO vo) {
		System.out.println("===> SPRING 기반으로 getStudent() 기능 처리");
		return spring.queryForObject(STUDENT_GET, new StudentRowMapper(), vo.getNum());
	}
	
	// 학생 목록 검색
	public List<StudentVO> getStudentList(StudentVO vo) {
		System.out.println("===> SPRING 기반으로 getStudentList() 기능 처리");
		if(vo.getSearchCondition().equals("NAME")) {
			return spring.query(STUDENT_LIST_N, new StudentRowMapper(), vo.getSearchKeyword() );
		}else if(vo.getSearchCondition().equals("ROLE")) {
			return spring.query(STUDENT_LIST_R,new StudentRowMapper(), vo.getSearchCondition());
		}
		return null;
		}
}
    
```

- JdbcTemplate 객체의 update() 메소드로 sql문의 insert, update, delete 문 처리할 수 있다.
- JdbcTemplate 객체의 queryForObject() 메소드로 객체 1개를 조회할 수 있다.
- JdbcTemplate 객체의 query() 메소드로 객체 여러 개를 조회할 수 있다. 
- JdbcTemplate 객체에서 sql문의 select를 처리하기 위해서는 RowMapper() 객체가 필요하다. 

StudentRowMapper.java

```java
import java.sql.ResultSet;
import java.sql.SQLException;

import org.springframework.jdbc.core.RowMapper;

public class StudentRowMapper implements RowMapper<StudentVO> {

	@Override
	public StudentVO mapRow(ResultSet rs, int rowNum) throws SQLException {
		StudentVO student = new StudentVO();
		student.setNum(rs.getInt("NUM"));
		student.setSex(rs.getString("SEX"));
		student.setName(rs.getString("NAME"));
		student.setRole(rs.getString("ROLE"));
		student.setRegDate(rs.getDate("REGDATE"));
		return student;
	}
}
```

- RowMapper 인터페이스를 구현해야 한다. 
- 매개변수로 받는 ResultSet의 ROW를 어떤 자바객체(VO)에 매핑할 것인지를 기술한다.

<br>

### Transaction(트랜잭션)
- 분리될 수 없는 업무처리의 단위을 의미한다.
- Transaction의 중요성 예시
> - 은행에서 A가 B에게 100만원을 송금을 한다. -> A의 통장에서 100만원이 빠져나간다. -> 전기가 끊긴다. -> B는 100만원을 받지 못했다. -> A의  통장에서는 100만원이 빠져나갔다.

- 인출과 입금처럼 부분 작업이 실패하면 모든 작업을 실패시켜야 하고 모든 작업이 성공해야 성공하는 작업 단위를 Transaction이라 한다.
- Commit(커밋) : 모든 부분 작업이 성공하면 한번에 DB에 반영하는 것을 의미한다.
- Rollback(롤백) : 부분 작업이 실패하면 Transaction 실행 전으로 되돌리는 것을 의미한다. 
- Spring은 Transaction 관리자를 지원한다.

<br>
### Spring의 Transaction(트랜잭션) 

- .xml 파일의 Namespaces-`tx` 체크 (Eclipse 기준)

```xml
<!-- Transaction -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>
```

- Transaction 매니저 bean 등록하고 setter 방식으로 dataSource 객체를 주입해준다.
- Transaction은 AOP의 개념을 이용한다.
> - [ 참고!! Spring AOP](https://ddungi.github.io/java/2022/06/09/springAOP/)

- Advice 지정

```xml
<!-- 모든 메소드에서 예외 발생하면 롤백되며, 예외가 발생하지 않으면 자동으로 커밋된다.-->
	<tx:advice id="txAdvice" transaction-manager="txManager">
		<tx:attributes>
			<tx:method name="*" rollback-for="Exception"/>
		</tx:attributes>
	</tx:advice>
```

- `<tx:attributes>`
> - `name` :  Transaction이 적용될 메소드 이름 지정 ( Ex: `*Service` : 이름이 Service로 끝나는 메소드)
> - `read-only` : 읽기 전용 여부를 지정 (default값 : false)
> - `no-rollback-for` : Transaction을 롤백하지 않을 예외 지정
> - `rollback-for` : Transaction을 롤백할 예외 지정

```xml
<!-- pointcut 지정 -->
	<aop:config>
		<aop:pointcut id="txPointcut" expression="execution(* com.naver.biz..*Service.*(..))"/>
		<aop:advisor pointcut-ref="txPointcut" advice-ref="txAdvice"/>
	</aop:config>
```

- aspect는 method 이름을 지정해주어야 한다.
- Transaction에서만 advisor 사용하여 txPointcut과 txAdvice를 연결해준다. 

<br>

Reference:
- [inflean_img](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC_renew)
- [[Spring JDBC] Spring JDBC를 이용한 데이터 접근 방법_gmlwjd9405](https://gmlwjd9405.github.io/2018/05/15/setting-for-db-programming.html)
- [java-jdbc_ahea](https://ahea.wordpress.com/2017/02/20/java-jdbc/)
- [트랜잭션(Transaction)이란?_ mommoo](https://mommoo.tistory.com/62)
- [[Spring 레퍼런스] 11장 트랜잭션 관리 #2_ Outsider's Dev Story](https://blog.outsider.ne.kr/870)