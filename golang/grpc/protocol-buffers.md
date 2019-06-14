# ProtocolBuffers

<!-- TOC -->

- [ProtocolBuffers](#protocolbuffers)
  - [What are protocol buffers?](#what-are-protocol-buffers)
  - [How do they work?](#how-do-they-work)
  - [proto3](#proto3)
    - [定义消息类型](#%E5%AE%9A%E4%B9%89%E6%B6%88%E6%81%AF%E7%B1%BB%E5%9E%8B)
    - [分配字段编号](#%E5%88%86%E9%85%8D%E5%AD%97%E6%AE%B5%E7%BC%96%E5%8F%B7)
    - [指定字段规则](#%E6%8C%87%E5%AE%9A%E5%AD%97%E6%AE%B5%E8%A7%84%E5%88%99)
    - [添加更多消息类型](#%E6%B7%BB%E5%8A%A0%E6%9B%B4%E5%A4%9A%E6%B6%88%E6%81%AF%E7%B1%BB%E5%9E%8B)
    - [添加注释](#%E6%B7%BB%E5%8A%A0%E6%B3%A8%E9%87%8A)
    - [保留字段](#%E4%BF%9D%E7%95%99%E5%AD%97%E6%AE%B5)
    - [Scalar值类型](#scalar%E5%80%BC%E7%B1%BB%E5%9E%8B)
    - [默认值](#%E9%BB%98%E8%AE%A4%E5%80%BC)
    - [枚举](#%E6%9E%9A%E4%B8%BE)
    - [import proto](#import-proto)
    - [内嵌类型](#%E5%86%85%E5%B5%8C%E7%B1%BB%E5%9E%8B)
    - [更新消息类型](#%E6%9B%B4%E6%96%B0%E6%B6%88%E6%81%AF%E7%B1%BB%E5%9E%8B)
    - [Any](#any)
    - [Oneof](#oneof)
      - [Oneof 特性](#oneof-%E7%89%B9%E6%80%A7)
      - [标签重用问题](#%E6%A0%87%E7%AD%BE%E9%87%8D%E7%94%A8%E9%97%AE%E9%A2%98)
    - [Maps](#maps)
    - [定义服务](#%E5%AE%9A%E4%B9%89%E6%9C%8D%E5%8A%A1)
    - [JSON 映射](#json-%E6%98%A0%E5%B0%84)
  - [参考](#%E5%8F%82%E8%80%83)

<!-- /TOC -->

这篇文章我们来介绍一下 Protocol Buffers , 官方地址 [https://developers.google.com/protocol-buffers/docs/overview](https://developers.google.com/protocol-buffers/docs/overview)

## What are protocol buffers?

`Protocol Buffers` 是一种灵活，高效、自动化的机制用来序列化结构化数据。有些类似于 json,xml，但是更简单，更小，更高效。我们可以再一个文件中定义好结构化数据，然后使用工具生成代码，和读取各种数据流。

## How do they work?

前面的文章中，我们介绍过，就是在 `.proto` 文件中定义消息类型，并指定这些消息的内部字段属性，就可以了。

例如下面这个基于golang的 proto 定义

```proto
message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phone = 4;
}
```

在上面的示例中，`Person`消息包含 `PhoneNumber`消息，而`AddressBook`消息包含`Person`消息。我们甚至可以定义嵌套在其他消息中的消息类型 ，例如`PhoneNumber`类型定义在`Person`内部，而且还可以定义enum，用来指定PhoneNum是哪里的，HOMW,WORK ...

使用了上面的定义之后，我们就可以使用protoc编译器，来生成相应语言的代码了，同时还会一并生成相应的Get和Set方法，以及marshal和unmarshal结构化数据的方法。参考我们前面的例子。

## proto3

接下来，我们来介绍一下 proto3(新版本的ProtocolBuffers),以及`.proto`文件的语法。

### 定义消息类型

先来看一个简单的例子。假设我们需要定义一个查询请求，这个message 包含一个查询语句 `query`，包含的页数 `page_number`,每页包含的结果数`result_per_page`.那么我么的 `proto` 文件可以像下面这样定义。

```proto
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

- `syntax = "proto3"` 必须位于首行,第一个非注释行，如果不指定，默认使用`proto2`
- `SearchRequest`指定了三个字段(name/value 对),每个field都有自己的type。

具体的field type 可以参考 [Scalar Value Types](https://developers.google.com/protocol-buffers/docs/proto3#scalar)

### 分配字段编号

从上面的定义中可以看出，每个字段都有一个唯一编号，这些数字，用于在将message二进制序列化时标识字段。这里有一点需要注意，1~15范围需要一个byte来进行编码，包括字段编号和字段类型，可以从[Protocol Buffer Encoding](https://developers.google.com/protocol-buffers/docs/encoding#structure)查到相关内容。 16~2047 占用两个字节。所以一般会把 1~15留给哪些频繁出现的字段。或者是预留一些空间给将来可能频繁出现的元素。

字段编号最小可以指定为1，最大  2^29-1 。但是 19000~19999 这1000个数不要使用，这是protocol保留的。

### 指定字段规则

字段规则一般就下面两种：

- `singular` : 可以包含零个或一个，默认的
- `repeated` : 可以重复多次，包括零次。同时，其重复的顺序也会被保留。

在 `proto3` 中，简单数字类型的 `repeated` 字段默认使用 `packed` 编码。

详细内容可以参考 [Protocol Buffer Encoding](https://developers.google.com/protocol-buffers/docs/encoding#packed)

### 添加更多消息类型

简单 略过

### 添加注释

简单 略过

### 保留字段

在更新message type时，假设需要彻底删除一个field，或者注释掉，这样未来其他人就可以继续使用之前分配给这个字段的编号。如果他们以后加载了相同 `.proto` 文件的旧版本，这可能会导致严重问题，包括数据损坏，隐私错误等。有一个办法就是将要删除的字段设置为保留字段，未来任何用户试图使用这个字段的时候，protocol buffer 的编译器就会告警提示。

```protocol
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

但是要注意不能在同一个保留字段声明中混合使用数字和 Field name.

### Scalar值类型

这个可以参考官方网站 [Scalar Value Types](https://developers.google.com/protocol-buffers/docs/proto3#scalar)

这里主要把java和golang的值列举一下，其他的可以参考官方网站。

.proto type | java type | golang type |
-|-|-
double|double|float64
float|float|float32
int32|int|int32
int64|long|int64
uint32|int|uint32
uint64|long|uint64
sint32|int|int32
sint64|long|int64
fixed32|int|uint32
fixed64|long|uint64
sfixed32|int|int32
sfixed64|long|int64
bool|boolean|bool
string|String|string
bytes|ByteString|[]byte

### 默认值

- 对于strings, 默认值是空字符串(注, 是"", 而不是null)
- 对于bytes, 默认值是空字节(注, 应该是byte[0], 注意这里也不是null)
- 对于boolean, 默认值是false.
- 对于数字类型, 默认值是0.
- 对于枚举, 默认值是第一个定义的枚举值, 而这个值必须是0.
- 对于消息字段, 默认值是null.

对于重复字段, 默认值是空(通常都是空列表)

注意: 对于简单字段, 当消息被解析后, 如果值恰巧和默认值相同(例如一个boolean设置为false)是没有办法知道这个字段到底是有设置值还是取了默认值。这样就要求，不要根据默认值来采取某些切换行为，例如当某个 boolean 值为false时，切换状态。同样的，如果一个字段被设置了默认值，这个值不会被序列化。

### 枚举

当定义消息类型时, 我们希望某个字段只能有预先定义的多个值中的一个. 例如, 为每个SearchRequest添加一个corpus字段, 而corpus可以是UNIVERSAL, WEB, IMAGES, LOCAL, NEWS, PRODUCTS 或 VIDEO . 这样就可以简单的添加一个枚举到消息定义, 为每个可能的值定义常量.

```protocol
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
```

举的第一个常量设置到0: 每个枚举定义必须包含一个映射到0的常量作为它的第一个元素. 这是因为:

- 必须有一个0值, 这样我们才能用0来作为数值默认值
- 0值必须是第一个元素, 兼容proto2语法,在proto2中默认值总是第一个枚举值

可以通过将相同值赋值给不同的枚举常量来定义别名. 为此需要设置allow_alias选项为true,否则protocol编译器会报错。

```protocol
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
enum EnumNotAllowingAlias {
  UNKNOWN = 0;
  STARTED = 1;
  // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}

```

枚举常量必须在32位整形的范围内. 由于枚举值使用 `varint encoding`, 负值是效率低下的因此不推荐使用.

### import proto

像编写代码一样,proto文件也支持从其他的proto文件中导入message 定义。

> import "myproject/other_protos.proto";

**但是这里要注意**

假设  `a.proto`引入了`b.proto`，但是`b.proto`更换了位置，路径变成了`test/b.proto`,那有下面的办法可以解决:

- 修改`a.proto`中的import语句，直接`import "test/b.proto"`
- 在`b.proto`文件原来的位置，创建一个`b.proto`文件，文件内容为`import public "test/b.proto"`，就可以了

假设 `a.proto`引入了`b.proto`，`b.proto` 中引用 `c.proto`,如果 `a.proto`想要引用`c.proto`，是不能直接用的，同样有下面两种办法：

- 在`a.proto`中新增`c.proto`的引用
- 在`b.proto`中将引用修改为 `import public "c.proto"``

### 内嵌类型

与其他编程语言一样，Protocol Buffer 支持内嵌类型，并且可以内嵌多层。

```proto
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result result = 1;
}
```

如果想在父消息类型之外重用消息类型, 可以使用 Parent.Type 来引用:

```proto
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

还可以嵌的更深

```proto
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

### 更新消息类型

如果现有的消息类型不再满足所有需求 (例如，添加额外的字段 )，但仍然希望使用使用旧格式创建的代码，不用担心，在不破坏任何现有代码的情况下更新消息类型非常简单。请记住以下规则：

- **不要**更改任何现有字段的字段编号；
- 如果添加新的字段时, 使用"老"消息格式序列化后的任何消息都可以被新生成的代码解析. 但是需要留意这些元素的默认值以便新的代码可以正确和老代码生成的消息交互(也就是说，新添加的字段此时会采用默认值，因为lao的消息传递过来的时候不会包含这些字段). 类似的, 新代码创建的消息可以被老代码解析: 解析时新的字段被简单的忽略. 当消息反序列化时未知字段会失效, 因此如果消息被传递给新代码, 新的字段将不再存在.
- 字段可以被删除, 但是要求在更新后的消息类型中原来的标签数字不再使用.可以考虑重命名这个字段, 或者添加前缀`OBSOLETE_`, 或者`reserved`, 以便其他用户在修改.proto文件不会不小心重用这个数字.
- `int32`, `uint32`, `int64`, `uint64`, 和 `bool` 是完全兼容的 ,也就是说可以将一个字段的类型从这些类型中的一个修改为另外一个，而不会打破向前或者向后兼容. 也就是说，类型解析时会做一下相应的类型转换。
- sint32 和 sint64 是彼此兼容的,但是和其他整型类型不兼容.
- string 和 bytes 是兼容的, 如果bytes是有效的UTF-8编码的.
- 如果bytes包含这个消息编码后的内容，内嵌的message type 和bytes兼容,.
- fixed32 兼容 sfixed32, 而 fixed64 兼容 sfixed64.

### Any

Any 消息类型可以让你使用消息作为嵌入类型而不必持有他们的.proto定义. Any把任意序列化后的消息作为bytes包含, 带有一个URL, 工作起来类似一个全局唯一的标识符. 为了使用Any类型, 需要导入google/protobuf/any.proto.

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated Any details = 2;
}
```

给定消息类型的 default type URL 是 `type.googleapis.com/packagename.messagename`.

不同语言实现将提供运行时类库来帮助以类型安全的方式封包和解包Any的内容,例如, 在java中, Any类型将有特别的pack()和unpark()访问器, 而在c++中有PackFrom() 和 PackTo()方法:

```cpp
// 在Any中存储任务消息类型
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// 从Any中读取任意消息
ErrorStatus status = ...;
for (const Any& detail : status.details()) {
  if (detail.IsType<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

### Oneof

如果你有一个有很多字段的消息, 而同一时间最多只有一个字段会被设值, 你可以通过使用`oneof`特性来强化这个行为并节约内存.

Oneof 字段和常见字段类似, 除了所有字段共用内存, 并且同一时间最多有一个字段可以设值. 设值`oneof`的任何成员都将自动清除所有其他成员. 可以通过使用特殊的case()或者WhichOneof()方法来检查oneof中的哪个值被设值了(如果有), 取决于不同语言.

使用oneof关键字来在.proto中定义oneof, 后面跟oneof名字, 在这个例子中是test_oneof:

```proto
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

然后再将oneof字段添加到oneof定义. 可以添加任意类型的字段, 但是不能使用重复(repeated)字段.

在生成的代码中, oneof字段和普通字段一样有同样的getter和setter方法. 也会有一个特别的方法用来检查哪个值(如果有)被设置了,可以点击 [API Reference](https://developers.google.com/protocol-buffers/docs/reference/overview) 查看更多。

#### Oneof 特性

- 设置一个oneof字段会自动清除所有其他oneof成员. 所以如果设置多次oneof字段, 只有最后设置的字段依然有值.

```cpp
SampleMessage message;
message.set_name("name");
CHECK(message.has_name());
message.mutable_sub_message();   // Will clear name field.
CHECK(!message.has_name());
```
- 如果解析器遇到同一个oneof的多个成员, 只有看到的最后一个成员在被解析的消息中被使用.
- oneof不能是重复字段
- Reflection APIs work for oneof fields. (oneof字段用反射api来实现?)
- 如果使用c++, 需要兼顾确认确认代码不会导致内存奔溃. 下面的示例代码会导致crash因为sub_message已经在调用set_name()方法时被删除.

```cpp
SampleMessage message;
SubMessage* sub_message = message.mutable_sub_message();
message.set_name("name");      // Will delete sub_message
sub_message->set_...            // Crashes here
```

- 同样在c++中, 如果通过调用Swap()来交换两个带有oneof的消息, 每个消息将会有另外一个消息的oneof: 在下面这个示例中, msg1将会有sub_message和msg2会有name.

```cpp
SampleMessage msg1;
msg1.set_name("name");
SampleMessage msg2;
msg2.mutable_sub_message();
msg1.swap(&msg2);
CHECK(msg1.has_sub_message());
CHECK(msg2.has_name());
```

#### 标签重用问题

- 将字段移入或者移出oneof: 消息被序列化和解析后, 可能丢失部分信息(某些字段可能被清除). 但是，可以安全地将单个字段移动到新的AAA中，并且如果已知只有一个字段被设置，还可以移动多个字段。
- 删除oneof的一个字段又回来: 消息被序列化和解析后, 可能清除你当前设置的oneof字段
- 拆分或者合并oneof: 和移动普通字段一样有类似问题

### Maps

如果想创建一个map作为数据定义的一部分, 可以使用下面的语法:

> map<key_type, value_type> map_field = N;

key_type可以是任意整型或者字符类型( 除了floating point和bytes外任何简单类型). value_type可以是任意类型.

这与大多数编程语言类似,例如下面的代码，projects 是另一个 message type。

```proto
map<string, Project> projects = 3;
```

**map语法和下面的代码等同** 

```proto
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

### 定义服务

前面的文章中已经介绍过多次，这里不再重复介绍。

```proto
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

### JSON 映射

Proto3支持JSON格式的标准编码, 让不同系统之间的通信变的兼容。

如果一个值在json编码的数据中丢失或者它的值是null, 在被解析成protocol buffer时它将设置为对应的默认值.如果一个字段的值正好是protocol buffer的默认值, 这个字段默认就不会出现在json编码的数据中以便节约空间.

点击 [JSON Mapping](https://developers.google.com/protocol-buffers/docs/proto3#json) 查看proto3 与JSON 的编码对应。

**关于proto3的介绍暂时先写这么多，实际应用中可以再去官方网站查看相应的详细内容**。

## 参考

- [Protocol Buffer 官网](https://developers.google.com/protocol-buffers/)
- [Protocol Buffer 3 学习笔记](https://skyao.io/learning-proto3/)
