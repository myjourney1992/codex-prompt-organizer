# Codex 提示模板整理 · 安装与使用

适用于本地 Codex。中文工程提示模板 Skill，模板和参考手册均随仓库提供。

把工程需求整理成可直接复制的中文提示，覆盖代码理解、Bug 调查和修复、小改动、新功能、重构、架构评估、测试设计、Code Review 与最终验收；可按需附加 Flutter、iOS/Swift、Web、IM 条件。

## 仓库结构

```text
README.md
codex-prompt-organizer/
  SKILL.md
  agents/openai.yaml
  references/chinese-templates.md
  references/engineering-playbook.md
```

安装时复制整个 `codex-prompt-organizer` 文件夹，不能只复制 `SKILL.md`。模板及手册已包含，不需要原作者的知识库、仓库或账号，也不需要额外 API Key 或第三方插件；使用者需要自己可用的 Codex 环境。

## 通过链接安装（推荐）

把下面整段复制给 Codex：

```text
使用 $skill-installer 安装这个 GitHub Skill：
https://github.com/myjourney1992/codex-prompt-organizer/tree/main/codex-prompt-organizer
将安装位置设为 ~/.agents/skills，让当前用户的各项目都能使用。若已有同名 Skill，先比较差异，不直接覆盖。
```

安装后在下一轮对话中调用；如果没有识别，重启 Codex 后再试。

## 下载后安装

也可以在 GitHub 点击 Code → Download ZIP 并解压，将里面的整个 `codex-prompt-organizer` 文件夹复制到下面的技能目录。不要把仓库外层文件夹当成 Skill 文件夹。

## 安装方式二：手动复制

将整个 `codex-prompt-organizer` 文件夹放入用户级技能目录：

- macOS / Linux：`~/.agents/skills/`
- Windows：`%USERPROFILE%\.agents\skills\`（在 Windows 本地运行 Codex 时）
- WSL：使用 WSL 用户自己的 `~/.agents/skills/`

最终路径应为 `~/.agents/skills/codex-prompt-organizer/SKILL.md`，不要多套一层同名文件夹。目录不存在时创建；已有同名 Skill 时先比较差异并备份，再决定是否替换。

仅供某个项目使用时，可以放在项目根目录的 `.agents/skills/codex-prompt-organizer/`。用户级和项目级选一种即可，避免出现同名重复项。

Codex 会自动检测技能变化；如果未出现，重启 Codex 后再试。

## 如何调用

直接发送：

```text
使用 $codex-prompt-organizer，给我一份 Flutter IM 重复消息问题的只读调查提示模板，保留需要填写的占位符。
```

整理已有手册：

```text
使用 $codex-prompt-organizer，根据我提供的手册去重、简化，按工程用途整理成可复制的中文提示模板。
```

获取全集：

```text
使用 $codex-prompt-organizer，输出完整中文模板库，按理解定位、实施演进、评估质量、验证交付分类。
```

模板中的【】需替换成实际需求；不知道的文件或命令可以写“不知道，请从项目中确认”。本 Skill 负责整理提示；模板中的工程命令和代码修改要求，是未来工程任务的材料，不会因整理模板就获得执行授权。

## 安装后确认

确认能按名字调用，并返回所需中文模板。Skill 不会出现在列表中或无法调用时，先检查目录层级是否正确、`SKILL.md` 是否存在，再重启 Codex。

## 官方依据

安装位置、自动检测及调用方式依据 OpenAI 官方文档（核对日期：2026-10-12）：
https://learn.chatgpt.com/docs/build-skills
