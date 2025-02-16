# pb2tarsgo

## 用法

* install protoc [https://github.com/protocolbuffers/protobuf/releases](https://github.com/protocolbuffers/protobuf/releases)

* Install protoc-gen-go and protoc-gen-go-tarsrpc

```bash
export PATH=$PATH:$GOPATH/bin
# < go 1.16
go get -u google.golang.org/protobuf/cmd/protoc-gen-go
go get -u github.com/TarsCloud/TarsGo/tars/tools/protoc-gen-go-tarsrpc
# >= go 1.16
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install github.com/TarsCloud/TarsGo/tars/tools/protoc-gen-go-tarsrpc@latest
```

## 示例

* 创建项目目录

```shell
mkdir pb2tarsgo
cd pb2tarsgo
go mod init pb2tarsgo
```

* helloworld.proto 文件

```protobuf
syntax = "proto3";
package helloworld;
option go_package = "pb2tarsgo/helloworld";

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

* 生成代码

```shell
protoc --go_out=. --go-tarsrpc_out=. helloworld.proto
```

* 服务端 `mian.go`

```go
package main

import (
    "github.com/TarsCloud/TarsGo/tars"
    "pb2tarsgo/helloworld" 
)

type GreeterImp  struct {
}

func (imp *GreeterImp) SayHello(input helloworld.HelloRequest)(output helloworld.HelloReply, err error) {
    output.Message = "hello" +  input.GetName() 
    return output, nil 
}

func main() {
	  // Init servant
    imp := new(GreeterImp)                                      // New Imp
    app := new(helloworld.Greeter)                              // New init the A JCE
    cfg := tars.GetServerConfig()                               // Get Config File Object
    app.AddServant(imp, cfg.App+"."+cfg.Server+".GreeterTestObj") //Register Servant
    tars.Run()
}
	
```

* 客户端 `client/client.go`

```go
package main

import (
    "fmt"
    "github.com/TarsCloud/TarsGo/tars"
    "pb2tarsgo/helloworld"
)

func main() {
    comm := tars.NewCommunicator()
    obj := fmt.Sprintf("StressTest.HelloPbServer.GreeterTestObj@tcp -h 127.0.0.1  -p 10014  -t 60000")
    app := new(helloworld.Greeter)
    comm.StringToProxy(obj, app)
    input := helloworld.HelloRequest{Name: "sandyskies"}
    output, err := app.SayHello(input)
    if err != nil {
        fmt.Println("err: ", err)
    }   
    fmt.Println("result is:", output.Message)
}
```

* 配置文件 `config.conf`

```xml
<tars>
    <application>
        <server>
            app=StressTest
            server=HelloPbServer
            local=tcp -h 127.0.0.1 -p 10014 -t 30000
            logpath=/tmp
            <StressTest.HelloPbServer.GreeterTestObjAdapter>
                allow
                endpoint=tcp -h 127.0.0.1 -p 10014 -t 60000
                handlegroup=StressTest.HelloPbServer.GreeterTestObjAdapter
                maxconns=200000
                protocol=tars
                queuecap=10000
                queuetimeout=60000
                servant=StressTest.HelloPbServer.GreeterTestObj
                shmcap=0
                shmkey=0
                threads=1
            </StressTest.HelloPbServer.GreeterTestObjAdapter>
        </server>
    </application>
</tars>
```