## Cookie 和 Session 的关系和区别是什么？

1. 位置：Cookie 在客户端（浏览器），Session 在服务器端。
2. 安全性： Cookie 的安全性一般，他人可通过分析存放在本地的 Cookie 并进行 Cookie 欺骗。在安全性第一的前提下，选择 Session 更优。重要交互信息比如权限等就要放在 Session 中，一般的信息记录放 Cookie 就好了。
3. 大小：单个 Cookie 保存的数据不能超过 4K。
4. 性能：Session 可以放在 文件、数据库或内存中，比如在使用 Node 时将 Session 保存在 redis 中。由于一定时间内它是保存在服务器上的，当访问增多时，会较大地占用服务器的性能。考虑到减轻服务器性能方面，应当适时使用 Cookie。
5. 依赖关系：Session 的运行依赖 Session ID，而 Session ID 是存在 Cookie 中的，也就是说，如果浏览器禁用了 Cookie，Session 也会失效（但是可以通过其它方式实现，比如在 url 中传递 Session ID）。
6. 使用场景：用户验证这种场合一般会用 Session。因此，维持一个会话的核心就是客户端的唯一标识，即 Session ID。

