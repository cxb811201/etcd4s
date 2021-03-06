# etcd4s

[![Build Status](https://travis-ci.org/mingchuno/etcd4s.svg?branch=master)](https://travis-ci.org/mingchuno/etcd4s)

A Scala etcd client implementing V3 API using gRPC and ScalaPB with optional Akka Stream support. This project is in beta stage with basic test coverage and usable APIs.

## Overview

This repo is a client library of [etcd](https://coreos.com/etcd/) implementing V3 [APIs](https://coreos.com/etcd/docs/latest/learning/api.html) using gRPC under the hood with optional Akka Stream support for stream APIs. This library implement the complete set of the APIs in the V3 protoal. More information about the APIs can be found here:

* [etcd V3 API overview](https://coreos.com/etcd/docs/latest/learning/api.html)
* [etcd V3 API Reference](https://coreos.com/etcd/docs/latest/dev-guide/api_reference_v3.html)
* [protobuf defination](https://github.com/mingchuno/etcd4s/tree/master/etcd4s-core/src/main/protobuf)

Note that this library do not support gRPC json gateway and use raw gRPC call instead (underlying is java-grpc). Also, this project cross build against Scala 2.11 and 2.12.

## Getting Started

The core lib

```scala
libraryDependencies += "com.github.mingchuno" %% "etcd4s-core" % "0.2.0"
```

To include akka stream support for stream API

```scala
libraryDependencies += "com.github.mingchuno" %% "etcd4s-akka-stream" % "0.2.0"
```

## Usage

```scala
import org.etcd4s.{Etcd4sClientConfig, Etcd4sClient}
import org.etcd4s.implicits._
import org.etcd4s.formats._
import org.etcd4s.pb.etcdserverpb._

import scala.concurrent.ExecutionContext.Implicits.global

// create the client
val config = Etcd4sClientConfig(
  address = "127.0.0.1",
  port = 2379
)
val client = Etcd4sClient.newClient(config)

// set a key
client.kvService.setKey("foo", "bar") // return a Future

// get a key
client.kvService.getKey("foo").foreach { result =>
  assert(result == Some("bar"))
}

// delete a key
client.kvService.deleteKey("foo").foreach { result =>
  assert(result == 1)
}

// set more key
client.kvService.setKey("foo/bar", "Hello")
client.kvService.setKey("foo/baz", "World")

// get keys with range
client.kvService.getRange("foo/").foreach { result =>
  assert(result.count == 2)
}

// remember to shutdown the client
client.shutdown()
```

If you want the Akka Stream support for the stream APIs, you should add the `etcd4s-akka-stream` depns into your `build.sbt`

```scala
import org.etcd4s.akkasupport._
import org.etcd4s.implicits._
import org.etcd4s.pb.etcdserverpb._
import akka.NotUsed

// assume you have the implicit value and client needed in the scope
val flow: Flow[WatchRequest, WatchResponse, NotUsed] = client.rpcClient.watchRpc.watchFlow
Source.single(WatchRequest().withCreateRequest(WatchCreateRequest().withKey("foo")))
  .via(flow)
  .runForeach { resp =>
    println(resp)
  }
```

More example usage under the test dir in the repo.

## Development

### Requirment

* Java 8+, Scala 12.12.X+, sbt and docker

```bash
# to start a background etcd for development
docker-compose up -d
```

### How to start?

Simple! Just `sbt test`

### Publish

This is to remind me how to publish and may switch to `sbt-release` later

1. make sure you have `~/.sbt/gpg/` ready with pub/sec key paris
2. make sure you have `~/.sbt/1.0/sonatype.sbt` ready with credentials
3. `sbt "+clean" "+compile"`
4. `sbt "+publishSigned"`
5. `sbt sonatypeReleaseAll`
