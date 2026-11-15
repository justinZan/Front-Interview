# 01｜前端基础与 Vue 工程化

本章对应岗位要求：HTML/CSS/JavaScript、TypeScript、Vue 3、Vite、Pinia、组件化、前端工程化和 UI 组件二次封装。建议边读边写一个“设备管理页”，而不是只记定义。

## 学习目标

完成本章后，应该能够：

1. 用语义化 HTML 编写可键盘操作的后台表单，并用 CSS 完成响应式大屏布局。
2. 解释盒模型、层叠、BFC、浏览器渲染和 CORS 对实际页面的影响。
3. 用 JavaScript 解释异步执行、闭包和请求竞态。
4. 用 TypeScript 为接口、设备状态和组件事件建模。
5. 解释 Vue 3 响应式系统，并正确选择 `ref`、`computed`、`watch`。
6. 判断状态应该放在组件、composable 还是 Pinia。
7. 设计一个类型清晰、可复用的业务组件。
8. 解释 Vite 开发和生产构建的流程，并正确管理环境变量。

## 1. HTML、CSS 与浏览器基础

这部分决定后台页面是否好用、复杂布局是否稳定，以及联调时能否判断问题发生在样式、浏览器还是接口。练习时优先使用原生 HTML/CSS 做一个简化页面，再迁移到 Vue 和组件库。

### 1.1 语义化 HTML 与表单可访问性

语义化是按内容和行为选对元素。后台系统可用 `<header>` 放全局导航，`<nav>` 放菜单，`<main>` 放主要内容，`<section>` 划分有标题的内容区。执行操作用 `<button>`，跳转页面用 `<a>`，录入数据用原生表单控件。这样浏览器、键盘和辅助技术都能理解页面结构与控件用途。

```html
<main>
  <h1>设备管理</h1>
  <form aria-label="设备筛选">
    <label for="site">站点</label>
    <select id="site" name="site">
      <option value="">全部站点</option>
      <option value="north">北区站点</option>
    </select>

    <label for="keyword">设备名称</label>
    <input id="keyword" name="keyword" type="search" />
    <button type="submit">查询</button>
    <button type="reset">重置</button>
  </form>
</main>
```

`label` 的 `for` 与控件 `id` 要对应；占位提示不能代替标签。错误提示要能被发现，除了颜色还要用文字说明，可使用 `aria-describedby` 把输入框关联到错误信息。弹窗打开后要考虑初始焦点、Tab 导航和关闭后的焦点恢复；若直接使用组件库弹窗，也要验证这些行为。

**面试问答：**“后台系统大量使用组件库，为什么还要关注原生语义？”答题应提到组件库最终仍输出 HTML；正确元素自带键盘行为、表单语义和辅助技术支持。比如把可点击的 `<div>` 伪装成按钮，需要额外补齐焦点、键盘触发、禁用态等行为。

**练习：**不用组件库做设备筛选表单和一张设备表。只用键盘完成选站点、输入关键字、提交、重置和打开详情；检查每个输入是否有可见标签，错误是否有文字提示。

