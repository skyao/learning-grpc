# URI 术语

> 注： 后面大量章节涉及到 `URI` 标准中的诸多术语，为了概念清晰，单独开一章来介绍。

## URI的语法

通用 URI 和绝对 URI 的语法参考最初定义在 [RFC 2396](https://en.wikipedia.org/wiki/Request_for_Comments),发布于1998年，并定稿于  RFC 3986， 发布于 2005年。

通用 URI 的形式如下：

	scheme:[//[user:password@]host[:port]][/]path[?query][#fragment]

它包括：

- ** scheme **

	由一系列字符组成，以字母开头，跟着有字母，数字，加号(+)，点号(.)或者中划线(-)的任何组合。

    虽然 scheme 是大小写不敏感，但是权威形式是小写并注明特定 scheme 必须也同样是小写。后面跟一个冒号(:)。流行的 scheme 例子包括 `http`, `ftp`, `mailto`, `file`, 和 `data` 。URI scheme 应该在 [Internet Assigned Numbers Authority (IANA)](https://en.wikipedia.org/wiki/Internet_Assigned_Numbers_Authority) 注册,虽然不注册的 scheme 实际也在使用。

- **双斜杠(//)**

	某些 scheme 要求有这个， 而有些 scheme 不要求。当 authority 部分缺失时， path 部分不能以双斜杠开始。

- **authority部分**

	包括：

    * 可选的认证部分，有用户名和密码，冒号分隔，带一个@符号
    * **host**,包括一个被注册名字(包括但是不限于hostname)，或者 IP 地址。IPv4 地址必须以点号分割的形式，而 IPv6 地址必须加[]符号。
    * 可选的**端口号**，和hostname之间用冒号分隔。

- **路径**

	包含数据，通常以层次形式组织，表现为一系列斜杠分隔的部分。这样一个序列可能类似或者映射到文件系统路径，但是并不暗示一定和某个路径有关系。如果有 authortity 部分，则路径必须以单斜杠开始，

- 可选的**query**

	和之前的部分以问号(?)分隔，包含非层次数据的 query string 。它的语法没有很好的定义，但是习惯上通常是一系列的属性值对，以分隔符分隔。

- 可选的**fragment/片段***

	和之前的部分以#分隔。fragment 包括一个 fragment identifier / 片段标识，提供到间接资源的引导，就像文章中的章节头，通过URI的剩余部分定位。当首要资源是 HTML 文档时，fragment 通常是一个特别原始的 id 属性，而web浏览器将滚动这个元素到视野中。

## 例子

下面的图片展示了两种URI的例子，和他们的组成部分。

```html
                    hierarchical part
        ┌───────────────────┴─────────────────────┐
                    authority               path
        ┌───────────────┴───────────────┐┌───┴────┐
  abc://username:password@example.com:123/path/data?key=value&key2=value2#fragid1
  └┬┘   └───────┬───────┘ └────┬────┘ └┬┘           └─────────┬─────────┘ └──┬──┘
scheme  user information     host     port                  query         fragment
```


```html
  urn:example:mammal:monotreme:echidna
  └┬┘ └──────────────┬───────────────┘
scheme              path
```

## 特别强调

authority代表URI中的 [userinfo@]host[:port]，包括host(或者ip)和可选的port和userinfo。

这个术语平时用的不多，但是在gRPC中频繁出现，请牢记：authority 代表 `username:password@example.com:123` 这一段。

## 参考资料

- https://en.wikipedia.org/wiki/Uniform_Resource_Identifier