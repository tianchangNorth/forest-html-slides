# 编写与交互约定

## 文件组织

复制 `assets/starter.html` 为本次输出。它包含原稿视觉系统、通用播放器和六个可替换的版式样本。CSS、内容与 JavaScript 同处一个文件，直接使用 `file://` 打开也应工作。

更完整的布局与交互可从 `assets/reference.html` 按需借用。不要直接复制其中整段主题脚本：它绑定了 Jev 专属样本、公式和页面 ID。保留参考文件原样，在新输出文件中工作。

骨架分为四块：`<style>` 视觉规则；`deck`、`sources`、`slides` 内容；`slideBehaviors` 按页面 ID 注册交互；通用播放器处理导航、弹层、全屏、触摸与打印。

## 内容数据

```js
const deck = {title: '演示主题 · 副标题', brand: 'TOPIC / EXPLAINED', lang: 'zh-CN'};
const sources = {guide: ['资料名称', 'https://example.com/official-guide']};
const slides = [{
  id: 'mechanism', // 全稿唯一，供交互注册使用
  title: '机制的具体名称',
  chapter: '原理', dark: false, cover: false,
  lead: '支持本页主问题的一句引导。',
  body: `<div class="pipeline">…</div>`,
  notes: '讲者可以使用的补充解释，而不是制作过程汇报。',
  refs: ['guide'], foot: '数据口径或必要边界'
}];
```

`body` 仅写经过作者控制的 HTML。来源文本、用户原文等不可信输入要转义，不直接拼成可执行 HTML。源码中的反引号、`${…}` 和 `</script>` 要正确处理，避免模板字符串或脚本块意外终止。普通元数据通过 `escapeHTML()` 输出，备注通过 `textContent` 输出。

封面 `cover: true` 时不生成普通页标题；在其 `body` 中放唯一主 `h1`。页数、目录、进度条和讲解从 `slides` 自动计算。不要把原稿 17 页写死。外部来源键必须存在于 `sources`。

## 页内行为

在 `slideBehaviors` 用语义化页面 ID 注册交互，仅查询该页元素：

```js
const slideBehaviors = {
  experiment: {
    init(section) {
      const slider = section.querySelector('[data-value]');
      const output = section.querySelector('[data-result]');
      const render = () => { output.textContent = slider.value; };
      slider.addEventListener('input', render);
      render();
    },
    leave(section) { /* 清理该页定时器或动画 */ },
    print(section) { /* 设置可读静态状态，可保留当前参数 */ },
    afterPrint(section) { /* 仅在 print 改了状态时恢复 */ }
  }
};
```

钩子可按需省略；也支持 `enter(section)`。删除或换序幻灯片时，未命中的行为不执行。自动动画在 `enter` 启动、`leave` 停止；`init` 只绑定一次事件。

- `input` 配 `label`，输出配 `output`；关键结果使用适量 `aria-live`。
- 选择按钮用 `aria-pressed` 同步状态，不用普通文字冒充交互控件。
- 公式、滑块、图表与结论来自同一状态，验证阈值相等情况和边界值。
- 动画启动前就准备可读默认结果。机制动画注明模拟，不用等待时长冒充性能实测。
- 处理重播、重置、离开页面、减少动态效果与打印的状态。

## 播放器约定

默认保留方向键、空格、PageUp/PageDown、Home/End，以及 `O` 目录、`N` 备注、`F` 全屏、`?` 帮助。Esc 关闭弹层。页码、章节、进度条和 `#slide-N` 链接从数据生成。

只对当前页解除 `inert`，同步 `aria-hidden`。输入框或滑块获得焦点时不拦截导航键；按钮与链接保留原生激活行为。手机横扫不抢占输入、按钮、链接或纵向滚动。全屏不可用时给出浏览器提示。

打印时显示所有页、解除隐藏状态并调用打印钩子；打印后恢复当前页与焦点语义。

## 复用高级布局

| 原稿片段 | 搜索词 | 迁移注意 |
| --- | --- | --- |
| 选择标签改变内容 | `primitiveData` / `renderPrimitive` | 新主题数据与页内查询 |
| 顺序与并行机制 | `startRace` / `cancelRace` | 清除定时器，不沿用速度暗示 |
| 参数改变分布 | `renderConfidence` | 核实公式，不沿用 Jev 公式 |
| 阈值改变分流 | `renderTicket` | 标明样本来源与阈值含义 |
| 逐步工作流 | `renderFlow` | 步数与文本一致，打印全部可读 |
| 场景切换 | `renderUse` | 相同信息维度，主题相关案例 |
| 费用估算 | `renderCost` | 单位、假设、日期与排除项 |

只移植叙述确实需要的逻辑，不把这些交互当成每套演示的必选项。

## 浏览器检查

用可用浏览器工具或项目已有 Playwright。不要把机器专属运行时路径写进技能或最终 HTML。需要依赖时先检查已有运行时。

桌面推荐 1440 × 940（包含导航），手机推荐 390 × 844。检查每页排版与来源页，对交互实际点击或输入，不仅检查语法。

可检查当前页叶子元素与页脚边界，同时看截图。截图前等待动画稳定或用减少动态效果模式。另在正常动画模式验证演示完成、重播且离开后停止。

打印使用 16 in × 9 in 页面。打印裁切需修版式。默认交付 HTML；用户要求 PDF 时再交付检查过的 PDF。
