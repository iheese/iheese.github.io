---
layout: post
title: '[JAVA, SPRING] SPRING MYBATIS'
subtitle: 'SqlSessionTemplate, MapperFactoryBean'
date: 2022-06-29 12:00:00 +0900
categories: 'JAVA'
background: '/img/posts/etc/spring.jpg'
---

## MyBatis

- SQL 명령어를 외부의 XML 파일이나 Annotation으로 분리하고 DB 연동 처리를 쉽게 도와주는 프레임워크이다.
-  SqlSessionTemplate과 MapperFactoryBean을 이용하는 2가지 방법을 알아볼 것이다. 

<br>

##  MyBatis 사용법
- maven 사용하지 않는다면 lib 폴더 아래 mybatis-x.x.x.jar,  mybatis-spring-x.x.x.jar 파일을 넣어주면 된다. 
- maven 사용시, pom.xml에 아래 의존성을 추가해주면 된다.

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>

<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>x.x.x</version>
</dependency>
```

- mybatis-spring은 MyBatis와 Spring의 연동을 위한 태그이다. 

<br>

![mybatis](/img/posts/spring/mybatis.jpg)

<br>

### SqlSessionTemplate을 이용한 방법

business-layer.xml(root-context.xml) 설정

```xml
<!--생략-->

<!-- DataSource -->
	<context:property-placeholder location="classpath:datasource.properties"/>
	
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
	destroy-method="close">
		<property name="driverClassName" value="${jdbc.driverClassName}"/>
		<property name="url" value="${jdbc.url}"/>
		<property name="username" value="${jdbc.username}"/>
		<property name="password" value="${jdbc.password}"/>
	</bean>
	
	<!-- Spring과 MyBATIS 연동 설정 -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="configLocation" value="classpath:sql-map-config.xml"/>
		<property name="dataSource" ref="dataSource"/>
	</bean>
	
	<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg ref="sqlSessionFactory"/>
	</bean>

<!--생략-->
```
- SqlSessionTemplate은 MyBatis 쿼리문을 수행해주는 역할을 한다. (MyBatis와 Spring 연동 모듈의 핵심)
- setter injection으로 SqlSessionFactoryBean에 dataSource, sql-map-config 설정 파일을 주입해준다. 
- SqlSessionFactoryBean을 생성자 주입하여 SqlSessionTemplate bean을 생성한다. 

<br>

sql-map-config.xml 설정

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<typeAliases>
		<typeAlias type="com.school.service.student.studentVO" alias="student"/>
	</typeAliases>
	
	<mappers>
		<mapper resource="mappings/student-mapping.xml"/>
	</mappers>
	
</configuration>
```

- `<typeAliases>` 을 이용해 객체 별칭을 정하여 사용할 수 있다.
- `<mappers>`을 통해 mapping 정보가 담긴 xml파일을 알려준다.

<br>

