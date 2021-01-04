# RESTful API Design

# 什么是 REST？

REST（Representational State Transfer，表述性状态转换），是 Roy Fielding 提议的使用表述性状态转移 （REST）作为设计 Web 服务的体系性方法。REST 是一种基于超媒体构建分布式系统的架构风格。匹配或兼容 `REST` 架构风格的 `Web` 服务称为 RESTful Web 服务。RESTful Web 服务允许客户端发出 `URI`（Uniform Resource Identifier，统一资源标识符）访问和操作 `Web` 资源的请求。

## 设计流程

设计指南建议在设计面向资源的 API 时采取以下步骤：

- 确定 API 提供的资源类型。
- 确定资源之间的关系。
- 根据类型和关系确定资源名称方案。
- 确定资源架构。
- 将最小的方法集附加到资源。

## 主要原则

下面是使用 HTTP 设计 RESTful API 时的一些主要原则：

- RESTful API 面向资源设计。

- 每个资源有一个标识符，即，唯一标识该资源的 URI。例如特定客户订单的 URI 可能是：

  ```
  https://example.com/orders/1
  ```

- 客户端通过交换资源的表示形式来与服务交互。 许多 Web API 使用 JSON 作为交换格式。 例如，对上面所列的 URI 发出 GET 请求可能返回以下响应正文：

  ```json
  {
      "orderId": 1,
      "amount": 99.9,
      "productId": 1,
      "quantity": 1
  }
  ```

- REST API 使用统一接口，这有助于分离客户端和服务实现。 对于基于 HTTP 构建的 REST API，统一接口包括使用标准 HTTP 谓词对资源执行操作。 最常见的操作是 GET、POST、PUT、PATCH 和 DELETE。

- REST API 使用无状态请求模型。 HTTP 请求应是独立的并可按任意顺序发生，因此保留请求之间的瞬时状态信息并不可行。 信息的唯一存储位置就在资源内，并且每个请求应是原子操作。 此约束可让 Web 服务获得高度可伸缩性，因为无需保留客户端与特定服务器之间的关联。 任何服务器可以处理来自任何客户端的任何请求。 也就是说，其他因素可能会限制可伸缩性。

