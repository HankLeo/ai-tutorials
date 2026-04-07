# Claude + Obsidian 打造本地知识库：Karpathy LLM Wiki 最先进玩法搭建教程

## 核心思路：从 RAG 到增量式 Wiki 维护

Andrej Karpathy 提出的 LLM Wiki 方案与传统 RAG 不同：

- **传统 RAG**：每次查询都从原始文档重新检索、重新推导，没有知识积累
- **Karpathy 方案**：让 LLM 增量式构建和维护一个结构化的、互相链接的 Markdown Wiki，知识持续复利增长

核心思想：Obsidian 是你的 IDE，LLM 是程序员，Wiki 是代码库。你负责找资料和提问，LLM 负责所有脏活累活——总结、交叉引用、分类整理、保持一致性。

## 系统架构：三层设计

整个系统分为三层：

```
├── Raw Sources/  # 原始资料层：你的原始文件，LLM只读不写，不可变
├── Wiki/         # 知识库层：LLM生成和维护的Markdown文件，完全由LLM负责
└── Schema/       # 规则层：定义Wiki组织方式和操作约定（如CLAUDE.md）
```

## 准备工作

需要安装的基础工具：

1. **Obsidian**：本地知识库管理工具，支持插件扩展
   - 下载：https://obsidian.md/

2. **Claude Code**：提供AI Agent能力，支持文件操作
   - 安装参考官方文档：https://docs.anthropic.com/claude-code

3. **Claudian 插件**：连接 Obsidian 和 Claude Code 的桥接插件
   - 下载地址：https://github.com/YishenTu/claudian/releases
   - 安装步骤：
     1. 下载 `main.js`、`manifest.json`、`styles.css` 三个文件
     2. 在你的 Obsidian vault 创建 `.obsidian/plugins/claudian/` 目录
     3. 将三个文件复制进去
     4. 在 Obsidian 设置 → 社区插件 中启用 "Claudian"
     5. 启用后左侧边栏会出现机器人图标，点击即可与 Claude 对话

4. **Skills 包配置**（可选但推荐）：为 Claude 添加 Obsidian 特定能力
```bash
# 克隆Skills仓库
git clone https://github.com/kepano/obsidian-skills.git ~/.claude/skills/obsidian-skills-main
# 复制到Skills目录
cp -r awesome-skill ~/.claude/skills/
```

## 第一步：初始化知识库结构

### 目录结构设计

推荐结合十进制分类法和PARA方法论的目录结构：

```
vault/
├── 00-收件箱/        # 临时存储，收集灵感和待处理信息
├── 01-内容创作/      # 日常创作、文章草稿
├── 02-软件开发/      # 技术笔记、代码片段
├── 03-产品设计/      # 产品需求、竞品分析
├── 04-数据分析/      # 数据报告、复盘总结
├── 10-项目进行/      # 正在执行的具体项目
├── 20-持续关注/      # 长期追踪的领域
├── 30-资源库/        # 工具教程、参考文献
├── 40-归档/          # 已完成或暂停的项目
├── aboutme/          # 个人信息，让AI了解你
├── RawSources/       # Karpathy方案的原始资料层
└── Wiki/             # Karpathy方案的LLM维护知识库
```

### 让 Claude 一键生成目录结构

在 Claude Code 中执行以下指令：

```
帮我创建以下完整的文件夹结构，每个文件夹下创建README文件说明其用途和命名规范。

文件夹清单：
- 00-收件箱：临时存储，收集灵感和待处理信息
- 01-内容创作：日常创作、文章草稿、发布记录
- 02-软件开发：技术笔记、代码片段、项目文档
- 03-产品设计：产品需求、竞品分析、设计文档
- 04-数据分析：数据报告、复盘总结、趋势观察
- 10-项目进行：正在执行的具体项目
- 20-持续关注：长期追踪的领域、技能提升计划
- 30-资源库：工具教程、参考文献、最佳实践
- 40-归档：已完成或暂停的项目
- aboutme：关于我的个人信息，帮助AI理解我的偏好
- RawSources：原始资料层，存储原始文档，LLM只读不写
- Wiki：LLM维护的知识库，存储结构化的摘要和分析页面

每个README需要包含该文件夹的定位、存储内容类型和严格的命名规范。并在末尾添加AI指令："AI指令: 在此文件夹创建文件前，请先读取本README并严格遵循命名规范。"
```

