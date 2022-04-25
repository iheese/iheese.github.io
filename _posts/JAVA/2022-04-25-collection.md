---
layout: post
title: "COLLECTION"
subtitle: "ArrayList, HashSet, HashMap 중심으로"
date: 2022-04-25 12:00:00 +0900
categories: "JAVA"
background: "/img/posts/javaetc/java.png"
---


![collection](/img/posts/javaetc/collection.png)

<br>

## ArrayList

```java
import java.util.ArrayList;
import java.util.List;

public class ArrayListTest {
	public static void main(String[] args) {
		ArrayList noTypeList = new ArrayList(); // 타입없는 리스트 생성
		ArrayList<Person> personList1 = new ArrayList<Person>(); //Person 타입 리스트 생성
        List<Person> personList2 = new ArrayList<Person>();
		// 묵시적 형 변환으로 생성(유지, 보수, 변경에 유리)
		ArrayList<Integer> integerList = new ArrayList<>();
        // new 뒤에 제너릭은 비워도 컴파일러가 자동으로 채워준다.
        //Integer 타입 리스트 생성
		integerList.add(1);
		integerList.add(2);
		integerList.add(3); //값 추가
		integerList.add(1,5); //1번 인덱스에 5 추가, 뒤의 데이터 밀림
		integerList.remove(1); //1번 인덱스 제거, 앞으로 땡겨짐
		System.out.println(integerList.size()); //리스트 크기 //3
		System.out.println(integerList.get(0)); //리스트 0번째 인덱스 // 1
		System.out.println(integerList.contains(2)); //2 데이터 값이 존재하는지 // true
		System.out.println(integerList.indexOf(2)); // 2 데이터의 인덱스 값 // 1
		
	}
}
```

- List 인터페이스를 구현한 선형 리스트 컬렉션
- 기본적으로 10의 저장공간이 주어지며 10의 저장공간을 넘어가면 10개씩 늘어난다. 
- 중복이 인정되며 넣는 순서대로 순서를 보장해준다.
- ArrayList를 생성할 때 타입을 선언해주지 않으면 다양한 타입의 데이터를 입력 받을 수 있다. 그러나 입력된 값을 꺼낼 때 캐스팅 과정이 필요하게 되므로 그 과정에서 에러가 발생할 수 있어 타입을 명시해준다.
> - Generics(제너릭)  : 타입 선언의 안정성을 위해 도입된 개념
> - `ArrayList<객체 타입>` 으로 선언할 수 있다. 
> - int 같은 기본자료형은 들어갈 수 없다. 기본자료형을 객체화시킨 wrapper 클래스를 이용해야 한다. (Ex: int > Integer) 
- add() : 데이터 입력받기
> - 데이터를 입력 받을 때는 무조건 Object 타입으로 변환되어 저장된다. 
- get([index]) : 인덱스 해당 데이터 꺼내오기
> - Object 타입으로 저장되었기 때문에 명시적 형 변환을 해주어야 한다.
- remove([index]) :  인덱스 해당 데이터 삭제
> - clear() : 모든 값 제거
- contains(data) : list에 data 있으면 true, 없으면 false
- indexOf(data) : list에 data 있으면 해당 인덱스 반환, 없으면 -1

<br>

## HashSet

```java
import java.util.HashSet;
import java.util.Set;

public class HashSetTest {
	public static void main(String[] args) {
		HashSet noTypeSet = new HashSet(); // 타입없이 생성
		Set<Person> personSet1 = new HashSet<Person>(); //묵시적 형 변환 	
		HashSet<Integer> integerSet = new HashSet<Integer>();
		// new 뒤의 제너릭은 생략 가능
		integerSet.add(1); 
		integerSet.add(2);
		integerSet.add(3); //데이터 추가
		System.out.println(integerSet.add(1)); // 이미 데이터가 존재하지 않으면 true, 존재하면 false // false
		System.out.println(integerSet.size()); // Set 크기 구하기 // 3
		integerSet.remove(3); // 데이터 삭제
		System.out.println(integerSet); //전체값 출력 // [1, 2]
		System.out.println(integerSet.contains(1)); //내부 데이터 존재하면 true, 아니면 false //true
	}
}
```

