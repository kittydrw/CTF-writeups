# CTF Crypto Writeup: Xor很心疼你

## 题目信息
- **题目名称**：Xor很心疼你
- **题目类型**：Crypto
- **难度**：中等
- **考点**：随机数种子、流密码、密钥空间爆破

---

## 一、题目分析

### 1.1 题目附件

题目提供了一个Python加密脚本 `easyXor.py`：

```python
# Python3
from secret import flag
import random
import base64

pool = 'qwertyuiopasdfghjklzxcvbnm1234567890QWERTYUIOPASDFGHJKLZXCVBNM'
r = random.randint(2, 250)
assert flag.startswith('hsctf{')

def generate(length):
    return ''.join(random.choices(pool, k=length))

def f(x):
    random.seed(x)
    return random.getrandbits(8)

def encrypt(plaintext, key):
    plaintext = list(map(ord, plaintext))
    for _ in range(20):
        key = f(key)
        assert key != 0
    for i in range(len(plaintext)):
        key = f(key)
        tmp = (key * r) % 251
        assert tmp != 0 and key != 0
        plaintext[i] = plaintext[i] ^ tmp
    plaintext = bytes(plaintext)
    return base64.b64encode(plaintext)

m = generate(random.randint(200, 300)) + flag + generate(random.randint(200, 300))
c = encrypt(m, random.getrandbits(128))
print(c)
```

### 1.2 加密流程分析

1. **明文构造**：
   ```
   plaintext = random_padding(200-300) + flag + random_padding(200-300)
   ```
   flag被夹在两段随机填充之间，flag位置未知。

2. **密钥生成**：
   - 初始密钥：128位随机数
   - 通过`f(key)`迭代20次后开始加密

3. **加密过程**（流密码）：
   ```
   对每个明文字节：
     key = f(key)
     tmp = (key * r) % 251
     ciphertext[i] = plaintext[i] ^ tmp
   ```

4. **关键约束**：
   - `tmp != 0` 且 `key != 0`
   - `r` 取值范围：2-250
   - `key` 取值范围：0-255（但实际加密中不能为0）

---

## 二、漏洞发现

### 2.1 核心漏洞：f(x)函数

```python
def f(x):
    random.seed(x)
    return random.getrandbits(8)
```

**关键特征**：
- 每次调用都会用`x`重置随机数种子
- 返回值只依赖于`x`
- 值域：0-255（8位）

这意味着`f(x)`是一个**确定性函数**，且只有256种可能的状态。

### 2.2 密钥空间缩小

虽然初始密钥是128位，但经过20次`f`迭代后：
```
key0 → f(key0) → f(f(key0)) → ... → key20
```

由于`f`的结果范围是0-255，实际加密使用的密钥`key20`只有256种可能。

同时`r`的范围是2-250，共249种可能。

**总密钥空间**：256 × 249 ≈ 63,744种组合

这完全可以在可接受时间内爆破。

---

## 三、解题思路

### 3.1 枚举策略

1. 枚举所有可能的初始密钥 `key0` (1-255)
2. 枚举所有可能的乘数 `r` (2-250)
3. 对每个组合尝试解密
4. 利用已知明文格式验证结果

### 3.2 验证条件

从题目代码中提取的约束条件：

1. **前填充验证**：
   - 长度：200-300字节
   - 字符集：`pool`中的62个字符

2. **Flag格式验证**：
   - 以`hsctf{`开头
   - 以`}`结尾
   - 中间为可打印ASCII字符

3. **后填充验证**：
   - 长度：200-300字节
   - 字符集：`pool`中的62个字符

### 3.3 优化策略

- 在前200个字节就验证字符集，提前剪枝
- 预计算f(x)映射表，避免重复计算
- 找到第一个符合条件的就输出

---

## 四、解题脚本

### 4.1 完整脚本

