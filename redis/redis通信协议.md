P173没看

CRLF代表"\r\n"
[理解 Redis 的 RESP 协议](https://moelove.info/2017/03/05/%E7%90%86%E8%A7%A3-Redis-%E7%9A%84-RESP-%E5%8D%8F%E8%AE%AE/)
* 单行回复：'+'开头，CRLF结尾
* 错误信息：'-'开头，CRLF结尾
* 整形数字：':'开头，CRLF结尾
* 多行字符串：'\$'+字节数+CRLF+数据+CRLF。空字符串为"\$0\r\n\r\n"，当要表示不存在的值时候，则以"$-1\r\n"返回
* 数组：'*'+数组长度+CRLF+数组内容，其中数组内容可以是这五种类型之一

redis6之后出现了RESP3，新增了很多内容。但是redis6默认还是使用RESP2，上面介绍的是RESP2。