- Set 인터페이스를 구현한 컬렉션
- add() :  데이터 입력
- remove(data) : 데이터값을 넣으면 해당 데이터를 삭제한다.
- size() :  Set 크기 구하기
- contains(data) : 데이터 존재하면 true, 없으면 false 
- 객체를 중복해서 저장할 수 없고 순서가 보장되지 않는다. (인덱스 없음)
> - wrapper class는 자동으로 중복을 막아준다. (기본자료형을 객체화 시킨 것들 : Byte,  Character, Integer, Float, Double, Boolean, 	Long, Short )
> - 일반적으로 생성한 객체는 중복 허용을 막기 위해 오버라이딩이 필요하다.
> > - hashCode() ,equals()
> > - 또한 toString을 오버라이딩하지 않으면 참조변수값이 출력된다. 

- hashCode() ,equals() 을 오버라이딩 하지 않았을 때

```java
import java.util.HashSet;
import java.util.Set;

public class HashFoodTest {
	public static void main(String[] args) {
    
    	Set<Food> foodSet= new HashSet<>();
		foodSet.add(new Food("kimbab", 2000));
		foodSet.add(new Food("ramyeon", 4000));
		foodSet.add(new Food("ramyeon", 4000));
		System.out.println(foodSet.toString());
        //[Food [name=kimbab, price=2000], Food [name=ramyeon, price=4000], Food [name=ramyeon, price=4000]]
		//라면이 중복되었다. 
	}
}

class Food{
	String name;
	int price;
	
	public Food(String name, int price) {
		this.name = name;
		this.price = price;
	}

	@Override
	public String toString() {
		return "Food [name=" + name + ", price=" + price + "]";
	}
}

```

- hashCode() ,equals() 을 오버라이딩했을 때

```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

public class HashSetTest {
	public static void main(String[] args) {
    
		Set<Food> foodSet= new HashSet<>();
		foodSet.add(new Food("kimbab", 2000));
		foodSet.add(new Food("ramyeon", 4000));
		foodSet.add(new Food("ramyeon", 4000));
		System.out.println(foodSet.toString());
        //[Food [name=ramyeon, price=4000], Food [name=kimbab, price=2000]]
        //중복 허용되지 않았고 순서도 보장되지 않았다.
	}
}



class Food{
	String name;
	int price;
	
	public Food(String name, int price) {
		this.name = name;
		this.price = price;
		}

	@Override
	public String toString() {
		return "Food [name=" + name + ", price=" + price + "]";
	}

	@Override
	public int hashCode() {
		return Objects.hash(name, price);
	}

	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (getClass() != obj.getClass())
			return false;
		Food other = (Food) obj;
		return Objects.equals(name, other.name) && price == other.price;
	}
}
```

- 생성자를 통해 들어온 값이 같다면 같은 hashCode 값을 반환해준다.
- 객체의 내용과 타입이 같다면 true를 반환한다.

<br>

## HashMap

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Map.Entry;

