---
name: obsidian-cli
description: 使用 Obsidian CLI 与 Obsidian 库交互，支持读取、创建、搜索、管理笔记、任务、属性等。同时支持插件和主题开发，提供重载插件、运行 JavaScript、捕获错误、截图和检查 DOM 等命令。当用户要求与 Obsidian 库交互、管理笔记、搜索库内容、从命令行执行库操作，或开发和调试 Obsidian 插件及主题时使用。
---

# Obsidian CLI

使用 `obsidian` CLI 与正在运行的 Obsidian 实例交互。需要 Obsidian 保持开启状态。

## 命令速查

运行 `obsidian help` 查看所有可用命令，该列表始终是最新的。完整文档：https://help.obsidian.md/cli

## 语法

**参数** 通过 `=` 传值。含空格的值需加引号：

```bash
obsidian create name="My Note" content="Hello world"
```

**标记** 为布尔开关，无需赋值：

```bash
obsidian create name="My Note" silent overwrite
```

多行内容中使用 `\n` 表示换行、`\t` 表示制表符。

## 文件定位

多数命令通过 `file` 或 `path` 指定目标文件。两者皆无时，默认使用当前活动文件。

- `file=<name>` —— 按双链方式解析（仅名称，无需路径或扩展名）
- `path=<path>` —— 从库根目录开始的精确路径，如 `folder/note.md`

## 库定位

命令默认 targeting 最近聚焦过的库。如需指定特定库，将 `vault=<name>` 作为首个参数：

```bash
obsidian vault="My Vault" search query="test"
```

## 常用模式

```bash
obsidian read file="My Note"
obsidian create name="New Note" content="# Hello" template="Template" silent
obsidian append file="My Note" content="New line"
obsidian search query="search term" limit=10
obsidian daily:read
obsidian daily:append content="- [ ] New task"
obsidian property:set name="status" value="done" file="My Note"
obsidian tasks daily todo
obsidian tags sort=count counts
obsidian backlinks file="My Note"
```

> **双链安全**：使用 `create` 或 `append` 写入包含 `[[双链]]` 的内容时，务必先用 `obsidian search query="笔记名"` 或 `obsidian read file="笔记名"` 验证目标笔记存在。严禁编造指向不存在笔记的双链。

任意命令后加 `--copy` 可将输出复制到剪贴板。加 `silent` 可防止文件被自动打开。列表命令加 `total` 可获取总数。

## 插件开发

### 开发/测试循环

修改插件或主题代码后，按以下流程操作：

1. **重载**插件使改动生效：
   ```bash
   obsidian plugin:reload id=my-plugin
   ```
2. **检查错误** —— 若出现报错，修复后从第 1 步重复：
   ```bash
   obsidian dev:errors
   ```
3. **视觉验证** 通过截图或 DOM 检查：
   ```bash
   obsidian dev:screenshot path=screenshot.png
   obsidian dev:dom selector=".workspace-leaf" text
   ```
4. **查看控制台输出** 有无警告或异常日志：
   ```bash
   obsidian dev:console level=error
   ```

### 其他开发者命令

在应用上下文中运行 JavaScript：

```bash
obsidian eval code="app.vault.getFiles().length"
```

检查 CSS 值：

```bash
obsidian dev:css selector=".workspace-leaf" prop=background-color
```

切换移动端模拟：

```bash
obsidian dev:mobile on
```

运行 `obsidian help` 查看更多开发者命令，包括 CDP 和调试器控制。
