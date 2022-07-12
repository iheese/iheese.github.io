---
layout: post
title: '[JAVA, SPRINGBOOT] JPA(Java Persistence API)'
subtitle: 'ORM, @Table, @Entity, @Id, @Repository'
date: 2022-07-12 12:00:00 +0900
categories: 'JAVA'
background: '/img/posts/etc/spring.jpg'
---

- 모든 실습은 Eclipse Spring Tool Suite 4 에서 진행되었습니다.

<br>

### ORM(Object Relational Mapping)
- 객체 관계 매핑을 의미한다.
- 관계형 데이터베이스와 Java 객체를 연결해주는 역할을 한다.

### JPA(Java Persistence API)

![jpa](/img/posts/spring/jpa.png)

- Java의 ORM의 기술 표준을 JPA라 한다. 
- 인터페이스로 구성되어 있으며, 인터페이스의 구현체가 Hibernate, EcipseLink, DataNucleus이다. 
- 이 포스트에서는 Hibernate를 사용할 것이고, Spring Boot는 Hibernate를 기본 JPA 구현체로 사용한다.

<br>

### JPA 설정(application.yml)
- 빌드툴이 Maven이라면 pom.xml에 아래와 같은 의존성을 추가해준다. 
- Spring Starter Project를 이용해 프로젝트를 만든다면 Spring Data JPA를 추가하면 된다.
- 버전은 각 상황에 맞게 달리 적용해주면 된다.

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
            <version>2.7.1</version>
		</dependency>
```

- 빌드툴이 Gradle이라면 build.gradle에 아래와 같은 의존성을 추가해준다.

```xml
implementation group: 'org.springframework.boot', name: 'spring-boot-starter-data-jpa', version: '2.7.1'

```

<br>

#### application.yml 설정
```yml
# datasource 설정
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:tcp://localhost/~/test
    username: username
    password: password

# JPA 설정
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
    show-sql: true  
    hibernate:
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl 
      ddl-auto: update
      use-new-id-generator-mappings: false
    properties[hibernate.format_sql]: true #포맷 유지
```

- spring.datasource : Datasource 연결 정보
- spring.jpa.database-platform :  DB 플랫폼 
> - [참고! DB마다 다른 플랫폼을 확인하여 넣어줘야 한다.](https://docs.jboss.org/hibernate/core/3.6/reference/en-US/html/session-configuration.html#configuration-optional-dialects)
> - JPA가 DB에 해당하는 SQL문을 만들어준다.
- JPA 테이블 컬럼 이름 생성 전략(spring.jpa.hibernate.naming)
> - implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyJpaCompliantImpl : 소문자, 언더바(_) 사용해서 컬럼명 생성, default값 <br>  Ex) userName -> user_name
> - physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl : 물리적으로 객체 VO에 적은 대로 적용됨
- spring.jpa.hibernate.ddl-auto 
> - none : 테이블 생성하지 않음
> - update : 테이블 내용 변경될 경우 변경 내용 반영
> - create : 기존 테이블 삭제 후 재생성
> - create-drop : 기존 테이블 삭제하고 프로그램 종료시 테이블 삭제
> - validate : Entity와 테이블이 정상 매핑되었는지 확인
- spring.jpa.hibernate.use-new-id-generator-mappings : Hibernate의 id 생성 전략을 따라갈지에 대한 선택
> - Hibernate v5부터 default값은 true이다.(v5 이전은 false가 기본값)
> - true일 때, DB가 Hibernate의 id 생성 전략을 따라간다. (AUTO)
> > - MYSQL, H2 DB는 TABLE 전략을 따라간다.
> - false일 때 DB(Dialect)에 의해 결정된다.
> > - H2는 그래도 TABLE 전략을 선택한다.
> > - MYSQL은 IDENTITY를 선택한다.
> - `@GeneratedValue(strategy=GenerationType.INDENTITY)` 를 각 엔티티 PK에 적용하는 것이 좋을 것 같다. (yml 파일보다 우선순위가 높다.)
> - PK 값 식별 생성 전략
> > - TABLE : 별도의 키 생성 테이블 사용하여 PK값 생성
> > - SEQUENCE : SEQUENCE를 이용해 PK값 생성
> > - IDENTITY : auto-increment, IDENTITY를 이용하여 PK값 생성
> > - AUTO :  Hibernate가 자동으로 정해준다.(default)
- spring.jpa.properties[hibernate.format_sql] : 콘솔창에 테이블 생성 포맷 유지(가독성을 위해)

<br>
### @Repository Method

Student.java

```java
@Data //lombok
@Table(name="STUDENT") //생성할 테이블 이름 지정
@Entity // DB에 테이블 생성
public class Student {
	@Id // 식별자 변수 (Primary Key) 선언
	// 1부터 시작하여 자동으로 1씩 값이  증가하도록 설정한다.
	@GeneratedValue(strategy=GenerationType.INDENTITY)
	private int seq; // 학생 일련 번호
	