mappings/student-mapping.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="StudentDAO">
	<resultMap id="student" type="student">
        <id property="seq" column="SEQ" />
        <result property="name" column="NAME" />
        <result property="sex" column="SEX" />
        <result property="role" column="ROLE" />
    </resultMap>

	<insert id="insertStudent" >
		 insert into student(seq, name, sex, role) values(#{seq}, #{name}, #{sex}, #{role})
	</insert>
	
	<select id="getStudent" resultMap="student">
		<![CDATA[
		select * from student where id = #{seq}
		]]>
	</select>
	
	<select id="getStudentList" resultMap="student">
		select * from student order by seq desc
	</select>
</mapper>

```

- `<mapper id="">, <insert id="">` id를 통해 mapper의 sql문을 호출할 수 있다. 
- VO 속성명과 컬럼명이 다르면 위의 resultMap 태그로 정해줄 수 있다.
- sql문 마다 태그가 존재한다.(`<insert>, <update>, <delete>, <select>`)
- `<, > ,&` 을 사용한 sql문을 사용하려면 `<![CDATA[ ]]>` 로 감싸주면 된다.
- resultMap 선언시 참조한 이름으로 select 태그의 resultMap으로 리턴값을 정해줄 수 있다. 
- resultType으로 리턴 타입을 정할 수 있는데 mapping하려는 클래스 경로, alias명, 타입명(ex: int, String 등)을 적어주면 된다.
- sql문 내 입력될 파라미터 값은 `#{}` 안에 적어주면된다. (JDBC의 ? 의 역할이다. )

<br>

StudentDAO.java

```java
@Repository
public class StudentDAOMybatis implements StudentDAO{
		
	@Autowired
	private SqlSessionTemplate sqlSession;

	// 학생 정보 등록
	public void insertStudent(StudentVO vo) {
		sqlSession.insert("StudentDAO.insertStudent",vo);
	}
	
	// 학생 상세 조회
	public StudentVO getStudent(StudentVO vo) {
		return (StudentVO) sqlSession.selectOne("StudentDAO.getStudent",vo);
	}
	
	// 학생 목록 검색
	public List<StudentVO> getStudentList(StudentVO vo) {
		return sqlSession.selectList("StudentDAO.getStudentList", vo);
	}
}
```

- SqlSessionTemplate의 insert(), delete(), update() 메소드가 각각 존재하여 sql문에 맞게 사용하면 된다. 
- selectOne()은 객체 하나를 조회할 때 사용한다.
- selectList()은 객체 여러 개를 조회할 때 사용한다. 

<br>

### MapperFactoryBean을 이용한 방법

- MapperFactoryBean 방법은 DAO interface에 직접 이용하여 구현체를 만들지 않아도 사용할 수 있는 방법이다.


business-layer.xml(root-context.xml) 설정

```xml
 	<!-- DataSource -->
	<context:property-placeholder
		location="classpath:datasource.properties" />
	
	<bean id="dataSource"
		class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName"
			value="${jdbc.driverClassName}" />
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
	</bean>
    
	<!-- Spring과 MyBATIS 연동 설정 -->
	<bean id="sqlSessionFactory"
		class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="configLocation"
			value="classpath:sql-map-config.xml" />
		<property name="dataSource" ref="dataSource" />
	</bean>

	<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg ref="sqlSessionFactory" />
	</bean>
    <!--위에까지는 일치한다.-->
    
    <bean id="StudentDAO"
		class="org.mybatis.spring.mapper.MapperFactoryBean">
		<property name="mapperInterface"
			value="com.school.service.student.studentDAO" />
	    <property name="sqlSessionTemplate" ref="sqlSessionTemplate"/>
	</bean> 
```

- MapperFactoryBean에 sqlSessionTemplate과 DAO interface를 setter 주입해준다.

<br>

StudentDAO.java

```java
public interface StudentDAO {
	
	// 학생 등록
	void insertStudent(studentVO vo);

	// 학생 상세 조회
	studentVO getStudent(studentVO vo);

	// 학생 목록 검색
	List<studentVO> getStudentList(studentVO vo);
}
```

<br>

sql-map-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<typeAliases>
		<typeAlias type="com.school.service.student.studentVO" alias="student"/>
	</typeAliases>
	
	<mappers>
		<mapper resource="mappings/student-mapping(mapper).xml"/>
	</mappers>
	
</configuration>
```

<br>

mappings/student-mapping(mapper).xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="StudentDAO">
	<resultMap id="student" type="student">
        <id property="seq" column="SEQ" />
        <result property="name" column="NAME" />
        <result property="sex" column="SEX" />
        <result property="role" column="ROLE" />
    </resultMap>

	<insert id="insertStudent" >
		 insert into student(seq, name, sex, role) values(#{seq}, #{name}, #{sex}, #{role})
	</insert>
	
	<select id="getStudent" resultMap="student">
		<![CDATA[
		select * from student where id = #{seq}
		]]>
	</select>
	
	<select id="getStudentList" resultMap="student">
		select * from student order by seq desc
	</select>
</mapper>

```

- 앞 내용과 거의 비슷하다.

<br>

StudentServiceImpl.java

```java
<!--생략-->

@Service
public class StudentServiceImpl implements StudentService {

	@Autowired 
	private StudentDAO studentDAO;

	public void insertStudent(StudentVO vo) {
		studentDAO.insertStudent(vo);
	}

	public StudentVO getStudent(StudentVO vo) {
		return studentDAO.getStudent(vo);
	}

	public List<StudentVO> getStudentList(StudentVO vo) {
		return studentDAO.getStudentList(vo);
	}
}
```

- DAO를 구현할 때 사용하는 것이 아닌 Service구현체에서 바로 사용할 수 있다. 
- DAO interface의 메소드명과 mapping 파일의 sql문의 id가 일치해야한다. 
- business-layer.xml의 MapperFactoryBean id와 @Autowired 되는 DAO의 타입이 일치해야 한다.  

<br>

#### 마무리

- SqlSessionTemplate
> - DAO interface의 구현체가 여러 개 필요할 때 더 효율적인 방법일 것 같다.

- MapperFactoryBean 
> -  DAO interface를 직접 이용한다는 점이 유지 보수 측면에서 효율적이지 않지만 DAO 구현체가 1개보다 많이 필요하지 않은 로직에서는 좋은 방법일 것 같다. 
> - Java 소스 내 MyBatis에 의존적이지 않다. 

<br>

Reference:
- [[MyBatis] MyBatis란? 개념 및 데이터구조, img_ 히진쓰의 서버사이드 기술 블로그](https://khj93.tistory.com/entry/MyBatis-MyBatis%EB%9E%80-%EA%B0%9C%EB%85%90-%EB%B0%8F-%ED%95%B5%EC%8B%AC-%EC%A0%95%EB%A6%AC)
- [mybatis-spring](https://mybatis.org/spring/ko/sqlsession.html)
- [mybatis-spring, Mapper](https://mybatis.org/spring/ko/mappers.html)
- [[Spring] MyBatis 사용하기, MapperFactoryBean (게시판 구현)
_Bigfat](https://bigfat.tistory.com/93?category=611606)