**官方参考：**[MDN 语义化 HTML](https://developer.mozilla.org/en-US/curriculum/core/semantic-html/) · [HTML 表单与按钮](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/HTML_forms) · [表单结构与标签](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/How_to_structure_a_web_form)

### 1.2 Flex、Grid 与响应式布局

Flex 适合沿单一主轴排列，如筛选栏按钮、卡片内指标；Grid 适合同时控制行列，如监控大屏的多图表区。响应式设计不仅是缩放：要决定窄屏时哪些区域换行、隐藏次要信息、允许横向滚动或改为单列。

```css
.dashboard {
  display: grid;
  grid-template-columns: repeat(12, minmax(0, 1fr));
  gap: 16px;
}
.summary { grid-column: span 3; }
.trend { grid-column: span 8; min-width: 0; }
.alarms { grid-column: span 4; min-width: 0; }

.filter-bar {
  display: flex;
  flex-wrap: wrap;
  align-items: end;
  gap: 12px;
}

@media (max-width: 900px) {
  .summary { grid-column: span 6; }
  .trend, .alarms { grid-column: 1 / -1; }
}
```

`minmax(0, 1fr)` 与子项 `min-width: 0` 可避免较长内容把网格列撑破。大屏常见问题是图表容器在切换布局后尺寸变化，需配合容器尺寸监听调用图表 `resize`。表格列太多时，应决定保留关键列、折叠详情或局部滚动，而不是把所有内容压到不可读。

**面试问答：**“Flex 与 Grid 怎么选，1920×1080 和 4K 大屏怎么适配？”答题应说明一维/二维布局差别；先确定内容优先级与最小可读尺寸，再使用弹性网格、相对单位和断点；检查不同纵横比和浏览器缩放，图表随容器尺寸重算。纯整体缩放可能导致文字太小或交互区域不合理。

**练习：**用四个指标卡、趋势图占位区和告警列表做大屏。至少检查 1366×768、1920×1080 与一个窄窗口；记录哪个断点开始改列数、为何这样选，以及图表容器变化时如何通知图表实例。

**官方参考：**[Flexbox 基础](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Flexible_box_layout/Basic_concepts) · [Grid 基础](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Grid_layout/Basic_concepts) · [响应式设计](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout/Responsive_Design)

### 1.3 盒模型、层叠与 BFC

标准盒模型中，声明的 `width` 是内容区宽度；实际占用还要加左右 padding、border 和 margin。`box-sizing: border-box` 让声明宽度包含 padding 与 border，后台卡片栅格更容易计算，但 margin 仍在盒外。

```css
*, *::before, *::after { box-sizing: border-box; }
.panel { width: 320px; padding: 16px; border: 1px solid #ddd; }
/* border-box 下，该元素边框盒宽 320px；margin 另计。 */
```

样式冲突要按层叠规则判断：来源与重要性、层、选择器权重、顺序等都会影响最终声明；不能只记“写在后面就覆盖”。在组件库二次封装时，优先使用主题变量、组件提供的样式入口和低权重的局部类；到处加 `!important` 会增加后续维护成本。

BFC（块级格式化上下文）是块盒布局的一块独立区域。它可容纳内部浮动，并阻止与外部相邻块发生某些 margin 折叠；`display: flow-root` 可明确创建新的 BFC。现代后台布局多用 Flex/Grid，但理解 BFC 有助于排查老页面的浮动、margin 折叠和容器高度问题。

**面试问答：**“设置 `width: 100%` 后卡片仍溢出，如何定位？”答题应检查盒模型、padding/border、`min-width`、长文本、网格/弹性子项的默认最小宽度，以及表格自身尺寸；在 DevTools 查看计算样式和实际盒尺寸，不直接盲目加 `overflow: hidden`。

**练习：**故意给两列卡片设置 `width: 50%` 加 padding，使其溢出；用 `border-box`、Grid `minmax(0, 1fr)` 和子项 `min-width: 0` 修复。再做一个浮动图片导致父容器高度异常的例子，用 `display: flow-root` 观察变化。

**官方参考：**[CSS 盒模型](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Box_model/Introduction) · [CSS 层叠](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Introduction) · [选择器权重](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Specificity) · [BFC](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Display/Block_formatting_context)

### 1.4 浏览器渲染与 CORS

浏览器把 HTML 解析成 DOM，把 CSS 解析成 CSSOM，形成渲染所需结构并进行布局、绘制。JavaScript 可以改变 DOM 和样式；频繁读写几何尺寸可能触发重复布局。监控页若每条实时消息都更新大量 DOM 和图表，容易出现掉帧，应先用 Performance 面板确认耗时，再考虑批量更新、减少节点或限制图表点数。

CORS 是浏览器对跨源脚本请求的访问控制机制。源由协议、主机、端口组成；不同源时，服务端通过响应头声明允许哪些源访问。某些方法或请求头会触发 `OPTIONS` 预检。前端 `fetch` 无法单方面“打开 CORS”；开发代理可以让浏览器请求同源的本地服务，再由代理转发，但生产环境仍需后端/网关正确配置。

**面试问答：**“本地设备接口在 Postman 正常，浏览器报 CORS，怎么排查？”答题应说明 Postman 与浏览器安全模型不同；检查浏览器 Network 中预检和实际响应、`Origin`、允许源/方法/请求头、凭证设置，以及网关是否处理 `OPTIONS`。不要把 `mode: 'no-cors'` 当成读取跨源 JSON 的解决方案。

**练习：**用浏览器 DevTools 记录一次设备列表页面的 DOM 加载、脚本执行和布局耗时；再搭一个本地不同端口的 mock API，观察无 CORS 响应、正确 CORS 响应及带自定义请求头的预检差异，并写出排查步骤。

**官方参考：**[关键渲染路径](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/Critical_rendering_path) · [CORS 指南](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS) · [CORS 错误排查](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS/Errors)

## 2. JavaScript：面试常考的执行模型

### 2.1 作用域、闭包和资源清理

函数能访问其创建位置的词法作用域；函数返回后，只要这个函数仍被引用，相关变量就可能继续保留。闭包常用于封装状态、回调和工厂函数。

```ts
function createCounter() {
  let count = 0
  return () => ++count
}

const next = createCounter()
next() // 1
next() // 2
```

在后台页面中，闭包常见于事件处理器和订阅回调。关注的是**谁持有回调，以及何时解除引用**：

```ts
function subscribeToResize(onResize: () => void) {
  window.addEventListener('resize', onResize)
  return () => window.removeEventListener('resize', onResize)
}

const unsubscribe = subscribeToResize(() => console.log(window.innerWidth))
// 页面退出或组件卸载时调用 unsubscribe()
```

面试回答：闭包本身不是“内存泄漏”；长期存活的监听器、定时器或订阅者持有不再需要的数据，且未清理，才可能形成泄漏。

### 2.2 Promise、事件循环与执行顺序

浏览器执行一段同步代码后，在进入下一个任务前会处理当前微任务队列。Promise 回调通常进入微任务队列，定时器回调进入任务队列。`async` 函数在遇到 `await` 时暂停后续代码，并把继续执行安排到 Promise 处理流程中。

```ts
console.log('A')
setTimeout(() => console.log('B'), 0)
Promise.resolve().then(() => console.log('C'))
console.log('D')
// 常见浏览器环境下输出：A D C B
```

不要把 `await` 理解为“阻塞整个浏览器”。它等待的是当前异步函数的后续执行；其他任务仍可以继续运行。

### 2.3 请求竞态：后台筛选页的真实问题

用户快速切换站点时，先发出的请求可能后返回，覆盖最新结果。下面用请求序号确保只接收最后一次查询的响应：

```ts
let latestRequest = 0

async function loadDevices(siteId: string) {
  const requestId = ++latestRequest
  const result = await fetch(`/api/devices?siteId=${encodeURIComponent(siteId)}`)
  if (!result.ok) throw new Error(`HTTP ${result.status}`)
  const data = await result.json()
  if (requestId !== latestRequest) return // 忽略过期结果
  renderDevices(data)
}
```

在实际 Vue 页面中还应管理 loading 和错误状态。若请求支持取消，可同时使用 `AbortController` 减少无用网络与解析工作。二者目的不同：取消请求节省资源；序号检查确保旧结果不会覆盖新结果。

### 本节面试追问

- 为什么 `setTimeout(fn, 0)` 不保证马上执行？答：它只安排后续任务，需等当前调用栈和相关微任务处理完成。
- `Promise.all` 什么时候不合适？答：多个请求可以并发，但只要一个拒绝整体就拒绝；如果需要逐项结果可考虑 `Promise.allSettled`。
- 防抖和节流如何选？答：搜索输入停止后查询用防抖；滚动或高频状态展示需限制调用频率时用节流/批处理。实时数据有延迟要求时要说明窗口大小。

## 3. TypeScript：把业务状态表达完整

### 3.1 基础类型与联合类型

设备状态可以用字面量联合类型约束，避免任意字符串进入 UI 分支：

```ts
type DeviceStatus = 'online' | 'offline' | 'fault'

interface Device {
  id: string
  name: string
  siteId: string
  status: DeviceStatus
  powerKw: number | null // 离线或无采集数据时可能为空
  updatedAt: string
}

type PageResult<T> = {
  items: T[]
  total: number
  page: number
  pageSize: number
}

async function getDevices(): Promise<PageResult<Device>> {
  const response = await fetch('/api/devices')
  if (!response.ok) throw new Error(`HTTP ${response.status}`)
  return (await response.json()) as PageResult<Device>
}
```

最后一行的类型断言**不会校验运行时数据**。实际项目要根据接口可靠性决定是否在边界做字段验证；对于设备状态、权限等关键数据，运行时验证更有价值。

### 3.2 判别联合：让状态分支可检查

```ts
type LoadState<T> =
  | { kind: 'idle' }
  | { kind: 'loading' }
  | { kind: 'success'; data: T }
  | { kind: 'error'; message: string }

function getLabel(state: LoadState<Device[]>) {
  switch (state.kind) {
    case 'idle': return '尚未查询'
    case 'loading': return '加载中'
    case 'success': return `共 ${state.data.length} 台设备`
    case 'error': return state.message
  }
}
```

相比同时维护 `loading`、`error`、`data` 三个松散变量，这种写法能减少“加载中却同时显示成功数据”等矛盾状态。复杂页面不一定都要采用这种模型，但面试时能体现类型设计服务于业务正确性。

### 3.3 `type`、`interface`、泛型与 `unknown`

- `interface` 常用于可扩展的对象契约；`type` 能表达联合、交叉、元组和条件类型。很多对象场景两者都能用，选择一套团队约定即可。
- 泛型用于**保留类型之间的关系**，例如 `PageResult<Device>` 使 `items` 自动变成 `Device[]`。
- `unknown` 表示暂不确定，使用前必须收窄；`any` 会绕过静态检查，适合极少数明确边界与迁移场景。

```ts
function isDeviceStatus(value: unknown): value is DeviceStatus {
  return value === 'online' || value === 'offline' || value === 'fault'
}

function parseStatus(raw: unknown): DeviceStatus {
  if (!isDeviceStatus(raw)) throw new Error('未知设备状态')
  return raw
}
```

### 本节面试追问

- DTO 与页面模型为什么可能分开？答：接口字段、时间格式、空值约定可能与 UI 所需表达不同；在 API 边界转换可减少页面重复判断。
- `as` 能否解决接口类型安全？答：只能告诉编译器按某类型看待数据，无法防止运行时响应不符合约定。
- 如何表达“在线设备有功率，离线设备没有有效功率”？答：可按状态建立更细的判别联合，或者在页面逻辑中显式处理 `null`，取决于真实协议。

## 4. Vue 3：响应式与组件生命周期

### 4.1 `ref`、`reactive` 和依赖追踪

Vue 在响应式值被读取时建立依赖关系，在值变化时通知相关计算与渲染更新。`reactive()` 对对象使用代理；`ref()` 提供有 `.value` 的响应式引用，常用于基本类型、可整体替换的数据和统一接口。在模板中顶层 ref 通常会自动解包；在脚本中需要 `.value`。

```vue
<script setup lang="ts">
import { computed, reactive, ref } from 'vue'

const search = reactive({ keyword: '', siteId: '' })
const devices = ref<Device[]>([])
const onlineCount = computed(() =>
  devices.value.filter((device) => device.status === 'online').length
)
</script>

<template>
  <p>在线设备：{{ onlineCount }}</p>
  <input v-model="search.keyword" placeholder="搜索设备" />
</template>
```

注意：上面的 `Device` 需从类型模块导入。对 `reactive` 对象直接普通解构可能丢失属性与原代理之间的响应式连接；需要独立响应式属性时可用 `toRefs`，或直接保持 `search.keyword` 访问。`ref` 持有对象时，其内部对象默认也会被转换为响应式对象；大体量不可变数据可考虑 `shallowRef`，但更新时要整体赋新值。

### 4.2 `computed`、`watch` 和 `watchEffect`

| API | 用途 | 典型场景 |
|---|---|---|
| `computed` | 纯派生值，可缓存 | 在线设备数量、表格展示标签 |
| `watch` | 明确观察源并执行副作用 | 站点 ID 变化后请求列表 |
| `watchEffect` | 自动收集同步执行中读取的依赖并执行副作用 | 依赖较多且适合自动追踪的简单副作用 |

```vue
<script setup lang="ts">
import { ref, watch } from 'vue'

const siteId = ref('site-a')
const devices = ref<Device[]>([])
const error = ref('')

watch(siteId, async (id, _previousId, onCleanup) => {
  const controller = new AbortController()
  onCleanup(() => controller.abort())
  error.value = ''
  try {
    const response = await fetch(`/api/devices?siteId=${encodeURIComponent(id)}`, {
      signal: controller.signal
    })
    if (!response.ok) throw new Error(`HTTP ${response.status}`)
    devices.value = (await response.json()) as Device[]
  } catch (cause) {
    if (!controller.signal.aborted) error.value = String(cause)
  }
}, { immediate: true })
</script>
```

这里的 cleanup 在下一次 watcher 回调运行前或 watcher 停止时执行，避免旧请求继续影响结果。`watchEffect` 的异步回调只自动追踪首次 `await` 之前同步访问到的依赖；接口筛选条件明确时，`watch` 往往更直观。

### 4.3 生命周期与副作用清理

`onMounted` 用于需要 DOM 容器的操作（例如图表初始化）；`onUnmounted` 用于清理图表、定时器、订阅等。组件可能因路由切换、条件渲染或缓存策略出现不同生命周期，设计时要明确资源由谁拥有。

```ts
import { onMounted, onUnmounted } from 'vue'

let timer: ReturnType<typeof setInterval> | undefined
onMounted(() => {
  timer = setInterval(() => console.log('刷新时间标签'), 1000)
})
onUnmounted(() => {
  if (timer) clearInterval(timer)
})
```

如果页面使用 `<KeepAlive>`，组件离开视图时可能是停用而非卸载；持续连接或计时器是否暂停，需要结合 `onActivated` / `onDeactivated` 与业务需求处理。

### 4.4 列表 `key` 与组件通信

可排序、可插入的设备列表应使用稳定设备 ID 作为 `key`，使 Vue 正确复用对应节点与组件内部状态。数组下标在插入/排序后可能指向不同设备，导致输入框、选中态错位。

```vue
<DeviceRow
  v-for="device in devices"
  :key="device.id"
  :device="device"
  @select="selectedId = $event"
/>
```

通信选择：父传子用 props，子通知父用 emits；多层局部共享可用 provide/inject；跨路由共享且有明确业务拥有者的状态放 Pinia。不要把每个表单输入都放全局 store。

### 本节面试追问

- 为什么 `computed` getter 不宜发请求？答：计算属性应表达纯派生值；请求是副作用，可能触发不可预测的重复调用与状态变化。
- `watch` 和 `watchEffect` 如何选？答：依赖明确且要比较新旧值时用 `watch`；自动收集若干依赖的简单副作用可用 `watchEffect`。
- `v-if` 与 `v-show` 如何选？答：前者按条件创建/销毁子树；后者保留 DOM 并切换显示。根据切换频率、初始成本和资源生命周期判断。

## 5. Pinia：状态边界与异步流程

Pinia store 可以包含 state、getter 和 action。适合放登录用户、权限、跨页面共享的站点选择等有明确共享需求的状态；一个页面自己的弹窗开关、临时输入通常留在组件中。

```ts
// stores/site.ts
import { defineStore } from 'pinia'
import { computed, ref } from 'vue'

export const useSiteStore = defineStore('site', () => {
  const currentSiteId = ref<string | null>(null)
  const recentSiteIds = ref<string[]>([])
  const hasSelection = computed(() => currentSiteId.value !== null)

  function selectSite(id: string) {
    currentSiteId.value = id
    recentSiteIds.value = [id, ...recentSiteIds.value.filter((x) => x !== id)].slice(0, 5)
  }

  return { currentSiteId, recentSiteIds, hasSelection, selectSite }
})
```

组件里需要解构响应式 state/getter 时使用 `storeToRefs`；action 可直接从 store 解构：

```ts
import { storeToRefs } from 'pinia'
import { useSiteStore } from '@/stores/site'

const siteStore = useSiteStore()
const { currentSiteId, hasSelection } = storeToRefs(siteStore)
const { selectSite } = siteStore
```

直接 `const { currentSiteId } = siteStore` 会拿到当前值，破坏该解构变量与 store 的响应式关联。设计 store 时先问：这个状态被几个页面使用？谁修改它？生命周期多长？页面刷新后是否需要恢复？

**进阶讨论：**实时设备消息不要不加选择地全部塞进单一全局 store。按设备 ID 归并、限制历史窗口、只保留页面需要的状态，并在订阅结束时清理。Pinia 解决共享问题，不自动解决高频更新性能问题。

### 本节面试追问

- Pinia getter 与组件 `computed` 的区别？答：两者都表达派生值；getter 适合跨组件共享的 store 派生逻辑，组件 `computed` 适合局部视图逻辑。
- 异步 action 失败怎样处理？答：明确是 action 转换成统一结果还是抛异常给调用方；页面负责展示反馈，避免 store 和页面重复提示。
- 用户退出登录要清理什么？答：会话、权限、动态路由、与用户绑定的订阅/缓存，以及不应跨账号保留的业务状态。

## 6. Vite 与前端工程化

### 6.1 开发过程与生产构建

Vite 开发服务器利用浏览器原生模块加载与按需转换，依赖会预处理以改善开发体验；生产构建会生成适合部署的优化产物。面试时说明**开发快**和**上线包体积小**属于不同问题，分别要看启动/HMR 与构建产物/首屏指标。

常见项目目录可按业务责任划分：

```text
src/
  api/          # 网络请求和接口数据转换
  components/   # 通用展示与交互组件
  composables/  # 可复用的组合逻辑
  features/     # 设备、站点、告警等业务模块
  router/       # 路由与导航控制
  stores/       # 跨页面状态
  types/        # 共享类型
```

目录不是固定标准；更关键的是同类逻辑有稳定位置、模块依赖方向清晰。

### 6.2 路由懒加载与代码拆分

```ts
const routes = [
  {
    path: '/devices',
    component: () => import('@/features/devices/DevicePage.vue')
  }
]
```

动态 `import()` 让非首屏页面在访问时加载。实践中还要看依赖共享、请求数量和用户常走路径；把每个微小组件都拆包不一定有收益。可以先分析构建产物，再定位最大的首屏模块。

### 6.3 环境变量与密钥

Vite 中通过 `import.meta.env` 读取环境信息。以 `VITE_` 前缀暴露给客户端的变量会进入前端产物，因此**不能放服务端密钥、数据库密码或私有访问凭证**。

```ts
const apiBaseUrl = import.meta.env.VITE_API_BASE_URL
```

例如 `.env.development` 配开发接口地址，`.env.production` 配生产公开接口地址；真正的秘密由后端或部署环境持有。开发代理只帮助本地请求转发，并不代替生产环境的跨域与网关配置。

### 6.4 接口层与错误处理

把传输协议、错误码转换和业务字段映射集中在 API 层，组件只关心加载/成功/失败状态。统一请求封装应该解决明确的共同问题，例如基地址、身份信息、超时和错误格式；保留接口个性化配置能力。

```ts
type ApiError = { code: string; message: string }

async function getJson<T>(path: string, signal?: AbortSignal): Promise<T> {
  const response = await fetch(`${apiBaseUrl}${path}`, { signal })
  if (!response.ok) {
    const detail = (await response.json().catch(() => null)) as ApiError | null
    throw new Error(detail?.message ?? `请求失败：${response.status}`)
  }
  return (await response.json()) as T
}
```

这只是示意：生产项目还要处理 204 无响应体、非 JSON 错误、鉴权过期、幂等重试与统一日志。不要把所有失败都无条件重试，尤其是写操作。

## 7. 业务组件二次封装

组件库负责基础交互和视觉规范，业务组件负责稳定的业务语义。例如设备状态标签可统一颜色、文案和空状态，避免每个页面重新写判断。

```vue
<!-- DeviceStatusTag.vue：内部可换成 Element Plus 或 Ant Design Vue 的 Tag -->
<script setup lang="ts">
type DeviceStatus = 'online' | 'offline' | 'fault'
const props = defineProps<{ status: DeviceStatus }>()

const labels: Record<DeviceStatus, string> = {
  online: '在线', offline: '离线', fault: '故障'
}
</script>

<template>
  <span :class="['device-status', `device-status--${props.status}`]">
    {{ labels[props.status] }}
  </span>
</template>
```

更复杂的设备选择器可提供 `modelValue` / `update:modelValue`、加载状态、禁用态、站点过滤、搜索和清空事件；表格组件可以抽取分页与远程排序的共性，但业务列、插槽和行操作应保留配置空间。

二次封装的判断标准：重复出现的业务规则是否稳定？封装后的调用是否比原组件清楚？升级底层组件库时是否能集中修改？如果只是转发几十个 props，封装可能增加维护成本。

### 组件设计面试追问

- 为什么 props 不应在子组件中直接改？答：单向数据流便于追踪状态所有者；子组件通过事件请求父级更新。
- 插槽和 props 如何选择？答：结构差异用插槽较自然；简单数据与行为配置用 props；保持 API 小且明确。
- 组件如何支持测试？答：把数据转换和规则提取为纯函数；组件输入输出明确；模拟加载、空、错误和权限场景。

## 8. 一次完整练习：设备管理页

### 最小功能

1. 用 `Device`、`PageResult<T>` 和设备状态联合类型定义数据。
2. 搜索区有站点、状态和关键字；切换条件时重置页码。
3. 通过 API 层请求数据，展示加载、空结果和错误重试。
4. 表格行使用稳定的设备 ID；点击后打开详情抽屉。
5. 把设备状态标签封装成业务组件。
6. 只在跨页面共享时使用 Pinia 保存当前站点。
7. 快速连续切换站点，验证旧请求不会覆盖新数据。

### 完成标准

- 页面刷新、接口慢响应、接口失败、空列表均有明确 UI。
- TypeScript 不依赖 `any` 掩盖接口字段。
- 组件销毁时清理请求和监听器。
- README 说明目录职责、状态放置理由、一个真实遇到的问题及修复过程。

### 口述演练

用 3 分钟回答：“这个设备管理页的状态分别放在哪里？接口响应乱序怎么处理？为什么选择当前的组件边界？”

## 9. 官方学习资料与 GitHub 源码

推荐按“文档 → 运行示例 → 源码或 issue → 自己复现”的顺序学习。

- [JavaScript Guide（MDN）](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)；[Promise 参考](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)；[日常类型](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)；[泛型](https://www.typescriptlang.org/docs/handbook/2/generics.html)；[GitHub 源码](https://github.com/microsoft/TypeScript)
- [Vue 响应式基础](https://vuejs.org/guide/essentials/reactivity-fundamentals.html)；[计算属性](https://vuejs.org/guide/essentials/computed.html)；[侦听器](https://vuejs.org/guide/essentials/watchers.html)；[TypeScript 与 Composition API](https://vuejs.org/guide/typescript/composition-api.html)；[GitHub 源码](https://github.com/vuejs/core)
- [Pinia 核心概念](https://pinia.vuejs.org/core-concepts.html)；[GitHub 源码](https://github.com/vuejs/pinia)
- [Vite 原理与定位](https://vite.dev/guide/why.html)；[环境变量](https://vite.dev/guide/env-and-mode.html)；[GitHub 源码](https://github.com/vitejs/vite)
- [Element Plus GitHub](https://github.com/element-plus/element-plus)；[Ant Design Vue GitHub](https://github.com/vueComponent/ant-design-vue)

> 阅读源码时先带着问题定位，例如 Vue 的 `ref` 如何追踪依赖、Pinia `storeToRefs` 为何能保留响应性。初学阶段不必从仓库第一页顺序通读。
