# 动画模式与参数

这些模式是按场景选择的参考。保留当前框架、动画上下文和布局约定；名称、数值、方向均可调整。代码片段中的 `gsap` 为项目现有实例，初始化逻辑放入当前工程的动画上下文。

## 目录

- [页面内容连续入场](#页面内容连续入场)
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
| 可反复切换的选项 | 内部标签轻弹或小幅回摆 | 外层热区不动，及时响应连续操作 |
| 多个轻量装饰 | 不同相位的缓慢呼吸 | 控制数量，避免争抢主视觉 |
| 状态由等待转为完成 | 由原来的局部元素接续完成标记 | 稳定业务状态驱动，不能靠动画回调判断成功 |
| 卡片之间有关联内容 | 同一视觉元素自然接续位置或形态 | 不为动画改变导航与信息层级 |

避免只因需要“活泼”就给所有按钮加闪光、所有卡片加旋转或所有文字加弹跳。选择与当前内容有关、可以被感知的一处细节，再确认它不会妨碍阅读与操作。
