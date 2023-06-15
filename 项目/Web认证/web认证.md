单点认证：session+cookie

分布式系统：jwt、token+redis、分布式session


多点认证：CAS系统


https://blog.nowcoder.net/n/7a6a4758eef4444ea7ea301640cb46db
https://www.cnblogs.com/54chensongxia/p/13491214.html


JWT: 生成并发给客户端之后，后台是不用存储，客户端访问时会验证其签名、过期时间等再取出里面的信息（如username），再使用该信息直接查询用户信息完成登录验证。jwt自带签名、过期等校验，后台不用存储，缺陷是一旦下发，服务后台无法拒绝携带该jwt的请求（如踢除用户）；

token+redis： 是自己生成个32位的key，value为用户信息，访问时判断redis里是否有该token，如果有，则加载该用户信息完成登录。服务需要存储下发的每个token及对应的value，维持其过期时间，好处是随时可以删除某个token，阻断该token继续使用