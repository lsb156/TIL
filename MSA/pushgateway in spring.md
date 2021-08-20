
```
# spring prometheus (with pushgateway)
management:
  metrics:
    export:
      prometheus:
        pushgateway:
          base-url: ... # pushgateway url
          enabled: true
          shutdown-operation: push
          push-rate: 1m
          grouping-key:
            instance: ${spring.cloud.client.ip-address:localhost}:${server.port}

```
