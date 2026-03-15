# 实验名称：管理员功能不是受保护，URL不可预测
## 基本信息
- **难度** 初级
- **类型** 访问控制
## 漏洞原理
- 敏感路径暴露在了前端文件里
- 管理面板没有受保护
## 攻击步骤
- CTRL+U 在前端文件中寻找（admin /admin panel等关键词可能是某个隐藏的 <script> 标签 var adminPanelPath = "/admin-xxxxxxxx";）
- 发现路径 /admin-ra8pzi -> 直接访问管理控制面板
- 删除目标
## 漏洞修复
- 混淆前端js内容
- 把内容移植到后端不把敏感路径写到前端
- 添加管理控制面板保护身份验证
## 总结
- 前端内容不做敏感内容
- "隐藏即安全" 不等于真正的安全（Security through obscurity 是反模式）
- 管理功能必须加身份验证，不能仅依赖 URL 不可预测性
## 链接
- https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/access-control-apprentice/access-control/lab-unprotected-admin-functionality-with-unpredictable-url