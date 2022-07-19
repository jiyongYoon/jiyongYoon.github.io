---
title: "[객체지향] 상속(Heritance) (다형성, 추상클래스, 인터페이스)"
layout: single
categories: java
typora-root-url: ..\..\images\
toc: true
---

# [객체지향] 상속(Heritance) (다형성, 추상클래스, 인터페이스)

------



## 1) 상속 (Heritance)

: (상속 받을 수 있는) 필드와 메서드를 넘겨받는 개념.

```java
// 상위 클래스(슈퍼 클래스, 부모 클래스, 기본 클래스)
public class KoreanPeople {
    public String name = "이사람";
    public int years;
    public String location = "대한민국";
}
```

```java
// 하위 클래스(서브 클래스, 자식 클래스, 파생 클래스)
public class Students extends KoreanPeople {
    public String name = "김학생";
    public String school;
    
    public void printInfo() {
        System.out.println("name: " + name);
        System.out.println("years: " + years);
        System.out.println("Student location: " + location);
        // 부모클래스 변수 출력방법
        System.out.println("People location" + super.location); 
    }
}
```

```java
// 제 3의 클래스
public class test1 {
    public static void main(String[] args) {
    	Student s1 = new Student();
        s1.name = "홍길동"; 
        s1.years = 30;
        s1.location = "서울";
        // 맴버 변수를 선언하지 않아도 부모 클래스에서 변수를 상속받아 사용 가능
        s1.printInfo();
        // 메서드도 사용 가능
    }
}
```

```java
// 제 4의 클래스
public class test2 {
    public static void main(String[] args) {
    	Student s2 = new Student();
        s2.years = 20;
        s2.location = "성남";
        s2.school = "성남대학교";

        s2.printInfo(); 
        // name = 김학생, years = 20, 
        // People location = 대한민국, Student location = 성남.
        // printInfo() 메서드는 Student 클래스 메서드이기 때문.
    }
}
```



- ### 단일 상속

  : 클래스는 한 곳에서만 상속 받을 수 있음.

  

- ### 상속과 생성자

  : 부모 클래스의 생성자를 먼저 호출하고, 자식 클래스의 생성자를 그 이후에 호출한다.

  ```java
  // 부모 클래스
  public class KoreanPeople {
      public String name;
      public int years;
      public String location = "대한민국";
      public Korean() {
          System.out.println("한국사람 생성");
          // 없으면 에러남. 
          // 자식의 기본생성자에서 부모의 기본생성자를 못찾기 때문
          // 자식의 기본생성자 안에 부모의 매개변수 생성자를 호출하면 괜찮음.
      }
      public Korean(int years, int location) {
        	this.years = years;
          this.location = location;
      }   
  }
  
  // 자식 클래스
  public class Student extends KoreanPeople {
      public String name = "김학생";
      public String school;
      public student() {
          System.out.println("학생 생성");
      }
      public Student(int years, String location, String school) {       
          super(years, location); // 부모 클래스 생성자로 넘기기
          this.school; // 자식 클래스에서 처리하기
      }
  }
  
  // 제 3의 클래스
  public class test {
  	public static void main(String args[]) {
  		Student s3 = new Student(30, "부산", "부산대학교");
  		// "한국사람 생성"
  		// "학생 생성"
        	// 30과 부산은 부모 클래스, 부산대학교는 자식 클래스 생성자로 감
  	}
  }
  ```

  - ### 접근 제어자 복습
  
    ![access-modifier](..\..\images\access-modifier.PNG)

------



## 2) 다형성 (Polymorphism)

: 상속관계에 따라 다른 클래스 형태로 표현하는 것.

### - 업캐스팅과 다운캐스팅

: 하위 객체가 상위 클래스형 변수에 대입될 때. 즉, 상위로의 자료형 변환이 되었을 때 <u>업캐스팅</u> 되었다고 함. <br>

업캐스팅된 객체는 하위 객체의 멤버를 참조할 수 없음.

<u>다운캐스팅</u>은 반대로 상위 클래스형을 하위 클래스 형으로 변환했을 때를 말하며, 명시적인 형변환 연산자가 필요함.

```java
// 부모 클래스
public class KoreanPeople {
    public String name;
    public int years;
    public String location = "대한민국";
    public Korean() {
    }
    public Korean(int years, int location) {
      	this.years = years;
        this.location = location;
    }   
}

// 자식 클래스
public class Student extends KoreanPeople {
    public String name = "김학생";
    public String school;
    public student() {
    }
    public Student(int years, String location, String school) {       
        super(years, location);
        this.school;
    }
}

// 제 3의 클래스
public class test {
	public static void main(String args[]) {
        // 객체 생성
        Student s3 = new Student(30, "부산", "부산대학교");
        
        // 업캐스팅
        KoreanPeople k3 = s3;
        // 업캐스팅 시 Student의 객체라 하더라도 캐스팅 된 클래스의 변수와 메서드만 사용 가능해짐.
        System.out.println(k3.location); // "대한민국"
        System.out.println(k3.school); // 불가능
        
        // 다운캐스팅
        Student s4 = (Student)k3;
        // k3의 본체는 Student였음. 
        // 업캐스팅을 해서 필요한 작업을 하고 나면 다운캐스팅해서 다시 하위 클래스의 객체에 넣어줄 수 있음.
        System.out.println(s4.school); // "부산대학교"
	}
}
```

