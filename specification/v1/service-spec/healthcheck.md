# Health check


## HTTP detection form of health check

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


## TCP detection form of health check

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


## Actively report a health check in the form of heartbeat

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
