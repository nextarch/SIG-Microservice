# Health check

```yaml
kind: HealthCheck
metadata:
  namespace:
  service:
spec:
  name:
  checker:
    type: http
    options:
      openTls:
      timeout:
      url: /api/v1/check
      method: GET
      body: |-
        {

        }
      expectCode: 200
      expectResp: |-
        {
            "code": "",
            "msg": ""
        }
```


```yaml
kind: HealthCheck
metadata:
  namespace:
  service:
spec:
  name:
  checker:
    type: tcp
    options:
      port: 
      timeout:
```


```yaml
kind: HealthCheck
metadata:
  namespace:
  service:
spec:
  name:
  checker:
    type: heartbeat
    options:
      beatInterval: 
      unhealthTime: 
      removeTime: 
```
