OurBits 站点 API 文档
----------------

2021年9月起，本站将迁移多数方法到新的 JSONAPI 中，抛弃并迁移 原有 ajax 加载网页的方法，以及原 `api.php` 路径下相关方法。
本处为站点新的 API 文档相关说明。

最近更新： `2023-02-05 13:02`

请注意： 
 - 此处所有 API 介绍均非 stable 状态，请勿依赖，或经常性检查本文档。
 - 请注意站点规则中 **`用户存在以下行为的将会被封禁账号: 1.对网站使用自动脚本或工具危害到网站正常运转;`**
 - 本文档不提供任何解释。

# API 介绍

- API 路径： `jsonapi.php`
- 请求方法： 除非接口特别说明，否则均为 GET 请求
- 请求参数： 
  - 无论是否为GET请求，请求字段 `&action=` （下称`接口请求动作`）均必备，格式说明见 [nikic/FastRoute](https://github.com/nikic/FastRoute#defining-routes) ，此处不再累述
  - 如果接口声明支持分页，除另有说明外，则分页使用以下参数，以下不再累述。
    | 参数名 | 必要 | 类型 | 说明 |
    |:---:|:---:|:---:|:---|
    | page | √ | int | 页数，以0为基底 |
    | limit | x | int | 单页返回数量，具体见各接口说明 `limit=<默认单页大小,?最大单页大小>`。当你传入的值超过`最大单页大小`，服务器会静默处理至设置值而不会报错。 |
  - 如果接口声明支持搜索，除另有说明外，则搜索使用以下参数，以下不再累述。
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

1. 凭证来源：~~使用 `/auth/login` 接口获取站点认证JWT，或也~~可以使用站点Cookies中的 `ourbits_jwt` 值。
2. 凭证使用：
   1. 推荐使用HTTP-Header `Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.xxxxxxxxxxxx` 的形式，以便符合 [rfc6750](https://www.rfc-editor.org/rfc/rfc6750) 规范。
   2. 但由于历史原因，我们同样对以HTTP-Header `Cookie: ourbits_jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.xxxxxxxxxxx` 这种以Cookies形式进行身份认证进行支持，且支持程度更高。 

## 频率限制

- 对于未携带用户凭证的请求，其请求频率基于ip地址进行限制，限制请求为每分钟5次；
- 对除下列具体说明的接口，其请求频率基于`用户uid和请求动作`进行限制，限制请求均为每分钟30次：
  - 种子搜索： 每分钟 10次；


# 1. 用户认证

## 1.1 用户登录




# 2. 用户相关

// TODO

# 3. 种子相关

> 如无特别说明，本节中 `接口请求动作` 中的 `id` 参数均为种子id。 

## 3.1 种子搜索

- 接口功能：进行种子搜索
- 接口请求动作： `torrent/search[/{mode:(?:rescue|rss|myrss)}]`
- 请求动作参数： mode不存在时为普通搜索，为`rescue`时搜索保种区，为`myrss`时搜索下载筐；为`rss`时，表现与普通搜索一致，仅因历史兼容问题，影响部分请求参数情况（详见下方说明） 
- 接口请求参数：

| 参数名 | 必要 | 类型 | 默认值 | 说明 |
|:--:|:---:|:---:|:---|:---|
| search | x | string | "" | 将要搜索的字符串，为空时则表现为“占位符搜索” |
| search_mode | x | enum{0,1,2} | 0 | 匹配模式： 0 - 与（NPHP默认模式）， 1 - 或， 2 - 精准（使用英文双引号确定存在词） |
| search_area | x | enum{0,3,4,5} | 0 | 范围：     0 - 标题、副标题（默认）， 3 - 发布者， 4 - imdb id， 5 - 豆瓣id |
| notnewword | x | boolean | 0 | 是否新热门词 |
| cat          | x      | string\|string[]   | null *    | 类型 |
| medium       | x      | string\|string[]   | null *    | 媒介 |
| codec        | x      | string\|string[]   | null *    | 编码 |
| standard     | x      | string\|string[]   | null *    | 分辨率 |
| processing   | x      | string\|string[]   | null *    | 地区 |
| team         | x      | string\|string[]   | null *    | 制作组 |
| audiocodec   | x      | string\|string[]   | null *    | 音频编码 |
| incldead     | x         | enum{0,1,2}       | x            | 0  *     | 显示断种/活种：  0 - 包括断种， 1 - 活种， 2 - 断种 | 
| spstate      | x         | enum{range(0, 7)}       | x            | 0  *     | 促销：   0 - 全部， 1-7 分别表示不同优惠等级 | 
| inclbookmarked | x       | enum{0,1,2}       | x            | 0  *     | 显示收藏：  0 - 全部， 1 - 仅收藏， 2 - 仅未收藏（因历史遗留问题，当mode为rss时，表示`保种区`） |
| size           | x       | {min?: int, max?: int} | null | 种子大小（MiB） |
| times_completed | x      | {min?: int, max?: int} | null | 完成数 |
| seeders         | x      | {min?: int, max?: int} | null | 做种数（不一定精准） |
| leechers        | x      | {min?: int, max?: int} | null | 下载数（不一定精准） |
| tag_gf          | x        | boolean           | 0       | 官方 |
| tag_diy         | x        | boolean           | 0       | DIY |
| tag_sf          | x        | boolean           | 0       | 首发 |
| tag_gy          | x        | boolean           | 0       | 国语 |
| tag_zz          | x        | boolean           | 0       | 中字 |
| tag_jz          | x        | boolean           | 0       | 禁转 |
| tag_yq         | x         | boolean           | 0       | 应求 |
| tag_db         | x         | boolean           | 0       | 杜比视界 |
| tag_hdr        | x         | boolean           | 0       | HDR10 |
| tag_hdrp       | x         | boolean           | 0       | HDR10+ |
| tag_hlg        | x         | boolean           | 0       | HLG |
| sort           | x           | enum{0,1,3,4,5,6,7,8,9} | 0 | 排序列： 0 - id, 1 - name, 3 - coments, 4 - added, 5 - size , 6 - times_completed , 7 - seeders , 8 - leechers, 9 - owner |
| type           | x           | enum{'asc', 'desc'} | 'desc  | 排列顺序（升序还是降序） |
| page           | x           | int                | 0      | 第？页（mode不存在或非rss时使用）  |
| rows           | x         | enum{10,20,30,40,50} | 10    | 每页返回的种子数量（mode为rss时使用） |
| startindex     | x         | enum{0,1,2,3}                | 0       | 第？页（mode为rss时使用） |

说明： 
1. 标`*`的请求项如果不存在时，会尝试从用户设置（`设置-网站设置-默认分类`）中获取。
2. 仅`mode`为rss时，支持设置分页的每页大小，其余情况默认为用户设置值（`设置-网站设置-种子页面`）

- 成功响应示例：
```json5
[
  {
    id: 208129,    // 种子 id
    info_hash: "a5b7dd163e7f8b39a651bd309ca9bd959c0d5bdf",  // 种子的 info_hash
    name: "The Italian Job 1969 UHD 2160p Blu-ray HEVC DTS-HD MA 5 1-BHYS@OurBits",   // 主标题
    small_descr: "意大利任务 [UHD 原盘DIY国配对应简体特效中英特效字幕  保留Dolby Vision]",    // 副标题
    size: 75274409544,   // 种子大小
    added: 1675528959,  // 上传日期（时间戳形式）
    
    category: 401,   // 种子对应的分类
    source: 0,   // 来源（站点未启用，均为 0）
    medium: 12,  // 媒介
    codec: 14,   // 编码
    standard: 5,  // 分辨率
    processing: 2,  // 地区
    team: 1,   //   制作组
    audiocodec: 1,   // 音频编码
    
    comments: 0,   // 评论数量
    seeders: 256,   // 做种数
    leechers: 47,  // 下载数
    times_completed: 281,  // 完成数
    
    anonymous: 1,  // 是否匿名
    owner: -1,  //  发布者，如果该种子为匿名，则为 -1，否则为对应用户的uid
    
    url: 64505,  // imdb编号
    dburl: 1305869,  // 豆瓣编号

    hr: 0,  // 是否启用H&R
    rescue: 0,   // 是否是保种区种子
    visible: 1,  // 是否可见
    banned: 0,  // 是否被禁用
    
    tag_gf: 1,  // 标签：官方（为1时表示存在该标签，下同）
    tag_diy: 1,  // 标签：DIY
    tag_sf: 0,  // 标签：首发
    tag_gy: 1,  // 标签：国语
    tag_zz: 1,  // 标签：中字
    tag_jz: 1,  // 标签：禁转
    tag_yq: 0,  // 标签：应求
    tag_db: 1,  // 标签：杜比视界
    tag_hdr: 1,  // 标签：HDR10
    tag_hlg: 0,  // 标签：HDR10+
    tag_hdrp: 0,  // 标签：HLG
    
    approved: 0,  // 种子审核状态  0=未审核 1=已审核 2=被驳回 3=被举报
    picktype: "hot",  // 'hot' - 热门， 'classic' - 经典，  'recommended' - 推荐， 'normal' - 普通
    pos_group: 3,   // 置顶类型
    sp_state: 2,   // 优惠类型
    promotion_time_type: 2,  // 优惠类型的时间类型 0 - 全局， 2 - 直到
    promotion_until: 1675615359,  // promotion_time_type为2时，优惠类型的到期时间

  },
  ...
]
```

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
