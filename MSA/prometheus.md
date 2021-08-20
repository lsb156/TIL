> 도커기반 prometheus 설명

## 설정 적용 및 재실행

`prometheus.yml` 파일을 수정하여 메트릭정보를 긁어갈 서비스를 정의해준다.

```yaml
  - job_name: {service-name}
    scrape_interval: 5s
    metrics_path: /actuator/prometheus
    static_configs:
      - targets:
        - {target-ip}:{target-port}
```
설정파일 수정 후에 `curl -X POST http://localhost:9090/-/reload` 옵션으로 재시작 가능 하지만
프로메테우스 실행시에 `--web.enable-lifecycle`를 넣어주고 실행시켜야 한다.

- https://www.robustperception.io/reloading-prometheus-configuration

아래와 같은 명령으로도 설정을 재 적용 할 수 있다.
```
docker exec prometheus kill -SIGHUP 1 
```


shell command
```
docker -D exec -it prometheus sh
```



## spring boot prometheus
```
# prometheus tag
management:
  metrics:
    tags:
      application: ${spring.application.name}

```
