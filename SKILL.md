---
name: "llm-wiki"
description: "LLM Wiki 搭建与维护助手（Karpathy 模式）。当用户说：搭建知识库/处理新资料/查询 Wiki/Wiki 体检/Ingest/Query/Lint 时激活。"
---

# LLM Wiki Skill

> 不是 RAG（查询时检索），而是让 LLM 平时把资料"编译"成持久化的 Markdown Wiki，查询时直接读 Wiki。

## 初始化协议（每次激活时先执行）

**在执行任何操作前，Agent 必须先确认项目位置。**

### 首次激活（从零搭建）

只问一个必填问题：

```
知识库想放在哪个目录？
   （如：D:\myProject\LLM_Wiki，或用当前目录）
```

**可选追问**（如果 Agent 需要初始化 index.md 的 L0 概览）：

```
这个知识库大概是什么方向？
   （可以是具体领域如"AI 前沿研究"，
     也可以是宽泛的如"个人学习笔记"、"综合知识库"，
     或者"暂时不确定，后面再定"）
```

**注意**：不要强迫用户指定领域。很多 Wiki 是综合性的，领域可以后续随内容自然形成。

拿到路径后，后续所有操作中的 `raw/`、`wiki/`、`schema.md` 都基于这个路径。

### 后续激活（已有项目）

用户说 Ingest / Query / Lint 时：

- **如果用户提供了路径或上下文明确** → 直接使用
- **如果不确定** → 先问：`你的 Wiki 项目目录是哪个？`
- **如果当前工作目录下有 schema.md** → 自动识别为项目根目录，无需询问

### 路径记忆

在同一次会话中，一旦确认过路径就记住，不再重复问。

---

## 何时激活

| 用户说的话 | 执行 |
|:----------|:-----|
| 搭建/初始化/从零开始 | → [§一 搭建](#一从零搭建) |
| 处理/Ingest/消化/摄入 | → [§二 Ingest](#二ingest-摄入新资料) |
| 查询/Query/Wiki 里 xxx | → [§三 Query](#三query-提问) |
| 检查/体检/Lint | → [§四 Lint](#四lint-定期体检) |

---

## 一、从零搭建

把 Karpathy 的原始设计文档丢给 Agent：

```
请读取以下文件，理解 LLM Wiki 的核心思想，
然后帮我在「<目标目录>」搭建一个完整的 LLM Wiki 项目。
（可选）大概方向：「<方向或留空>」

<skill-path>/references/karpathy-gist.md
```

> `<skill-path>` 由 Agent 自动替换为 skill 实际路径，用户直接复制使用即可，无需手动修改。

Agent 会按以下步骤执行：

1. **读 Gist**：理解 LLM Wiki 核心模式
2. **建骨架**：创建 raw/ + wiki/ 目录
3. **生成 schema.md**：基于 Gist 原文动态生成 schema.md，同时初始化 index.md（索引）和 log.md（变更日志）
4. **逐条优化**：读取社区优化建议，逐条与你确认是否采纳
5. **就绪**：搭建完成！把第一篇资料（.md 格式）放到 raw/ 目录下，然后对我说"处理新资料"开始第一次 Ingest

> **提示**：搭建完成后，可自行初始化 Git 仓库并配置云同步，方便版本管理与多设备访问。

> - Karpathy 原文：`references/karpathy-gist.md`
> - 社区优化建议：`references/community-optimizations.md`（6 条，逐条确认）
> - 全部离线可用

---

## 二、Ingest（摄入新资料）

最常用的操作。每次有新资料时执行。

### 标准 Prompt

```
请读取 schema.md，按其中的 Ingest 工作流处理 raw/ 目录下的新资料。
```

> **核心规则**：schema.md 是唯一权威来源。所有操作细节（分类、模板、流程）都由 schema 定义，SKILL 不重复规定。
> 如果 schema 中没有定义 Ingest 工作流，则遵循 Karpathy 原文中的基本流程：读取 → 讨论 → 确认 → 写入 → 更新 index/log → 建立关联。

---

## 三、Query（提问）

### 标准 Prompt

```
请读取 schema.md，按其中的 Query 工作流回答我的问题：「xxx」
```

> 如果 schema 中没有定义 Query 工作流，则 Agent 按以下流程处理：读取 index 定位相关页面 → 阅读并综合回答 → 标注来源 → 输出回答。

---

## 四、Lint（定期体检）

### 标准 Prompt

```
请读取 schema.md，按其中的 Lint 工作流对 Wiki 执行健康检查。
```

> 如果 schema 中没有定义 Lint 工作流，则 Agent 执行基础检查：检查孤立页面 → 检查断裂链接 → 检查矛盾或过时内容 → 建议后续优化方向。

每 **5-10 次 Ingest** 后执行一次。

---

## 五、进阶优化

搭建完成后，Agent 会读取 `references/community-optimizations.md` 中的 **6 条社区验证过的优化建议**，逐条询问是否采纳。

> 这些不是必须的——基础 Wiki 不加任何优化也能正常工作。优化项在搭建完成后按需选择，也可以之后随时补充。

详细内容：[community-optimizations.md](references/community-optimizations.md)

---

## 文件速查

| 文件 | 用途 |
|:-----|:-----|
| `references/karpathy-gist.md` | Karpathy 原文（离线可用） |
| `references/community-optimizations.md` | 社区 6 条优化建议（搭建后逐条确认） |
| `schema.md` | Wiki 核心配置文件（搭建时生成，定义wiki规则） |

---