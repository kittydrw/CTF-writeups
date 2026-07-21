# [LitCTF 2023]Flag点击就送！  
题目地址：https://www.nssctf.cn/problem/3872  
考点为 **flask session** 伪造  
密钥为 `LitCTF` 有点猜谜成分  

### 🧩Flask Session 是什么？

Flask 默认把 Session **整个存在客户端的 Cookie 里**（不是存在服务器内存或数据库）。  
Cookie 长这样：

```
eyJ1c2VyIjoiZ3Vlc3QifQ.Zm9v.YWJjZA==
```

它其实是用 `.` 分隔的三部分：

1. **数据部分**（Base64 编码的 JSON）  
2. **时间戳**（Base64 编码）  
3. **签名**（HMAC-SHA1，用 `SECRET_KEY` 计算）

服务器收到 Cookie 后，会用 `SECRET_KEY` 重新计算签名，对比第3部分是否一致。  
**只要密钥保密，用户就改不了数据**（改了签名就对不上）。  
**一旦密钥泄露，攻击者就能伪造任意数据并算出合法签名**。

---
### 🎯 本题目标

题目要求你访问 `/flag` 接口，但只有 **管理员** 才能看到 Flag。  
你注册登录后拿到的 Session 身份是普通用户（比如 `{"name":"guest"}`），所以直接访问 `/flag` 会被拒绝。

所以思路：**伪造一个管理员身份的 Session**。

---

### 🔍 第一步：获取普通用户的 Session Cookie

- 访问网站，打开 **F12 → 应用（Application）→ Cookies**，找到名为 `session` 的 Cookie，复制它的值。
- 
<img width="638" height="177" alt="image" src="https://github.com/user-attachments/assets/6ee30576-31cb-4260-89b4-54d07aedcfe4" />

---

### 🧪 第二步：解码 Cookie，看数据长什么样

Flask Session 的数据部分是 **第一段**（第一个 `.` 之前的内容），它是 Base64 编码的 JSON。  
你应该**只取第一段**进行 Base64 解码，而不是把整个 Cookie 放进去。

**正确做法**：

```python
import base64
cookie = "eyJuYW1lIjoiZ3Vlc3QifQ.ZG1haw.6X5T..."
data_part = cookie.split('.')[0]   # 取第一段
# 因为 Base64 可能包含填充 '='，要补全
decoded = base64.b64decode(data_part + '==')  # 如果长度不够补 '='
print(decoded)  # 输出 b'{"name":"guest"}'
```
或使用在线工具Cyberchef进行Base64解码

---

### 🗝️ 第三步：找到 SECRET_KEY（最关键）

题目告诉你密钥是 `LitCTF`。  
这个密钥通常通过以下方式泄露：

- 查看页面源码中的注释  
- 扫描到 `.git`、`config.py` 备份  
- 报错信息（Debug 模式开启）  
- 弱密码爆破  

本题中，密钥就是硬编码在源码里的 `LitCTF`。

---

### ✍️ 第四步：伪造 Session

现在你有密钥，知道数据结构是 `{"name":"xxx"}`，就可以把 `name` 改成 `admin`（或题目要求的管理员标识）。  
然后重新签名，生成一个合法的新 Cookie。
运行 `flask_session.py` 就是做这件事的：  

<img width="1306" height="340" alt="image" src="https://github.com/user-attachments/assets/9a8e1ced-6213-4f88-8cf7-dd46a2f85d88" />

```python
import subprocess
SECRET_KEY = "LitCTF"
# 用 flask-unsign 工具，基于密钥和新的数据生成签名 Cookie
cmd_out = subprocess.check_output([
    'flask-unsign', '--sign',
    '--cookie', '{\'name\': \'admin\'}',   # 要伪造的数据
    '--secret', SECRET_KEY
])
cookie = {'session': cmd_out.decode().rstrip()}
```

`flask-unsign --sign` 命令内部做了三件事：

1. 把 `{"name":"admin"}` 转成 JSON 字符串  
2. Base64 编码得到数据段  
3. 用 `SECRET_KEY` 算出时间戳和签名，拼接成完整的 Cookie  

最终生成的 Cookie 类似于：
```
eyJuYW1lIjoiYWRtaW4ifQ.ZG1haw.新的签名
```

---

### 🚀 第五步：用伪造的 Cookie 请求 `/flag`

脚本最后用 `requests.get()` 携带这个新 Cookie 去访问 `/flag`：

```python
response = requests.get('http://node4.anna.nssctf.cn:27126/flag', cookies=cookie)
print(response.text)
```

服务器收到 Cookie 后，用 `SECRET_KEY` 验证签名，发现合法，于是解析数据得到 `{"name":"admin"}`，判断你是管理员，返回 Flag。  

<img width="858" height="106" alt="image" src="https://github.com/user-attachments/assets/91f01de3-c425-443e-ad31-561bd0d13a82" />

---

### 📌 为什么能成功？

**因为服务器信任了签名，而签名是你用正确的密钥算出来的**。  
密钥泄露 → 信任体系崩塌 → 你可以伪装成任何人。

---

### ✅ 总结整个攻击链条

| 步骤 | 操作 | 目的 |
|------|------|------|
| 1 | 获取普通 Session Cookie | 拿到样本，了解数据结构 |
| 2 | 解码数据段 | 找到身份字段名（如 `name`） |
| 3 | 找到 `SECRET_KEY` | 获得伪造权限 |
| 4 | 修改数据并用密钥重新签名 | 生成管理员 Cookie |
| 5 | 用新 Cookie 访问 `/flag` | 获得 Flag |

整个过程的核心就是 **密钥泄露 + 客户端存储 Session** 的设计缺陷。
