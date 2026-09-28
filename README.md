# 从零打造一个企业级转盘抽奖页面 — Art Deco 奢华风格

> 一个纯前端实现的转盘抽奖应用，支持自定义奖品、概率配置、旋转动画和中奖庆祝特效。

## 前言

转盘抽奖是营销活动和用户运营中常见的需求。本文将带你从零开始，使用纯 HTML + CSS + JavaScript 打造一个具有 **Art Deco 几何奢华风格** 的转盘抽奖页面，无需任何框架依赖，开箱即用。

### 你将学到

- 如何用 Canvas 绘制可配置的转盘
- 基于概率的随机算法实现
- CSS 动画与缓动函数的高级运用
- 粒子系统实现彩纸庆祝效果
- Art Deco 风格的 UI 设计思路

## 效果预览

打开 `index.html` 即可在浏览器中运行，无需构建工具或服务器。

核心功能：

- **转盘旋转** — 点击中心按钮触发，5.5 秒缓动动画，精准落点
- **奖品配置** — 右侧面板支持增删改奖品、调整概率、更换颜色
- **概率校验** — 实时检测概率总和是否为 100%，不合法时阻止抽奖
- **中奖弹窗** — 中奖触发彩纸庆祝动画，未中奖显示鼓励文案
- **数据统计** — 实时展示总抽奖次数、中奖次数、中奖率
- **抽奖记录** — 保留最近 20 次抽奖结果及时间

## 技术实现

### 1. 转盘绘制

使用 Canvas 2D API 绘制转盘。每个奖品扇区通过径向渐变（`createRadialGradient`）实现立体感，分隔线使用香槟金色调保持整体风格统一。

```javascript
function drawWheel() {
  ctx.clearRect(0, 0, W, H);
  const total = prizes.reduce((s, p) => s + p.prob, 0);
  let startAngle = -Math.PI / 2; // 从顶部开始

  prizes.forEach((prize, i) => {
    const sliceAngle = (prize.prob / total) * 2 * Math.PI;
    const endAngle = startAngle + sliceAngle;

    // 径向渐变填充
    const grad = ctx.createRadialGradient(CX, CY, 0, CX, CY, R);
    grad.addColorStop(0, lighten(prize.color, 40));
    grad.addColorStop(0.4, lighten(prize.color, 15));
    grad.addColorStop(0.75, prize.color);
    grad.addColorStop(1, darken(prize.color, 25));

    ctx.beginPath();
    ctx.moveTo(CX, CY);
    ctx.arc(CX, CY, R, startAngle, endAngle);
    ctx.closePath();
    ctx.fillStyle = grad;
    ctx.fill();

    startAngle = endAngle;
  });
}
```

### 2. 概率算法

采用**累积概率法**：生成 0-100 的随机数，遍历奖品列表累加概率值，第一个使累积值超过随机数的奖品即为中奖结果。

```javascript
const rand = Math.random() * 100;
let cumulative = 0;
let winnerIndex = 0;
for (let i = 0; i < prizes.length; i++) {
  cumulative += prizes[i].prob;
  if (rand < cumulative) {
    winnerIndex = i;
    break;
  }
}
```

### 3. 旋转动画

旋转动画通过 CSS `transform: rotate()` + `transition` 实现，使用 `cubic-bezier(0.15, 0.75, 0.1, 1)` 缓动曲线模拟真实物理减速效果。

关键计算：根据中奖结果反推目标角度，加上 6-8 圈完整旋转确保视觉冲击力。

```javascript
const rotations = 6 + Math.floor(Math.random() * 3);
currentRotation += rotations * 360 + (360 - target);

canvas.style.transition = 'transform 5.5s cubic-bezier(0.15, 0.75, 0.1, 1)';
canvas.style.transform = `rotate(${currentRotation}deg)`;
```

### 4. 彩纸庆祝效果

使用独立的 Canvas 实现粒子系统。每个粒子具有位置、速度、旋转、重力等属性，支持矩形、圆形、条状三种形状，180 个粒子同时运动营造庆祝氛围。

```javascript
particles.forEach(p => {
  p.x += p.vx;
  p.vy += p.gravity;
  p.y += p.vy;
  p.rotation += p.rotSpeed;
  p.vx *= 0.99; // 空气阻力
  if (frame > 50) p.opacity -= 0.012; // 渐隐
});
```

### 5. Art Deco 设计系统

整体视觉采用 Art Deco 几何奢华风格，核心设计决策：

| 设计元素 | 实现方式 |
|---------|---------|
| 色彩体系 | 深靛蓝底色 + 香槟金点缀，`#08090e` + `#d4b574` |
| 背景层次 | 三层叠加：渐变网格 + 几何线条 + 噪点纹理 |
| 转盘外圈 | `conic-gradient` 模拟拉丝金属质感 |
| LED 灯带 | 20 个圆点交替呼吸动画 |
| 刻度线 | 60 个刻度，每 5 个一组大刻度 |
| 字体搭配 | Cormorant Garamond（标题）+ DM Sans（正文）+ Noto Serif SC（中文） |
| 毛玻璃面板 | `backdrop-filter: blur(24px) saturate(1.3)` |

## 奖品配置说明

默认配置了 4 个奖品：

| 奖品 | 概率 | 颜色 |
|-----|------|------|
| 一等奖 | 5% | 红色 |
| 二等奖 | 15% | 橙色 |
| 三等奖 | 30% | 绿色 |
| 谢谢参与 | 50% | 蓝色 |

你可以在右侧配置面板中：
- 修改奖品名称
- 拖动滑块调整概率（总和须为 100%）
- 点击色块更换颜色
- 添加或删除奖品（最少保留 2 项）

## 项目结构

```
转盘抽奖/
├── index.html    # 单文件应用（HTML + CSS + JS）
└── README.md     # 项目文档
```

## 浏览器兼容

支持所有现代浏览器（Chrome、Firefox、Safari、Edge），无需 polyfill。

## 总结

本项目展示了如何用纯前端技术实现一个功能完整、视觉精美的转盘抽奖应用。通过 Canvas 绘制、概率算法、CSS 动画和粒子系统的组合，在零依赖的前提下达到了企业级的设计品质。

---

**在线演示**：直接克隆仓库，打开 `index.html` 即可体验。

```bash
git clone git@github.com:lfl171/zhuanpan.git
```
