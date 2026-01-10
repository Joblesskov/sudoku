# Sudoku 逆向分析

## 项目概况
- 单页数独游戏，Svelte + TailwindCSS 实现，UI 组件化，逻辑通过 Svelte store 管理。
- 支持难度选择、键盘/鼠标输入、候选标注、提示、胜利检测；分享关卡通过自定义 sencode 编码。
- 入口 `src/main.js` 实例化 `App.svelte`；挂载时显示欢迎模态并暂停计时，开始游戏后恢复。

## 代码结构（关键目录）
- `src/App.svelte`：布局 Header/Board/Controls/Modal，监听胜利状态显示 GameOver。
- `src/components`：
  - `Board` 渲染 9x9 棋盘与单元格高亮、候选显示。
  - `Controls` 虚拟键盘与顶部操作栏（暂停/提示/笔记）。
  - `Header` 下拉选择难度、打开分享与设置。
  - `Modal` 各类模态（欢迎、分享、二维码、设置、确认/提示、胜利）。
- `src/node_modules/@sudoku`：核心逻辑与状态。
  - `constants.js` 常量（尺寸、颜色、模态、难度、sencode 分隔符等）。
  - `game.js` 游戏生命周期：开始新局/自定义、暂停、恢复。
  - `sudoku.js` 生成与求解：`fake-sudoku-puzzle-generator` 生成题面，`@mattflow/sudoku-solver` 求解。
  - `sencode/` 编码/解码/校验关卡字符串，Base62 实现。
  - `stores/` Svelte stores：
    - `grid` 原始题面；`userGrid` 玩家填充；`invalidCells` 查重用于冲突高亮与胜利判定。
    - `cursor` 光标位置；`keyboardDisabled` 控制输入可用性。
    - `difficulty` 持久化难度；`settings` 本地存储设置；`hints` 剩余提示与已用次数；`notes` 笔记模式；`timer` 计时；`gamePaused`/`gameWon` 状态。

## 运行与构建
- 开发：`npm install` 后运行 `npm run dev`（并行 watch + 本地静态服务）。
- 生产构建：`npm run build`（Rollup + Svelte preprocess + Tailwind/PostCSS），构建后 `scripts/postbuild.js` 内联关键 CSS、指纹化 `bundle.js`。
- 模板/静态：`src/template.html` 产出 `dist/index.html`；`static/` 存放 manifest、favicon、logo、未启用的 service worker（注册已注释）。

## 主要功能流程
- 开局：欢迎模态选择难度或输入 sencode -> `game.startNew`/`startCustom` 生成/解码题面 -> 重置光标/计时/提示。
- 互动：
  - 光标在单元格上时，虚拟键盘/物理键盘写入数字；Notes 模式写入候选，支持清除。
  - 高亮：同 row/col/box、同数字、冲突数字，均由设置开关控制。
  - 提示：`userGrid.applyHint` 对当前棋盘求解并写入该格，同时消耗提示计数。
- 胜利：`gameWon` 派生 store 确认无空格且无冲突 -> 暂停计时并弹出 GameOver 模态（展示时间/难度/已用提示，可分享或重新开局）。
- 分享：`grid.getSencode` 编码当前题面；分享模态提供复制、社交链接、邮件与第三方 API 生成二维码。

## 构建/部署流水线
- Rollup（`rollup.config.js`）：复制模板与静态资源、Svelte 编译、CSS 抽离/关键 CSS 输出、开发时 livereload，生产时 terser 压缩。
- GitHub Actions（`.github/workflows/deploy.yml`）：推送 master -> npm install -> build -> 写 CNAME -> 用 `JamesIves/github-pages-deploy-action` 发布 `dist` 至 GitHub Pages。

## 已知缺口与风险
- 功能未完：撤销/重做、关卡创作模式（UI 按钮占位但无实现）。
- 提示实现直接对 `userGrid` 求解，若已有矛盾数字可能求解失败或写入不一致；缺少错误处理。
- 候选存储用 `delete` 删除数组元素，留下空洞；虽不影响 `includes` 渲染，但后续序列化/排序可能异常。
- 二维码依赖外部 API（需联网），无离线兜底。
- Service worker 文件存在但注册被注释，当前无离线缓存。

## 快速定位
- UI 入口：`src/App.svelte`
- 棋盘交互：`src/components/Board/*`
- 键盘/动作栏：`src/components/Controls/*`
- 模态：`src/components/Modal/*`
- 游戏逻辑与状态：`src/node_modules/@sudoku/*`
- 构建脚本：`rollup.config.js`, `scripts/dev.js`, `scripts/postbuild.js`
- 部署：`.github/workflows/deploy.yml`
