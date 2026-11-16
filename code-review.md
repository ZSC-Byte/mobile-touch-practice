# 前端代码评审报告

- 项目：移动端触摸事件课堂实践
- 评审人：邹思成（AI 辅助初版，逐行复核）
- 日期：2026-11-16
- 范围：`index.html`、`mobile-probe.html`、`mobile-gesture.html`、`mobile-zoom.html`、`mobile-sensor.html`、`assets/css/wy-common.css`

---

## 一、规范符合性检查

| 检查项 | 结果 | 说明 |
|---|---|---|
| 每个页面含 viewport meta | 通过 | 五个 HTML 全部具备 |
| `-webkit-tap-highlight-color` 全局 | 通过 | `wy-common.css` 的 `*` 选择器 |
| `body` 上无 `touch-action` | 通过 | 全文检索 `touch-action` 仅出现在三个手势容器与按钮上 |
| 四个触摸事件齐全 | 通过 | 探针页 / 卡片页 / 查看器均绑定 touchcancel |
| `touchend` 使用 `changedTouches[0]` | 通过 | 检索确认 `touchend` 内无 `touches[0]` |
| 坐标系统一 client | 通过 | 手势链路内无 `pageX/pageY` 混用 |
| `wy_` 前缀 | 通过 | 类名 / id / 变量 / 函数统一前缀 |
| 文件头姓名学号 | 通过 | 五个 HTML 首行 + CSS 注释 |
| 无第三方依赖 | 通过 | 零外部请求，离线可跑 |
| 中文注释说明「为什么」 | 通过 | 关键判断处均写明原因 |

## 二、发现的问题与处理

| 级别 | 位置 | 问题 | 处理 |
|---|---|---|---|
| 高 | `index.html` `.wy_task_name` / `.wy_task_desc` | 两者是 flex 容器内的 `span`，未块级化，任务名与描述会挤在同一行，`text-overflow` 也不生效 | 已修：补 `display: block` |
| 高 | `mobile-zoom.html` `wy_zoomTo()` | 先改 `wy_scale` 再算锚点，导致换算比例恒为 1，双击放大后画面偏到左上角 | 已修：先存旧倍数 `wy_old`，锚点用旧倍数换算、位移用新倍数反推 |
| 中 | `mobile-zoom.html` 双指分支 | 两指几乎重合时 `wy_pinchDist0` 接近 0，存在除零得到 `Infinity` 的风险 | 已修：加 `wy_baseDist` 下限保护 |
| 中 | `mobile-zoom.html` `touchend` | 1x 时不进入拖动分支，无法通过 `wy_moved` 判断「拖了一下」，可能被误判为双击的一半 | 已修：抬手时用 `changedTouches[0]` 与起始点比对补算位移 |
| 低 | `mobile-zoom.html` `.wy_photo` | 图片未加载时元素无尺寸，原图 1832px 会先撑出容器再被裁掉，出现闪动 | 已修：CSS 兜底 `width/height:100%` + `object-fit: contain` |
| 观察 | `mobile-sensor.html` `.catch()` | 在 ES3 引擎（JScript）下保留字作属性名会报语法错，现代浏览器完全正常 | 不改，属于引擎限制，不影响交付 |

## 三、逻辑复核要点

1. **手势冲突**：卡片页用 `pan-y` 把纵向让给浏览器、横向留给自己；查看器用 `none` 全接管。两处都只加在容器，页面其他区域照常滚动。
2. **状态收尾**：三处手势都有 `touchcancel` 兜底；长按类逻辑虽未使用，定时器清理模式已在探针页演示。
3. **冷却与阈值**：卡片页 300ms 冷却，摇一摇 1.5s 冷却 + 回落再上膛，两者都不会一次动作触发两次。
4. **边界处理**：卡片在首尾有阻尼与文字提示；查看器拖动时软阻尼、松手硬夹紧、1x 强制居中。
5. **动画期间的状态**：查看器双击后 260ms 内拒绝新手势，避免过渡中抢夺 transform。
6. **无传感器分支**：三层判定（无事件 / 2s 无数据 / 拒绝授权）最后都汇到同一个降级按钮，不会白屏。

## 四、结论

规范相关项全部通过，6 处问题已修复 5 处、保留 1 处引擎限制说明。
本地静态服务器七个资源均返回 200，四个页面内联脚本语法检查通过。
可以进入真机人工验证阶段。
