# Artillery

성능 측정 도구로 Node가 설치되어있어야한다.
https://artillery.io/

``` bash
$ npm install -g artillery
```



``` yaml
config:
  target: "https://shopping.service.staging"
  phases:
    - duration: 60
      arrivalRate: 5
      name: Warm up
    - duration: 120
      arrivalRate: 5
      rampTo: 50
      name: Ramp up load
    - duration: 600
      arrivalRate: 50
      name: Sustained load
  payload:
    # Load search keywords from an external CSV file and make them available
    # to virtual user scenarios as variable "keywords":
    path: "keywords.csv"
    fields:
      - "keywords"
scenarios:
  # We define one scenario:
  - name: "Search and buy"
    flow:
      - post:
          url: "/search"
          body: "kw={{ keywords }}"
          # The endpoint responds with JSON, which we parse and extract a field from
          # to use in the next request:
          capture:
            json: "$.results[0].id"
            as: "id"
      # Get the details of the product:
      - get:
          url: "/product/{{ id }}/details"
      # Pause for 3 seconds:
      - think: 3
      # Add product to cart:
      - post:
          url: "/cart"
          json:
            productId: "{{ id }}"
```

## Start Test
``` bash
$ artillery run --output report.json test.yml
```

위에 작성된 test.yml 파일을 토대로 테스트를 시작한다.

``` yaml
- duration: 60 # 60초동안
  arrivalRate: 5 # 5명의 가상유저
```

## report json viewer
``` bach
$ artillery report report.json
```
보고서 작성이 완료된 json 파일을 html 형태로 보여준다.