### 配置 About Me 让 AI 真正了解你

这是最容易被忽视但最有价值的一步。在 `aboutme/` 文件夹创建以下文件：

```
aboutme/
├── README.md         ← AI读取指南
├── 我的介绍.md         ← 个人背景、职业经历、核心身份
├── 写作风格.md         ← 语言偏好、格式习惯、表达特点
├── 性格特点.md         ← MBTI性格、沟通风格、工作偏好
├── 内容定位.md         ← 目标受众、内容方向、差异化定位
└── 审稿标准.md         ← 文章质量检查清单、红线标准
```

示例 `我的介绍.md`：
```
- 职位：产品经理，AI产品方向
- 工作经验：5年互联网，2年AI工具领域
- 核心能力：产品策略、用户研究、AI应用设计
- 当前身份：技术博主、独立创作者
```

示例 `写作风格.md`：
```
- 语气：理性、客观，避免夸张修辞
- 结构：问题→分析→实践→总结
- 长度：深度文章1500-3000字，短文300-500字
- 禁用词汇：最强、颠覆、革命性等绝对化表述
```

配置完成后，Claude 每次生成内容都会自动匹配你的风格和要求。

## 第二步：配置 LLM Wiki 规则

在项目根目录创建 `CLAUDE.md`（规则文件），内容参考：

> 注意：schema（规则文件）不是一次配置就永久不变的，它是你和 LLM **共同演化**的产物。随着使用，可以不断调整规则，让系统越来越贴合你的使用习惯。

```markdown
# LLM Wiki 操作规则

遵循 Karpathy 的 LLM Wiki 方法论：

## 三层架构
- RawSources/: 原始资料，只读不修改
- Wiki/: LLM维护的知识库，可自由创建和更新
- CLAUDE.md: 本文件，规则定义

## 核心操作流程

### 1. Ingest（录入新来源）
当有新的原始资料添加到 RawSources/:
- 阅读整个原始资料
- 提取关键信息、核心观点、重要实体和概念
- 在 Wiki/ 创建对应的摘要页面
- 更新相关实体和概念页面，整合新信息
- 标注新信息与旧结论的矛盾之处
- 更新 index.md 索引
- 记录操作到 log.md

一个来源通常需要更新 10-15 个相关 Wiki 页面。

### 2. Query（提问探索）
当用户提问时：
- 搜索 Wiki 中相关页面
- 综合信息生成回答
- **如果回答有长期保留价值，将其保存为新的 Wiki 页面**（关键闭环）

Karpathy 强调：好的回答本身就是知识，应该存入 Wiki。这样不只是 Ingest 在喂料，Query 探索本身也在持续产生新知识，让知识库复利增长。

输出形式可以多种多样：
- Markdown 页面（最常用）
- 对比表格
- Marp 幻灯片
- matplotlib 图表
- Obsidian Canvas 可视化

无论什么格式，有长期价值都应该回存到 Wiki 中，让知识库越用越丰富。
- 更新索引和交叉引用

### 3. Lint（健康检查）
定期执行：
- 查找页面之间的矛盾
- 标记过时信息
- 发现没有入链的孤儿页面
- 识别提到但没有独立页面的重要概念
- 补全缺失的交叉引用
- 建议下一步应该研究的问题

## 命名规范
- 所有文件使用 Markdown 格式，扩展名 .md
- 文件名使用小写字母和连字符，不包含空格
- 索引文件：Wiki/index.md
- 操作日志：Wiki/log.md

## index.md 索引机制

在百页级知识库规模下，**不需要复杂的向量数据库**，`index.md` 就足够用了：

1. LLM 回答问题时，**先读 `index.md`**，它记录了所有页面的主题和概要
2. LLM 根据问题从 `index.md` 定位到相关页面
3. 再去精读这些相关页面的具体内容，综合生成回答

这就是 Karpathy 方案能够轻量运行的核心原因之一。`index.md` 由 LLM 自动维护，每次新增页面都会更新。

## log.md 日志技巧

Karpathy 的一个实用小 tip：

log 条目使用统一的前缀格式，比如：
```
# 2026-04-01: Ingested raw article "..."
```
这样可以用 `grep` 快速搜索历史操作，方便追踪知识库演进过程。

## 交叉引用
- 使用 Obsidian 格式 [[页面名称]] 进行内部链接
- 每个新页面至少添加 2-3 个相关页面的反向链接
```

