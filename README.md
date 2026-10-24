# mobile-gsap-motion

为手机端网页设计、调优和排查 GSAP 动画的 Codex Skill。覆盖有层次的入场、差异化错峰、卡片回弹、弹窗开合、序列帧收尾，以及背景呼吸和适合局部使用的灵动细节。

适用于不同前端框架与设计体系，不绑定品牌、活动、页面名称、素材路径或固定数值。沿用项目已有 hook、缓存生命周期与验证权限。

路由动画优先使用项目当前默认效果；没有时保留现状。只有明确要求特殊切换、展开/缩回等效果时，才针对对应页面编写适配。新增普通长页动画优先连续入场，滚动联动按明确需求或现有项目约定处理。

## 全局安装

以下安装方式任选一种；已有同名技能时先核对内容，避免重复安装到多个技能目录。

### 使用 npx skills

需要已安装 Node.js/npm。全局安装到 Codex：

```sh
npx skills add yulin96/mobile-gsap-motion --global --agent codex --skill mobile-gsap-motion
```

`--global` 表示跨项目使用，`--agent codex` 指定 Codex，`--skill` 指定本技能。省略 `--agent codex` 可交互选择其他支持的 Agent；这类安装由 Skills CLI 管理。参数说明见 [Skills CLI 文档](https://github.com/vercel-labs/skills/blob/main/docs/commands.md)。

### 使用 Git

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

### 使用 Codex 内置安装技能

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

指定特殊路由效果：

```text
使用 $mobile-gsap-motion 为当前详情页单独实现返回时缩回菜单卡片的效果，内容依次退场，并衔接菜单入场。
```

在项目 `AGENTS.md` 中加入调用规则：

```markdown
- 新增、调整或排查手机端 GSAP 动画时，使用 `mobile-gsap-motion` Skill，按需参考其动画模式；当前用户要求、项目规范及验证权限优先。
```

## 内容

- [SKILL.md](SKILL.md)：路由边界、节奏调优、按场景选用的布局换位与媒体播放、弹窗退出、首帧与终态、缓存和验证。
- [动画模式与参数](references/patterns.md)：连续入场与完成标签、行程与回弹、稳定绘制层、弹窗中断、序列帧收尾、扩散回声及滚动逐行排队。
- [场景验收](references/scenarios.md)：用于维护指令的典型请求与预期行为。
- `agents/openai.yaml`：Codex 展示与调用元数据。

## 更新

通过 `npx skills` 安装时，可使用：

```sh
npx skills update mobile-gsap-motion --global
```

通过 Git 克隆安装、且本地没有自行修改时，进入技能目录执行：

```sh
git pull --ff-only
```

通过安装器下载的目录可能没有 Git 元数据；更新时先备份本地修改，再通过相同方式重新安装。此 Skill 不要求自动运行 build、浏览器或 E2E；验证范围由当前项目和用户授权决定。
