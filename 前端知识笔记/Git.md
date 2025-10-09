---
date: 2025-06-17
tags:
  - Git
---
### **Git 核心概念**

- **版本控制系统（VCS）**
  - 跟踪代码变更历史，支持回退、分支管理、协作开发。

- **Git 工作流程**
  ```mermaid
  graph LR
  A[工作目录] --> B[暂存区] --> C[本地仓库] --> D[远程仓库]
  ```

  - **工作目录**：本地修改的文件
  - **暂存区 (Stage)**：`git add` 后的临时存储区
  - **本地仓库**：`git commit` 后生成永久快照
  - **远程仓库**：云端存储（如GitHub）

  ​

### **Git 核心命令**

| **操作**         | **命令**                                 | **说明**                            |
| :--------------- | :--------------------------------------- | :---------------------------------- |
| **初始化仓库**   | `git init`                               | 创建新仓库                          |
| 克隆仓库         | `git clone <url>`                        | 下载远程仓库                        |
| **提交更改**     | `git add <file>` → `git commit -m "msg"` | 提交到本地仓库                      |
| **查看状态**     | `git status`                             | 显示工作区/暂存区状态               |
| 查看提交历史     | `git log`                                | 显示提交记录（加 `--oneline` 简化） |
| **创建分支**     | `git branch <branch-name>`               | 新建分支                            |
| **切换分支**     | `git checkout <branch-name>`             | 切换到指定分支                      |
| **合并分支**     | `git merge <branch>`                     | 将目标分支合并到当前分支            |
| **拉取远程更新** | `git pull origin <branch>`               | 拉取代码并自动合并                  |
| **推送到远程**   | `git push origin <branch>`               | 上传本地提交到远程                  |
| 撤销工作区修改   | `git restore <file>`                     | 丢弃未暂存的修改                    |
| 撤销暂存区文件   | `git restore --staged <file>`            | 将文件移出暂存区                    |



### **Git 高级操作**

- **分支管理策略**
  - **主分支**：`main`/`master`（稳定版本）
  - **开发分支**：`dev`（集成新功能）
  - **功能分支**：`feature/xxx`（按功能拆分）

- **==解决冲突==**
  1. 发现冲突
     - 当执行`git merge` 或 `git pull`时，如果发生冲突，Git会在终端中提示

  2. **手动解决冲突**
     - 手动打开冲突文档（例如：`your-file.txt`）
          ```
       <<<<<<< HEAD
       print("Hello, World!")
       =======
       print("Hi, there!")
       >>>>>>> other-branch-name
       ```
     - `<<<<<<< HEAD` 到 `=======` 之间，是自己修改的代码（`HEAD` 代表目前所在的分支）
     - `=======` 到 `>>>>>>> other-branch-name` 之间，是别人（或另一分支）修改的代码
     - **刪除 `<<<`, `===`, `>>>` 标记，並決定最终保存的内容**

  3. 标记为「已解決」并提交
     - 使用`git add`标记为已解决
       ```bash
       git add your-file.txt
       ```
     - 执行`git commit`完成合并
       ```bash
       git commit
       ```

- **版本回退**
  - `git reset --hard <commit-id>`：回退到指定提交（**谨慎使用**）

- **恢复删除的分支**
  - `git reflog` 查找分支最后提交ID → `git branch <name> <commit-id>`

- **撤销已推送提交**
  ```bash
  git revert <commit-id>  # 新建抵消提交（推荐）
  git reset --hard HEAD~1 && git push -f  # 强制覆盖（慎用）
  ```



### **GitHub 核心功能**

- **远程仓库操作**
  - 关联远程仓库：`git remote add origin <url>`
  - 推送到远程：`git push -u origin main`（首次加 `-u`）

- **Pull Request（PR）流程**
  ```mermaid
  graph LR
  A[Fork仓库] --> B[本地修改] --> C[推送分支] --> D[创建PR] --> E[代码审查] --> F[合并]
  ```


### **重要提示**

1. 频繁 `commit`，小步提交
2. 推送前先 `pull` 避免冲突
3. 合并前使用 `git diff` 检查变更
4. 敏感数据勿提交（用 `.gitignore` 过滤）