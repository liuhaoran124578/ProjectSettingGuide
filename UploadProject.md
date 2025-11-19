# 1. 初始化 Git 仓库
这步操作会在你本地生成一个 `.git` 隐藏文件夹，让你的项目变成一个 Git 仓库。
```bash
git init
```

# 2. 将文件添加到暂存区
这一步是告诉 Git：“把你看到的这些文件（除了 .gitignore 里忽略的）准备好，我要存起来了。”
```bash
git add .
```
*(注意：`add` 后面有个空格和一个点 `.`，代表当前目录下的所有文件)*

# 3. 提交文件到本地仓库
这一步相当于给当前的代码拍一张快照，并写下备注。
```bash
git commit -m "第一次提交我的项目"
```

# 4. 关联 GitHub 远程仓库
*   首先，去 [GitHub 官网](https://github.com) 登录你的账户。
*   点击右上角的 **+** 号 -> **New repository** (新建仓库)。
*   起个名字，点击 **Create repository**。
*   在创建成功的页面，你会看到一个 HTTPS 的链接（例如 `https://github.com/你的用户名/项目名.git`），复制它。

回到你的终端，输入：
```bash
git remote add origin https://github.com/你的用户名/你的项目名.git
```
*(请将 URL 替换为你刚才复制的链接)*

# 5. 推送代码到 GitHub
这是最后一步，把本地的代码上传到云端。
```bash
git branch -M main
git push -u origin main
```
*(解释：第一行是将分支重命名为 main，这是现在的标准；第二行是推送到远程的 main 分支)*