	@Column(nullable=false, length=50)
	private String name; // 이름
	@Column(nullable=false, length=10)
	private String sex; // 성별
	@Column(nullable=false,length=100)
	private String role; // 역할
	
	@CreationTimestamp //현재 시간 정보(자동 저장) 
	private Timestamp createDate;
}
```

<br>

StudentRepository.java

```java
@Repository
public interface StudentRepository extends JpaRepository<Student, Integer>{
        // findBy컬럼명
        // 예시 : name 컬럼으로 객체를 찾는 메소드
		Optional<Student> findByName(String name);
```

- JPQL(Java Persistence Query Language)을 메소드로 쉽게 처리할 수 있다.
- 쿼리 메소드 : find + 엔티티이름(생략가능)+ By + 변수명
- And, Or, Between, LessThan, GreaterThan 등등 많은 유형이 있다.

<br>

StudentService.java

```java
@Service
public class StudentService {

	@Autowired
	private StudentRepository studentRepository;
	
    //학생 정보 입력
	@Transactional
	public void insertStudent(Student student) {
		studentRepository.save(student);
	}
	//학생 정보 상세 조회
	@Transactional
	public Student getStudent(int seq) {	
		Optional<Student> student = studentRepository.findById(seq);
		if(student.isPresent()) {
			return student.get();
		}
		return new Student();		
	}
    //학생 정보 수정
    //updateStudent 구현 첫번째
    public String updateStudent(Student student) {
		Optional<Student> findEntity= studentRepository.findById(student.getSeq());
		if(findEntity.isPresent()) {
		Student findStudent = findEntity.get();
		findStudent.setName(student.getName());
		findStudent.setSex(student.getSex());
		findStudent.setRole(student.getRole());
		studentRepository.save(findStudent);
		return "학생 정보 수정 성공";
	} else {
		return student.getSeq()+" 번 학생 정보는 없습니다.";
	}
}
    //updateStudent 구현 두번쨰(@Transactional 차이)
    @Transactional
    public String updateStudent(Student student) {
		Optional<Student> findEntity= studentRepository.findById(student.getSeq());
		if(findEntity.isPresent()) {
			Student findStudent = findEntity.get();
			findStudent.setName(student.getName());
			findStudent.setSex(student.getSex());
			findStudent.setRole(student.getRole());
		return "학생 정보 수정 성공";
	} else {
		return student.getSeq()+" 번 학생 정보는 없습니다.";
	}
}
	//학생 정보 삭제
	public void deleteStudent(int seq) {
		studentRepository.deleteById(seq);
	}
    
    //학생 정보 목록 조회
	public List<Student> getStudentList() {
		return studentRepository.findAll();
	}
```

- @Transactional : 로직 진행 중 예외 없이 성공하면 커밋한다.

<br>

StudentController.java

```java
@Controller
public class StudentController {
	
	@Autowired
	private StudentService studentService;
	
    //학생 정보 상세 조회
    @GetMapping("/student/{seq}") 
	public Student getStudent(@PathVariable int seq) { 
		return studentService.getStudent(seq);
	}
    
    //학생 정보 입력
	@PostMapping("/student")
    @ResponseBody
	public String insertStudent(@RequestBody Student student) {
	 Student findStudent = studentService.getStudent(student.getSeq());
		
		if(findStudent.getName()==null) {
			studentService.insertStudent(student);
			return student.getName() + " 학생 정보 등록 성공";
		}else {
			return student.getName() + "학생 정보는 존재합니다.";
		}
	}
        
    //학생 정보 수정
    @PutMapping("/")
    public String updateStudent(@RequsetBody Student student){
  		return studentService.updateStudent(student);
 	}    
    
    //학생 정보 삭제
    @ResponseBody
    @DeleteMapping("/student/{seq}")
    public String deleteList(@PathVariable int seq){
    	 studentService.deleteStudent(seq);
         return "학생 정보 삭제 성공";
    }

    //학생 정보 목록 조회
    @GetMapping("/student/list")
    public List<Student> getStudentList(){
    	return studentService.getStudentList();
    }
    
}

```

<br>
Reference:
- [inflean_img](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC_renew)
- [ JPA 소개(2) img _ jsj3282](https://velog.io/@jsj3282/2.-JPA-%EC%86%8C%EA%B0%9C2)
- [ 2.2 Naming strategies _ HIBERNATE](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#naming)
- [Spring Boot Data JPA 2.0 에서 id Auto_increment 문제 해결_기억보다 기록을](https://jojoldu.tistory.com/295)
- [How do Identity, Sequence, and Table (sequence-like) generators work in JPA and Hibernate](https://vladmihalcea.com/hibernate-identity-sequence-and-table-sequence-generator/)
