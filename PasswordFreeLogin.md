# 免密登录（本地到服务器）
1.  **生成密钥对：**
    *   打开本地电脑的终端（Windows 下打开 PowerShell 或 CMD，Mac 打开 Terminal）。
    *   输入以下指令并一路回车（如果提示覆盖，说明你以前生成过，可以选择覆盖或直接用旧的）：
        ```bash
        ssh-keygen -t rsa
        ```
    *   这会在你本地的 `~/.ssh/` (Windows是 `C:\Users\你的用户名\.ssh\`) 目录下生成 `id_rsa` (私钥) 和 `id_rsa.pub` (公钥)。
2.  **获取【本地】的公钥内容：**
    *   在本地终端输入(Windows)： `type .ssh\id_rsa.pub`
    *   **复制**打印出来的所有内容（以 `ssh-rsa` 开头的那一大串字符）。

3.  **将公钥放入【服务器】：**
    *   输入以下指令创建目录（如果不存在）并打开权限文件：
        ```bash
        mkdir -p ~/.ssh
        vim ~/.ssh/authorized_keys
        ```
    *   **将刚才复制的本地公钥粘贴进去**。

4.  **测试：**
    *   连接。此时应该不再询问密码。   