public class HashMapTest {
	public static void main(String[] args) {
		HashMap<Integer, String> intStrMap =new HashMap<Integer, String>();
		// 키를 정수로 밸류를 스트링 타입으로 가지는 HashMap 생성
		Map<String, String> strMap =new HashMap<>();
		// new 뒤의 제너릭에는 타입 생략 가능, 컴파일러가 채워줌
		// 키와 밸류 모두 스트링 타입을 가지는 HashMap 생성
		strMap.put("NUM001", "Lee");
		strMap.put("NUM002", "Kim");
		strMap.put("NUM003", "Park");
		strMap.put("NUM004", "Jung"); //데이터 입력
		System.out.println(strMap.get("NUM002")); //데이터 검색 //Kim
		
		strMap.put("NUM004", "Kang"); //데이터 중복
		System.out.println(strMap.get("NUM004")); //키는 유일한 값이고 중복되면 덮어쓴다. //Kang
		
		strMap.remove("NUM004"); //데이터 제거
		
        //key, valuse 출력하기
        
        System.out.println(strMap);
        //{NUM001=Lee, NUM003=Park, NUM002=Kim}
        
		System.out.println("entrySet 이용 ");
		for (Entry<String, String> entry : strMap.entrySet()) {
		    System.out.println("Key : " + entry.getKey() + "  Value : " + entry.getValue());
		}
		
		System.out.println("keySet 이용 ");
		for(String key : strMap.keySet()){ 
        //저장된 key값과 해당 value 값 출력
			    System.out.println("Key : " + key + " Value : "+ strMap.get(key));
			}	
            
        //entrySet, keySet
        //Key : NUM001 Value : Lee
		//Key : NUM003 Value : Park
		//Key : NUM002 Value : Kim
            
		System.out.println("Values 이용");
		for(String value : strMap.values()){ //value 만  출력
		    System.out.println("Values : " + value);
		}	
       
        //Values 이용
		//Values : Lee
		//Values : Park
		//Values : Kim
	}
}
```

- Map 인터페이스를 구현한 컬렉션
- Key(키), Value(밸류) 형태로 데이터를 저장하고 관리한다. 
- 쌍으로 관리되며 리스트 두 개를 세로로 붙여놓은 형태이다. 
- Key 값이 중복을 인정하지 않고 중복하게 입력되면 덮어쓰기 된다. 
- put(key, value) : 데이터 입력
- get(key) : 데이터 검색
- remove(key) : 데이터 제거
- clear() : 모든 데이터 제거
- 데이터 출력하기
> - 그냥 출력하면 
> - entrySet은 key, value 모두 필요할 때 사용한다.
> - keySet과 get(key) 을 이용한 방법은 key를 이용해 value를 찾는 과정에서 시간이 많이 소요된다.
- 물론 value에 new를 이용해 인스턴스 객체를 생성해 넣을 수 있다.

```java
	Map<String, Food> foodMap = new HashMap<>();
	foodMap.put("NUM001", new Food("ttukbbokki", 8000));
```

<br>
### HashTable

```java
import java.util.Hashtable;
import java.util.Map;

public class HashtableTest {
	public static void main(String[] args) {
    Map<String, String> strTable =new Hashtable<>();    
    
    
    strTable.put("NUM001", "Lee"); 
    strTable.put("NUM002", "Kim"); //데이터 입력
    strTable.remove("NUM002");  //데이터 삭제
    
    for(String key : strtable.keySet()){ 
		System.out.println("Key : " + key + ", Value : "+ strtable.get(key));
            //저장된 key값과 해당 value 값 출력   
            //Key : NUM001, Value : Lee
    }
}   
```

- HashMap과 사용방법과 구조가 거의 같다. 
- Hashtable은 key, value에  Null이 들어갈 수 없다.
- Hashtable은 동기화를 지원하여 thread-safe하다.(multi-thread 상황에서 shared resource에 대해 순서를 기다린다.)
- Hashtable은 구형 자료구조이므로 multi-thread 상황에서는 ConcurrentHashMap 사용을 권고한다.(HashMap은 단일 쓰레드 상황에서만 사용이 가능하다.)

<br>

Reference:
- [java img_크레벅스](https://www.crebugs.com/product/view.php?idx=7382&code=1412)  
- [collection img _Dev.GA](https://gangnam-americano.tistory.com/41)
- [[Java] 자바 ArrayList 사용법 & 예제 총정리_ 코딩팩토리](https://coding-factory.tistory.com/551)
- [[JAVA] HASHSET개념과 자세한 사용방법, 예제_reakwon](https://reakwon.tistory.com/83)
- [[Java] 자바 HashMap 사용법 & 예제 총정리_코딩팩토리](https://coding-factory.tistory.com/556)
- [[JAVA] HashMap과 HashTable 차이_Odol87](https://odol87.tistory.com/3)