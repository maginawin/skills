---
name: change-log
description: 对比当前代码与最近一次 git tag 版本之间的功能差异，生成更新日志。当用户说"change-log"时自动触发。
---

# 生成更新日志 (Change Log)

当识别到用户想要生成更新日志时，执行以下步骤：

1. 获取 git 最近一次的 tag 作为 `oldVersion`（示例：`1.0.0`）
2. 获取用户传入的版本号作为 `newVersion`（示例：`1.0.1`）
3. 分析 `oldVersion` 到当前代码（包括未提交的代码）之间的差异
4. 总结功能更新，保存到项目中的 `./change_log/CHANGELOG_{oldVersion}_to_{newVersion}.md` 文件中
5. 报告执行结果

## 版本号处理规则

- 如果用户传入了版本号（如 `1.0.1`），则使用该版本号作为 `newVersion`
- 如果用户未传入版本号：
- 存在 `oldVersion` 时：递增最后一位版本号（如 `1.0.0` → `1.0.1`）
- 不存在任何 tag 时：`oldVersion` 为空，`newVersion` = `1.0.0`

## Changelog 输出格式

```markdown
# 更新日志：v{oldVersion} 到 v{newVersion}

## 概述
<简要描述本次更新的主要内容>

## 新功能
<列出新增的功能>

## 改进
<列出改进和优化>

## 修复
<列出修复的问题>

## 配置更改
<列出配置文件的更改>

## 修改的文件
<表格列出修改的文件>

## 提交记录
<列出相关的 git 提交>

注意事项

- 如果不存在任何 git tag，则分析总结整个代码的功能，以简洁为主
- Changelog 内容使用简体中文
- 省略没有内容的章节
```