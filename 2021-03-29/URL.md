## 从输入 URL 到展现页面的全过程

总结来说，从输入从URL开始，会发生下面一系列过程：
1. DNS域名解析,浏览器获得对应的IP地址
1. 浏览器和服务器的TCP链接（3次握手）
1. 浏览器发送HTTP/HTTPS请求
1. 服务器处理请求，并返回请求的资源（HTML，CSS，JS）
1. 浏览器解析并渲染界面，并呈现给用户
1. 断开TCP链接（4次挥手）
