# 1zjs  
题目地址：https://www.nssctf.cn/problem/3871  

<img width="483" height="244" alt="image" src="https://github.com/user-attachments/assets/534985f7-0907-4c29-8043-ae2b3d958253" />

打开网址，看到一个魔方，按F12，找到提示  
<img width="694" height="316" alt="image" src="https://github.com/user-attachments/assets/65ff7738-6ecf-463d-80a3-8d9bd9c2d0fc" />

然后去源代码中的index.umd.js代码，找到注释中的提示信息：`Your gift just take it : /f@k3f1ag.php`  
<img width="1043" height="279" alt="image" src="https://github.com/user-attachments/assets/5cd5252e-9a76-4741-b3eb-99030e808be9" />

在访问的网址后加上`/f@k3f1ag.php`，得到这个页面  
<img width="1914" height="378" alt="image" src="https://github.com/user-attachments/assets/60dce4cb-db37-491a-852f-aa2356e974bb" />

由于我不了解JSfuck的相关内容，所以我去查找学习了一下，以下是有关JSfuck的相关信息  
><img width="804" height="839" alt="image" src="https://github.com/user-attachments/assets/fa4d2688-b53c-45fd-acc6-037d659576f9" />

>JSFuck 是一种极其特殊的 **JavaScript 代码混淆（Obfuscation）技术**。在 CTF 比赛中，它常作为一种编码/混淆手段，出现在 Web 或 Misc 类型的题目里。
>它的核心特点是用 `[]()!+` 这**六种字符**来重写任何 JavaScript 代码。
>
>### 🤔 什么是 JSFuck？它如何工作？
>*   **它是什么**：一种深奥的编程风格，仅用 `[ ] ( ) ! +` 六个字符编写可执行的 JavaScript 代码。
>*   **如何工作**：利用 JavaScript 是弱类型语言的特性，通过这六个字符的组合和强制类型转换，构造出数字、字符串，进而拼凑出任意代码。
>*   **特征**：编码后的代码**极其冗长**，且仅由 `()[]+!` 组成。例如，`alert("Hello")` 会被转换成超过 2 万字符的乱码。
>
>### 🎯 CTF 中如何结合 JSFuck 出题？  
>在 CTF 中，JSFuck 主要扮演“绊脚石”的角色，用于隐藏关键信息。常见的出题方式有：  
>1.  **隐藏在网页源码中**：将 Flag 用 JSFuck 编码后藏在 HTML 注释里。解题者需查看源码发现并解码。
>2.  **作为 JavaScript 代码执行**：将一段关键 JS 逻辑（如生成 Flag、验证输入）用 JSFuck 混淆。代码在浏览器中可正常运行，但人类无法直接阅读。
>3.  **作为独立文件提供**：提供一个内容全是 `()[]+!` 的文件作为附件。解题者需要识别并解码以提取信息。
>4.  **结合其他混淆技术**：将 JSFuck 与其他编码（如 Base64）叠加使用，增加题目复杂度。
>
>### 🧑‍💻 作为出题人，如何构造 JSFuck 代码？  
>1.  **准备原始代码**：编写要实现功能的 JavaScript 代码，例如：`console.log("flag{...}");` 或 `var flag = "...";`。
>2.  **使用在线工具编码**：
>    *   访问 [JSFuck.com](https://jsfuck.com/)。
>    *   将你的 JS 代码粘贴到输入框。
>    *   点击编码按钮，即可生成仅含 `()[]+!` 的混淆代码。
>3.  **将编码结果植入题目**：
>    *   **Web 题**：放入 HTML 的 `<script>` 标签内，或作为 `onclick` 等事件属性值。
>    *   **Misc 题**：保存为 `.js` 文件或 `.txt` 文本文件供下载。
>
>### 🔧 作为解题人，如何解码 JSFuck？  
>识别出 JSFuck 后，有几种简单的解码方法：  
>1.  **浏览器控制台（最直接）**：
>   *   按 `F12` 打开开发者工具，切换到 **Console（控制台）** 选项卡。
>   *   将 JSFuck 代码粘贴进去，按回车。
 >   *   浏览器会直接执行并输出结果。这是最快的方法，但要注意，如果代码有恶意行为（如弹窗），也会被执行。  
>2.  **在线解码工具（更安全）**：
 >   *   使用在线 JSFuck 解码器，例如 [JSUnFuck](http://codertab.com/JsUnFuck)、[Decoder-JSFuck](https://enkhee-osiris.github.io/Decoder-JSFuck/)或 `ctf.ssleye.com` 上的工具。将代码粘贴进去，即可获得可读的 JavaScript 源码。
>
>总的来说，JSFuck 是一种有趣且极端的混淆方式。在 CTF 中，只要识别出其 `()[]+!` 的特征，利用浏览器控制台或在线工具就能快速解码，关键在于识别它。

然后ctrl+A复制全部内容后，按F12，在控制台中粘贴，按回车运行结果就是flag  

<img width="848" height="869" alt="image" src="https://github.com/user-attachments/assets/1b16461f-884f-4708-a353-25ed5dcad9fc" />  

得到flag:  **NSSCTF{6a0107e2-b664-4a29-90e8-b90313199015}**
<img width="744" height="55" alt="image" src="https://github.com/user-attachments/assets/8c5b0459-08d0-4e0a-87c8-82f0967d665d" />



