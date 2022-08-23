---
layout: post
title: '[JAVA] DTO, ENTITY와 DTO 변환'
subtitle: 'DTO, Entity, 생성자, 메소드, ModelMapper, MapStruct'
date: 2022-08-23 12:00:00 +0900
categories: 'JAVA'
background: '/img/posts/etc/datastructure.jpg'
---

- 모든 실습은 Eclipse Spring Tool Suite 4 에서 진행되었습니다.

<br>

## DTO(Data Transfer Object)
- 데이터를 전달하기 위해 사용하는 객체를 의미한다.
- 요청이나 응답값을 전달하는 클래스이다.
- getter/setter 만을 가지며 다른 로직은 가지고 있지 않다. 
- 필드를 final로 선언하고 setter를 없애 생성자로 객체를 생성하게 하면 불변 객체로 만들어 안정성을 보장할 수 있다. 

<br>

## Entity
- DB와 매핑된 핵심 클래스이며 Entity 클래스를 기준으로 테이블이 생성되고 스키마가 변경된다.
- setter 사용을 지양하여 데이터의 일관성을 보장하면 좋다.

<br>

## DTO와 Entity를 구분하는 이유
- Entity는 정말 중요한 클래스이다. 수많은 클래스와 로직이 Entity를 기준으로 동작한다. Entity가 자주 변경되는 것은 많은 클래스와 로직에도 영향을 미치게 되는 것을 의미한다. View에서 사용되는 데이터는 많은 변경이 이루어지기 때문에 Entity는 적합하지 않아 DTO를 사용하는 것이다. 
- Entity가 직접 View 데이터를 구성하다보면 민감한 정보도 함께 움직이기 때문에 보안성을 떨어지기도 할 가능성이 있다.  

<br>

## Entity, DTO, 변환 실습
- Entity와 DTO 변환 위치에 대한 많은 이야기가 있으나 필자는 서비스단에서 처리하는 것이 가장 낫다고 판단하였다.


Member.java

```java
@Entity
@Data
public class Member {
    private String memberId;
    private String name;
    private int age;
    private String role;
}
```

MemberDto.java

```java
@Data
public class MemeberDto {
    private String memberId;
    private String name;
    private int age;
    private String role;
}
```

<br>

### 생성자 방식으로 Entity to DTO

MemberDto.java

```java
@Data
public class MemeberDto {
    private String memberId;
    private String name;
    private int age;
    private String role;

	public MemberDto(Member member){
    	this.memberId = member.getMemberId();
        this.name = member.getName();
        this.age = member.getAge();
        this.role = member.getRole();
    }
}
```

<br>

### 변환 메소드 생성으로 DTO to Entity

MemberDto.java

```java
@Data
public class MemeberDto {
    private String memberId;
    private String name;
    private int age;
    private String role;
    
    //builder 방식
    public Member toEntity1(){
    	return Member.builder()
        			 .memberId(memberId)
                     .name(name)
                     .role(role)
                     .build();
    }
    
    //기본 생성자와 setter이용
    public Member toEntity2(){
    	Member member = new Member();
        
        member.setMemberId(this.memberId);
        member.setName(this.name);
        member.setAge(this.age);
        member.setRole(this.role);
        
        return member;
    }
}
```

MemberService.java

```java
@RequiredArgsConstructor
@Service
public class MemberService {
	
    //생성자 주입
	private final MemberRepository memberRepository;

	//DTO로 받아서 Entity로 변환 후 저장
    @Transactional
    public void insertMemeber(MemberDto memberDto){
    	// 두 가지 방법 존재
 	   memberRepository.save(memberDto.toEntity1());
   	 //memberRepository.save(memberDto.toEntity2());
    }
}
```


<br>

### ModelMapper 라이브러리를 이용한 변환
- 런타입 시점에 Reflection API를 이용해 비용이 큰 편이다.
- 일반적으로 Setter를 이용한다.  


maven 이용 시 의존성 추가

```pom
		<dependency>
			<groupId>org.modelmapper</groupId>
			<artifactId>modelmapper</artifactId>
			<version>2.4.0</version>
		</dependency>
```

gradle 이용 시 의존성 추가

```gradle
implementation group: 'org.modelmapper', name: 'modelmapper', version: '2.4.0'
```

ApplicationConfig.java
- 설정 파일에 ModelMapper 빈을 등록해주었다.

```java
@Configuration
public class ApplicationConfig {
	
	@Bean
	ModelMapper modelMapper() {
		return new ModelMapper();
	}
}
```

MemberService.java

```java
@RequiredArgsConstructor
@Service
public class MemberService {
	
    //생성자 주입
	private final MemberRepository memberRepository;
    private final ModelMapper modelMapper;

	//DTO로 받아서 Entity로 변환 후 저장
    @Transactional
    public void insertMemeber(MemberDto memberDto){
    
    Member member = modelMapper.map(memberDto, Member.class);
    memberRepository.save(member);
    }
	
    //DB로부터 Entity 객체를 받아 DTO로 변환 후 전달
    @Transactional(readOnly=true)
	public MemberDto getMember(int memberId){
		Member member = memberRepository.findById(memberId).orElseGet(Member::new);
        MemberDto memberDto = modelMapper.map(member, MemberDto.class);
		return memberDto;
	}
	
    //DB로부터 객체 리스트 받아 DTO 리스트로 변환 후 전달
	@Transactional(readOnly=true)
	public List<MemberDto> getMemberList(){
		List<Member> memberList =memberRepository.findAll();
		List<MemberDto> memberDtoList = memberList.stream().map(member -> modelMapper.map(member, MemberDto.class)).collect(Collectors.toList());
		return memberDtoList;
	}
}
```

