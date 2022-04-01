# Instance

```yaml
kind: Instance
metadata:
  namespace:
  service:
spec:
  # Health check
  checker:
    # Name of health check strategy
    name: 
  instance:
    # Instance IP information
    host:
    # listen port
    port:
    # Port protocol
    protocol:
    # weight information
    weight:
    # health status identification
    healthy:
    # When an instance is isolated, the state is true, and any 
    # request cannot be routed to this example.
    isolate:
    # The version information of the instance is mainly used for version routing
    version:
    # Geographic location information is mainly used to close the route
    location:
      region:
      zone:
      campus:
    # Instance label, store metadata information related to this instance
    labels:
      - key:
        value:
      - key:
        value: 
```
