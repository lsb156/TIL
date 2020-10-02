# JMeter

Thread Group를 생성하여 테스트를 진행 환경 생성
![jmeter-thread-group](../asset/Test/jmeter-thread-group.png)

## Thread-group
![jmeter-thread-group-config](../asset/Test/jmeter-thread-group-config.png)
- Number of Threads : 요청을 보낼 Thread의 갯수를 지정 (유저의 숫자)
- Ramp-up period : 몇초만에 상단에 적은 Thread를 모두 생성할건지
- Loop Count : 총 몇번을 반복할건지 Count
- Action to taken after a Sampler error : 에러가 났을 경우에 대한 행동 정의


## Regist Sampler
![jmeter-sampler-action-http](../asset/Test/jmeter-sampler-action-http.png)
테스트를 진행 할 Http 요청을 정의
![jmeter-sampler-http-method](../asset/Test/jmeter-sampler-http-method.png)
테스트를 진행할 IP와 Port, Http Request Method와 Path를 적는다.

## Listener
![jmeter-listener-list](../asset/Test/jmeter-listener-list.png)
보낸 요청에 대해서 리스너를 등록하여 어떻게 요청을 했고 무슨 응답이 왔는지 시각화하여 볼 수 있다.
- View Result Tree
  - ![jmeter-listener-view-result-tree](../asset/Test/jmeter-listener-view-result-tree.png)
- View Result in Table
  - ![jmeter-listener-view-result-in-table](../asset/Test/jmeter-listener-view-result-in-table.png)
- Summary Report
  - ![jmeter-listener-summary-reposrt](../asset/Test/jmeter-listener-summary-reposrt.png)
- Aggregate
  - ![jmeter-listener-aggregate-report](../asset/Test/jmeter-listener-aggregate-report.png)
- Response Time Graph
  - ![jmater-listener-response-time-graph](../asset/Test/jmater-listener-response-time-graph-001.png)
  - ![jmeter-listener-response-time-graph](../asset/Test/jmeter-listener-response-time-graph-002.png)

## Assertion
![jmeter-assertion-setting](../asset/Test/jmeter-assertion-add.png)
![jmeter-assertion-setting](../asset/Test/jmeter-assertion-setting.png)
Response의 Code가 200인것만 허용하는 설정을 추가


## 레포트
`jmeter/bin/jmeter -n -t ./jmeter-thread-group.jmx`
- `-n` : UI 없이 실행
- `-t` : 설정파일 기반으로 바로 실행


