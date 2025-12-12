# Obsidian+Git 意外关机应急检查清单

## 🔍 第一步：先检查错误提示

优先识别报错关键词，定位问题类型：

- 含 `reference broken`/`cannot lock ref` → 处理**损坏的引用文件**；
- 含 `merge failed` → 先处理文件冲突（Obsidian 会标记冲突文件）。

## 🛠️ 第二步：通用修复流程（核心）

1. **关闭 Obsidian**
    
    避免程序占用 Git 相关文件，导致无法修改。
    
2. **进入仓库的`.git`目录**
    
    打开 Obsidian 对应的仓库文件夹，进入隐藏的 `.git` 目录（Windows 需先开启「显示隐藏文件」）。
    
3. **删除损坏的引用文件**
    
    根据报错提示，删除对应的损坏文件（常见损坏文件：`refs/remotes/origin/main`、`ORIG_HEAD`、`FETCH_HEAD`）。
    
4. **执行 Git 清理命令**
    
    在仓库目录的终端中运行：
    
    bash
    
    运行
    
    ```bash
    # 清理无效的远程引用
    git remote prune origin
    
    # 重建正确的远程引用
    git fetch origin
    ```
    
5. **重启 Obsidian**
    
    测试 Git 同步功能（拉取 / 推送），确认恢复正常。
    

## 🚫 第三步：避免重复踩坑的小习惯

1. 关机 / 重启前，**手动执行一次「提交 + 推送」**，确保本地修改已同步到 GitHub；
2. Obsidian 的「自动提交间隔」设置为 **5-10 分钟**（避免 Git 文件频繁被占用）；
3. 定期在仓库终端运行 `git gc`，优化 Git 仓库文件，降低损坏概率。