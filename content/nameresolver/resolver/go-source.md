---
date: 2018-09-29T06:54:00+08:00
title: Resolver的Go代码实现
weight: 10530
menu:
  main:
    parent: "nameresolver-resolver"
description : "Resolver的Go代码实现"
---

> 备注：以 v1.32.0 的代码为准

## resolver 包定义

### schema builder的注册

```go
var (
   // m is a map from scheme to resolver builder.
   m = make(map[string]Builder)
)

// Register registers the resolver builder to the resolver map. b.Scheme will be
// used as the scheme registered with this builder.
//
// NOTE: this function must only be called during initialization time (i.e. in
// an init() function), and is not thread-safe. If multiple Resolvers are
// registered with the same name, the one registered last will take effect.
func Register(b Builder) {
	m[b.Scheme()] = b
}

// Get returns the resolver builder registered with the given scheme.
//
// If no builder is register with the scheme, nil will be returned.
func Get(scheme string) Builder {
	if b, ok := m[scheme]; ok {
		return b
	}
	return nil
}
```

### 默认schema

默认 schema 是 passthrough。

```go
var (
   // defaultScheme is the default scheme to use.
   defaultScheme = "passthrough"
)

// SetDefaultScheme sets the default scheme that will be used. The default
// default scheme is "passthrough".
//
// NOTE: this function must only be called during initialization time (i.e. in
// an init() function), and is not thread-safe. The scheme set last overrides
// previously set values.
func SetDefaultScheme(scheme string) {
	defaultScheme = scheme
}

// GetDefaultScheme gets the default scheme that will be used.
func GetDefaultScheme() string {
	return defaultScheme
}
```

### Address 结构定义

Address 用于表示客户端连接到的一个服务器。

```go
// This is the EXPERIMENTAL API and may be changed or extended in the future.
type Address struct {
   // Addr is the server address on which a connection will be established.
   Addr string

   // ServerName is the name of this address.
   // If non-empty, the ServerName is used as the transport certification authority for
   // the address, instead of the hostname from the Dial target string. In most cases,
   // this should not be set.
   //
   // If Type is GRPCLB, ServerName should be the name of the remote load
   // balancer, not the name of the backend.
   //
   // WARNING: ServerName must only be populated with trusted values. It
   // is insecure to populate it with data from untrusted inputs since untrusted
   // values could be used to bypass the authority checks performed by TLS.
   ServerName string

   // Attributes contains arbitrary data about this address intended for
   // consumption by the load balancing policy.
   Attributes *attributes.Attributes

   // Type is the type of this address.
   //
   // Deprecated: use Attributes instead.
   Type AddressType

   // Metadata is the information associated with Addr, which may be used
   // to make load balancing decision.
   //
   // Deprecated: use Attributes instead.
   Metadata interface{}
}
```

小心这个新的替代 naming 的 resolver 也还是 EXPERIMENTAL API，而且 Type 和  Metadata 这两个字段已经 Deprecated，改为使用 Attributes。

```go
// Attributes是一个不可变的结构体，用于存储和检索通用键/值对。 key必须是可哈希的，用户应该为key定义自己的类型。
type Attributes struct {
   m map[interface{}]interface{}
}
```

### builder定义

Builder 创建 resolver， resolver将用于观察名称解析更新。

```go
type Builder interface {
   // Build creates a new resolver for the given target.
   //
   // gRPC dial calls Build synchronously, and fails if the returned error is
   // not nil.
   Build(target Target, cc ClientConn, opts BuildOptions) (Resolver, error)
   // Scheme returns the scheme supported by this resolver.
   // Scheme is defined at https://github.com/grpc/grpc/blob/master/doc/naming.md.
   Scheme() string
}
```

Target的说明：

```go
// Target represents a target for gRPC, as specified in:
// https://github.com/grpc/grpc/blob/master/doc/naming.md.
// It is parsed from the target string that gets passed into Dial or DialContext by the user. And
// grpc passes it to the resolver and the balancer.
//
// If the target follows the naming spec, and the parsed scheme is registered with grpc, we will
// parse the target string according to the spec. e.g. "dns://some_authority/foo.bar" will be parsed
// into &Target{Scheme: "dns", Authority: "some_authority", Endpoint: "foo.bar"}
//
// If the target does not contain a scheme, we will apply the default scheme, and set the Target to
// be the full target string. e.g. "foo.bar" will be parsed into
// &Target{Scheme: resolver.GetDefaultScheme(), Endpoint: "foo.bar"}.
//
// If the parsed scheme is not registered (i.e. no corresponding resolver available to resolve the
// endpoint), we set the Scheme to be the default scheme, and set the Endpoint to be the full target
// string. e.g. target string "unknown_scheme://authority/endpoint" will be parsed into
// &Target{Scheme: resolver.GetDefaultScheme(), Endpoint: "unknown_scheme://authority/endpoint"}.
type Target struct {
   Scheme    string
   Authority string
   Endpoint  string
}
```

