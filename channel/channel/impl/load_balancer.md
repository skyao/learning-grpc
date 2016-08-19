# Load Balancer

```java
private final LoadBalancer.Factory loadBalancerFactory;

// channel处于空闲模式时 channel 为 null
@Nullable
private LoadBalancer<ClientTransport> loadBalancer;
```