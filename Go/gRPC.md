1. 这里的例子写得很清楚了，gRPC提供专门的protocol buffer compiler（叫做`protoc`）编译任何语言定义的（用户自己选择）.proto文件，生成相应的data access classes，大大减小了用户自己编写RPC消息结构体内部成员access method等操作的工作量
	https://grpc.io/docs/languages/cpp/quickstart/
2. 其次：
gRPC uses `protoc` with a special gRPC plugin to generate code from your proto file: you get generated gRPC client and server code, as well as the regular protocol buffer code for populating, serializing, and retrieving your message types
