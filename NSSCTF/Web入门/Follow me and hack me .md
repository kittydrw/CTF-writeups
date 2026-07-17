# Follow me and hack me
题目地址：https://www.nssctf.cn/problem/3864  

## 题目需要我们传入GET和POST对应的参数  
### GET 参数:  
- 直接在浏览器地址栏输入：http://node5.anna.nssctf.cn:28255/?CTF=Lit2023 （原来网址/?CTF=Lit2023）
<img width="675" height="655" alt="image" src="https://github.com/user-attachments/assets/a1736480-b077-42db-ac9b-37b90b958fb9" />

### POST 参数:  
#### 用 Burp Suite（专业抓包工具）：
1. 打开 Burp
2. 访问目标页面，Burp 会拦截请求。
3. 在 Burp 的 **Proxy → Intercept** 中抓到 GET 请求后，可以：
   - 右键 → **Send to Repeater**（发送到重放器）。
   ><img width="1147" height="559" alt="image" src="https://github.com/user-attachments/assets/1367250f-8d39-4f27-a076-05634d92c191" />

   - 在 Repeater 中，右键选择 **Change request method** 将请求方法改为 `POST`。
   ><img width="562" height="533" alt="image" src="https://github.com/user-attachments/assets/327d51c7-7f8c-4621-930e-fc495445d244" />
   
   - 在请求头部下面空一行，添加请求体（Body），例如：
     ```
     Challenge=i'm_c0m1ng
     ```
   ><img width="570" height="396" alt="image" src="https://github.com/user-attachments/assets/b2d99fd0-5a48-4956-92dd-4edfed3952db" />

   - 同时，如果还需要 GET 参数，可以直接在 URL 后面加 `?CTF=Lit2023`。
   - 点击 **Go** 发送，查看响应。
     
4. 如果题目需要同时提交 GET 和 POST，你可以在同一个请求中**同时**：
   - 在 URL 中加 GET 参数。
   - 在 Body 中加 POST 数据。  

> 如果题目只允许一次请求就同时传递 GET 和 POST，你可以把 GET 参数写在 URL 里，同时用 fetch 发送 POST，这样一次请求就同时包含了两种方式。

<img width="575" height="593" alt="image" src="https://github.com/user-attachments/assets/f27b6090-f545-4449-bbda-a85b3d10e389" />  

最后在Response的Render中找到flag  

`NSSCTF{d72aee58-14ba-46b9-92dc-4d9b9bc2f0cd}`

