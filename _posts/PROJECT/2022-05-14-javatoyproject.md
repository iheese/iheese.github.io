---
layout: post
title: "JAVA TOY PROJECT 회고록"
subtitle: "JDBC, DAO, SQL, H2"
date: 2022-05-14 11:00:00 +0900
categories: "PROJECT"
background: "/img/posts/javaetc/JDBC.jpg"
---

### 프로젝트 설명

- Java를 이용한 회원 관리 프로그램입니다.
- 개인 프로젝트로 진행하였습니다. 쿼리를 직접 작성하고 JDBC를 직접 구현해보았습니다. 
- ##### [ 어떤 프로젝트인지 자세한 내용 + 소스 코드 클릭!! ](https://github.com/iheese/JavaToyProject)

<br>
## 수정할 점

- DAO 의 회원 상세 조회 부분에 아쉬움이 있다.

MemberDAO.java

```java
//회원 상세 조회(아이디 입력값으로 존재 유무 확인)
//내 코드
		private String MEMBER_SEARCH="select member_id from member";
		public boolean searchMemberId(String inputId) {
			List<Member> memberIdList = new ArrayList<>();
			try {
				conn=JDBCUtil.getConnection();
				stmt=conn.prepareStatement(MEMBER_SEARCH);
				rs=stmt.executeQuery();
				
				while(rs.next()) {
					Member member = new Member();
					member.setMemberId(rs.getString("MEMBER_ID"));	
					memberIdList.add(member);
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}finally {
				JDBCUtil.close(rs, stmt, conn);
			}	
			
			for(Member m :memberIdList){
				if(m.getMemberId().equals(inputId)){
					return true;
				}	
			}
		 return false;
		}
```

- 내가 생각한 회원 상세 조회 기능은 DB 내 멤버 아이디를 모두 조회해서 어떤 inputId 값이 들어오면 그 값의 존재 여부를 boolean으로 리턴하는 함수로 만들었다.
- 회원 등록(insertMember), 회원 삭제(deleteMember) 시 회원 존재 여부를 판단하는 함수로 이용했다. 
- 위 함수는 뚜렷한 목적성이 좋은 함수지만 다양한 부분에 활용하기에는 어려운 함수라는 생각이 들었다. 

<br>
- 회원 상세 조회 DAO 부분을 수정한 것이다.

```java
	private String MEMBER_GET="select member_id from member";
	public Member getMember(Member member) {
		Member member =null;
		try {
			conn = JDBCUtil.getConnection();
			stmt = conn.prepareStatement(MEMBER_GET);
			stmt.setString(1, member.getMemberId());
			rs = stmt.executeQuery();
			if(rs.next()) {
				member = new Member();
				member.setMemberId(rs.getString("MEMBER_ID"));
				member.setName(rs.getString("NAME"));
				member.setPhoneNumber(rs.getString("PHONE_NUMBER"));
			}
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			JDBCUtil.close(rs, stmt, conn);
		}
		return member;
	}
```

- boolean이 아닌 member 객체를 리턴하는 함수로 만들었다.
- 조회하고자 하는 member 객체를 입력받으면 DB에 객체가 존재하면 해당 객체를 반환하고 객체가 없다면 null을 반환한다.
- 위처럼 구현하면 member의 아이디만 비교하는 전의 함수와 다르게 memberId, name, phoneNumber 의 중복여부도 확인할 수 있게 된다.
- 또한 객체의 멤버 변수가 늘어나도 조금의 수정을 통해 함수의 목적을 유지할 수 있게 된다. 
- 함수의 활용도, 유지보수 등에 대해 더 생각하며 구현할 필요가 있을 것 같다. 

<br>

### 어려웠던 점 및 더 필요한 점
- 유효성 검증을 구현하는 로직을 구성하는데 어려움이 있었다.
> - 원하는 로직을 구현하기 위해 더 많은 코딩 경험과 다양한 알고리즘 문제 풀이들을 학습할 필요가 있을 것 같다.
- JDBC를 통해 DB에 접근하여 DAO를 구현하는데 어려움이 있었다.
> - JDBC, DB에 대한 이해와 반복적인 학습이 필요할 것 같다. 

<br>

Reference:
- 위 프로젝트는 핀테크 서비스 개발자 양성 과정_fastCampus 과정으로 학습하며 진행했습니다.
