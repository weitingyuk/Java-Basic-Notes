## HTTP 中 GET 和 POST 区别
### 1. 两种 HTTP 请求方法：GET 和 POST
在客户机和服务器之间进行请求-响应时，两种最常被用到的方法是：GET 和 POST。

- GET - 从指定的资源请求数据。
- POST - 向指定的资源提交要被处理的数据。

### 2. 比较 GET 与 POST
- 后退按钮/刷新:  
    - GET无害
    - POST数据会被重新提交（浏览器应该告知用户数据会被重新提交）。
- 缓存:   GET能被缓存, POST不能缓存
- 编码类型:
    - GET application/x-www-form-urlencoded 
    - POST application/x-www-form-urlencoded or multipart/form-data。为二进制数据使用多重编码。
- 历史: GET参数保留在浏览器历史中。POST参数不会保存在浏览器历史中。
- 对数据长度的限制:
    - 当发送数据时，GET 方法向 URL 添加数据；URL 的长度是受限制的（URL 的最大长度是 2048 个字符）。
    - POST无限制。
- 对数据类型的限制:
    - GET只允许 ASCII 字符。
    - POST没有限制。也允许二进制数据。
- 安全性:
    - 与 POST 相比，GET 的安全性较差，因为所发送的数据是 URL 的一部分。
    - 在发送密码或其他敏感信息时绝不要使用 GET ！  POST 比 GET 更安全，因为参数不会被保存在浏览器历史或 web 服务器日志中。
- 可见性:
    - GET数据在 URL 中对所有人都是可见的。
    - POST数据不会显示在 URL 中。