# NGC 7741 Meridian

单页 Canvas 粒子天空。浏览器标题是 `NGC 7741 Meridian — Meridian Observatory`。页眉写着 Meridian Observatory · Deep Field Survey · Plate 047，主标题是 NGC 7741 *Meridian*。分类文字把它写成棒旋星系 SB(s)b：两条由年轻蓝星组成的主旋臂从较老的琥珀色核球展开，并写明这块底片上的光在 2930 万年前离开、现在才到达。画布的无障碍标签描述的是同一幅图：暖色核球、两条蓝白旋臂、边缘淡红尘埃。

页面不加载本目录里的预览图，也不请求任何网络资源。

用户指定本页的创建模型是 `qwen4-max`。分类文字下方有一行可见标注，原文是 `创建模型：qwen4-max`。这行不接收指针事件，脚本未改。这是用户提供的创作来源，不是页面要加载的运行时依赖。

## 文件

| 文件 | 说明 |
|---|---|
| `index.html` | 页面本体。HTML、CSS 和脚本都在这一个文件里。 |
| `preview.png` | 静止预览。界面处于默认尺度附近：Particles 42,000，Distance 29.3 Mly，Diameter 61.8 kly，Tilt 59.0°，Zoom 1.00×。`index.html` 没有引用它。 |
| `preview-orbit.png` | 另一张静止预览。同一界面拉近之后：Tilt 50.0°，Zoom 5.37×，粒子数、距离和直径仍是上面三个终值。`index.html` 没有引用它。 |
| `README.md` | 本说明。 |

## 如何打开

用浏览器打开 `index.html`。本地文件和静态站点都可以，因为页面没有 `fetch`、外链脚本、样式表或图片。需要 JavaScript。脚本被关闭时，页底显示：This instrument requires JavaScript to turn the sky.

线上路径按仓库相对路径对应：

- https://reports.bluemeat.work/web-visualization/ngc-7741-meridian/
- https://reports.bluemeat.work/web-visualization/ngc-7741-meridian/index.html
- https://reports.bluemeat.work/web-visualization/ngc-7741-meridian/preview.png
- https://reports.bluemeat.work/web-visualization/ngc-7741-meridian/preview-orbit.png

首页目录不在本页维护。

## 交互

界面提示原文是 `Drag — orbit · Scroll — zoom · Space — hold`。脚本里的行为如下。

- 在星空画布上拖拽：水平位移改变航向，垂直位移改变倾角。倾角限制在 0.12 到 1.48 弧度。拖拽时光标变为 grabbing。
- 滚轮改变缩放，范围 0.35× 到 6×。两个指针同时按下时，捏合距离也按同样范围改变缩放。
- 右侧四个按钮：放大（当前目标乘 1.4）、缩小（除以 1.4）、暂停或继续自转、重置。重置把航向设为 -0.65、倾角设为 1.03、缩放设为 1，并把模拟时间清零。
- 空格键切换暂停。焦点在按钮上时不处理空格，避免和按钮激活叠在一起。
- 左下角读数：粒子数、距离、直径、倾角、缩放。距离终值是脚本常量 29.3 Mly，直径终值是 61.8 kly。粒子数、距离和直径在约 3.2 秒内从 0 缓动到终值。
- 右上角罗盘指针随航向转动。
- 未暂停时，粒子按半径分箱做差速自转，内圈角速度大于外圈。
- 系统开启 `prefers-reduced-motion: reduce` 时，粒子数从 42000 降到 14000，页面一开始就暂停，页眉入场动画也会关掉。

## 外部依赖

没有。字体只用页面里写的系统衬线栈和等宽栈，没有网络字体。渲染用两块 `<canvas>` 的 2D 上下文：背景星场和远处模糊星系在改变窗口大小时画一次，星系每帧重绘。脚本注释里把 WebGL 写成以后若需要更多余量时的升级方向；当前代码没有使用 WebGL，也没有引入 Three.js 或其他库。

## 画面是怎么生成的

这些数字都来自 `index.html` 里的脚本，不是另行测量的天文数据。

- 默认生成 42000 个粒子，其中约 18% 是高斯分布的核球，其余落在两条对数螺线旋臂上。
- 颜色按半径分成暖色核、蓝白旋臂和外缘暗红三段，另有约 6% 的旋臂粒子被画成更大的亮结。
- 沿旋臂另有 90 团星云尘埃，中心有随倾角压扁的暖色光晕。
- 若平均帧间隔大于 40 毫秒，绘制步长会从 1 增加到最多 3；低于 20 毫秒再降回去。