<BR>

### MapStruct 라이브러리를 이용한 변환
- 컴파일 시점에 매핑 정보를 생성하고 객체를 매핑한다.
- Annotation processor를 이용해 인자 값과 리턴될 객체에 필요한 메소드를 호출하여 자동으로 객체 간 매핑을 제공한다. 

maven 이용 시 의존성 추가

```pom
<properties>
  <org.mapstruct.version>1.4.2.Final</org.mapstruct.version>
  <org.projectlombok.version>1.18.24</org.projectlombok.version>
</properties>
		....
		<dependency>
			<groupId>org.mapstruct</groupId>
			<artifactId>mapstruct</artifactId>
			<version>${org.mapstruct.version}</version>
		</dependency>
        <dependency>
   		 <groupId>org.projectlombok</groupId>
    		<artifactId>lombok</artifactId>
    		<version>${org.projectlombok.version}</version>
    		<scope>provided</scope>
  	  </dependency>
      ....
```


```pom
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
      <source>1.8</source>
      <target>1.8</target>
 	 <annotationProcessorPaths>
        <path>
          <groupId>org.mapstruct</groupId>
          <artifactId>mapstruct-processor</artifactId>
          <version>${org.mapstruct.version}</version>
        </path>
        <path>
          <groupId>org.projectlombok</groupId>
          <artifactId>lombok</artifactId>
          <version>${org.projectlombok.version}</version>
        </path>
      </annotationProcessorPaths>
</plugin>
```

- MapStruct는 lombok과 충돌 이슈가 있어 위와 같은 annotationProcessorPaths 설정을 해주어야 한다.
> - Lombok annotation processor가 getter, builder를 만들기 전에 mapStruct annotation processor가 동작하여 매핑할 방법을 찾지 못해 오류가 발생한다고 한다.
> - [MapStruct에서 제시해주는 lombok과 함께 의존성 추가 예시](https://github.com/mapstruct/mapstruct-examples/blob/main/mapstruct-lombok/pom.xml)

gradle 이용 시 의존성 추가

```gradle
implementation "org.mapstruct:mapstruct:1.4.2.Final"
annotationProcessor "org.mapstruct:mapstruct-processor:1.4.2.Final"
```

MemberMapper.java

```java
@Mapper
public interface MemberMapper {
	MemberMapper INSTANCE = Mappers.getMapper(MemberMapper.class);

	//@Mapping(source = "member.필드명", target = "바뀐 필드명")
	MemberDto toDto(Member member); //Entity to DTO
	
	Member toEntity(MemberDto memberDto); //DTO to Entity
 }
```

- @Mapping 어노테이션은 변환 전 필드명과 변환 후 필드 명이 다를 때 사용한다.


MemberService.java

```java
@RequiredArgsConstructor
@Service
public class MemberService {
	
    //생성자 주입
	private final MemberRepository memberRepository;
    //Mapper 변수 객체 생성
	MemberMapper memberMapper = MemberMapper.INSTANCE;

	//DTO로 받아서 Entity로 변환 후 저장
    @Transactional
    public void insertMemeber(MemberDto memberDto){
    
    Member member = memberMapper.toEntity(memberDto);
    memberRepository.save(member);
    }
	
    //DB로부터 Entity 객체를 받아 DTO로 변환 후 전달
    @Transactional(readOnly=true)
	public MemberDto getMember(int memberId){
		Member member = memberRepository.findById(memberId).orElseGet(Member::new);
        MemberDto memberDto =memberMapper.toDto(member);
		return memberDto;
	}
	
    //DB로부터 객체 리스트 받아 DTO 리스트로 변환 후 전달
	@Transactional(readOnly=true)
	public List<MemberDto> getMemberList(){
		List<Member> memberList =memberRepository.findAll();
		List<MemberDto> memberDtoList = memberList.stream().map(member -> memberMapper.toDto(member)).collect(Collectors.toList());
		return memberDtoList;
	}
}
```

<BR>


Reference:
- [2) 스프링부트로 웹 서비스 출시하기 - 2. SpringBoot & JPA로 간단 API 만들기_기억보단 기록을](https://jojoldu.tistory.com/251)
- [Convert Entity To DTO In Spring REST API_amitph](https://www.amitph.com/spring-entity-to-dto/)
- [Quick Guide to MapStruct_Baeldung](https://www.baeldung.com/mapstruct)
- [Spring MapStruct Getting Started - 1_호식이](https://blog.naver.com/PostView.nhn?blogId=gngh0101&logNo=221992848918&redirect=Dlog&widgetTypeCall=true&directAccess=false)
- [객체 변환하기. 자바 코드 매핑 vs MapStruct vs ModelMapper?
](https://lob-dev.tistory.com/entry/%EA%B0%9D%EC%B2%B4-%EB%B3%80%ED%99%98%ED%95%98%EA%B8%B0-%EC%9E%90%EB%B0%94-%EC%BD%94%EB%93%9C-%EB%A7%A4%ED%95%91-vs-MapStruct-vs-ModelMapper)
