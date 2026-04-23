# 数据架构、模板使用与存储

补充 SKILL.md 的「HTML 模板」「文件与存储」「笔记整理规则」——记录 chapters 数据结构详细说明、4 种 block 字段、outline section 写法、myEdits 合并流程、笔记错误处理细节、window.storage 工作机制、备份提醒时机。SKILL.md 主体提示需要时来这里查。

## chapters 数据结构

template.html 里的 chapters 是一个数组，三种类型的条目按顺序混排：

```js
const chapters = [
  // 第一项是 meta（整体标题/副标题/breadcrumb）
  { id: 'meta', type: 'meta', breadcrumb: '...', title: '...', subtitle: '...' },

  // 第一次生成笔记时插入的 outline section（详见下文「outline section」）
  { id: 'outline', type: 'section', navLabel: { num: '★', text: '完整大纲' }, title: '完整大纲', blocks: [...] },

  // chapter 条目：一级标题（章），在正文和侧边栏都渲染为大标题
  { type: 'chapter', num: '第 1 章', title: '章节名' },

  // section 条目：二级标题（节），承载实际内容的 blocks
  {
    id: 'section-id',
    type: 'section',
    navLabel: { num: '1.1', text: '侧边栏显示的名字' },
    title: '小节标题',
    blocks: [
      { kind: 'prose', html: '<p>...</p><p>...</p>' },
      { kind: 'callout', label: '反直觉判断', html: '<p>...</p>' },
      { kind: 'proCon', pros: '...', cons: '...' },
      { kind: 'compareTable', headers: [...], rows: [[...], [...]] }
    ]
  },

  // 下一章
  { type: 'chapter', num: '第 2 章', title: '另一章' },
  { id: '...', type: 'section', ... }
];
```

chapter 和 section 在数组里**平级顺序排列**——chapter 出现即代表"后续 section 都属于这一章"，直到下一个 chapter 条目为止。

## 四种 block 字段详解

### prose

- `kind: 'prose'`
- `html`：模板字符串，可以包含 `<p>`、`<ul><li>`、`<ol><li>`、`<strong>` 等任意 HTML
- 用法：所有正文段落、bullet list、有序步骤都用这个——大部分内容都是 prose

### callout

- `kind: 'callout'`
- `label`：左上角的标题文字（中文直接写，不用大写或英文）
- `html`：内容（同 prose）
- 用法：反直觉判断、独立陷阱、精华原话——只用这三种场景。详细决策见 SKILL.md「卡片克制使用」 + [teaching-detail.md](teaching-detail.md#callout-反例清单)

### proCon

- `kind: 'proCon'`
- `pros`：优点描述（一段文字，1-2 句）
- `cons`：缺点描述（一段文字，1-2 句）
- 用法：单一方案的优缺点对比，比纯文字"优点是…缺点是…"好扫读。一边超过两三句就拆出去用 prose

### compareTable

- `kind: 'compareTable'`
- `headers`：列标题数组
- `rows`：每行是数组，元素数和 headers 一致
- 用法：多方案多维度对比。**列数控制在 4 列以内最舒服**，超过 5 列建议用「行列转置」（维度作为行、方案作为列），第一列自动有浅色背景适合放维度名

## outline section（大纲目录）

第一次生成笔记时，chapters 必须包含一个 `id: 'outline'` 的 section，紧跟在 meta 之后，作为正文区第一节。让用户打开笔记时立刻看到完整大纲。

写法：

```js
{
  id: 'outline',
  type: 'section',
  navLabel: { num: '★', text: '完整大纲' },
  title: '完整大纲',
  blocks: [
    { kind: 'prose',
      html: `<p>本笔记的完整学习大纲（已学的小节会持续填充内容）：</p>
<ol>
<li><strong>第 1 章 章节名</strong>
<ul>
<li>1.1 小节名</li>
<li>1.2 小节名</li>
</ul></li>
<li><strong>第 2 章 章节名</strong>
<ul>
<li>2.1 小节名</li>
</ul></li>
</ol>`
    }
  ]
}
```

**每次大纲变更**（增节、删节、改章节标题）时同步更新这个 outline section 的内容，让目录始终反映最新大纲。

## myEdits 合并流程

**重写某节时先合并 myEdits**——避免覆盖用户的编辑：

1. 读取 storage 里 myEdits 中和这节相关的条目（key 形如 `<sectionId>:block:N:html` 或 `<sectionId>:block:N:pros` 等）
2. 把 myEdits 里的修改**合并进 chapters 对应位置**（即把用户改过的内容覆盖回 chapters 里）
3. 清空 myEdits 里这节相关的条目
4. 基于合并后的 chapters 改写

这样用户的修改成为 Claude 改写的起点，而不是被 Claude 的旧版本覆盖。

## 笔记错误处理流程

Claude 在回答问题时意识到笔记里之前写的内容有错——**不要直接改**（违反「核心原则 3」）。流程：

1. **指出问题**："我注意到笔记里 XX 这段写得不准确，正确应该是 YY"
2. **给出建议版本**——直接把改好的文字贴出来
3. **优先建议用户自己复制粘贴进编辑模式**——点几下就完成了，比调用工具重新生成快得多
4. **用户明确要求 Claude 改时**，才进入"合并 myEdits → 改写"流程（见上节）

为什么优先让用户自己改：写入新版会触发 myEdits 合并 + 整个 section 重渲染，操作开销大；用户在编辑模式里直接改 5 个字、按 Ctrl+S 保存——更轻量、更安全。

## 持久化机制

笔记基于 Claude.ai Artifact 的账号级云端存储 `window.storage`：

- **启动时**：从 storage 读 myEdits 和 myNotes（用户的修改和批注）
- **编辑后**：自动存回 storage
- **chapters**：始终以 HTML 文件里的定义为准，**不存 storage**——这样 Claude 改了 HTML 文件，新内容会立刻生效，不会被 storage 里的旧快照覆盖

HTML 文件丢失也能从 storage 恢复用户的修改和批注（重新打开同名笔记会自动加载 myEdits/myNotes）。

`window.storage` **需要 Pro/Max/Team/Enterprise 订阅**。免费用户要在第一次创建笔记时提前告知"无法云端保存，需要 Download 到本地"。

## 备份时机提醒

在以下时机主动提醒用户备份：

- **学完一个大章节**
- **用户说"今天学到这里"或类似的暂停信号**

提醒话术："建议用 Artifact 右上角的 Add to project 或 Download 备份。"

不要每次更新都提醒——只在关键节点。频繁提醒等于不提醒，用户会麻木。
