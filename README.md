# rpcx

[**中文介绍**](README-CN.md)

[![GoDoc](https://godoc.org/github.com/smallnest/rpcx?status.png)](http://godoc.org/github.com/smallnest/rpcx) [![Drone Build Status](https://drone.io/github.com/smallnest/rpcx/status.png)](https://drone.io/github.com/smallnest/rpcx/latest) [![Go Report Card](https://goreportcard.com/badge/github.com/smallnest/rpcx)](https://goreportcard.com/report/github.com/smallnest/rpcx)
![travis](https://travis-ci.org/smallnest/rpcx.svg?branch=master)

rpcx is a distributed RPC framework like [Alibaba Dubbo](http://dubbo.io/) and [Weibo Motan](https://github.com/weibocom/motan).
It is developed based on Go net/rpc and provides extra governance features.


![Benchmark](_documents/benchmark.png)

![Throughput](_documents/throughput.png)

When we talk about RPC frameworks, Dubbo is first framework we should introduced, and there is also Dubbox mantained by dangdang.
Dubbo has been widely used in e-commerce companies in China, for example, Alibaba, Jingdong and Dangdang.

Though Dubbo has still used Spring 2.5.6.SEC03 and seems has not been supported by Alibaba no longer, 
some other companies still use it and maintained their branches.

> DUBBO is a distributed service framework , provides high performance and transparent RPC remote service call. 
> It is the core framework of Alibaba SOA service governance programs. 
> There are 3,000,000,000 calls for 2,000+ services per day, 
> and it has been widely used in various member sites of Alibaba Group.

Motan is open source now by Weibo. As Zhang Lei said, he is current main developer of Motan:

> Motan started in 2013. There are 100 billion calls for hundreds of service callsevery day.

Those two RPC frameworks are developed by Java. 
There are other famous RPC frameworks such as [thrift](https://thrift.apache.org/)、[finagle](https://github.com/twitter/finagle)。

Goal of [rpcx](https://github.com/smallnest/rpcx) is implemented a RPC framework like Dubbo in Go ecosphere.
It is developed by Go, and for Go. 

It is a distributed、plugable RPC framework with governance (service discovery、load balancer、fault tolerance、monitor, etc.).

As you know, there are some RPC frameworks, for example, [net/rpc](https://golang.org/pkg/net/rpc/)、[grpc-go](https://github.com/grpc/grpc-go)、[gorilla-rpc](http://www.gorillatoolkit.org/pkg/rpc),
Then why re-invent a wheel?

Although those Go RPC frameworks work well, but their function is relatively simple and only implement end-to end communications.
Some product features of service management functions are lack, such as service discovery,
Load balancing, fault tolerance. 

So I created rpcx and expect it could become a RPC framework like Dubbo.


The similar project is [go-micro](https://github.com/micro/go-micro). 


## What's RPC
From wikiPedia:

> In distributed computing, a remote procedure call (RPC) is when a computer program causes a procedure (subroutine) to execute in another address space (commonly on another computer on a shared network), which is coded as if it were a normal (local) procedure call, without the programmer explicitly coding the details for the remote interaction. That is, the programmer writes essentially the same code whether the subroutine is local to the executing program, or remote.[1] This is a form of client–server interaction (caller is client, executer is server), typically implemented via a request–response message-passing system. The object-oriented programming analog is remote method invocation (RMI). The RPC model implies a level of location transparency, namely that calling procedures is largely the same whether it is local or remote, but usually they are not identical, so local calls can be distinguished from remote calls. Remote calls are usually orders of magnitude slower and less reliable than local calls, so distinguishing them is useful.
>
>RPCs are a form of inter-process communication (IPC), in that different processes have different address spaces: if on the same host machine, they have distinct virtual address spaces, even though the physical address space is the same; while if they are on different hosts, the physical address space is different. Many different (often incompatible) technologies have been used to implement the concept.


Sequence of events during an RPC
1. The client calls the client stub. The call is a local procedure call, with parameters pushed on to the stack in the normal way.
2. The client stub packs the parameters into a message and makes a system call to send the message. Packing the parameters is called marshalling.
3. The client's local operating system sends the message from the client machine to the server machine.
4. The local operating system on the server machine passes the incoming packets to the server stub.
5. The server stub unpacks the parameters from the message. Unpacking the parameters is called unmarshalling.
6. Finally, the server stub calls the server procedure. The reply traces the same steps in the reverse direction.


There are two ways to implement RPC frameworks. 
One focusses on cross-language calls and the other focusses on service governance.

Dubbo、DubboX、Motan are RPC framework of service governance .
Thrift、gRPC、Hessian、Hprose are RPC framework of cross-language calls.

rpcx is a RPC framework of service governance.

## Features

[more features](feature)


* bases on net/rpc. a Go net/prc project can be converted rpcx project whit few changes.
* Plugable. Features are implemented by Plugins such as service discovery.
* Commnuicates with TCP long connections.
* support many codec. for example, Gob、Json、MessagePack、gencode、ProtoBuf.
* Service dicovery. support ZooKeeper、Etcd.
* Fault tolerance：Failover、Failfast、Failtry.
* Load banlancer：support randomSelecter, RoundRobin, consistent hash etc.
* scalable.
* Other: metrics、log.
* Authorization.

## Architecture
rpcx contains three roles : RPC Server，RPC Client and Registry.
* Server registers services on Registry
* Client queries service list and select a server from server list returned from Registry.
* When a Server is down, Registry can remove this server and then client can remove it too.

![](https://raw.githubusercontent.com/smallnest/rpcx/master/_documents/images/component.png)

So far rpcx support zookeeper, etcd as Registry，Consul support is developing。

## Benchmark

**Test Environment**
* CPU:    Intel(R) Xeon(R) CPU E5-2620 v2 @ 2.10GHz, 24 cores
* Memory: 16G
* OS:     Linux Server-3 2.6.32-358.el6.x86_64, CentOS 6.4
* Go:     1.6.2

Test request is copied from protobuf and encoded to a proto message. Its size is 581 bytes.
The response is same to test request but server has handled the decoding and encoding processing.

The concurrent clients are 100, 1000,2000 and 5000. Count of the total requests for all clients are 1,000,000.

**Test Result**

concurrent clients|mean(ms)|median(ms)|max(ms)|min(ms)|throughput(TPS)
-------------|-------------|-------------|-------------|-------------|-------------
100|1|0|96|0|100694
500|3|2|151|0|121212
1000|6|4|167|0|119146
2000|11|10|472|0|32047
5000|27|24|442|0|15799

If you use too many clients, the throughput (transations per second) will be worse. It looks 1000 clients is feasible.

When you use clients, clients should be shared as possible.


you can use test code in `_benchmark` to test.
`server` is used to start a server and `client` is used as clients via protobuf.



The above test is that client and server are running on the same mechine.
If I run them on separate servers, test results are:


concurrent clients|mean(ms)|median(ms)|max(ms)|min(ms)|throughput(TPS)
-------------|-------------|-------------|-------------|-------------|-------------
100|1|0|18|0|91066
500|4|1|1230|0|103241
1000|5|1|1420|0|95219
2000|12|2|3092|0|97323
5000|26|2|12726|0|69454


If they are running on cluster mode, one is for the client and two are for two servers, test results are:

concurrent clients|mean(ms)|median(ms)|max(ms)|min(ms)|throughput(TPS)
-------------|-------------|-------------|-------------|-------------|-------------
100|0|0|10|0|136440
500|2|1|808|0|157927
1000|5|2|7275|0|114916
2000|8|1|7584|0|96627
5000|21|1|6387|0|97096

Basically its throughput is greater than throughput of single node and less than 2 times of throughput of single node.




The below lists benchmarks of serialization libraries:

```
[root@localhost rpcx]# go test -bench . -test.benchmem
PASS
BenchmarkNetRPC_gob-16            100000             18742 ns/op             321 B/op          9 allocs/op
BenchmarkNetRPC_jsonrpc-16        100000             21360 ns/op            1170 B/op         31 allocs/op
BenchmarkNetRPC_msgp-16           100000             18617 ns/op             776 B/op         35 allocs/op
BenchmarkRPCX_gob-16              100000             18718 ns/op             320 B/op          9 allocs/op
BenchmarkRPCX_json-16             100000             21238 ns/op            1170 B/op         31 allocs/op
BenchmarkRPCX_msgp-16             100000             18635 ns/op             776 B/op         35 allocs/op
BenchmarkRPCX_gencodec-16         100000             18454 ns/op            4485 B/op         17 allocs/op
BenchmarkRPCX_protobuf-16         100000             17234 ns/op             733 B/op         13 allocs/op
```

## Comparision with gRPC
[gRPC](https://github.com/grpc/grpc-go) is the RPC framework by Google. It support multiple programming lanaguage.
I have compared three cases for prcx and gRPC. It shows rpcx is much better than gRPC.

Test results of rpcx has been listed on the above. Here is test results of gRPC.

### one client and one server in a same machine
concurrent clients|mean(ms)|median(ms)|max(ms)|min(ms)|throughput(TPS)
-------------|-------------|-------------|-------------|-------------|-------------
100|1|1|25|0|50040
500|8|7|63|0|57313
1000|16|13|115|0|60488
2000|30|26|115|0|62367
5000|73|67|349|0|59421 

![](_documents/images/rpcx-grpc-1.png)

### one client and one server in two machines
concurrent clients|mean(ms)|median(ms)|max(ms)|min(ms)|throughput(TPS)
-------------|-------------|-------------|-------------|-------------|-------------
100|1|1|20|0|59168
500|5|1|4350|0|73524
1000|10|2|3233|0|79974
2000|17|2|9735|0|49185
5000|44|2|12788|0|52770 

![](_documents/images/rpcx-grpc-2.png)

### one client on a machine and three servers in three machines
concurrent clients|mean(ms)|median(ms)|max(ms)|min(ms)|throughput(TPS)
-------------|-------------|-------------|-------------|-------------|-------------
100|1|0|17|0|71895
500|5|3|670|0|88347
1000|10|5|2456|0|85273
2000|19|12|2465|0|86169
5000|51|40|6358|0|82243 

![](_documents/images/rpcx-grpc-2.png)
