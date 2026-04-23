# study-notebook

一个 Claude Skill，带你**先学后记**，先和 Claude 对话学习一个领域，每讲完一节，Claude 同步把这节内容整理进一份结构化、可编辑、可批注、可持续迭代的 HTML 学习笔记。

![学习笔记渲染效果](https://github.com/user-attachments/assets/50261564-df7e-4ff3-9e4b-9addb5fc76e4)

![横向对比表格](https://github.com/user-attachments/assets/71b979f3-5cda-4938-aea4-66206414ed91)

![优缺点对比](https://github.com/user-attachments/assets/905cc397-f400-4de5-bc44-0566b2ac0c39)

---

## 推荐使用场景

适合**需要快速系统掌握某个领域基础知识的人**：

- 准备面试的候选人（把一个领域的核心概念系统过一遍）
- 学生或实践者补齐某块知识
- 需要长期回看同一主题的人（笔记可以持续追加，每次复习不需要从头）

**不适合**：

- 只想问一个具体问题（用普通对话即可，不需要触发 skill）
- 想要 Claude 直接给一份完整教程的人（这个 skill 强调"对话产生的个性化笔记"，不是模板化教程）

---

## 安装

### Claude.ai（网页 / 桌面 App）

1. 把这个仓库 Download 到本地（GitHub 页面右上角 Code → Download ZIP）
2. 解压后**重命名根文件夹为 `study-notebook`**（GitHub 默认带 `-main` 后缀），然后重新打包成 ZIP
3. 在 Claude.ai 右上角点击 **Settings → Customize → Skills**
4. 点击 **+ Create skill → Upload a skill**，上传 ZIP 文件，skill 会自动出现在列表中并启用
5. 注意：需在 Settings 里开启 "Code execution and file creation" 才能看到 Skills 选项。

### Claude Code

```bash
# Mac / Linux
git clone https://github.com/ZhuosiYang-01/study-notebook-skill ~/.claude/skills/study-notebook

# Windows (PowerShell)
git clone https://github.com/ZhuosiYang-01/study-notebook-skill $env:USERPROFILE\.claude\skills\study-notebook
```


---

## 使用方式

### 1. 跟 Claude 对话开始学习

在 Claude.ai 或 Claude Code 里直接说：

- "教我SQL的基础知识和操作"
- "带我过一遍这门课的知识来准备期末考试，给你课程大纲"
- "我想系统学一下大模型原理"

Claude 会先和你确认大纲，然后**一节一节**带你学，每节讲完等你反应，可以追问，不会一次性灌输。

### 2. 用 Artifact 渲染笔记

每节讲完后，Claude 把内容整理成 HTML 笔记，在 Claude.ai 里以 **Artifact** 形式显示在右侧。Artifact 可直接看到渲染效果：

- 左侧侧边栏目录，点击跳转
- 中间正文区，每节的内容随学习进度填充
- 顶部「编辑」按钮——点击进入编辑模式，可直接修改任意文字
- 每节末尾有「我的批注」区，写自己的思考、疑问、联想

如果是 Claude Code，则用默认浏览器打开 HTML。

### 3. 定期保存到项目

如果要开启新对话，把笔记 Artifact 通过 **Add to Project** 保存到项目空间，新对话能找到这份笔记继续学

但要注意：**Add to Project 是"快照保存"，不是实时同步**

- 加进 project 之后，project 里的 HTML 是当时那一刻的版本
- 你之后在对话里继续学习、Claude 继续追加内容到 Artifact，**project 里的快照不会自动更新**
- 想让 project 里有最新版本，**要重新 Add to Project 手动覆盖旧快照**
- 当前对话窗口里的 Artifact 会一直保留最新版（包括 Claude 的更新和你的编辑），只在对话里继续学的话不会丢

下次新对话开始学同一主题时，Claude 会从 project 读到最新快照，从这版接着追加。你的编辑和批注通过 `window.storage` 云端持久化（账号级），跨对话自动恢复——前提是 Pro / Max / Team / Enterprise 订阅。

### 4. 定期手动备份到本地

云端 storage 和 project 都依赖 Anthropic 服务，建议关键节点也手动 Download 到本地：

Artifact 右上角有 Download 按钮，保存为 `.html` 文件。

---

## 项目结构

```
study-notebook/
├── README.md
├── LICENSE
├── SKILL.md                       skill 的入口指令（Claude 触发时读）
└── references/
    ├── template.html              笔记模板（含完整 CSS / JS / 4 种 block 示例）
    ├── teaching-detail.md         讲解风格、追问处理细节
    ├── data-and-storage.md        数据架构、模板使用、storage 机制
    └── troubleshooting.md         常见问题排查
```

---

## 注意事项

- **`window.storage` 需要 Pro / Max / Team / Enterprise 订阅**。免费用户无法云端保存编辑和批注，请定期 Download 到本地。
- **没有特殊需求不建议新建版本化文件**（`-v2.html`、`-new.html` 等）。这会让 storage key 不一致、之前的编辑批注全部失效。Skill 内置规则会拒绝这么做——如果你自己想保留旧版做新版本，请明确告诉 Claude。
- **笔记是用于复习的**，不是对话的回放。不要期待笔记里包含所有对话细节——只保留复习时需要的核心内容（专业定义、对比、判断、口诀）。

---

## License

[MIT](LICENSE)
