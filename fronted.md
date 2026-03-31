# CSS 常用样式速查表

## 一、布局与盒模型

| 样式属性               | 生效范围       | 效果描述                                                                             |
| ------------------ | ---------- | -------------------------------------------------------------------------------- |
| `width` / `height` | 块级元素、行内块元素 | 设置元素的宽高（对行内元素无效）                                                                 |
| `margin`           | 所有元素       | 控制元素外部间距，可让块级元素水平居中（`margin: 0 auto`）                                            |
| `padding`          | 所有元素       | 控制元素内部内容与边框的间距                                                                   |
| `border`           | 所有元素       | 边框样式、宽度、颜色，如 `border: 1px solid #ccc`                                            |
| `box-sizing`       | 所有元素       | 改变盒模型计算方式：<br>`content-box`（默认，宽高只算内容）<br>`border-box`（宽高包含 padding 和 border，推荐） |

---

## 二、显示模式

| 样式属性 | 生效范围 | 效果描述 |
|---------|---------|---------|
| `display` | 所有元素 | 改变元素类型：<br>`block`（块级，独占一行）<br>`inline`（行内，宽高无效）<br>`inline-block`（行内块，可设宽高且不换行）<br>`none`（隐藏元素，不占位）<br>`flex` / `grid`（弹性/网格布局） |
| `visibility` | 所有元素 | `hidden` 隐藏但占位，`visible` 显示 |

---

## 三、定位与层叠

| 样式属性 | 生效范围 | 效果描述 |
|---------|---------|---------|
| `position` | 所有元素 | 定位方式：<br>`static`（默认）<br>`relative`（相对自身偏移，原位置保留）<br>`absolute`（相对最近的非 static 祖先定位，脱离文档流）<br>`fixed`（相对视口定位）<br>`sticky`（粘性定位，滚动到阈值时固定） |
| `top` / `right` / `bottom` / `left` | 非 static 元素 | 偏移位置 |
| `z-index` | 定位元素（非 static） | 层叠顺序，数值越大越靠上 |

---

## 四、文本与字体

| 样式属性 | 生效范围 | 效果描述 |
|---------|---------|---------|
| `color` | 所有元素（文本类） | 文字颜色 |
| `font-size` | 所有元素 | 字体大小（px、rem、em 等） |
| `font-weight` | 所有元素 | 字体粗细（`normal`、`bold`、数值 100-900） |
| `font-family` | 所有元素 | 字体族，如 `"Microsoft YaHei", sans-serif` |
| `text-align` | 块级元素内的文本或行内元素 | 水平对齐（`left`、`center`、`right`） |
| `line-height` | 所有元素 | 行高，常用于垂直居中单行文本 |
| `text-decoration` | 所有元素 | 文本装饰（`none`、`underline`、`line-through`） |
| `white-space` | 所有元素 | 处理空白与换行，`nowrap` 禁止换行 |

---

## 五、背景与边框

| 样式属性 | 生效范围 | 效果描述 |
|---------|---------|---------|
| `background-color` | 所有元素 | 背景色 |
| `background-image` | 所有元素 | 背景图片（`url(...)`） |
| `background-size` | 所有元素 | 背景图尺寸（`cover`、`contain`、具体尺寸） |
| `background-position` | 所有元素 | 背景图位置 |
| `border-radius` | 所有元素 | 圆角边框，可做圆形（`50%`） |
| `box-shadow` | 所有元素 | 阴影效果，如 `0 2px 8px rgba(0,0,0,0.1)` |

---

## 六、Flex 布局（弹性布局）

| 样式属性 | 生效范围 | 效果描述 |
|---------|---------|---------|
| `display: flex` | 父容器 | 开启弹性布局 |
| `flex-direction` | flex 容器 | 主轴方向（`row`、`column` 等） |
| `justify-content` | flex 容器 | 主轴对齐（`flex-start`、`center`、`space-between`） |
| `align-items` | flex 容器 | 交叉轴对齐（`center`、`stretch` 等） |
| `flex-wrap` | flex 容器 | 是否换行 |
| `flex` | flex 子项 | 伸缩比例，如 `flex: 1` 占满剩余空间 |

---

## 七、Grid 布局（网格布局）

| 样式属性 | 生效范围 | 效果描述 |
|---------|---------|---------|
| `display: grid` | 父容器 | 开启网格布局 |
| `grid-template-columns` | grid 容器 | 定义列宽，如 `repeat(3, 1fr)` |
| `grid-template-rows` | grid 容器 | 定义行高 |
| `gap` | grid 容器 | 网格间隙（也可用 `row-gap` / `column-gap`） |
| `grid-column` / `grid-row` | grid 子项 | 合并单元格，如 `grid-column: 1 / 3` |

---

## 八、过渡与动画

| 样式属性 | 生效范围 | 效果描述 |
|---------|---------|---------|
| `transition` | 所有元素 | 属性变化过渡效果，如 `all 0.3s ease` |
| `transform` | 所有元素 | 变换：`translate()`（位移）、`rotate()`（旋转）、`scale()`（缩放） |
| `animation` | 所有元素 | 配合 `@keyframes` 实现关键帧动画 |

---

## 九、其他常用

| 样式属性 | 生效范围 | 效果描述 |
|---------|---------|---------|
| `opacity` | 所有元素 | 透明度 0~1，子元素继承 |
| `overflow` | 块级元素 | 溢出处理（`auto`、`hidden`、`scroll`） |
| `cursor` | 所有元素 | 鼠标样式（`pointer`、`default`、`not-allowed`） |
| `filter` | 所有元素 | 滤镜效果，如 `blur(5px)`、`grayscale(1)` |

---

## 生效范围理解要点

- **继承属性**（如 `color`、`font-size`）会传递给后代元素
- **非继承属性**（如 `border`、`background`、`margin`）仅影响当前元素
- 部分属性对**块级**与**行内**元素表现不同（如宽高仅块级/行内块有效）
- 可通过 **CSS 选择器**精确控制生效范围（类、ID、标签、伪类、伪元素等）