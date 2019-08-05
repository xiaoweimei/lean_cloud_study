# user用户
- 不仅在移动应用上，还在其他系统中，很多应用都有一个统一的登陆流程，通过REST API访问用户的账户让你可以在leanCloud上简单实现这一功能。通常来说，用户这个
类的功能与其他的对象时相同的，比如都没有限制模式。user对象和其他对象不同的是一个用户必须有用户名(username)和密码(password)，密码会被自动地加密和存储。
leanCloud强制要求username和email这两个字段必须是没有重复的(大小写敏感)
1. 注册
- 注册一个新用户与创建一个新的普通对象之间的不同点在于username和password字段都是必须的。password字段会以和其他的字段不一样的方式处理，它在存储时会被加密
而且永远不会被返回给任何来自客户端的请求。
- 为了注册一个新的用户，需要向user路径发送一个POST请i去，你可以加入一个新的字段，比如，创建一个新的用户有一个电话号码
- 当创建成功时，HTTP返回为201 Created，location头包含了新用户的URL
- 返回的主体是一个JSON对象，包含objectId、createdAt时间戳表示创建对象时间，sessionToken可以被用来验证这名用户随后的请求。
2. 登陆
- 在你允许用户注册之后，在以后你需要让它们用自己的用户名和密码登陆。为了做到这一点，发送一个POST请求到/1.1/login，加上username和password作为body
- 用户也可以通过邮箱地址和密码登陆，只需要将body中的username换成email
- 返回的主体是一个JSON对象包括所有除了password以外的自定义字段。它同样包含了createAt、updateAt、objectId和sessionToken字段
- 可以将 sessionToken 理解为用户的登录凭证，每个用户的 sessionToken 在同一个应用内都是唯一的， 类似于 Cookie 的概念。
- 正常情况下，用户的 sessionToken 是固定不变的，但在以下情况下会发生改变：
- 客户端调用了忘记密码功能，重设了密码。
- 开发者在 控制台 > 存储 > 设置 > 用户账号 中勾选了 密码修改后，强制客户端重新登录，那么在修改密码后 sessionToken 也将强制更换。
- 调用 refreshSessionToken 主动重置。
- 在 sessionToken 变化后，已有的登录如果调用到用户相关权限受限的 API，将返回 403 权限错误。
3. 已登陆的用户信息
- 用户成功注册或登陆后，服务器会返回sessionToken并保存在本地，后续请求可以通过传递sessionToken来获取该用户信息(如访问权限等)，返回的JSON数据与/login登陆请求所返回的相同
4. 重置登陆sessionToken
- 可以主动重置用户的sessionToken
5. 使用手机号码注册或登陆
- 请参考[地址](https://leancloud.cn/docs/rest_sms_api.html#hash-1925130390)
6. 验证Email
- 设置email验证是app设置中的一个选项，通过这个表示，应用层可以对提供真实email的用户更好的功能挥着体验。Email验证会在User对象中加入emailVerified字段，当一个用户
的email被新设置或者修改过的话，emailVerified会被重置为false。leanCloud后台会往用户填写的邮箱发送一个验证链接，用户点击这个链接可以让emailVerified被设置为true
7. 请求验证Email
- 发送给用户的邮箱验证邮件在一周内失效，你可以调用/1.1/requestEmailVerify来强制重新发哦是那个
8. 请求密码重设
- 在用户将email与他们的账户关联起来后，你可以通过邮件来重设密码。操作方法为，发送一个POST请求到/1.1/requestPasswordReset，同时在request的body部分带上email字段
- 如果成功的话，返回的值是一个JSON对象
- 关于自定义邮件模板和验证链接请看这篇博客文章[文章地址](https://blog.leancloud.cn/607/)
9. 手机号码验证
- 请参考下面[地址](https://leancloud.cn/docs/rest_sms_api.html#hash915522908)
10. 获取用户
- 你可以发送一个GET请求到URL以获取用户的账户信息，返回的内容就是当创建用户时返回的内容。
- 返回的body是一个JSON对象，包含所有用户提供的字段，除了密码以外，也包括了createdAt、updateAt和objectId字段
- 用户不存在时返回400Bad Request错误
11. 更新用户
- 在通常的情况下，没有人会允许别人来改动他们自己的数据。为了做好权限认证，确保只有用户自己可以修改个人数据，在更新用户信息的时候，必须在 HTTP 头部加入一个 X-LC-Session 项来请求更新，这个 session token 在注册和登录时会返回。
- 了改动一个用户已经有的数据，需要对这个用户的 URL 发送一个 PUT 请求。任何你没有指定的 key 都会保持不动，所以你可以只改动用户数据中的一部分。username 和 password 也是可以改动的，但是新的 username 不能和既有数据重复。
- 返回的body是一个JSON对象，只有一个updateAt字段表明更新发生的时间
12. 安全地修改用户密码
- 修改密码，可以直接使用上面的PUT /1.1/users/:objectId的 API，但是很多开发者会希望让用户输入一次旧密码做一次认证，旧密码正确才可以修改为新密码，因此我们提供了一个
单独的API PUT 1.1/users/:objectId/updatePassword 来安全地更新密码：注意：仍然需要传入 X-LC-Session，也就是登录用户才可以修改自己的密码。
13. 查询
- 你可以一次获取多个用户，只要向用户的根 URL 发送一个 GET 请求。没有任何 URL 参数的话，可以简单地列出所有用户：
- 返回的值是一个 JSON 对象包括一个 results 字段，值是包含了所有对象的一个 JSON 数组。
- 所有的对普通对象的查询选项都适用于对用户对象的查询，所以可以查看 查询 部分来获取详细信息。
14. 删除用户
- 为了在 LeanCloud 上删除一个用户，可以向它的 URL 上发送一个 DELETE 请求。同样的，你必须提供一个 X-LC-Session 在 HTTP 头上以便认证。
15. 连接用户账户和第三方平台
- LeanCloud 允许你连接你的用户到其他服务，比如新浪微博和腾讯微博，这样就允许你的用户直接用他们现有的账号 id 来注册或登录你的应用。在进行注册和登录时，
需要附带上 authData 字段来提供你希望连接的服务的授权信息。一旦关联了某个服务，authData 将被存储到你的用户信息里。
- authData 是一个普通的 JSON 对象，它所要求的 key 根据 service 不同而不同，具体要求见下面。每种情况下，你都需要自己负责完成整个授权过程(一般是通过 OAuth 协议，1.0 或者 2.0) 来获取授权信息，提供给连接 API。
16. 安全
- 当你用 REST API key 来访问 LeanCloud 时，访问可能被 ACL 所限制，就像 iOS 和 Android SDK 上所做的一样。你仍然可以通过 REST API 来读和修改，只需要通过 ACL 的 key 来访问一个对象。
- CL 按 JSON 对象格式来表示，JSON 对象的 key 是 objectId 或者一个特别的 key（*，表示公共访问权限）。ACL 的值是权限对象，这个 JSON 对象的 key 即是权限名，而这些 key 的值总是 true。
- 个例子，如果你想让一个 id 为 55a47496e4b05001a7732c5f 的用户有读和写一个对象的权限，而且这个对象应该可以被公共读取，符合的 ACL 应该是:
```
{
  "55a47496e4b05001a7732c5f": {
    "read": true,
    "write": true
  },
  "*": {
    "read": true
  }
}
```