- REST API 由表示形式中包含的超媒体链接驱动。 例如，下面显示了某个订单的 JSON 表示形式。 该表示形式包含一些链接，用于获取或更新与该订单关联的客户。

  ```json
  {
      "orderID": 3,
      "productID": 2,
      "quantity": 4,
      "amount": 16.6,
      "links": [
          {
              "rel": "product",
              "href": "https://example.com/customers/3",
              "action": "GET"
          },
          {
              "rel": "product",
              "href": "https://example.com/customers/3",
              "action": "PUT"
          }
      ]
  }
  ```

  2008 年，Leonard Richardson 提议对 Web API 使用以下[成熟度模型](https://martinfowler.com/articles/richardsonMaturityModel.html)：

  - 级别 0：定义一个 URI，所有操作是对此 URI 发出的 POST 请求。
  - 级别 1：为各个资源单独创建 URI。
  - 级别 2：使用 HTTP 方法来定义对资源执行的操作。
  - 级别 3：使用 Hypermedia（HATEOAS，下面会详细介绍）。

  根据 Fielding 的定义，级别 3 对应于某个真正的 RESTful API。 在实践中，许多发布的 Web API 大致处于级别 2。

## 面向资源设计

### 资源

面向资源的 API 通常被构建为资源层次结构，其中每个节点是一个“简单资源”或“集合资源”。 为方便起见，它们通常被分别称为资源和集合。

- 一个集合包含**相同类型**的资源列表。 例如，一个用户拥有一组联系人。
- 资源具有一些状态和零个或多个子资源。 每个子资源可以是一个简单资源或一个集合资源。

## HTTP 请求方法的幂等性

### 什么是幂等性？

在 `HTTP/1.1` 规范中，幂等性的定义是：

> Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request.

从定义上看，HTTP 方法的幂等性是指一次和多次请求某一个资源应该具有同样的作用。

通俗点说，`幂等` 就是一次或多次同样的请求，得到的结果也是同样的。

- `GET` 不管多少次请求同一个资源，得到的结果是完全相同的，不会有任何副作用。
- `POST` 每请求一次都会新增资源，所以是非幂等的。
- `PUT` 方法同样的数据提交多少次，该资源都与第一次请求后的结果相同（ `PUT` 也可以认为是创建或更新资源，当指定 ID 的资源不存在时创建，存在时则更新），`PUT` 是直接将请求数据覆盖原数据。
- `DELETE` 每请求一次都会删除资源，所以是非幂等的。
- `PATCH` 方法修改资源部分值，是非幂等的，比如将 version+1，每次执行后的结果都不相同。
- `OPTIONS` 方法通常是浏览器执行真正请求之前发送的预请求（pre-flight），它是幂等的。

### 常见方法的幂等性与安全性

| 方法名  | 幂等性 | 安全性 |
| ------- | ------ | ------ |
| GET     | Y      | Y      |
| POST    | N      | N      |
| PUT     | Y      | N      |
| PATCH   | N      | N      |
| DELETE  | Y      | N      |
| OPTIONS | Y      | Y      |
| HEAD    | Y      | Y      |

### 如何在分布式系统中实现幂等性

**Token 机制**

**数据库去重表**

**Redis实现**

**状态机**

很多业务是有一个业务流转状态的，每个状态都有前置状态和后置状态以及最后的结束状态。例如订单的待支付，已支付，已取消，已完成。

以订单为例，已支付的状态的前置状态只能是待支付，而已取消状态的前置状态只能是待支付，通过这种状态机的流转我们就可以控制请求的幂等。

## 使用 HATEOAS

REST 背后的主要动机之一是它应能够导航整个资源集，而无需事先了解 URI 方案。 每个 HTTP GET 请求应通过响应中包含的超链接返回查找与所请求的对象直接相关的资源所需的信息，还应为它提供描述其中每个资源提供的操作的信息。 此原则称为 HATEOAS 或作为应用程序状态引擎的超文本。 该系统实际上是有限状态机，每个请求的响应包含从一种状态转为另一种状态所需的信息；任何其他信息都不应是必需的。

例如，若要处理订单与客户之间的关系，可以在订单的表示形式中包含链接，用于指定下单客户可以执行的操作。 下面是可能的表示形式：

```json
{
  "orderID":3,
  "productID":2,
  "quantity":4,
  "orderValue":16.60,
  "links":[
    {
      "rel":"customer",
      "href":"https://example.com/customers/3",
      "action":"GET",
      "types":["text/xml","application/json"]
    },
    {
      "rel":"customer",
      "href":"https://example.com/customers/3",
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"customer",
      "href":"https://example.com/customers/3",
      "action":"DELETE",
      "types":[]
    },
    {
      "rel":"self",
      "href":"https://example.com/orders/3",
      "action":"GET",
      "types":["text/xml","application/json"]
    },
    {
      "rel":"self",
      "href":"https://example.com/orders/3",
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"self",
      "href":"https://example.com/orders/3",
      "action":"DELETE",
      "types":[]
    }]
}
```

## 数据塑形

所谓 `数据塑形` 就是不同请求，根据请求所需要的字段来返回 JSON 数据。减少不必要的网络支出和数据冗余。通过 param 传入所需要的字段 fields=orderID,productID 返回 JSON 数据就只包含 fields 中所传入的字段。当然，使用 HATEOAS 还会含有 links。在 Java 中通过反射，塑形后放入 Map，返回 JSON 给前端。

```json
{
  "orderID":3,
  "productID":2,
}
```

## 使用 Header 返回分页信息

使用 X-Pagination 头返回分页信息，而不是在 JSON 中返回直接返回分页信息。

```
X-Pagination: {"previousPageLink":"https://example.com/orders?pageNum=1&pageSize=10","nextPageLink":"https://example.com/orders?pageNum=3&pageSize=10","totalCount":50,"pageSize":3,"currentPage":2,"totalPages":3}
```

## 使用 JSON Patch

`PATCH` 方法用于对资源的局部更新，普通。JSON Patch（[RFC6902](https://tools.ietf.org/html/rfc6902)）是一种用于描述对JSON文档所做的更改的格式（JSON Patch本身也是 JSON 结构）。当只更改了一部分时，可用于避免发送整个文档。可以与 HTTP PATCH 方法结合使用时，它允许以符合标准的方式对 HTTP API 进行部分更新。

**原始资源**

```json
{
    "id": "1",
    "username": "unufolio",
    "email": "unufolio@gmail.com",
    "mobile": "13000000000",
    "nickname": "Unufolio",
    "roles": [
        "admin",
        "member",
        "superadmin"
    ]
}
```

当我们需要更新 id 为 1 的用户的 nickname 的时候，我们只需要提交

**PATCH https://example.com/users/1**

```json
[
    {
        "op": "replace",
        "path": "/nickname",
        "value": "new nickname"
    }
]
```

当我们需要为 id 为 1 的用户的添加角色的时候，我们只需要提交(当然这里只是做一个示范，实际设计中不应该这样对用户的角色进行操作)

**PATCH https://example.com/users/1**

```json
[
    {
        "op": "add",
        "path": "/roles/-",
        "value": [
            "role name"
        ]
    }
]
```

当我们需要为 id 为 1 的用户的删除角色的时候，我们只需要提交(当然这里只是做一个示范，实际设计中不应该这样对用户的角色进行操作)

**PATCH https://example.com/users/1**

```json
[
    {
        "op": "add",
        "path": "/roles/-",
        "value": [
            "member"
        ]
    }
]
```

**JSON Pointer**

JSON Pointer ([IETF RFC 6901](https://tools.ietf.org/html/rfc6901)) 定义了一种字符串格式，用于标识 JSON 文档中的特定值。 JSON Patch 程序中的所有操作都使用它来指定要操作的文档部分。

JSON Pointer 是由一串由 `/` 分隔的字符串，这些令牌要么指定对象中的键，要么指定数组的索引。例如，给定 JSON：

```json
{
    "biscuits": [
        {
            "name": "Digestive"
        },
        {
            "name": "Choco Leibniz"
        }
    ]
}
```

- `/biscuits` 将指向 biscuits 数组
- `/biscuits/1/name` 将指向 "Choco Leibniz"。
- 如果要指向 JSON 文档的“根”，那么使用空字符串 `""` ，而不是 `\` ,因为 `\` 指向了 root 节点的下一个 "" key。
- 如果你要指向名称中带有 `~` 或者 `/` 的 key，那没需要将其用 ~0 和 ~1 进行转译，例如，要从 `{"foo/bar~": "baz"}` 获取 `baz`，则可以使用指针 `/foo〜1bar〜0`。
- 可以使用 `-` 指向数组的最后元素，例如：`/biscuits/-`

**Json Patch 的所有操作**

- **Add**

```json
{
    "op": "add",
    "path": "/biscuits/1",
    "value": {
        "name": "Ginger Nut"
    }
}
```

将值添加到对象或插入到数组中。如果是数组，则将值插入到给定索引之前。如果要插入到数组尾部，那么使用 `-` 字符。

- **Remove**

```json
{
    "op": "remove",
    "path": "/biscuits"
}
```

从对象或数组中删除值。

```json
{
    "op": "remove",
    "path": "/biscuits/0"
}
```

移除 `biscuits` 数组的第一个元素（如果 `biscuits` 是一个对象，则移除其 key 为“0”的值）

- **Replace**

```json
{
    "op": "replace",
    "path": "/biscuits/0/name",
    "value": "Chocolate Digestive"
}
```

替换掉一个值，等同于移除并添加。

- **Copy**

```json
{
    "op": "copy",
    "from": "/biscuits/0",
    "path": "/best_biscuit"
}
```

将值从 JSON 文档中的一个位置复制到另一个位置。from 和 path 都是 JSON Pointer。

- **Move**

```json
{
    "op": "move",
    "from": "/biscuits",
    "path": "/cookies"
}
```

将值从一个位置移动到另一个位置。from 和 path 都是 JSON Pointer。

- **Test**

```json
{
    "op": "test",
    "path": "/best_biscuit/name",
    "value": "Choco Leibniz"
}
```

测试是否在文档中设置了指定的值。如果测试失败，则整个 patch 都不适用。

## 使用 Problem Json

**problem+json 中这么定义错误输出**

```
HTTP/1.1 404 Not found
Content-Type: application/problem+json
Response Body:
{
    type: 'http://www.restapitutorial.com/httpstatuscodes.html',
    title: 'User data not found',
    detail: 'The user JO not found',
    status: 404,
    instance: 'http://localhost/users/JO'
}
```

- `type`: 提供一个描述问题的连接(required)
- `title`: 对错误做一个简短的描述(required)
- `status`: HTTP status code(required)
- `detail`: 详细描述错误信息(optional)
- `instance`: 返回错误产生的URL, 绝对地址(optional)

采用 `problem+json` 格式我们可以让错误输出更具有描述性，可以让 API Consumer 更好进行错误处理。

**problem+json 中也可以附加更多有意义的信息**

例如采用如下信息描述订单问题:

```
HTTP/1.1 403 Forbidden
Content-Type: application/problem+json
Response Body:
{
    type: 'http://example.com/docs/api/problems/out-of-credit',
    title: 'You do not have enough credit',
    detail: 'Your current balance is 30, but that costs 50.',
    instance: 'http://example.com/users/unufolio/orders/1002222',
    balance: 30,
    users: [
        'http://example.com/users/unufolio'
		]
}
```

## 常见 HTTP 状态码

### 请求成功

- 200 **OK** : 请求执行成功并返回相应数据，如 `GET` 成功
- 201 **Created** : 对象创建成功并返回相应资源数据，如 `POST` 成功；创建完成后响应头中应该携带头标 `Location` ，指向新建资源的地址
- 202 **Accepted** : 接受请求，但无法立即完成创建行为，比如其中涉及到一个需要花费若干小时才能完成的任务。返回的实体中应该包含当前状态的信息，以及指向处理状态监视器或状态预测的指针，以便客户端能够获取最新状态。
- 204 **No Content** : 请求执行成功，不返回相应资源数据，如 `PATCH` ， `DELETE` 成功

### 重定向

**重定向的新地址都需要在响应头 `Location` 中返回**

- 301 **Moved Permanently** : 被请求的资源已永久移动到新位置
- 302 **Found** : 请求的资源现在临时从不同的 URI 响应请求
- 303 **See Other** : 对应当前请求的响应可以在另一个 URI 上被找到，客户端应该使用 `GET` 方法进行请求。比如在创建已经被创建的资源时，可以返回 `303`
- 307 **Temporary Redirect** : 对应当前请求的响应可以在另一个 URI 上被找到，客户端应该保持原有的请求方法进行请求

### 条件请求

- 304 **Not Modified** : 资源自从上次请求后没有再次发生变化，主要使用场景在于实现数据缓存。
- 409 **Conflict** : 请求操作和资源的当前状态存在冲突。主要使用场景在于实现并发控制。
- 412 **Precondition Failed** : 服务器在验证在请求的头字段中给出先决条件时，没能满足其中的一个或多个。主要使用场景在于实现并发控制。

### 客户端错误

- 400 **Bad Request** : 请求体包含语法错误
- 401 **Unauthorized** : 需要验证用户身份，如果服务器就算是身份验证后也不允许客户访问资源，应该响应 `403 Forbidden` 。如果请求里有 `Authorization` 头，那么必须返回一个 `[WWW-Authenticate](https://tools.ietf.org/html/rfc7235#section-4.1)` 头
- 403 **Forbidden** : 服务器拒绝执行
- 404 **Not Found** : 找不到目标资源
- 405 **Method Not Allowed** : 不允许执行目标方法，响应中应该带有 `Allow` 头，内容为对该资源有效的 HTTP 方法
- 406 **Not Acceptable** : 服务器不支持客户端请求的内容格式，但响应里会包含服务端能够给出的格式的数据，并在 `Content-Type` 中声明格式名称
- 410 **Gone** : 被请求的资源已被删除，只有在确定了这种情况是永久性的时候才可以使用，否则建议使用 `404 Not Found`
- 413 **Payload Too Large** : `POST` 或者 `PUT` 请求的消息实体过大
- 415 **Unsupported Media Type** : 服务器不支持请求中提交的数据的格式
- 422 **Unprocessable Entity** : 请求格式正确，但是由于含有语义错误，无法响应
- 428 **Precondition Required** : 要求先决条件，如果想要请求能成功必须满足一些预设的条件

### 服务端错误

- 500 **Internal Server Error** : 服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。
- 501 **Not Implemented** : 服务器不支持当前请求所需要的某个功能。
- 502 **Bad Gateway** : 作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。
- 503 **Service Unavailable** : 由于临时的服务器维护或者过载，服务器当前无法处理请求。这个状况是临时的，并且将在一段时间以后恢复。如果能够预计延迟时间，那么响应中可以包含一个 `Retry-After` 头用以标明这个延迟时间（内容可以为数字，单位为秒；或者是一个 [HTTP 协议指定的时间格式](http://tools.ietf.org/html/rfc2616#section-3.3)）。如果没有给出这个 `Retry-After` 信息，那么客户端应当以处理 500 响应的方式处理它。

`501` 与 `405` 的区别是：`405` 是表示服务端不允许客户端这么做，`501` 是表示客户端或许可以这么做，但服务端还没有实现这个功能

## 自定义 ACTION

当 HTTP Mothed 不足以描述 API，对 API 细粒度要求更高，需要对 API 更精细地鉴权的时候，参考 Google 的 API 设计指南我们可以使用自定义的方法。当然，在 REST 完全能够覆盖 API 的情况下，还是推荐使用遵守 REST 的方式来定义我们的 API。

### HTTP 映射

对于自定义方法，应该使用像下面这种 HTTP 映射：

```
https://api.example.com/some/resource/id:action
```

使用 `:` 而不是 `/` 分隔的原因是让动作（动词）和资源定义分开，降低 API 层级。例如恢复已经删除的资源可以使用 `POST` `https://api.example.com/some/resource/id:recover`

定义自定义 HTTP API 时，应遵循以下尊则。

- 除了作为代替 `get` 或 `list` 的方法（尽可能使用 `GET`）以外，自定义方法应该使用 HTTP `POST` 方法，因为该方法具有更灵活的语义。
- 自定义方法不应该使用 HTTP `PATCH` 方法，但可以使用其他 HTTP 方法。在这种情况下，方法必须遵循其标准 [HTTP 语义](https://tools.ietf.org/html/rfc2616#section-9) 。
- 注意：使用 HTTP `GET` 的自定义方法必须具有幂等性，且无负面影响。例如，一些自定义视图，可以使用自定义 HTTP `GET` 。

# 设计一个好的 RESTful API

## 协议

API 与用户的通信协议，总是使用`HTTPs`协议。

## 域名

应该尽量将API部署在专用域名之下。

```
https://api.example.com
```

如果确定 API 不会有进一步扩展，可以考虑放在主域名下。

```
https://example.org/api/
```

## 版本控制

常见的三种版本控制方式：

1. 在 URI 中放版本信息：`GET /v1/users/1`
2. Accept Header：`Accept: application/json+v1`
3. Accept Header：`Accept: vnd.example-com.foo+json; version=1.0`
4. 自定义 Header：`X-Api-Version:1`

## 建议 URI 规范

1. 不用大写
2. 用 `-` 不用 `_`
3. 参数列表要 encode
4. URI 中的名词表示资源集合，使用复数形式。

## RESTful 设计

**写在最前面：最好不要在 URI 中出现动词！类似 login、login、auth 等特定操作的除外。**

### 方法 + 宾语

RESTful 的核心思想就是，客户端发出的数据操作指令都是“方法 + 宾语”的结构。

### 方法

方法指的就是 `HTTP` 请求方法，常用的有 `GET` , `DELETE` , `POST` , `PUT` , `PATCH` 。

### 宾语

宾语就是 `API` 的 `URI`，是 `HTTP` 动词作用的对象，是资源。它应该是名词，不能是动词。

### URI

URI 表示资源，资源一般对应服务器端领域模型中的实体类。

### 资源

URI 表示资源的两种方式：资源集合、单个资源。

**资源集合：**

```
# 所有类别
/categories 
# 类别为 1 的所有文章
/categories/1/articles
```

**单个资源**

```
# 类别 1
/categories/1
# ID 为 14321 的文章
/articles/14321
```

### 避免层级过深的 URI

`/` 在url中表达层级，用于**按实体关联关系进行对象导航**，一般根据id导航。

过深的导航容易导致url膨胀，不易维护。

如 `GET /zoos/1/areas/3/animals/4`，尽量使用查询参数代替路径中的实体导航，如 `GET /animals?zoo=1&area=3`；

如 `GET users/1/categories/1/articles/14321` ，只需 `GET /articles/14321` 就足够。

### 资源不能是动词，但是可以是一种服务

如 ID 为 1 的账户给 ID 为 2 的账户转账 500

**错误写法**

```
POST /accounts/1/transfer/500/to/2
```

**正确写法**

```
POST /transanctions
```

```json
{
    "from": "1",
    "to": "2",
    "amount": 500
}
```

### HTTP 方法

**GET 查询**

```
GET /categories
GET /categories/1
GET /categories/1/articles
```

**POST 创建单个资源**

```
# 新增用户
POST /users
# 为 ID 为 1 的用户分配角色
POST /users/1/roles
```

**PUT 创建或更新资源（全量）**

```
# 新增 ID 为 10 的用户
PUT /users/10
# 修改(覆盖) ID 为 10 的用户的信息
PUT /users/10
```

**DELETE 删除资源**

```
# 删除 ID 为 10 的用户
DELETE /users/10
# 删除 ID 为 10 的用户的所有角色
DELETE /users/10/roles
# 批量删除 ID 为 10 的用户的 id in ids 的所有角色
DELETE /users/10/roles/(id1,id2,id3)
```

**PATCH 更新资源（局部）**

```
# 局部更新 ID 为 10 的用户的信息
PATCH /users/10
```

### 复杂查询

查询可以带类似以下参数

| .        | 示例                       | 备注                                         |
| -------- | -------------------------- | -------------------------------------------- |
| 过滤条件 | `?type=1&age=16`           | 允许一定的uri冗余，如`/zoos/1`与`/zoos?id=1` |
| 排序     | `?sort=age,desc`           |                                              |
| 投影     | `?whitelist=id,name,email` |                                              |
| 分页     | `?limit=10&offset=3`       |                                              |
| 分页     | `?pageNum=1&pageSize=10`   |                                              |

### Response（不用使用 Problem Json）

在操作成功时，返回成功的 HTTP 状态码 2xx，以及具体数据。若非需要，**不建议**进行类似以下包装，直接返回数据 JSON 对象即可：

```json
{
    "success": true,
    "data": {
        "id": 1,
        "name": "Unufolio"
    }
}
```

在操作失败时，返回失败的 HTTP 状态码如 400，以及具体错误信息（包括自定义错误码及描述等）。

```json
{
    "code": 400011,
    "message": "username cannot be blank",
    "status": "BAD_REQUEST",
    "timestamp": "2019-05-21T12:40:08Z"
}
```

若错误验证需要字段细分，则如下

```json
{
    "code": 40010,
    "message": "Validation Failed",
    "errors": [
        {
            "code": 40011,
            "field": "username",
            "message": "username cannot have fancy characters"
        },
        {
            "code": 40012,
            "field": "password",
            "message": "password cannot be blank"
        }
    ]
}
```

### Response（使用 Problem Json）

**成功**

```
HTTP/1.1 200 OK
```

**失败**

```
HTTP/1.1 404 Not found
Content-Type: application/problem+json
Response Body:
{
    type: 'http://example.com/httpstatuscodes.html',
    title: 'User data not found',
    detail: 'The user JO not found',
    status: 404,
    instance: 'http://example.com/users/JO'
}
```

### JSON 风格

### 时间格式

遵循 [ISO 8601](https://www.iso.org/obp/ui/#iso:std:iso:8601:ed-3:v1:en)([Wikipedia](https://en.wikipedia.org/wiki/ISO_8601)) 建议的格式：

- 日期 `2014-07-09`
- 时间 `14:31:22+0800`
- 具体时间 `2007-11-06T16:34:41Z`
- 持续时间 `P1Y3M5DT6H7M30S` （表示在一年三个月五天六小时七分三十秒内）
- 时间区间 `2007-03-01T13:00:00Z/2008-05-11T15:30:00Z` 、 `2007-03-01T13:00:00Z/P1Y2M10DT2H30M` 、 `P1Y2M10DT2H30M/2008-05-11T15:30:00Z`
- 重复时间 `R3/2004-05-06T13:00:00+08/P0Y6M5DT3H0M0S` （表示从2004年5月6日北京时间下午1点起，在半年零5天3小时内，重复3次）

### 货币名称

货币名称可以参考 ISO 4217([Wikipedia](http://en.wikipedia.org/wiki/ISO_4217)) 中的约定，标准为货币名称规定了三个字母的货币代码，其中的前两个字母是 ISO 3166-1([Wikipedia](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2)) 中定义的双字母国家代码，第三个字母通常是货币的首字母。在货币上使用这些代码消除了货币名称（比如 dollar ）或符号（比如 $ ）的歧义。

### 时区

客户端请求服务器时，如果对时间有特殊要求（如某段时间每天的统计信息），则可以参考 [IETF 相关草案](http://tools.ietf.org/html/draft-sharhalakis-httptz-05) 增加请求头 `Timezone` 。

```
Timezone: 2016-11-06 23:55:52+08:00;;Asia/Shanghai
```

具体格式说明：

```
Timezone: RFC3339 约定的时间格式;POSIX 1003.1 约定的时区字符串;tz datebase 里的时区名称
```

客户端最好提供所有字段，如果没有办法提供，则应该使用空字符串

如果客户端请求时没有指定相应的时区，则服务端默认使用最后一次已知时区或者 [UTC](http://zh.wikipedia.org/wiki/%E5%8D%8F%E8%B0%83%E4%B8%96%E7%95%8C%E6%97%B6) 时间返回相应数据。

PS 考虑到存在[夏时制](https://en.wikipedia.org/wiki/Daylight_saving_time)这种东西，所以不推荐客户端在请求时使用 Offset 。
