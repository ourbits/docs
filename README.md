OurBits 站点 API 文档
----------------

2021年9月起，本站将迁移多数方法到新的 JSONAPI 中，抛弃并迁移 原有 ajax 加载网页的方法，以及原 `api.php` 路径下相关方法。
本处为站点新的 API 文档相关说明。

最近更新： `2021-10-10 21:10`

请注意： 
 - 此处所有 API 介绍均非 stable 状态，请勿依赖，或经常性检查本文档。
 - 请注意站点规则中 **`用户存在以下行为的将会被封禁账号: 1.对网站使用自动脚本或工具危害到网站正常运转;`**
 - 本文档不提供任何解释。

# API 介绍

- API 路径： `jsonapi.php`
- 请求方法： 除非接口特别说明，否则均为 GET 请求
- 请求参数： 
  - 无论是否为GET请求，请求字段 `&action=` （下称`接口请求动作`）均必备，格式说明见 [nikic/FastRoute](https://github.com/nikic/FastRoute#defining-routes) ，此处不再累述
  - 如果接口声明支持分页，则分页使用以下参数，以下不再累述。
    | 参数名 | 必要 | 类型 | 说明 |
    |:---:|:---:|:---:|:---|
    | page | √ | int | 页数，以0为基底 |
    | limit | x | int | 单页返回数量，具体见各接口说明 `limit=<默认单页大小,?最大单页大小>`。当你传入的值超过`最大单页大小`，服务器会静默处理至设置值而不会报错。 |
  - 如果接口声明支持搜索，则搜索使用以下参数，以下不再累述。
    | 参数名 | 必要 | 类型 | 说明 |
    |:---:|:---:|:---:|:---|
    | search | x | string | 搜索字符串 |
  - 其余参数（在 `params` 还是 `body` 中）见各接口方法介绍
- 除用户认证接口（ user/{~~signup~~, login} ）外，其余方法均需要用户凭证，之后不做额外说明（凭证提供方法见下）
- 接口响应格式：
```json5
{
  "success": false, 
  "data": null,  // 响应数据
  "pager": {   // 如果接口支持分页，则还有此项存在
    "page": 0, // 当前分页
    "limit": 50,  // 当前分页大小
    "count": 10  // 记录总条数
  },
  "debug": {  // 如果传入参数 `&debug=1` （对所有用户均可提供）
    "mysql": 3,  // 执行次数
    "redis": {
      "read": 1,
      "write": 0,
      "delete": 0,
      "magic": 0
    }
  },
  /**
   * 如果用户开启高级debug（admin以上级别使用特定方法开启），
   * 则此时debug项信息如下
   *
  "debug": {
    "mysql": [
      "SELECT xxxx FROM xxxxx WHERE xxxxx",
      "UPDATE xxxx SET xxxx = xxxx WHERE xxxxx"
    ],  // 执行的具体SQL语句
    "redis": {
      "read": {
        "xxxxx1": 1,   // 执行的具体SQL语句 -> 执行次数
        "xxxxx2": 4,
      },
      "write": [],   // 因为后端没有开启 JSON_FORCE_OBJECT，所以此处会被json函数转成 empty array， 实质应该当作对象处理
      "del": [],
      "magic": {
        "xxxx:xxxx": 3
      }
    }
  }
   */
  
}
```
   - 如果响应成功，则响应数据在 `data` 字段中（下方成功响应实例均在该字段中），同时 `success` 字段值为真。
   - 如果相应失败，则 `data` 字段为 `null` ，此外还会存在 `errorCode`, `errorMsg` 字段，请根据 [ErrorCode 说明](#99-errorcode-说明) 进行检查。

## 用户凭证

1. 凭证来源：使用 `/auth/login` 接口获取站点认证JWT，或也可以使用站点Cookies中的 `ourbits_jwt` 值。
2. 凭证使用：
   1. 推荐使用HTTP-Header `Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.xxxxxxxxxxxx` 的形式，以便符合 [rfc6750](https://www.rfc-editor.org/rfc/rfc6750) 规范。
   2. 但由于历史原因，我们同样对以HTTP-Header `Cookie: ourbits_jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.xxxxxxxxxxx` 这种以Cookies形式进行身份认证进行支持，且支持程度更高。 

## 频率限制

- 对于未携带用户凭证的请求，其请求频率基于ip地址进行限制，限制请求为每分钟5次；
- 对除下列具体说明的接口，其请求频率基于`用户uid和请求动作`进行限制，限制请求均为每分钟10次：
  - 种子搜索： 每分钟 5次；


# 1. 用户认证

## 1.1 用户登录




# 2. 用户相关

// TODO

# 3. 种子相关

> 如无特别说明，本节中 `接口请求动作` 中的 `id` 参数均为种子id。 

## 3.1 种子搜索

- 接口功能：进行种子搜索
- 接口请求动作： `torrent/search`

## 3.2 获取种子基本信息

// TODO

## 3.3 种子文件列表

- 接口功能：获取文件树形式的种子文件列表
- 接口请求动作： `torrent/{id:\d+}/files`
- 成功响应示例：
```json5
{
  "torrentName": {   // 种子名
    "Dir1": {   // 子文件夹（如有）
      "file1": 12345678,   //  文件+大小
    },
    "file2": 12345678,
    "file3": 12345678
  }
}
  ```

## 3.4 种子做种情况



## 3.x 感谢种子发布

 - 接口功能：感谢种子发布者，你和种子发布者都能获得魔力值奖励
 - 接口请求动作：`POST torrent/{id:\d+}/thanks`
 - 成功响应示例：
 ```json5
 {
     "torrentId": 1234,   // 种子id
     "msg": "Say Thanks to Torrent 1234 success!"
 }
 ```

## 3.x 赠送魔力给种子发布者

 - 接口功能：赠送一定数量的魔力值给对应种子发布者，发布者会收到税后的魔力值。
 - 接口请求动作：`POST torrent/{id:\d+}/magic`
 - 接口请求参数：

  | 参数名 | 必要 | 类型 | 说明 |
  |:---:|:---:|:---:|:---|
  | value | √ | enum{ 50, 100, 200, 500, 1000 } | 需要赠送的魔力值 |

 - 成功响应示例：
```json5
{
  "torrentId": 1234,  // 种子id
  "point": 100,  // 赠送的魔力值
  "msg": "Sent Bonus 100 to Torrent 1234 Successful"
 }
```

// TODO

# // TODO

# 5. 站点相关

## 5.1 站点基本情况

// TODO

## 5.2 新闻

// TODO

## 5.3 趣味盒

### 5.3.1 获取一条趣味盒内容

- 接口功能：获取一条趣味盒的内容，如果未传入id，则默认获取最新一条。此接口不返回有趣程度为 `dull`或`banned` 的趣味盒信息。
- 接口请求动作： `site/fun/view[/{id:\d+}]`
- 成功响应示例：
```json5
{
  "id": "1234",  // * 趣味盒的id
  "userid": "12345", // * 该趣味盒发布者的uid
  "userHtml": "",   // 以html形式展示的发布者信息
  "added": "2021-09-26 10:57:52",  // 趣味盒发布时间
  "title": "Title",  // 标题
  "body": "Body",  // 趣味盒主题内容，BBcode格式，需要进行转换
  "status": "normal",  // 该趣味盒的有趣程度 enum ('normal', 'dull', 'notfunny', 'funny', 'veryfunny', 'banned')
  "vote": {    // 对该趣味盒投票信息
    "total": "2333",  // * 总数
    "fun": "1234"  // * 感觉有趣的数量
  }
  // * 由于缓存等原因，这几个值类型可能为 `string<int> | int`
}
```

### 5.3.2 获取趣味盒列表

- 接口功能：获取趣味盒列表。此接口不返回有趣程度为 `banned` 的趣味盒信息，以及趣味盒的投票信息 `vote`。
- 接口请求动作： `site/fun/list`，支持搜索、分页（`limit=<10,50>`）
- 接口请求参数：

  | 参数名 | 必要 | 类型 | 说明 |
  |:---:|:---:|:---:|:---|
  | field | x | enum{ 'title','body','both' } | 搜索字符串的位置，`default='title'` |

- 成功响应实例：
```json5
{
  search: 'search_text',  // 如果使用搜索，则存在该字段
  list: [
    {
      "id": "1234",  // * 趣味盒的id
      "added": "2021-09-26 10:57:52",  // 趣味盒发布时间
      "title": "Title",  // 标题
      "body": "Body",  // 趣味盒主题内容，BBcode格式，需要进行转换
      "status": "normal"  // 该趣味盒的有趣程度 enum ('normal', 'dull', 'notfunny', 'funny', 'veryfunny', 'banned')
    },
    // ....
  ]
}
```

# 第三方支持

// TODO

# 99. ErrorCode 说明

| ErrorCode | 说明 |
|:-----:|:------|
| 40001 | 请求方法、请求接口、请求参数中存在错误，未能检索到对应处理方法或请求参数不合法。 |
| 40002 | 请求的对应资源不存在 |
| 40003 | 请求处理过程中不满足相关要求，具体内容见`errorMsg` |
| 40101 | 访问需要认证的接口但没有提供凭证 |
| 40102 | 用户状态不正常（如封存状态），无法完全授权 | 
| 42901 | 频率限制，请等待一段时间后重试 |
| 499xx | 为各接口自定义错误，具体请见接口说明 |
| 99999 | 未指定错误 |
