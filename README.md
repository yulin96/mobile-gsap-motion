# mobile-gsap-motion

为手机端网页设计、实现和排查 GSAP 动画的 Codex Skill。覆盖有层次的入场、逐行错峰、滚动揭示、背景呼吸，以及适合局部使用的灵动细节。

适用于不同前端框架与设计体系，不绑定品牌、活动、页面名称、素材路径或固定数值。沿用项目已有 hook、缓存生命周期与验证权限。

## 全局安装

需要已安装 Codex 和 Git。以下命令将仓库直接安装到个人全局技能目录；已有同名目录时，先核对内容，不直接覆盖。

Windows PowerShell：

```powershell
$skillBase = if ($env:CODEX_HOME) { $env:CODEX_HOME } else { Join-Path $HOME '.codex' }
$skillPath = Join-Path $skillBase 'skills/mobile-gsap-motion'
git clone https://github.com/yulin96/mobile-gsap-motion.git $skillPath
```

macOS / Linux：

```sh
git clone https://github.com/yulin96/mobile-gsap-motion.git "${CODEX_HOME:-$HOME/.codex}/skills/mobile-gsap-motion"
```

安装后在下一轮对话使用 `$mobile-gsap-motion`。如果当前客户端未刷新技能列表，重新打开对话或重启客户端后再使用。

也可以在 Codex 中调用内置安装技能：

```text
使用 $skill-installer 从 yulin96/mobile-gsap-motion 安装技能，仓库内路径为 .，技能名称为 mobile-gsap-motion。
```

此仓库提供给 AI 读取的指南和示例。业务项目需要使用 GSAP 时，仍按项目现有依赖和插件注册方式接入。

## 使用

直接调用：

```text
使用 $mobile-gsap-motion 给当前手机端页面添加 GSAP 动画。主视觉幅度明显一些，局部可以有灵动、有巧思的细节，保持最终构图与交互稳定。
```

针对性调用：

```text
使用 $mobile-gsap-motion 检查逐行入场：避免首帧闪现、后面的内容插队，以及修改 stagger 后排队时间没有同步。
```

在项目 `AGENTS.md` 中加入调用规则：

```markdown
- 新增、调整或排查手机端 GSAP 动画时，使用 `mobile-gsap-motion` Skill，按需参考其动画模式；当前用户要求、项目规范及验证权限优先。
```

## 内容

- [SKILL.md](SKILL.md)：选效、编排、首帧与终态、缓存和验证边界。
- [动画模式与参数](references/patterns.md)：收拢与一次扩散、逐行排队、贴边背景及灵动细节。
- `agents/openai.yaml`：Codex 展示与调用元数据。

## 更新

通过 Git 克隆安装、且本地没有自行修改时，进入技能目录执行：

```sh
git pull --ff-only
```

通过安装器下载的目录可能没有 Git 元数据；更新时先备份本地修改，再通过相同方式重新安装。此 Skill 不要求自动运行 build、浏览器或 E2E；验证范围由当前项目和用户授权决定。
