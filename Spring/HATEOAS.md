# HATEOAS

## HATEOAS References

기본적으로 2가지의 형태로 제공
### WebLink(RFC5988)
https://tools.ietf.org/html/rfc5988
- Target URI: 
  - 전이, 요청을 할 수 있는 link.
  - `href`라고 표현
- Link Relation Type: 
  - target resource의 URI와 현재 URI가 어떻게 연결되어있는지 관계에 대해 정의 
  - `rel`라고 표현
- Attribute for target URI: 
  - target link를 부가 설명할 수 있는 요소 
  - title, media, type 및 정의에 따라 사용

### HAL (Hypermedia API Language)

- HAL은 json또는 xml의 content에 link를 담는 방식

아래 2개의 contentType 사용하여 표현
- `application/hal+xml`, `application/hal+json`
```yaml
{
  "_links": {
    "self": {
      "href": "/boards"
    }
  },
  "_embeded": {
    "boards": [ {
      "idx": 1,
      "title": "1 title",
      "contents": "1 contents"
        "_links": {
          "self": {
            "href": "/boards/1"
          }
        }
    },{
      "idx": 2,
      "title": "2 title",
      "contents": "2 contents"
        "_links": {
          "self": {
            "href": "/boards/2"
          }
        }
    } ]
  }
}
```

- `_link` : 호출한 resource와 연관있는 link들을 collection의 형식으로 표현
- `_embedded` : 해당 resource가 포함하고 있는 요소를 표현
- `self` : 각 Resource에 대해서 본인의 Resource에 대한 link는 반드시 포함
