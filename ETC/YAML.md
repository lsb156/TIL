# YAML 
## YAML?
https://yaml.org/

`YAML Ain’t Markup Language`
데이터 Serialization 표준

- `YAML`은 `JSON`의 Superset
- `YAML`은 파싱이 어렵지만 가독성이 좋고
- `JSON`은 파싱이 쉽지만 가독성이 안좋다.

> YAML Test Site
> YAML 문법을 JSON으로 컨버팅해주어 연습하기 좋은 사이트
> https://onlineyamltools.com/convert-yaml-to-json

## Version
- https://yaml.org/spec/1.0/ (2004-01-29)
- https://yaml.org/spec/1.1/ (2005-01-18)
- https://yaml.org/spec/1.2/spec.html (2009-10-01)

## V1.2에서의 변화
JSON의 Superset처럼 동작하기 위해서 많은 사항들이 변경
부분 혹은 전체를 JSON으로 작성하여도 대부분 인식 가능 
> All other characters, including the form feed (#x0C), are considered to be non-break characters.
Note that these include the non-ASCII line breaks: next line (#x85), line separator (#x2028) and paragraph separator (#x2029).
YAML version 1.1 did support the above non-ASCII line break characters; however, JSON does not.
Hence, to ensure JSON compatibility, YAML treats them as non-break characters as of version 1.2. In theory this would cause incompatibility with version 1.1;
in practice these characters were rarely (if ever) used.
YAML 1.2 processors parsing a version 1.1 document should therefore treat these line breaks as non-break characters, with an appropriate warning.

## 기본 문법
### 문서 시작, 문서 종료
- ```---``` : 문서 시작
- ```...``` : 문서 종료

```yaml
---
spring:
  profiles: local
  datasource:
      url: jdbc:mysql://local
...
---
spring:
  profiles: dev
  datasource:
      url: jdbc:mysql://dev
...
```
위와 같이 표시하는게 정석이긴 하지만 보통 아래와 같이 정의하면서 사용한다.
```yaml
---
spring:
  profiles: local
  datasource:
      url: jdbc:mysql://local
---
spring:
  profiles: dev
  datasource:
      url: jdbc:mysql://dev
```

### Sequence Node
list 형태의 자료형
`-` 뒤에 공백이 있어야 한다.
```yaml
# 1) Block Style
- a
- b
  
# 2) Flow Style
[a, b] 
```
```
["a", "b"]
```

### Mapping node
key value 형태의 자료형
hash, dictionary라고도 불리운다.
`:`, `?` 뒤에 공백이 있어야 한다.
```yaml
# 1) Block Style
test:
  key1: value1
  key2: value2
  
# 2) Block Style
test:
  ? key1
  : value1
  ? key2
  : value2

# 3) Flow Style 
test: {
  key1: value1,
  key2: value2
}
```
```json
// toJSON
{
  "test": {
    "key1": "value1",
    "key2": "value2"
  }
}
```
### Annotation
주석은 `#`로 시작한다.

### Anchor property / Alias node
> 참고 : YAML은 단지 표현문으로 사용되어야해서 병합키가 들어가는 형태를 지양하는 우려들이 있다.
> 
> In YAML 1.3, loaders will be encouraged not to support merge keys by default. It will become an optional feature. YAML is a data language in the same sense that JSON is. Programatic features should not have ever been encouraged.

```yaml
anchor: &name value
alias: *name
```
```json
// toJSON
{
  "anchor": "value",
  "alias": "value"
}
```
```yaml
bird: &default_bird
  cry: 짹짹
  fly: true
  color: white

참새:
  <<: *default_bird
  color: brown

오리:
  <<: *default_bird
  cry: 꽉꽉

닭:
  <<: *default_bird
  cry: 꼬끼오
  fly: false

```


### MultiLine
```yaml
include_newline: |
  Lorem 
  ipsum 
  dolor 
  sit 
  amet

fold_newline: >
  Lorem 
  ipsum 
  dolor 
  sit 
  amet
```
```json
{
  "include_newline": "Lorem \nipsum \ndolor \nsit \namet\n",
  "fold_newline": "Lorem  ipsum  dolor  sit  amet\n"
}
```

> YAML
> https://perfectacle.github.io/2018/08/19/yaml/