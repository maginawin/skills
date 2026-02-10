---
name: change-log
description: 对比当前代码与最近一次 git tag 之间的差异，生成更新日志并自动完成版本发布流程（commit、tag、push）。当用户说"change-log"、"更新日志"、"发版"、"发布版本"时触发。
---

# 生成更新日志 (Change Log)

## 前置检查

执行任何操作前，先完成以下检查：

1. **确认当前目录是 git 仓库**：运行 `git rev-parse --is-inside-work-tree`，若失败则提示用户不在 git 项目中并终止
2. **检查是否有可分析的变更**：运行 `git status --porcelain` 和 `git log`，若仓库为空（无任何提交）则提示用户先提交代码
3. **获取远程仓库信息**：运行 `git remote -v`，记录是否存在远程仓库（影响后续 push 步骤）

## 第一步：确定版本号

1. 获取最近一次 tag 作为 `oldVersion`：
   ```bash
   git describe --tags --abbrev=0 2>/dev/null
   ```
2. 确定 `newVersion`：
   - 用户传入了版本号 → 直接使用
   - 用户未传入版本号：
     - 存在 `oldVersion` → 递增最后一位（如 `1.0.0` → `1.0.1`）
     - 不存在任何 tag → `newVersion` = `1.0.0`，`oldVersion` 为空
3. **校验 tag 不冲突**：运行 `git tag -l "newVersion"`，若 tag 已存在则提示用户更换版本号

## 第二步：分析差异

根据是否存在 `oldVersion`，使用不同策略：

**有 `oldVersion` 的情况：**
```bash
# 查看文件级别的变更统计
git diff --stat {oldVersion}..HEAD
# 查看未提交的变更
git diff --stat
git diff --stat --cached
# 查看详细的代码差异（重点关注功能性变更）
git diff {oldVersion}..HEAD -- '*.py' '*.js' '*.ts' '*.java' '*.go' '*.rs' '*.vue' '*.jsx' '*.tsx' (根据项目实际语言调整)
# 查看提交记录
git log {oldVersion}..HEAD --oneline --no-merges
```

**无 `oldVersion` 的情况（首次发版）：**
```bash
# 查看所有提交记录
git log --oneline --no-merges
# 查看项目结构
find . -type f -not -path './.git/*' | head -100
```
分析项目整体功能，以简洁概述为主。

**分析重点：**
- 关注功能性代码变更，忽略格式化/空白变更
- 从提交消息和代码差异中提取：新增功能、改进优化、Bug 修复、配置变更
- 如果 diff 内容过大，优先分析提交记录和文件变更统计，再选择性查看关键文件的详细 diff

## 第三步：生成 Changelog

将分析结果保存到 `./change_log/CHANGELOG_{oldVersion}_to_{newVersion}.md`（首次发版时文件名为 `CHANGELOG_init_to_{newVersion}.md`）。

### 输出格式

```markdown
# 更新日志：v{oldVersion} → v{newVersion}

## 概述
（用 2-3 句话概括本次更新的核心内容）

## 新功能
- 功能描述（涉及的模块/文件）

## 改进
- 改进描述

## 修复
- 修复描述

## 配置更改
- 变更描述

## 修改的文件

| 文件 | 变更类型 | 说明 |
|------|----------|------|
| path/to/file | 新增/修改/删除 | 简要说明 |

## 提交记录
- `hash` 提交信息
```

**格式要求：**
- 使用简体中文
- 省略没有内容的章节（如无修复则不写"修复"章节）
- 每个条目简洁明了，一行说清楚

## 第四步：Git 操作

⚠️ **执行 git 操作前，先向用户展示生成的 Changelog 摘要和即将执行的命令列表，等待用户确认。**

确认后依次执行：

```bash
# 1. 暂存所有变更（包括新增、修改、删除的文件，不过滤任何目录）
git add -A

# 2. 提交（不包含 Claude 相关信息，不添加 Co-Authored-By）
git commit -m "release: v{newVersion}

{一句话概括主要更新}"

# 3. 打 tag
git tag v{newVersion}

# 4. 推送（仅在存在远程仓库时执行）
git push
git push --tags
```

**注意事项：**
- commit message 使用 `release: v{newVersion}` 格式，正文为一句话概括
- 不要在 commit 中包含 Claude、AI、Co-Authored-By 等信息
- 如果没有远程仓库，跳过 push 步骤并告知用户
- 如果 push 失败，告知用户错误信息，不要重试

## 第五步：报告结果

输出执行摘要：
- ✅ Changelog 已生成：`{文件路径}`
- ✅ 已提交并打 tag：`v{newVersion}`
- ✅ 已推送到远程 / ⚠️ 无远程仓库，需手动推送