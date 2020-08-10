# Spring Security OAuth2.0
`스프링 시큐리티(spring security)`는 막강한 `인증(Authentication)`과 `인가(Authorization)` 기능을 가진 프레임워크이다.
사실상 스프링 기반의 어플리케이션에서는 보안을 위한 표준
인터셉터, 필터, 기반의 보안 기능을 구현하는 것보다 스프링 시큐리티를 통해 구현하는 것을 적극적으로 권장.

## Google Cloud 연동
[Google Cloud Platform](https://console.cloud.google.com/)
1. 프로젝트 선택
2. 새프로젝트
3. 햄버거 메뉴 -> API 및 서비스 -> 대시보드
4. 사용자 인증 정보 만들기 -> OAuth 클라이언트 ID 만들기
5. 동의 화면 구성
	- 애플리케이션 이름 : 구글 로그인 시 사용자에게 노출될 어플리케이션 이름
	- 지원 이메일 : 사용자 동의 화면에서 노출될 이메일 주소 보통은 help@domain.com 이메일을 사용
	- Google Api의 범위 : 이번에 등록할 구글 서비스에서 사용할 범위 목록 (기본값은 email, profile, openid)
6. 상단 메뉴 -> OAuth 클라이언트 ID 만들기
7. 웹 어플리케이션 -> 승인된 리디렉션 URI
	- `http://localhost:8080/login/oauth2/code/google`
	- 서비스에서 파라미터로 인증 정보를 주었을 때 인증이 성공하면 구글에서 리다리렉트할 URL
	- SpringBoot Security에서는 기본적으로 `/login/oauth2/code/{social code}`를 리다이렉트로 사용
	- 별도의 Controller를 구성할 필요 없이 시큐리티에서 이미 구현 완료


### 클라이언트 ID, 클라이언트 보안 비밀번호 등록
```
spring.security.oauth2.client.registration.google.client-id={클라이언트 ID}
spring.security.oauth2.client.registration.google.client-secret={클라이언트 보안 비밀}
spring.security.oauth2.client.registration.google.scope=profile,email
```
- 다른 예제에서는 scope를 별도로 등록하지 않음
- 기본값이 openid, profile, email 이기 때문
- 강제로 profile, email을 등록한 이유는 openid라는 scope가 있으면 Open Id Provider로 인식하기 때문
- 이렇게 되면 Openid Provider인 서비스(구글)와 그렇지 않은 서비스(네이버/카카오)로 나눠서 각각 OAuth2Service를 만들어 줘야 한다.
- 하나의 OAuth2Service로 사용하기 위해 일부러 openid scope를 빼고 등록

```
@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
                    .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console")
	                    .permitAll()
                    .antMatchers("/api/v1/**")
	                    .hasRole(Role.USER.name())
                    .anyRequest().authenticated()
                .and()
                    .logout()
                        .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                        .userInfoEndpoint()
                            .userService(customOAuth2UserService);
    }
}
```
1. `@EnableWebSecurity`
	- Spring Security 설정을 활성화
2. `csrf().disable().headers().frameOptions().disable()`
	- h2-console 화면을 사용하기 위해 해당 옵션들을 disable
3. authorizeRequests
	- URL 별 권한 관리를 설정하는 옵션의 시작점
	- authorizeRequests가 선언되어야만 antMatchers 옵션 사용
4. antMatchers
	- 권한 관리 대상을 지정하는 옵션
	- URL, HTTP 메소드별로 관리가 가능
	- Role 별로 접근 권한 설정 가능
5. anyRequest
	- 설정된 값들 이외의 나머지 URL 설정
6. oauth2Login
	- OAuth2 로그인 기능에 대한 여러 설정의 진입점
7. userInfoEndpoint
	- OAuth2 로그인 성공 이후 사용자 정보를 가져올 때의 설정들을 담당




``` java
@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {

    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        String usernameAttributeName = userRequest.getClientRegistration().getProviderDetails().getUserInfoEndpoint().getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes.of(registrationId, usernameAttributeName, oAuth2User.getAttributes());
        User user = saveOrUpdate(attributes);

        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())), attributes.getAttributes(), attributes.getnameAttributeKey());
    }

    private User saveOrUpdate(OAuthAttributes attributes) {
        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(), attributes.getPicture()))
                .orElse(attributes.toEntity());
        return userRepository.save(user);
    }
}
```
1. registrationId
	- 현재 로그인 진행중인 서비스를 구분하는 코드
	- 네이버나 카카오 구글인지 식별하는 코드
2. usernameAttributeName
	- OAuth2 로그인 진행 시 키가 되는 필드값 Primary Key와 같은 의미
	- 구글만 지원하고 카카오 네이버는 기본 지원하지 않음 (구글의 기본 코드는 sub)
3. OauthAttributes
	- OAuth2UserService를 통해 가져온 OAuth2User의 attribute를 담을 클래스
4. SessionUser
	- 세션에 사용자 정보를 저장하기 위한 Dto 클래스

``` java
@Getter
public class OAuthAttributes {
    private final Map<String, Object> attributes;
    private final String nameAttributeKey;
    private final String name;
    private final String email;
    private final String picture;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes, String nameAttributeKey, String name, String email, String picture) {
        this.attributes = attributes;
        this.nameAttributeKey = nameAttributeKey;
        this.name = name;
        this.email = email;
        this.picture = picture;
    }

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
        return ofGoogle(userNameAttributeName, attributes);
    }

    private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes) {
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    public User toEntity() {
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.GUEST)
                .build();
    }
}
```
1. of
	- OAuth2User에서 반환하는 사용자 정보는 Map이라 값 하나하나를 반환해야함
2. toEntity
	- User 엔티티를 생성
	- OAuthAttributes에서 엔티티를 생성하는 시점은 처음 가입할 때
	- 가입할 떄의 기본 권한을 GUEST로 주기 위해 rople 빌더값에는 Role.GUEST를 사용합니다.
	- OAuthAttributes 클래스 생성이 끝났으면 같은 패키지에 SessionUser 클래스를 생성

``` java
@Getter
public class SessionUser implements Serializable {
    private String name;
    private String email;
    private String picture;

    public SessionUser(User user) {
        this.name = user.getName();
        this.email = user.getEmail();
        this.picture = user.getPicture();
    }
}
```
`SessionUser`에는 인증된 사용자 정보만 필요함
User를 그냥 사용하기엔 Entity이고 Serializable이 있어야함으로 따로 구현하는게 안전하고 좋다.
User Entity가 OneToMany ManyToOne을 가지고 있다면 자식 엔티티도 직렬화에 포함이 되어 성능이슈, 부수효과가 발생할 확률이 높다.

## Session Storage
현업에서 사용하는 세션 저장소 종류
- 톰캣 세션
	- 기본값
	- 2대 이상의 톰캣이 구동될 경우 톰캣들간의 세션 공유를 위한 추가 설정이 필요
- MySQL (RDB)
	- 여러 WAS 간의 공용 세션을 사용할 수 있는 가장 쉬운 방법.
	- 로그인 요청마다 DB IO가 발생하여 성능상 이슈
	- 로그인 요청이 많이 없는 백오피스, 사내시스템 용도에서 사용
- Redis, Memcached  (Memory DB)
	- B2C 서비스에서 가장 많이 사용
	- 실제 서비스로 사용하기 위해서는 Embedded Redis와 같은 방식이 아닌 외부 메모리 서버가 필요

