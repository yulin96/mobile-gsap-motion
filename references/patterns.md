# 动画模式与参数

这些模式是按场景选择的参考。保留当前框架、动画上下文和布局约定；名称、数值、方向均可调整。代码片段中的 `gsap` 为项目现有实例，初始化逻辑放入当前工程的动画上下文。

## 目录

- [页面内容连续入场](#页面内容连续入场)
- [差异化间隔与实际完成标签](#差异化间隔与实际完成标签)
- [柔和行程与回弹落定](#柔和行程与回弹落定)
- [卡片换位的稳定绘制层](#卡片换位的稳定绘制层)
- [弹窗开合与中断](#弹窗开合与中断)
- [序列帧收尾与缓存恢复](#序列帧收尾与缓存恢复)
- [主体收拢 + 一次扩散回声](#主体收拢--一次扩散回声)
- [逐行揭示与跨批排队](#逐行揭示与跨批排队)（仅用于滚动揭示）
- [贴边背景的呼吸与漂移](#贴边背景的呼吸与漂移)
- [可选的灵动细节](#可选的灵动细节)

## 页面内容连续入场

普通长页内容优先接入已有入场 timeline；“取消滚动触发，连续展示”也使用这一模式。`contents` 按阅读顺序排列，目标首帧在 CSS 隐藏并保留尺寸；容器可见时机与路由转场按当前项目约定衔接。

```js
const motion = { duration: 0.56, stagger: 0.08, distance: '4rem' }

entrance.addLabel('content')
entrance.fromTo(
  contents,
  { autoAlpha: 0, y: motion.distance },
  {
    autoAlpha: 1,
    y: 0,
    duration: motion.duration,
    stagger: motion.stagger,
    ease: 'power2.out',
  },
  'content',
)
```

此模式不依赖滚动位置。地图等组合场景可将自身分层 timeline 接入同一入场序列；控制长列表的累计错峰时间，结束后确认所有目标可见。不要为了此模式改写默认路由动画。

## 差异化间隔与实际完成标签

间隔控制起点关系，时长控制单个元素运动。标题后可略停，同排卡片紧接、换排稍停；“取中间值”应同时对照两版的行程与累计等待，而不是统一缩放所有循环、停顿和路由时长。

以下 `contents` 不含页脚；`data-after-gap` 表示当前项开始到下一项开始的间隔。`createReveal` 返回该项及其内部层次的有限 timeline，不包含装饰循环。

```js
let nextStart = entrance.labels.content ?? 0
let contentEnd = nextStart

contents.forEach((content) => {
  const reveal = createReveal(content)
  const start = nextStart
  if (content.dataset.startLabel) entrance.addLabel(content.dataset.startLabel, start)
  entrance.add(reveal, start)
  contentEnd = Math.max(contentEnd, start + reveal.duration())
  nextStart = start + Number(content.dataset.afterGap ?? 0.1)
})
entrance.addLabel('content-ready', contentEnd)
```

- 页脚跟随正文实际收尾，可留少量重叠；地图箭头循环等待地图自己的完成标签，不等待无关主视觉或整个页面的总时长。
- 父面板下的缩略图从面板开始可见的标签衔接，不从页面零时刻提前播放。主视觉距离与时长调整后，相关标签应随实际 timeline 计算。
- 密集元素的起点跨度可用 `Math.min(desiredGap, maxSpan / Math.max(1, count - 1))` 约束；组间停顿、最后一项时长另计。按行编排时可用 `row * rowGap + column * columnGap`，行坐标与列坐标须来自实际布局。

## 柔和行程与回弹落定

需要从远处靠近、换位后回弹时，可组合缓入行程与弹性落定。以下仅示意一条可调曲线；不同距离、尺寸和节奏需重新检查，不能把示例秒数当作全站默认值。

```js
const travel = gsap.parseEase('power1.inOut')
const landing = gsap.parseEase('elastic.out(1, 0.85)')
const move = { duration: 1, ease: (progress) => landing(travel(progress)) }

timeline
  .to(positionLayer, { x: target.x, y: target.y, ...move }, 0)
  .to(positionLayer, { autoAlpha: target.visible ? 1 : 0, duration: move.duration * 0.6, ease: 'power1.inOut' }, 0)
```

几何变换共用同一进度，透明度单独渐变。确认位置实际越过落点并逐渐归位；只增加 `duration` 或加 `elastic` 字样不能证明效果自然。远距离越界还可能被父层裁切，需要按实际容器核对。

若要求逐张贴卡，可将“接近”和“落定”拆为两段，并按视觉层级使用不等的开始偏移。若用户只要求换位回弹，让整个卡片的布局层参与运动，内部标题与图案不额外倾斜、挤压或重播入场。

## 卡片换位的稳定绘制层

- 位置层负责卡片路径；背景与热区使用目标卡片的实际边界。标题、canvas 等媒体保留稳定绘制尺寸，由独立 layout 层设置 `x/y/scale`。不要用基准外层的固定大尺寸推断小卡实际可见边界。
- 避免同时动画父卡宽高和子图片百分比宽高：`object-fit: contain` 可能在中途切换限制方向，canvas 尺寸观察也可能逐帧重设分辨率。布局重排时再计算绘制尺寸，切换时只改变布局包装层的等比缩放。
- 图片加载前先预留尺寸。含百分比居中变换的自动宽度图片，若初始宽度为零再交给 GSAP，可能丢失偏移；让外层负责居中、内层负责入场。
- 等高图片标题可根据同组最大宽高比确定共同高度：`Math.min(heightLimit, widthLimit / maxRatio)`。多行标题另设适当块高，不拉伸素材，也不让窄框单独压小长标题。
- 缓存页隐藏或脱离文档时，尺寸观察可能回报零值。先验证节点连接且宽高为正，再计算比例；无效期间保留已有布局。重排时结束旧布局 tween、恢复当前目标布局并释放切换锁，沿用项目的缓存重播约定。

跨多个槽位且要求不中停时，让途经槽位只作路径点，行程结束后统一回弹。槽位路径按业务遍历顺序计算，不能直接把物理槽位索引当作切换距离。GSAP 关键帧内部时间线会在终点截住外层超出 1 的进度，直接把弹性曲线套在整段关键帧外层，可能没有预期的越界回弹。

以下 `path` 包含起点及至少一个目标点；先让行程经过中间点到达轻微超出终点的位置，再统一落定。偏移量和两段占比仅为示意。

```js
const end = path[path.length - 1]
const previous = path[path.length - 2]
const approach = [...path.slice(0, -1), {
  x: end.x + (end.x - previous.x) * 0.04,
  y: end.y + (end.y - previous.y) * 0.04,
}]
const travelDuration = move.duration * 0.65

timeline
  .to(positionLayer, {
    duration: travelDuration,
    ease: 'power1.inOut',
    keyframes: {
      x: approach.map((point) => point.x),
      y: approach.map((point) => point.y),
      easeEach: 'none',
    },
  }, 0)
  .to(positionLayer, {
    x: end.x,
    y: end.y,
    duration: move.duration - travelDuration,
    ease: 'elastic.out(1, 0.85)',
  }, travelDuration)
```

背景尺寸和素材 layout 层按相同路径进度处理，`path` 含起点。透明度需要经过中途显隐时，用单独的无弹性时间线；仅在终点变化时直接淡到终态。业务活动项按最终目标设置，防止途经槽位触发额外序列帧或导航。完成、中断重排后都恢复明确终态，避免变换累积。

## 弹窗开合与中断

先确认组件库负责哪些属性和节点隐藏时机。以下是单弹窗的局部示意：组件库外层与遮罩负责渐隐，共用 `popupTiming.close`；内部 `root` 只负责几何变换，不由 `v-if` 在关闭瞬间移除。`context` 为当前页面动画上下文，`waitForDom` 使用当前框架的 DOM 更新机制。

```js
const popupTiming = { open: 0.7, close: 0.4 }
let revision = 0
let disposed = false
let transition = null

async function enterPopup(getRoot, waitForDom) {
  const current = ++revision
  transition?.kill()
  await waitForDom()
  const root = getRoot()
  if (disposed || current !== revision || !root) return
  context.add(() => {
    transition = gsap.fromTo(root,
      { y: '4rem', scale: 0.84 },
      { y: 0, scale: 1, duration: popupTiming.open, ease: 'back.out(1.4)' },
    )
  })
}

function closePopup(root) {
  revision++
  transition?.kill()
  if (disposed || !root) return
  context.add(() => {
    transition = gsap.to(root, { y: '3rem', scale: 0.9, duration: popupTiming.close, ease: 'power2.in' })
  })
}

function disposePopupMotion() {
  disposed = true
  revision++
  transition?.kill()
}
```

打开、关闭分别由组件事件衔接，清理接入实际卸载生命周期；不要据此改写缓存页的离开行为。多个独立弹窗各自跟踪动画和本次开合标记，不能共享标记而互相取消。Vant 等已有组件过渡可沿用其渐隐与结束事件，确保遮罩、外层与内部退出时长一致。

关闭时取消入场后，从当前变换继续退出，组件库淡出完成后再隐藏。稳定的 `modal` / `show` 状态决定显示关系，不靠旧动画的完成回调关闭新一轮弹窗。验证应覆盖入场中关闭、快速重开和最终遮罩残留；不能用立即隐藏内层或把过渡时长设为零消除冲突。

## 序列帧收尾与缓存恢复

只有需求明确要求“离开活动位后播完当前轮”时使用。优先复用现有播放器的播放、目标帧与暂停接口，不另建计时器，也不等待尾段完成后才切换或导航。

| 当前条件 | 播放决策 |
| --- | --- |
| 成为活动项 | 取消收尾模式，恢复现有循环播放 |
| 刚离开活动位，当前轮尚未结束 | 记录收尾状态，播放到本轮末帧后暂停 |
| 已在末帧 | 显式暂停原循环；仅调用目标帧接口可能立即返回，无法终止原循环 |
| 从未开始的小卡 | 按原展示规则保留，不因“播到结束”而自动启动 |
| 收尾中再次激活 | 新的活动项状态优先，不能被旧目标帧回调稍后暂停 |
| 缓存返回 | 按项目生命周期恢复主循环或未完成的尾段，已停末帧的小卡继续静止 |

如果项目规定离开缓存页时暂停，保存“未完成尾段”与帧号，不能只用当前 `isPlaying` 判断是否还需收尾。验证时检查切出后帧号继续变化、到末帧后稳定、再次激活能循环；临时 DOM 帧号与采样代码在定位后移除。

## 主体收拢 + 一次扩散回声

适合短标题、图标组、抽象主视觉。主体按顺序从较大尺寸收回正常大小，副本在对应主体落定后扩散淡出。保持最终构图稳定。

结构示意，`motion-hidden` 应在项目样式中定义为 `visibility: hidden`，容器预留素材尺寸：

```html
<div class="motion-item">
  <img data-motion-main class="motion-hidden" src="asset.png" alt="内容描述" />
  <img data-motion-echo class="motion-hidden" src="asset.png" alt="" aria-hidden="true" />
</div>
```

外层 `position: relative`；复制层 `position: absolute; inset: 0; pointer-events: none`，尺寸与主体一致。多个元素的主体和副本数组须按相同顺序一一对应。

```js
const motion = {
  start: 0.4,
  duration: 0.8,
  stagger: 0.18,
  fromScale: 2,
  echoScale: 1.8,
  echoDuration: 0.75,
}

timeline.fromTo(
  mainElements,
  { scale: motion.fromScale, autoAlpha: 0 },
  {
    scale: 1,
    autoAlpha: 1,
    duration: motion.duration,
    stagger: motion.stagger,
    ease: 'power3.out',
  },
  motion.start,
)
timeline.fromTo(
  echoElements,
  { scale: 1, autoAlpha: 0.4 },
  {
    scale: motion.echoScale,
    autoAlpha: 0,
    duration: motion.echoDuration,
    stagger: motion.stagger,
    ease: 'power2.out',
    immediateRender: false,
  },
  motion.start + motion.duration,
)
```

不要给整组外层一起做同样缩放，否则主体与副本的空间关系会被叠加。检查放大时的越界、邻近文字遮挡及最终 scale；复制层不循环、不拦截触摸。

## 逐行揭示与跨批排队

仅在用户要求滚动揭示或当前任务需要保留该机制时使用。普通连续入场使用“页面内容连续入场”模式，无需创建 observer 或维护跨批队列。

### 稳定遮罩

每行使用不移动的裁切外层，内部内容从下方进入。隐藏状态只影响内部内容，外层保留完整高度供排版与可见性观察。

```html
<div data-reveal-row style="overflow: hidden">
  <div data-reveal-content style="visibility: hidden">当前行内容</div>
</div>
```

图片行切片时，保持原始行范围与段落空白；SVG 使用正确的 `viewBox` 并显式 `overflow="hidden"`。动画施加在 HTML 内层，SVG 负责静态裁切。

### 统一时序

在一个页面动画上下文中维护以下参数和状态，observer 回调通过该上下文创建动画：

```js
const motion = { duration: 0.9, stagger: 0.1, initialDelay: 0.5 }
let nextIndex = 0
let nextStart = gsap.globalTimeline.time() + motion.initialDelay
let lastEnd = nextStart
```

一次回调中，将可见行按原文索引处理。`endIndex` 为当前可见行最大索引加一；需要连续阅读时，把 `nextIndex` 到 `endIndex` 之间尚未排队的行一起加入，而不只加入回调刚报告的行。这样快速滚动不会越过未处理行。

以下片段在回调的动画上下文内执行；`contents` 是按正文顺序排列的内容层数组：

```js
if (endIndex > nextIndex) {
  const pending = contents.slice(nextIndex, endIndex)
  const now = gsap.globalTimeline.time()
  const start = Math.max(now, nextStart)

  gsap.to(pending, {
    yPercent: 0,
    autoAlpha: 1,
    duration: motion.duration,
    stagger: motion.stagger,
    delay: start - now,
    ease: 'power3.out',
    clearProps: 'transform',
  })

  nextStart = start + pending.length * motion.stagger
  lastEnd = start + (pending.length - 1) * motion.stagger + motion.duration
  nextIndex = endIndex
}
```

初始化时对 `contents` 设置 `{ yPercent: 100, autoAlpha: 0 }`，避免继承不确定起点。不要把示例中的时间换成多个互不关联的常量。

- `nextStart` 控制下一批第一行开始的时刻，相邻行无需等待上一行动画全部完成。
- `lastEnd` 是已排队最后一行的完成时刻。末尾标记等需要等正文结束时，在自身满足可见条件后使用 `Math.max(0, lastEnd - now)` 作为延迟。
- 已排队行取消观察；末尾元素触发时若还有未排队正文，先按相同规则补齐，再计算等待时间。
- observer 回调可能包含多批条目，不能每批独立从零计算 stagger。动画时钟与延迟计算使用同一时钟。
- 页面整体入场仍在移动时，必要时等待外层落定再开启观察；最终方案取决于实际滚动和过渡结构，不用固定延迟掩盖布局问题。

## 贴边背景的呼吸与漂移

按锚点选择变换：右上角素材的缩放原点设在右上角；底部波纹可锚定底边。动画前后均检查覆盖区域。

- 背景有足够延展时，可在裁切容器内做慢速位移和缩放。
- 素材的直边本来贴着容器边缘时，优先缩放或向容器外移动，谨慎旋转，避免暴露图像矩形边缘。
- 背景入场与循环若同时修改同一属性，应串联或使用不同包装层，避免相互覆盖和跳变。
- 前景保持稳定；不要通过同步移动整个页面制造背景动感。

## 可选的灵动细节

| 视觉或交互线索 | 可尝试的做法 | 保持的边界 |
| --- | --- | --- |
| 连续弧线、层叠图形 | 沿轮廓方向追随入场，局部错峰 | 顺着素材形态，最终对齐原构图 |
| 需要选择反馈的选项 | 内部标签轻弹或小幅回摆 | 外层热区不动，用户要求取消时移除对应反馈 |
| 多个轻量装饰 | 不同相位的缓慢呼吸 | 控制数量，避免争抢主视觉 |
| 状态由等待转为完成 | 由原来的局部元素接续完成标记 | 稳定业务状态驱动，不能靠动画回调判断成功 |
| 卡片之间有关联内容 | 同一视觉元素自然接续位置或形态 | 不为动画改变导航与信息层级 |

避免只因需要“活泼”就给所有按钮加闪光、所有卡片加旋转或所有文字加弹跳。选择与当前内容有关、可以被感知的一处细节，再确认它不会妨碍阅读与操作。
