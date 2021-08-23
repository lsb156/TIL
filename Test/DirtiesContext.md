## DirtiesContext 사용 이유

1. 스프링 test 에서는 applicationContext가 딱 한개만 만들어짐
2. 이 context를 모든 테스트에서 공유하여 사용
3. 따라서 애플리케이션 컨텍스트의 구성이나 상태를 테스트 내에서 변경하지 않는 것이 원칙
4. 혹시나 context가 변경이 된다면 다른곳에서 계속해서 변경된 context를 재활용한다.
5. 이럴때 `@DirtiesContext`를 사용
6. 스프링의 테스트 컨텍스트 프레임워크에게 해당 클래스의 Test는 ApplicationContext 상태를 변화한다는것을 알려준다.
7. `@DirtiesContext`가 붙은 테스트 클래스에는 재활용되는 ApplicationContext를 공유하지 않는다.
8. 테스트중에 변경된 Context를 다른곳에서 사용하지 않게 하기 위함니다.
토비의 스프링 中
