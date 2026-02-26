---
name: git-push
description: 执行 Git 提交并推送。当用户说"push"、"推送"、"提交并推送"时自动触发。
---

# Git 提交并推送

当识别到用户想要推送代码时，执行以下步骤：

1. 运行 `git status` 查看当前更改
2. 如果有未提交的更改：
   - 优先使用 `git add <具体文件>` 逐个添加相关文件，避免 `git add .` 误提交敏感文件
   - 提交信息遵循 CLAUDE.md 中的 Commit Convention：`type(scope): description`
     - Types: feat, fix, refactor, perf, docs, test, release
     - Scope 可选：enocean, proxy, config, sunricher...
     - 提交信息中**不要**包含 Claude 相关信息
     - **不要**添加 Co-Authored-By 署名

3. 运行 `git push` 推送到远程仓库
   1. 如果有新 tag 则还要 `git push --tags`
4. 报告执行结果
