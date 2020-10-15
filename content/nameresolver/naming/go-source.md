---
date: 2018-09-29T06:54:00+08:00
title: Naming的Go代码实现
weight: 10330
menu:
  main:
    parent: "nameresolver-naming"
description : "Naming的Go代码实现"
---

> 备注: 由于 naming package 在grpc-go v1.30.0 版本之后被删除，所以源码实现以最后一个版本 v1.29.1 为准。

## Naming的定义

代码在 `naming/naming.go` 中。

### 数据结构定义

Update struct 定义命名解析的更新。Addr和metadata在同一个update中不能同时为空。

```go
// Update defines a name resolution update. Notice that it is not valid having both
// empty string Addr and nil Metadata in an Update.
//
// Deprecated: please use package resolver.
type Update struct {
	// Op indicates the operation of the update.
	Op Operation
	// Addr is the updated address. It is empty string if there is no address update.
	Addr string
	// Metadata is the updated metadata. It is nil if there is no metadata update.
	// Metadata is not required for a custom naming implementation.
	Metadata interface{}
}
```

其中 operation 定义了用于命名解析变更的对应操作：

```go
// Operation defines the corresponding operations for a name resolution change.
//
// Deprecated: please use package resolver.
type Operation uint8

const (
	// Add indicates a new address is added.
	Add Operation = iota
	// Delete indicates an existing address is deleted.
	Delete
)
```

### 接口定义

Resolver 接口创建 Watcher，用于跟踪指定目标的解析变更。

```go
// Resolver creates a Watcher for a target to track its resolution changes.
//
// Deprecated: please use package resolver.
type Resolver interface {
	// Resolve creates a Watcher for target.
	Resolve(target string) (Watcher, error)
}
```

Watcher 接口用于监控特定目标的更新：

```go
// Watcher watches for the updates on the specified target.
//
// Deprecated: please use package resolver.
type Watcher interface {
  // Next 方法阻塞直至更新或者错误发生。
  // 可能返回一个或者多个更新。
  // 第一次调用因该得到结果的全集。
  // 当且仅当 Watcher 无法恢复时返回错误。
	Next() ([]*Update, error)
	// Close closes the Watcher.
	Close()
}
```

## dns resolver的实现

gRPC 只内置了这一个实现: `naming/dns_resolver.go`

### dns resolver的结构

dnsResolver 处理遵循dns模式的名称的命名解析。

```go
type dnsResolver struct {
   // 该解析器创建的 watcher 将使用的 DNS 服务器的轮询频率。
   freq time.Duration
}
```

### 构建dns resolver

```go
const (
  // 默认频率是30分钟。
	defaultFreq = time.Minute * 30
)

// NewDNSResolverWithFreq 创建 DNS Resolver，用于解析 DNS 名称，并创建watcher，使用 freq 参数设置的频率来查询DNS 服务器
func NewDNSResolverWithFreq(freq time.Duration) (Resolver, error) {
	return &dnsResolver{freq: freq}, nil
}

// NewDNSResolverWithFreq 创建 DNS Resolver，用于解析 DNS 名称，并创建watcher，使用默认的频率来查询DNS 服务器
func NewDNSResolver() (Resolver, error) {
	return NewDNSResolverWithFreq(defaultFreq)
}
```

### 解析dns

Resolve 方法创建watcher，watcher用来监控目标的命名解析。

```go
func (r *dnsResolver) Resolve(target string) (Watcher, error) {
	host, port, err := parseTarget(target)
	if err != nil {
		return nil, err
	}

  // 尝试一下检测host是不是一个IP地址
	if net.ParseIP(host) != nil {
    // 如果是IP地址，不用做dns解析，走 ipWatcher
		ipWatcher := &ipWatcher{
			updateChan: make(chan *Update, 1),
		}
		host, _ = formatIP(host)
		ipWatcher.updateChan <- &Update{Op: Add, Addr: host + ":" + port}
		return ipWatcher, nil
	}

  // 如果host不是IP地址，则走 dnsWatcher
	ctx, cancel := context.WithCancel(context.Background())
	return &dnsWatcher{
		r:      r,
		host:   host,
		port:   port,
		ctx:    ctx,
		cancel: cancel,
		t:      time.NewTimer(0),
	}, nil
}
```

细节代码：parseTarget 处理用户输入的 target 字符串，返回格式化后的 host 和 port 信息。

```go
// parseTarget takes the user input target string, returns formatted host and port info.
// If target doesn't specify a port, set the port to be the defaultPort.
// If target is in IPv6 format and host-name is enclosed in square brackets, brackets
// are stripped when setting the host.
// examples:
// target: "www.google.com" returns host: "www.google.com", port: "443"
// target: "ipv4-host:80" returns host: "ipv4-host", port: "80"
// target: "[ipv6-host]" returns host: "ipv6-host", port: "443"
// target: ":80" returns host: "localhost", port: "80"
// target: ":" returns host: "localhost", port: "443"
func parseTarget(target string) (host, port string, err error) {
   if target == "" {
      return "", "", errMissingAddr
   }

   if ip := net.ParseIP(target); ip != nil {
      // target is an IPv4 or IPv6(without brackets) address
      return target, defaultPort, nil
   }
   if host, port, err := net.SplitHostPort(target); err == nil {
      // target has port, i.e ipv4-host:port, [ipv6-host]:port, host-name:port
      if host == "" {
         // Keep consistent with net.Dial(): If the host is empty, as in ":80", the local system is assumed.
         host = "localhost"
      }
      if port == "" {
         // If the port field is empty(target ends with colon), e.g. "[::1]:", defaultPort is used.
         port = defaultPort
      }
      return host, port, nil
   }
   if host, port, err := net.SplitHostPort(target + ":" + defaultPort); err == nil {
      // target doesn't have port
      return host, port, nil
   }
   return "", "", fmt.Errorf("invalid target address %v", target)
}
```

