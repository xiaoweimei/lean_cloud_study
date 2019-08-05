## 对象
1. 对象格式
- LeanCloud的数据存储服务是建立在AVObject(对象)基础上的，每个AVObject包含若干属性值对(key-value，也称键值对)，属性的值是与JSON格式兼容的数据。通过REST API
保存对象需要将对象的数据通过JSON来编码。这个数据是无模式化的，这意味着你不需要提前标注每个对象上有哪些key，你只要随意设置键值对就可以，后端会保存它。
- 当你从 LeanCloud 中获取对象时，一些字段会被自动加上，如 createdAt、updatedAt 和 objectId。这些字段的名字是保留的，值也不允许修改。
createdAt 和 updatedAt 都是 UTC 时间戳，以 ISO 8601 标准和毫秒级精度储存：YYYY-MM-DDTHH:MM:SS.MMMZ。objectId 是一个字符串，在类中可以唯一标识一个实例。 
2. 创建对象
- 为了在leanCloud上创建一个新的对象，应该向class的URL发送一个POST请求，其中应该包含对象本身。class的名称必须以字母开头，只能包含字母、数字和下划线。当创建
成功时，HTTP的返回是201 Created，而header中的Location表示新的object的URL
相应的主体是一个JSON对象，包含新的对象的objectId和createAt时间戳。如果希望返回新创建的对象的完整信息，可以在 URL 里加上 fetchWhenSave 选项，并且设置为 true：
3. 获取对象
- 当你创建了一个对象时，你可以通过发送一个GET请求到返回header的Location以获取它的内容。
- 返回的主体是一个JSON对象包含所有用户提供的field加上createdAt、updateAt和objectId字段，类不存在时，返回404 Not Found错误，objectId不存在时，返回一个空对象
(HTTP状态吗为200 OK)
4. 更新对象
- 为了更改一个对象已经有的数据，你可以发送一个PUT请求到对象相应的URL上，任何你未指定的key都不会更改，所以你可以知更新对象数据的一个子集。
- 返回的JSON对象只会包含一个updateAt字段，表示更新发生的时间 计数器、位运算、按条件更新对象、__op操作汇总
5. 删除对象
- 为了在leanCloud上删除一个对象，可以发送一个DELETE请求到指定的对象的URL，还可以使用DELETE操作删除一个对象的一个字段（注意此时HTTP Method还是PUT）
- 按条件删除对象，使用where参数，特别强调：where一定要作为URL的Query Parameters传入
6. 遍历Class
- 因为更新和删除都是基于单个对象的，都要求提供objectId，但有时候用户需要高效地遍历一个Class，做一些批量的更新或者删除的操作。
- 通常情况下，如果Class的数量规模不大，使用查询加上skip和limit分页配合排序order就可以遍历所有数据。但是当Class数量规模比较大的时候，skip的效率就非常低了
(这跟MySQL等关系数据库的原因一样，深度翻页比较慢)，因此我们提供了scan协议，可以按照特定字段排序来高效地遍历一张表，默认这个字段是objectId升序，同时支持设置limit限定
每一批次的返回数量，默认limit为100，最大可设置为1000
- 默认情况下按onjectId升序排序，增加scan_key参数可以使用其他字段来排序
7. 批量操作
- 为了减少网络交互的次数太多带来的时间浪费，你可以在一个请求中对多个对象进行vreate、update、delete操作
- 在一个批次中每一个操作都有相同的方法、路径和主体，这些参数可以代替你通常会使用的HTTP方法，这些操作会以发送过去的顺序来执行。
- 批量操作还有一个冷门用途，代替URL国长的GET（比如使用containedin等方法构造查询）和delete（比如批量删除）请求，以绕过服务端和某些客户端对URL长度的限制
8. 数据类型
- 到现在为止我们只使用可以被标准JSON编码的值，leanCloud移动客户端SDK library同样支持日期、二进制数据和关系型数据。在REST API中，这些值都被编码了，同时有一个
__type字段来表示出它们的类型，所以如果拟采用正确的编码的话就可以读或者写这些字段。
- Date类型包含了一个iso字段，其值是一个UTC时间戳，以ISO 8601格式和毫秒级的精度来存储的是兼职格式为：YYYY-MM-DDTHH：MM：SS.MMMZ
- Byte类型包含了一个base64字段，这个字段是一些二进制数据编码过的base64字符串。base64是MIME使用的标准，不包含空白符
- Pointer类型是用来设定AVObject作为另一个对象的值时使用的，它包含了className和objectId两个属性值，用来提取目标对象
- Relation类型被用在多对多的类型上，移动端使用AVRelation作为值，它有一个className字段表示目标对象的类名