```python
import base64
import random

# 题目给出的 Base64 密文
ct_b64 = b'8OcTbAfL6/kOMQnC9v8SNmmSzvQMeGTT8vANM1T+7vIce2fo0fc2RnScrNxTSmeSyuMjMF//w8BWaXX91dsGcnvmreg0NQTw96ceVVXj3sQ3Znn51OU1S0bOyaMtNHTj36AcWFqewN4zRUXD6agGbAPE+tQtd3XG0doAa1Ll9fhcQ1zk0McTM1bv8PIQOAnn3vQ3UgLD3PsONXLs4KkXMnjTyMEQOFn/0uYVUwOY1PsleEHCyNopRVDr+Kc0e2PH9v0XNXfprfIPU3nw7KYTNX/G7twLSkHoyaUlQHXi3v02UHmdy/4iNgme3Pc8bgPp+tYWV1+YzPkXYkXM4ulUc27DrM4SNUPT2fQlckj1qP4Fal+YoPYJMlyZ8qhXfF3Y0tUDdUXl3vg0dFTi++VVOFfH/dgMS1ru9N8WU0HF9cUCTgPe+qVdSn/u7Mkda0GTw/QDcWPZ9KYGN2jSzfk0OVrMzt0yRHD64KMrUgPF2sFWcmP56KZSTAD61PUGeXrd49MgU1bL8OsVNWj91vIsalXwqf0qaWbwzv0lWETA4eElS3L99cYmU1nv9dRQTWbDyclScQTN6NIhV2j//+ZWbH7Z68kwM3Dy4dcUc1PQy8kRTl/4zcU9WGWfoakOMXuf69MXZQTEz+kJT1Dar8UN'

# Base64 解码得到密文字节
ct = base64.b64decode(ct_b64)

# 填充字符集
pool = b'qwertyuiopasdfghjklzxcvbnm1234567890QWERTYUIOPASDFGHJKLZXCVBNM'
pool_set = set(pool)

# 预计算 f(x) 的映射表，x = 0..255
f = [0] * 256
for i in range(256):
    rand = random.Random(i)
    f[i] = rand.getrandbits(8)


def try_decrypt(key0, r):
    """尝试用 key0 和 r 解密，如果明显错误则返回 None"""
    x = key0
    out = bytearray()

    for i, b in enumerate(ct):
        x = f[x]
        tmp = (x * r) % 251

        # 加密过程中 tmp 和 key 都不能为 0
        if tmp == 0:
            return None

        p = b ^ tmp

        # 前 200 个字节一定是随机填充，必须来自 pool
        if i < 200 and p not in pool_set:
            return None

        out.append(p)

    return out


print("[*] 开始爆破 key0 和 r ...")

for key0 in range(1, 256):
    for r in range(2, 251):
        plain = try_decrypt(key0, r)
        if plain is None:
            continue

        # 查找 flag 开头
        pos = plain.find(b'hsctf{')
        if pos == -1:
            continue

        # 前填充长度必须是 200~300
        if not (200 <= pos <= 300):
            continue

        # 前填充必须全部来自 pool
        if not all(b in pool_set for b in plain[:pos]):
            continue

        # 查找 flag 结尾 }
        end = plain.find(b'}', pos + 1)
        if end == -1:
            continue

        flag = plain[pos:end + 1]

        # flag 内容应为可打印 ASCII
        if any(c < 32 or c > 126 for c in flag):
            continue

        # 后填充长度必须是 200~300
        tail = plain[end + 1:]
        if not (200 <= len(tail) <= 300):
            continue

        # 后填充必须全部来自 pool
        if not all(b in pool_set for b in tail):
            continue

        print(f"[+] 找到！r = {r}, key0 = {key0}")
        print(flag.decode())
```

### 4.2 脚本说明

1. **预计算优化**：将256种可能的`f(x)`结果预先计算并存储，避免重复计算

2. **提前剪枝**：在解密前200个字节时就开始验证字符集，快速排除错误组合

---

## 五、运行结果

<img width="421" height="115" alt="image" src="https://github.com/user-attachments/assets/d2ace0a5-6eca-4de4-9953-bd095e88483e" />

---

## 六、解题技巧总结

### 6.1 关键思路

1. **找确定性**：发现`f(x)`是确定性函数，减少了密钥空间

2. **利用约束**：充分使用题目中所有的约束条件进行验证

3. **提前剪枝**：在解密过程中尽早验证，减少无效计算

4. **并行优化**：利用多核CPU加速爆破

### 6.2 CTF Crypto常见漏洞模式

1. **随机数种子可控**：如果随机数种子是已知或可枚举的，可以预测随机数

2. **密钥空间过小**：密钥取值范围有限，可以暴力破解

3. **确定性函数**：如本题的`f(x)`，给定输入永远输出相同结果

4. **已知明文攻击**：知道部分明文格式（如flag前缀）可以验证解密结果

### 6.3 防御建议

1. 使用密码学安全的随机数生成器（CSPRNG）
2. 避免使用可预测的随机数种子
3. 密钥空间应足够大（至少128位）
4. 不要依赖确定性函数生成密钥流

---

## 七、扩展思考

### 7.1 如果密钥空间更大怎么办？

如果`f(x)`返回16位而不是8位，密钥空间变为65536 × 249 ≈ 1630万，仍然可以爆破但时间更长。可以使用：
- GPU加速
- 分布式计算
- 更优化的剪枝策略

### 7.2 如果没有已知明文格式怎么办？

可以使用：
- 熵分析（随机数据的熵较高）
- 统计特征（如英文文本的频率分布）
- 已知的压缩或编码格式头

### 7.3 这类漏洞在真实世界中的应用

类似的漏洞在实际系统中也出现过：
- PHP的mt_srand()种子可预测
- Java的Random类在特定场景下可预测
- 使用时间戳作为种子的随机数生成

---
**最终Flag**: `hsctf{x0r_i5_v4ry@eAsy_1oakn29gm3m3k93}`
