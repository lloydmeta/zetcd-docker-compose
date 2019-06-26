# zetcd docker-compose

Bare-bones [`docker-compose`](https://docs.docker.com/compose/) file that gets you a [zetcd](https://github.com/etcd-io/zetcd) ([Zookeeper](https://zookeeper.apache.org) "personality" for [etcd](https://etcd.io/)) running at `2181`.

# Usage

* Clone this repo, do `docker-compose up` in the directory you've cloned into.

* Your `etcd`-backed ZK will be at `2181`

    ```
    15:56 $ zkctl watch / &
    [1] 17542
    watch dir /
    2019/06/26 16:33:53 Connected to 127.0.0.1:2181
    2019/06/26 16:33:53 Authenticated: id=112426793504996108, timeout=1000
    2019/06/26 16:33:53 Re-submitting `0` credentials after reconnect
    [] &{Czxid:0 Mzxid:0 Ctime:0 Mtime:0 Version:0 Cversion:-1 Aversion:0 EphemeralOwner:0 DataLength:0 NumChildren:1 Pzxid:0}
    ✔ ~/Documents/dokka/zetcd-compose [master L|…3]
    16:33 $ zkctl create /abc "foo"
    2019/06/26 16:33:55 Connected to 127.0.0.1:2181
    2019/06/26 16:33:55 Authenticated: id=112426793504996111, timeout=1000
    2019/06/26 16:33:55 Re-submitting `0` credentials after reconnect
    {Type:EventNodeChildrenChanged State:Unknown Path:/ Err:<nil> Server:}
    [1]+  Done                    zkctl watch /
    ```

## Findings

zetcd isn't compatible with [3.5 opcodes](https://github.com/etcd-io/zetcd/issues/49), so the latest `CuratorFramework` won't work.

In particular, `create2` is not supported (see [other opcodes](https://zookeeper.apache.org/doc/r3.5.3-beta/api/org/apache/zookeeper/ZooDefs.OpCode.html#create2))

```scala
16:30 $ amm
Loading...
Welcome to the Ammonite Repl 1.0.5
(Scala 2.12.4 Java 1.8.0_152)
If you like Ammonite, please support our development at www.patreon.com/lihaoyi
@ import $ivy.{`org.apache.curator:curator-framework:4.2.0`}
import $ivy.$

@ import org.apache.curator.framework.CuratorFrameworkFactory
import org.apache.curator.framework.CuratorFrameworkFactory

@ import org.apache.curator.retry._
import org.apache.curator.retry._

@
CuratorFrameworkFactory.builder().connectString("localhost:2181").retryPolicy(new RetryOneTime(0)).build()
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
res3: org.apache.curator.framework.CuratorFramework = org.apache.curator.framework.imps.CuratorFrameworkImpl@3dc39412

@ res3.start
SLF4J: Failed to load class "org.slf4j.impl.StaticMDCBinder".
SLF4J: Defaulting to no-operation MDCAdapter implementation.
SLF4J: See http://www.slf4j.org/codes.html#no_static_mdc_binder for further details.
r

@ res3.blockUntilConnected


@ res3.create().orSetData().forPath("/test", Array())
org.apache.zookeeper.KeeperException$ConnectionLossException: KeeperErrorCode = ConnectionLoss for /test
  org.apache.zookeeper.KeeperException.create(KeeperException.java:102)
  org.apache.zookeeper.KeeperException.create(KeeperException.java:54)
  org.apache.zookeeper.ZooKeeper.create(ZooKeeper.java:1549)
  org.apache.curator.framework.imps.CreateBuilderImpl$17.call(CreateBuilderImpl.java:1180)
  org.apache.curator.framework.imps.CreateBuilderImpl$17.call(CreateBuilderImpl.java:1156)
  org.apache.curator.connection.StandardConnectionHandlingPolicy.callWithRetry(StandardConnectionHandlingPolicy.java:64)
  org.apache.curator.RetryLoop.callWithRetry(RetryLoop.java:100)
  org.apache.curator.framework.imps.CreateBuilderImpl.pathInForeground(CreateBuilderImpl.java:1153)
  org.apache.curator.framework.imps.CreateBuil
```

You'll see the following logs:

```
zetcd_1  | unknown opcode  15
zetcd_1  | I0626 07:31:50.126451       1 server.go:128] accepted remote connection "172.21.0.1:34314"
zetcd_1  | I0626 07:31:50.126820       1 authconn.go:53] auth(&{ProtocolVersion:0 LastZxidSeen:5 TimeOut:60000 SessionID:112426793504996099 Passwd:[48 227 153 200 59 247 69 155 193 11 94 238 16 121 18 32]})
zetcd_1  | I0626 07:31:50.129784       1 pool.go:92] authresp=&{ProtocolVersion:0 TimeOut:60000 SessionID:112426793504996099 Passwd:[48 227 153 200 59 247 69 155 193 11 94 238 16 121 18 32]}
zetcd_1  | I0626 07:31:50.129929       1 server.go:73] serving serial session requests on id=18f6b92b219fb03
zetcd_1  | I0626 07:31:50.129958       1 session.go:59] starting the session... id=112426793504996099
zetcd_1  | unknown opcode  15
zetcd_1  | I0626 07:31:50.131183       1 server.go:110] zkreq={xid:5 err:"ptr expected"}
zetcd_1  | I0626 07:31:50.131455       1 session.go:61] finishing the session... id=112426793504996099; expect revoke...
zetcd_1  | I0626 07:31:50.970954       1 server.go:128] accepted remote connection "172.21.0.1:34316"
zetcd_1  | I0626 07:31:50.971482       1 authconn.go:53] auth(&{ProtocolVersion:0 LastZxidSeen:5 TimeOut:60000 SessionID:112426793504996099 Passwd:[48 227 153 200 59 247 69 155 193 11 94 238 16 121 18 32]})
zetcd_1  | I0626 07:31:50.975287       1 pool.go:92] authresp=&{ProtocolVersion:0 TimeOut:60000 SessionID:112426793504996099 Passwd:[48 227 153 200 59 247 69 155 193 11 94 238 16 121 18 32]}
zetcd_1  | I0626 07:31:50.977250       1 session.go:59] starting the session... id=112426793504996099
zetcd_1  | I0626 07:31:50.976731       1 server.go:73] serving serial session requests on id=18f6b92b219fb03
zetcd_1  | I0626 07:31:50.978180       1 server.go:110] zkreq={xid:7 req:*zetcd.GetDataRequest:&{Path:/zookeeper/config Watch:true}}
```
