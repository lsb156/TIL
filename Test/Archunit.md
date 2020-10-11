# Archunit

[ArchUnit](https://www.archunit.org/)이란 애플리케이션의 아키텍처를 테스트 할 수 있는 오픈 소스 라이브러리  
패키지, 클래스, 레이어, 슬라이스 간의 의존성을 확인할 수 있는 기능을 제공

## 설정
``` gradle
testImplementation("com.tngtech.archunit:archunit-junit5-engine:0.12.0")
```

##  아키텍처 테스트 유즈 케이스
- A 라는 패키지가 B (또는 C, D) 패키지에서만 사용 되고 있는지 확인 가능.
- *Serivce라는 이름의 클래스들이 *Controller 또는 *Service라는 이름의 클래스에서만 참조하고 있는지 확인.
- *Service라는 이름의 클래스들이 ..service.. 라는 패키지에 들어있는지 확인.
- A라는 애노테이션을 선언한 메소드만 특정 패키지 또는 특정 애노테이션을 가진 클래스를 호출하고 있는지 확인.
- 특정한 스타일의 아키텍처를 따르고 있는지 확인.



## 패키지 레벨 테스트

## 클래스 레벨 테스트

## Freezing Arch Rule

## PlantUML
