# gRPC

## gRPC란 무엇인가

-   json을 사용하는 RESTful HTTP API와 같이 **client-server 또는 server-server간의 데이터를 주고 받을 때 사용되는 통신방법**이다.
-   가장 큰 차이점은 요청, 응답하는 데이터의 규격이다. RESTful HTTP API에선 uri로 요청하고 json이나 xml과 같은 데이터 형식으로 응답했다. 사람이 읽을 수 있는 readable data이다. gRPC에선 proto request와 proto response라는 데이터로 통신하며, 이는 사람이 읽을 수 없는 데이터 형식이기 때문에 네트워크 단에서 데이터를 보고 싶으면 추가적인 작업들이 필요하다.
-   RPC는 Remote Procedure Call로 원격 프로시저 콜이다. 이는 분산환경 시스템에서 서로 다른 컴퓨터 프로그램들이 서로의 네트워크 주소 등에 대한 정보 없이 함수호출만으로 데이터를 주고 받는 방식이다. gRPC의 g는 google의 약자로 google에서 만든 protobuf 데이터 방식을 사용하는 RPC의 형태라고 볼 수 있다.
-   MSA에서 사용하기에 적합하다.
-   **장점**
    -   JSON이나 XML에 비해 데이터의 크기가 매우 작고 속도도 빠르다.
    -   protobuf로 어떻게 통신하는지를 proto파일에 정의해야한다. 코드로 가이드를 생성하기 때문에 따로 API문서를 만들 필요없이 엄격한 규격이 생겨서 협업에 도움이 된다.
    -   HTTP/2를 지원하여 동일한 연결로 병렬적인 요청을 처리할 수 있고, 연결을 유지해서 connection을 매번 하는데 사용되는 cost도 줄일 수 있다.
-   **단점**
    -   아직 브라우저-서버간의 통신을 지원하지 않는다. (하지만 grpc-gateway으로 해결가능)
    -   사람이 읽을 수 없는 데이터 형식이기 때문에 네트워크 단에서 데이터를 보고 싶으면 추가적인 작업들이 필요하다.

## .proto파일 컴파일 방법

1. [컴파일러 다운로드](https://developers.google.com/protocol-buffers/docs/gotutorial#compiling-your-protocol-buffers)
2. Golang protobuf plugin 설치
    ```
    go install google.golang.org/protobuf/cmd/protoc-gen-go
    ```
3. 컴파일

    ```
    // 예시
    protoc -I=. \
            --go_out . --go_opt paths=source_relative \
            --go-grpc_out . --go-grpc_opt paths=source_relative \
            protos/v1/user/user.proto

    // 결과
    // protos/v1/user 디렉토리에 user.proto를 컴파일해서 만들어진 user.pb.go와 user_grpc.pb.go 생성
    ```

## gRPC구현

-   proto파일로 rpc에 대한 정의와 요청, 응답 규격을 정의 및 컴파일 (ex: [/protos/v1/user](https://github.com/GreatLaboratory/go-grpc-example/tree/master/protos/v1/user))
-   정의해놓은 rpc 구현 및 gRPC서버 등록 (ex: [/simple-client-server/user-server/main.go](https://github.com/GreatLaboratory/go-grpc-example/tree/master/simple-client-server/user-server/main.go))

## gRPC server들간의 통신

1.  Unary : client가 request를 보내면 server로부터 response가 올때까지 기다리는 방식

    ```
    rpc GetUser(GetUserRequest) returns (GetUserResponse);
    ```

    만약 post grpc server에서 user grpc server를 호출한다면 post server의 rpc가 호출되었을 때 user server한테 user id를 전달하고 유저의 정보를 리턴할 때까지 기다렸다.

2.  Server-side streaming : client가 Unary와 같이 request를 보내는데 server는 stream으로 메세지를 리턴하는 방식

    ```
    rpc GetUser(GetUserRequest) returns (stream GetUserResponse);
    ```

    Client는 단일의 메세지를 바로 받고 끝내는 것이 아니라, server가 전달한 stream을 구독하고 있고 메세지가 더 없을 때까지 계속 구독하는 것이다.

    한번에 큰 데이터를 리턴하게 되면 client는 데이터를 받기 까지 계속 blocking이 되어있어서 다른 작업들을 하지 못하게 된다. 이를 해결하기 위해 server-side streaming 방식을 사용할 수 있다고 보면 된다.

3.  Client-side streaming : Server-side streaming의 반대

    ```
    rpc GetUser(stream GetUserRequest) returns (GetUserResponse);
    ```

4.  Bidirectional streaming : client와 server가 둘다 stream방식으로 서로 주고 받는 방식

    ```
    rpc GetUser(stream GetUserRequest) returns (stream GetUserResponse);
    ```

    2개의 stream은 각각 독립적이여서 client나 server는 어떤 순서로도 동작이 가능하다. 예를 들면, server는 client가 stream으로 request를 다 보낼때까지 기다리고 나서 response를 주던지, 혹은 request가 올 때마다 바로 response를 보낼 것인지 자율적으로 할 수 있다.

    이런 stream을 지원하는 것이 gRPC의 큰 장점중 하나이고 이는 gRPC가 HTTP2를 지원하기 때문에 가능하다. HTTP2에서는 동시에 그리고 순서에 상관없이 양방향으로 데이터를 stream으로 받아볼 수 있기 때문이다.

출처: [DevJin-Blog](https://devjin-blog.com/golang-grpc-server-1/)