ClientConn 包含用于 resolver 的 callback，以将更新通知到grpc ClientConn。

```go
// ClientConn contains the callbacks for resolver to notify any updates
// to the gRPC ClientConn.
//
// This interface is to be implemented by gRPC. Users should not need a
// brand new implementation of this interface. For the situations like
// testing, the new implementation should embed this interface. This allows
// gRPC to add new methods to this interface.
type ClientConn interface {
   // UpdateState updates the state of the ClientConn appropriately.
   UpdateState(State)
   // ReportError notifies the ClientConn that the Resolver encountered an
   // error.  The ClientConn will notify the load balancer and begin calling
   // ResolveNow on the Resolver with exponential backoff.
   ReportError(error)
   // NewAddress is called by resolver to notify ClientConn a new list
   // of resolved addresses.
   // The address list should be the complete list of resolved addresses.
   //
   // Deprecated: Use UpdateState instead.
   NewAddress(addresses []Address)
   // NewServiceConfig is called by resolver to notify ClientConn a new
   // service config. The service config should be provided as a json string.
   //
   // Deprecated: Use UpdateState instead.
   NewServiceConfig(serviceConfig string)
   // ParseServiceConfig parses the provided service config and returns an
   // object that provides the parsed config.
   ParseServiceConfig(serviceConfigJSON string) *serviceconfig.ParseResult
}
```

BuildOptions 包含builder用于创建resolver的额外信息。

```go
type BuildOptions struct {
   // DisableServiceConfig 指示resolver的实现是否要获取service config数据
   DisableServiceConfig bool
   // DialCreds is the transport credentials used by the ClientConn for
   // communicating with the target gRPC service (set via
   // WithTransportCredentials). In cases where a name resolution service
   // requires the same credentials, the resolver may use this field. In most
   // cases though, it is not appropriate, and this field may be ignored.
   DialCreds credentials.TransportCredentials
   // CredsBundle is the credentials bundle used by the ClientConn for
   // communicating with the target gRPC service (set via
   // WithCredentialsBundle). In cases where a name resolution service
   // requires the same credentials, the resolver may use this field. In most
   // cases though, it is not appropriate, and this field may be ignored.
   CredsBundle credentials.Bundle
   // Dialer is the custom dialer used by the ClientConn for dialling the
   // target gRPC service (set via WithDialer). In cases where a name
   // resolution service requires the same dialer, the resolver may use this
   // field. In most cases though, it is not appropriate, and this field may
   // be ignored.
   Dialer func(context.Context, string) (net.Conn, error)
}
```

## resolver定义

Resolver 监控指定目标的更新。更新包括地址更新和服务配置的更新。

```go
type Resolver interface {
   // ResolveNow 将被 gRPC 调用来尝试再次解析目标名称。
   // 这仅仅是一个提示（hint），resolver 可以忽略它，如果没有必要。
   // 可以并发的多次调用。
   ResolveNow(ResolveNowOptions)
   // Close closes the resolver.
   Close()
}

// ResolveNowOptions includes additional information for ResolveNow.
type ResolveNowOptions struct{}
```



## passthrough resolver代码实现

Package passthrough实现了一个直通式的解析器。它将不含scheme的目标名称作为解析地址发回给gRPC。

```go
const scheme = "passthrough"

type passthroughBuilder struct{}

func (*passthroughBuilder) Build(target resolver.Target, cc resolver.ClientConn, opts resolver.BuildOptions) (resolver.Resolver, error) {
   r := &passthroughResolver{
      target: target,
      cc:     cc,
   }
   r.start()
   return r, nil
}

func (*passthroughBuilder) Scheme() string {
   return scheme
}
```

start方法中就直接进行解析了：

```go
func (r *passthroughResolver) start() {
   r.cc.UpdateState(resolver.State{Addresses: []resolver.Address{{Addr: r.target.Endpoint}}})
}
```

resolver 接口定义的方法都置空：