## 第三步：开始使用

### 1. 录入资料（Ingest）

工作流程：
1. 将新的原始文件（文章、论文、笔记）放到 `RawSources/` 目录
2. 在 Claude Code 中说：
```
我在 RawSources 添加了一个新文件 [文件名]，请按照 CLAUDE.md 的规则执行 Ingest 操作。
```
3. LLM 会自动读取文件，提取信息，创建/更新 Wiki 中的相关页面
4. 你可以在 Obsidian 中实时浏览更新结果

### 2. 提问探索（Query）

直接向 Claude 提问：
```
关于 [主题]，Wiki 里都有什么内容？请帮我综合一下当前的认识。
```
生成的回答如果有价值，可以让 LLM 保存为新页面，丰富你的知识库。

### 3. 定期健康检查（Lint）

每隔一段时间（比如每周）运行一次：
```
请按照 CLAUDE.md 的规则对整个 Wiki 执行 Lint 健康检查，修复发现的问题。
```
LLM 会自动清理矛盾、补全链接、保持 Wiki 健康。

### 4. 实际体验

Karpathy 本人的使用方式：
> 我把 LLM Agent 打开在一边，Obsidian 打开在另一边。LLM 根据对话编辑 Wiki，我在 Obsidian 里跟着链接点点看，看看图谱视图，读读更新后的页面。

## 推荐工具和插件

- **Obsidian Web Clipper**：浏览器扩展，快速把网页文章转成 Markdown 保存到 RawSources
- **图片本地化**：在 Obsidian 设置中把附件目录设为固定路径，使用 Web Clipper 一键下载文章图片到本地，让 LLM 能直接读图而不依赖外部 URL，这对研究类知识库非常实用
- **Dataview**：基于元数据做动态查询，方便知识检索
- **Canvas**：Obsidian 原生支持，可用于创建知识可视化图谱
- **qmd**（可选）：Wiki 变大后提供更好的全文搜索能力
- **Git**：天然支持版本控制，整个 Wiki 就是一个 Git 仓库

## 完整工作流示例

1. **收集**：看到一篇好文章，用 Web Clipper 保存到 `RawSources/`
2. **录入**：让 Claude Ingest，它会自动生成摘要页面，更新相关概念页，处理交叉引用
3. **探索**：你在 Obsidian 里浏览，跟着链接发现关联，提出新问题
4. **沉淀**：新问题的回答保存为新页面，知识库持续增长
5. **维护**：定期 Lint，保持知识库一致性

## 成本和优势

- **成本**：主要是 Claude API 调用费用，百页级知识库月成本通常不到几十元
- **优势**：
  - 知识真正沉淀，持续复利增长
  - LLM 承担所有繁琐的维护工作，人类只需要思考和探索
  - 完全本地，数据隐私可控
  - 不需要复杂的向量数据库（百级来源规模下，索引文件就够用）
  - Obsidian 生态完善，浏览体验极佳

## 参考资源

- Karpathy 原文 Gist：https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- Claudian 插件：https://github.com/YishenTu/claudian
- Obsidian Skills：https://github.com/kepano/obsidian-skills
- sage-wiki 实现：https://github.com/xoai/sage-wiki