- ### A instanceof B

A(객체)가 B(클래스)의 부모클래스 객체가 맞는지 확인. 즉, 형변환이 가능한지(A가 B로 형변환이 가능한가?). 맞으면 A는 B로 다운캐스팅이 가능해짐. <br>

=> if문에 넣어서 다운캐스팅 오류를 제한함.

```java
if(k3 instanceof Student) {
	Student s4 = (Student)k3;
} else {
    System.out.println("k3는 Student 객체가 아닙니다.");
}
```

- ### 오버라이드(override)

: 메서드를 수정, 보완, 재정의 하는 것을 말함. <br>

메서드의 1) 이름, 2) 매개변수, 3) 반환값 이 같아야 오버라이드 개념이 성립함. <br>

=> 상속받은 하위 클래스에서 생성자 및 메서드를 오버라이드 하는 경우가 많으며, 오버라이드 된 메서드는 **<u>가장 하위 객체의 메서드를 실행하게 된다.</u>**<br>

**메서드가 final, private인 경우는 오버라이딩 불가능* <br>

**오버로딩(overload)은 1) 이름, 2) 반환값 이 같으며 3) 매개변수의 개수나 매개변수의 자료형이 다르면 오버로딩 개념이 성립함.*

------

## 3) 추상 클래스 (abstract) 와 인터페이스 (interface)

### - 추상클래스

: 하위 클래스들을 대표하는 클래스 개념. 다만, '구체적'이지 않음. <br>=> 직접 객체화(instantiation) 될 수 없다. 상속해서 쓰라는 의미.

### - 인터페이스

: 하위 클래스가 수행해야 하는 메서드와 필요한 상수만을 미리 추상적으로 정의해놓은 특별한 클래스. 정의만 있고 구현체는 없음.<br>=> 다중상속이 안되는 java의 특성을 보완. 인터페이스는 다중상속 지원.<br>**=> 같은 인터페이스를 implement 한다면, 같은 메서드가 있다는 것을 보증할 수 있다!!**



------



## **실전예제**

```java
// 부모 클래스
public (abstract) class 유저 {
    String 직업명;
    int 체력;
    int 마나;
    private int level = 1;
    
    public void 마을에서다시태어나기() {
        체력 = full;
        마나 = full;
    }
    
    public void 공격() {
        System.out.println("각 직업에 맞는 공격 설정");
    }
    // 추상클래스로 구현 시 아래처럼.
    public abstract void 공격();
    
}

// 직업 인터페이스
public interface 캐릭터 {
    public void 레벨업();
    public void 전직();
}

// 자식 클래스 1
public class 전사 extends 유저 implements 캐릭터 {
    public 전사() {
        직업명 = "전사";
        체력 = 100;
        마나 = 20;
    }
    @override from class
    public void 공격() {
        System.out.println("몽둥이로 공격");
    }
    @override from interface
    public void 레벨업() {
        level++;
        System.out.println("전사 LV." + level + "이 되었습니다.");
    }
    public void 전직() {
        System.out.println("광전사 또는 갑옷전사로 전직이 가능합니다.");
    }
    
}

// 자식 클래스 2
public class 마법사 extends 유저 implements 캐릭터 {
    public 마법사() {
        직업명 = "마법사";
        체력 = 50;
        마나 = 80;
    }
    @override from class
    public void 공격() {
        System.out.println("파이어볼 공격");
    }
    @override from interface
    public void 레벨업() {
        level++;
        System.out.println("마법사 LV." + level + "이 되었습니다.");
    }
    public void 전직() {
        System.out.println("불마법사 또는 물마법사로 전직이 가능합니다.");
    }
}

// 자식 클래스 3
public class 용병 implements 캐릭터 {
    @override from interface
    public void 레벨업() {
        System.out.println("레벨이 1 올랐습니다.");
    }
    public void 전직() {
        System.out.println("용병은 전직이 불가능합니다.");
    }
}

// 실행 클래스
public class Game {
    public static void main(String args[]) {
        
        Scanner sc = new Scanner(System.in);
        System.out.print("캐릭터를 고르세요. 1=전사, 2=마법사   (1/2?)");
        int input = sc.nextInt();
        
        유저 user;
        if (input==1) {
            user = new 전사();
        } else if (input==2) {
            user = new 마법사();
        } else {
            System.out.println("캐릭터를 선택하지 않았습니다.");
        }
        
        user.공격(); // 전사면 몽둥이 공격, 마법사면 파이어볼 공격
        user.마을에서다시태어나기(); // 오버라이드 메서드가 없으니 둘다 체력, 마나 가득 회복
    }
}

```



------

> 마지막 수정일시: 2022-06-06 17:53