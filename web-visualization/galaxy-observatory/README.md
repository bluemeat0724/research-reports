# Galaxy Observatory

单页 Canvas 粒子天空。浏览器标题是 `NGC 7741 Meridian — Observatory Instrument`。页眉第一行是 Observatory archive / object 07741 / deep field vii，主标题是 NGC 7741 *Meridian*。分类文字把它写成偏离盘面观察的 SAB(rs)cd 旋涡星系：两条对数螺线旋臂从一个紧凑的暖色核球展开，到边缘散成钴蓝色尘埃和稀疏的年轻蓝星亮结，并写明内盘比外臂转得快。

用户指定本页的创建模型是 `hy4-flash`。分类文字下方有一行可见标注，原文是 `创建模型：hy4-flash`。这行不接收指针事件。这是用户提供的创作来源，不是页面要加载的运行时依赖。

`final.png` 是一张静止预览，画面读数约为 Particles 42,000、Distance 39.6 Mly、Diameter 74.2 kly、Tilt 58.4°、Zoom 1.00×。`index.html` 没有引用这张图。

## 文件

| 文件 | 说明 |
|---|---|
| `index.html` | 页面本体。HTML、CSS 和脚本都在这一个文件里。相对用户源文件，只增加了创建模型标注和它的样式。 |
| `final.png` | 静止预览。页面不引用它。 |
| `README.md` | 本说明。 |

## 如何打开

用浏览器打开 `index.html`。本地文件和静态站点都可以，因为页面没有 `fetch`、外链脚本、样式表或图片。需要 JavaScript。

线上路径按仓库相对路径对应：

- https://reports.bluemeat.work/web-visualization/galaxy-observatory/
- https://reports.bluemeat.work/web-visualization/galaxy-observatory/index.html
- https://reports.bluemeat.work/web-visualization/galaxy-observatory/final.png

首页目录不在本页维护。

## 交互

脚本里的行为如下。

- 在星空画布上拖拽：水平位移改变航向，垂直位移改变倾角。倾角限制在 0.08 到 1.50 弧度。按下时光标变为 grabbing。
- 滚轮改变缩放，范围 0.35× 到 6×。两指捏合也按这个范围改变缩放。
- 右侧四个按钮：放大（当前目标乘 1.35）、缩小（除以 1.35）、暂停或继续自转、重置。重置把航向设为 0.35、倾角设为 1.02、缩放设为 1，并把模拟时间清零；如果当时处于暂停，重置会恢复自转。
- 空格键切换暂停。焦点在按钮上时不处理空格。
- 左下角读数：粒子数、距离、直径、倾角、缩放。距离终值是脚本常量 39.6 Mly，直径终值是 74.2 kly。粒子数、距离和直径在约 3.2 秒内从 0 缓动到终值。
- 右上角罗盘指针随航向转动。
- 未暂停时，每个粒子的角速度是 `0.35 / (0.25 + r)`，内圈快于外圈。
- 系统开启 `prefers-reduced-motion: reduce` 时，粒子数从 42000 降到 14000，页眉入场动画缩短到 0.6 秒。页面不会因此一开始就暂停。

## 外部依赖

没有。字体只用页面里写的系统衬线栈和等宽栈。渲染用一块全屏 `<canvas>` 的 2D 上下文：背景星场在改变窗口大小时画一次，粒子先画进离屏缓冲再贴到画布上，尘埃和核球光晕用径向渐变精灵。没有 WebGL，也没有引入其他库。

## 画面是怎么生成的

这些数字都来自 `index.html` 里的脚本，不是另行测量的天文数据。

- 默认生成 42000 个粒子，其中约 18% 是核球，其余落在两条对数螺线旋臂上。
- 颜色按半径分成暖色核、蓝白旋臂和外缘尘埃三段，另有约 6% 的旋臂粒子被画成更大的亮结。
- 沿旋臂另有 90 团尘埃，中心有暖色光晕。
- 投影使用焦距常量 4，并按深度做透视缩放。