细节代码：formatIP方法，如果addr不是一个有效的IP地址文本表示方式，则返回 ok=false。如果addr是IPv4地址，则返回addr和ok = true。如果addr是ipv6地址，则返回包含在"[]"中的addr和ok=true。

```go
func formatIP(addr string) (addrIP string, ok bool) {
   ip := net.ParseIP(addr)
   if ip == nil {
      return "", false
   }
   if ip.To4() != nil {
      return addr, true
   }
   return "[" + addr + "]", true
}
```

## ipWatcher的实现

ipWatcher 监控IP地址的命名解析更新

```go
type ipWatcher struct {
   updateChan chan *Update
}
```

Next方法 返回目标的地址解析更新。对于IP地址，解析结果是IP地址自身，因此不需要轮询 name server。因此，Next()在第一次调用时将返回一个Update，在关闭Watcher之前，由于不存在Update，因此后续的所有调用都会被阻塞。

```go
func (i *ipWatcher) Next() ([]*Update, error) {
   u, ok := <-i.updateChan
   if !ok {
      return nil, errWatcherClose
   }
   return []*Update{u}, nil
}

// Close closes the ipWatcher.
func (i *ipWatcher) Close() {
   close(i.updateChan)
}
```

结合 Resolve() 的调用一起看：

```go
if net.ParseIP(host) != nil {
	ipWatcher := &ipWatcher{
		// 构建update channel，缓存大小为1
		updateChan: make(chan *Update, 1),
	}
  // 格式化IP地址
	host, _ = formatIP(host)
  // 往 updateChan 中发一个Update：操作为Add，Addr为IP地址+端口
	ipWatcher.updateChan <- &Update{Op: Add, Addr: host + ":" + port}
	return ipWatcher, nil
}
```

## dnsWatcher的实现

```go
// dnsWatcher watches for the name resolution update for a specific target
type dnsWatcher struct {
   r    *dnsResolver
   host string
   port string
   // The latest resolved address set
   curAddrs map[string]*Update
   ctx      context.Context
   cancel   context.CancelFunc
   t        *time.Timer
}
```

Next 方法返回被解析的目标地址（增量）更新。如果没有变化，则默认为sleep30分钟然后尝试再次解析。

```go
func (w *dnsWatcher) Next() ([]*Update, error) {
   for {
      select {
      case <-w.ctx.Done():
         return nil, errWatcherClose
      case <-w.t.C:
      }
      result := w.lookup()
     // 下一次 lookup 应该在 w.r.freq 定义的间隔之后发生
      w.t.Reset(w.r.freq)
      if len(result) > 0 {
         return result, nil
      }
   }
}

func (w *dnsWatcher) Close() {
   w.cancel()
}
```

实现细节：lookup()方法先查查 SRV 记录，找不到再查 A 记录。

```go
func (w *dnsWatcher) lookup() []*Update {
   newAddrs := w.lookupSRV()
   if newAddrs == nil {
      // If failed to get any balancer address (either no corresponding SRV for the
      // target, or caused by failure during resolution/parsing of the balancer target),
      // return any A record info available.
      newAddrs = w.lookupHost()
   }
   result := w.compileUpdate(newAddrs)
   w.curAddrs = newAddrs
   return result
}
```

### SVR 记录的查询

查询名为 “grpclb” 的 SRV 记录：

```go
func (w *dnsWatcher) lookupSRV() map[string]*Update {
   newAddrs := make(map[string]*Update)
   _, srvs, err := lookupSRV(w.ctx, "grpclb", "tcp", w.host)
   if err != nil {
      grpclog.Infof("grpc: failed dns SRV record lookup due to %v.\n", err)
      return nil
   }
   for _, s := range srvs {
      lbAddrs, err := lookupHost(w.ctx, s.Target)
      if err != nil {
         grpclog.Warningf("grpc: failed load balancer address dns lookup due to %v.\n", err)
         continue
      }
      for _, a := range lbAddrs {
         a, ok := formatIP(a)
         if !ok {
            grpclog.Errorf("grpc: failed IP parsing due to %v.\n", err)
            continue
         }
         addr := a + ":" + strconv.Itoa(int(s.Port))
         newAddrs[addr] = &Update{Addr: addr,
            Metadata: AddrMetadataGRPCLB{AddrType: GRPCLB, ServerName: s.Target}}
      }
   }
   return newAddrs
}
```



### A记录的查询

```go
func (w *dnsWatcher) lookupHost() map[string]*Update {
   newAddrs := make(map[string]*Update)
   addrs, err := lookupHost(w.ctx, w.host)
   if err != nil {
      grpclog.Warningf("grpc: failed dns A record lookup due to %v.\n", err)
      return nil
   }
   for _, a := range addrs {
      a, ok := formatIP(a)
      if !ok {
         grpclog.Errorf("grpc: failed IP parsing due to %v.\n", err)
         continue
      }
      addr := a + ":" + w.port
      newAddrs[addr] = &Update{Addr: addr}
   }
   return newAddrs
}
```