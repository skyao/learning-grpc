# Transport

```java
private static final ClientTransport SHUTDOWN_TRANSPORT =
  new FailingClientTransport(Status.UNAVAILABLE.withDescription("Channel is shutdown"));

static final ClientTransport IDLE_MODE_TRANSPORT =
  new FailingClientTransport(Status.INTERNAL.withDescription("Channel is in idle mode"));

private final ClientTransportFactory transportFactory;
```