```go
func (*passthroughResolver) ResolveNow(o resolver.ResolveNowOptions) {}

func (*passthroughResolver) Close() {}
```

package初始化时就自动注册

```go
func init() {
   resolver.Register(&passthroughBuilder{})
}

// 配合 passthrough.go
import _ "google.golang.org/grpc/internal/resolver/passthrough" // import for side effects after package was moved
```

## manual resolverd代码实现

package manual 定义了一个解析器，可以用来手动发送解析地址到ClientConn。

用于测试目标，在构造时给出一个初始化状态，之后就直接用这个状态通知 ClientConn：

```go
// InitialState adds initial state to the resolver so that UpdateState doesn't
// need to be explicitly called after Dial.
func (r *Resolver) InitialState(s resolver.State) {
	r.bootstrapState = &s
}

// Build returns itself for Resolver, because it's both a builder and a resolver.
func (r *Resolver) Build(target resolver.Target, cc resolver.ClientConn, opts resolver.BuildOptions) (resolver.Resolver, error) {
   r.CC = cc
   if r.bootstrapState != nil {
      r.UpdateState(*r.bootstrapState)
   }
   return r, nil
}
```

## dns resolver代码实现

### dns resolver的builder

```go
// Build creates and starts a DNS resolver that watches the name resolution of the target.
func (b *dnsBuilder) Build(target resolver.Target, cc resolver.ClientConn, opts resolver.BuildOptions) (resolver.Resolver, error) {
   host, port, err := parseTarget(target.Endpoint, defaultPort)
   if err != nil {
      return nil, err
   }

   // IP address.
   if ipAddr, ok := formatIP(host); ok {
      addr := []resolver.Address{{Addr: ipAddr + ":" + port}}
      cc.UpdateState(resolver.State{Addresses: addr})
      return deadResolver{}, nil
   }

   // DNS address (non-IP).
   ctx, cancel := context.WithCancel(context.Background())
   d := &dnsResolver{
      host:                 host,
      port:                 port,
      ctx:                  ctx,
      cancel:               cancel,
      cc:                   cc,
      rn:                   make(chan struct{}, 1),
      disableServiceConfig: opts.DisableServiceConfig,
   }

   if target.Authority == "" {
      d.resolver = defaultResolver
   } else {
      d.resolver, err = customAuthorityResolver(target.Authority)
      if err != nil {
         return nil, err
      }
   }

   d.wg.Add(1)
   go d.watcher()
   d.ResolveNow(resolver.ResolveNowOptions{})
   return d, nil
}
```

### dns resolver的结构体

```go
// dnsResolver watches for the name resolution update for a non-IP target.
type dnsResolver struct {
   host     string
   port     string
   resolver netResolver
   ctx      context.Context
   cancel   context.CancelFunc
   cc       resolver.ClientConn
   // rn channel is used by ResolveNow() to force an immediate resolution of the target.
   rn chan struct{}
   // wg is used to enforce Close() to return after the watcher() goroutine has finished.
   // Otherwise, data race will be possible. [Race Example] in dns_resolver_test we
   // replace the real lookup functions with mocked ones to facilitate testing.
   // If Close() doesn't wait for watcher() goroutine finishes, race detector sometimes
   // will warns lookup (READ the lookup function pointers) inside watcher() goroutine
   // has data race with replaceNetFunc (WRITE the lookup function pointers).
   wg                   sync.WaitGroup
   disableServiceConfig bool
}
```

### resolver接口方法的实现

```go
// ResolveNow invoke an immediate resolution of the target that this dnsResolver watches.
func (d *dnsResolver) ResolveNow(resolver.ResolveNowOptions) {
   select {
   case d.rn <- struct{}{}:
   default:
   }
}

// Close closes the dnsResolver.
func (d *dnsResolver) Close() {
   d.cancel()
   d.wg.Wait()
}
```

### watcher的实现

```go
func (d *dnsResolver) watcher() {
   defer d.wg.Done()
   for {
      select {
      case <-d.ctx.Done():
         return
      case <-d.rn:
      }

      state, err := d.lookup()
      if err != nil {
         d.cc.ReportError(err)
      } else {
         d.cc.UpdateState(*state)
      }

      // Sleep to prevent excessive re-resolutions. Incoming resolution requests
      // will be queued in d.rn.
      t := time.NewTimer(minDNSResRate)
      select {
      case <-t.C:
      case <-d.ctx.Done():
         t.Stop()
         return
      }
   }
}
```