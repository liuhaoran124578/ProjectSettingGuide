# 服务器连接 GitHub 
1.  **在【服务器】生成秘钥：**
    *   在 VS Code 的远程终端中输入：
        ```bash
        ssh-keygen -t rsa -C "你的GitHub邮箱@example.com"
        ```
    *   一路回车即可。

2.  **获取【服务器】的公钥：**
    *   输入：
        ```bash
        cat ~/.ssh/id_rsa.pub
        ```
    *   **复制**打印出来的内容。

3.  **添加到 GitHub 网站：**
    *   打开 GitHub 网页 -> 点击右上角头像 -> **Settings**。
    *   左侧栏找到 **SSH and GPG keys**。
    *   点击 **New SSH key**。
    *   Title 随便填（比如 "My Cloud Server"），在 Key 栏中**粘贴**刚才复制的内容。
    *   点击 **Add SSH key**。

4.  **验证连接：**
    *   在【服务器】终端输入：
        ```bash
        ssh -T git@github.com
        ```
    *   如果出现询问 `Are you sure...`，输入 `yes` 回车。
    *   看到 `Hi <你的用户名>! You've successfully authenticated...` 就成功了。