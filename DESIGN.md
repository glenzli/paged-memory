# Paged-Memory (P-Mem): Isomorphic Cross-Context Memory System

**Status**: Design Draft  
**Reference/Dependency**: [Paged-Context-Protocol (PCP)](https://github.com/glenzli/paged-context-protocol)

---

## 1. 核心理念：跨越 Context 与的完全同构 (Core Philosophy: Isomorphism)

**Paged-Memory (P-Mem)** 并不是一个传统的外部“知识库”或“向量检索 RAG 系统”，而是 **PCP (Paged-Context-Protocol) 在时间维度上的持久化映射层**。

它的核心设计哲学在于**抹平 LLM 单次任务上下文 (Context) 与全局持久化记忆 (Memory) 的物理差异**。
在 P-Mem 的架构下，Memory 中存储的实体**必须且只能是 PCP 协议中定义的 Logical Page (逻辑页)**。
无论是一次临时对话产生的当前语境，还是跨越十次会话沉淀下的长期记忆，它们共享**同一套数据结构 (Pages)、同一套寻址逻辑 (Router)** 以及**同一套整理法则 (Consolidator)**。

唯一的区别仅仅在于：**Context 只存在于单次任务的内存分配中，而 Memory 处于全局持久化存储区。**

---

## 2. 内存实体同构：逻辑页的套娃 (Isomorphic Entities: Logical Pages)

P-Mem 强制沿用 PCP 中的逻辑实体规范。内存中的数据就是一棵巨大的、由**综述页 (Consolidated Page)** 嵌套而成的逻辑树。

### 2.1 原始页 (Original Page)的重新定义
在 PCP 中，“原始页”通常对应最底层的物理切片（如一段生肉对话、一句具体的代码片段）。但在 P-Mem 这套持久化跨会话系统中：
*   **内存原始页** 同样是逻辑树的最小原子单位（不再具备子节点）。包含 `id`, `depth`, `timestamp`, `origin: Storage`, `summary`, `keywords`。
*   **但它的来源可能不再是“原始物理块”**。它很有可能是在上次 PCP 会话结束时，由 PCP Consolidator 输出的“结论性 Summary”。
*   在 P-Mem 看来，只要它被定标为逻辑尽头（depth 最深），它就是“原子”，哪怕这个原子本身已经是高度提纯过的信息。

### 2.2 综述页 (Consolidated Page)的容器属性
*   内存中的数据通过综述页组成宏观的逻辑树。
*   它包含 `id`, `depth`, `timestamp`, `summary`, `keywords` 以及 `source_ids`（指向更深层的 Consolidated Pages 或 Original Pages）。
*   **递归嵌套 (Recursive Nesting)**：通过 `source_ids` 的递归嵌套支持，P-Mem 构建起 **无限递归变焦 (Infinite Recursive Zooming)** 的能力。这允许系统从最高维度的“项目主旨（Level 1）”，通过数次 PCP 的 `Consult` 指令，一直变焦下钻到“几个月前某一次聊天的特定结论（Level N）”，形成具有纵深的**逻辑树 (Logic Tree)**。

---

## 3. 系统架构同构：双引擎驱动 (Isomorphic Architecture)

P-Mem 将自身定义为一个拥有与 PCP 高度相似结构的**异步记忆管理器**，核心同样由 **Router** 和 **Consolidator** 构成：

### 3.1 独立神经寻址器 (Independent Memory Router)
传统的 RAG 系统往往使用单一的向量余弦相似度进行召回。P-Mem 的 Router 则对外暴露标准契约，并内部完全复刻 PCP 的两级匹配机制：

*   **调用契约 (Memory Interface)**：PCP 直接向 P-Mem 抛出当前的完整**意图提示词 (Intent Prompt)**。
*   **阶段一：关键词海选 (Keyword Broad Match)**
    *   P-Mem 的 Router 独立从 Intent 中提取/匹配 `Keywords`。
    *   基于 `keywords` 的高维索引，在 P-Mem 持久化空间内进行海量筛选，召回潜在相关的 Logical Pages。
*   **阶段二：综述精选 (Summary Precision Selection)**
    *   P-Mem 的 Router 利用其内置 LLM 能力，针对阶段一召回的候选 Pages 的 `summary` 与最初 PCP 发来的 `Intent Prompt` 进行深度对比评估。
    *   决定哪些 Pages 作为“激活记忆”，并格式化为标准的 XML `Node` 返回至 PCP。

### 3.2 记忆整理器 (Memory Consolidator)
PCP 中的 Consolidator 负责单次 Context 内部的话题整理。
P-Mem 中的 Consolidator 则是一个运行在**后台的长期 GC (Garbage Collection) / 演化进程**，它的逻辑拆分和总结规则与 PCP 内的 Consolidator **完全一致**。

*   **同构的水平合并 (Horizontal Merge)**：
    *   定期扫描 Memory 中新增的、`keywords` 相似且 `depth` 相同的 Pages。
    *   如果发现多个跨会话产生的零散记忆具有高相关性，Consolidator 会自动将它们合并为一个新的更高维度的 **Consolidated Page**，并把之前的零散页变为它的子节点 (`source_ids`)。
*   **同构的逻辑坍缩、遗忘与负面标定 (Logic Collapse, Fade & Negative Weighting)**：
    *   基于 `timestamp` (时间戳时间定标) 和被 Router 召回的频率，对过时、冗余的旧分支进行降权。
    *   **独立负向记忆标定系统**：提供专门的遗忘/免疫接口。注意，这个信号**并非**由 PCP 会话内的 `Purge` 触发（因为 Context 内的 Purge 只是表示“本次无关”）。当 PCP （未来可能引入扩展协议）明确判定某块知识在事实上发生长期错误时，定向向 P-Mem 下发指令，由 P-Mem 通过后台 Consolidator 对该记忆进行彻底剔除或施加永久负权。

---

## 4. 调用流转：PCP 与 P-Mem 之间的无缝对接 (Workflow)

由于两者底层组件与数据结构实现了 $100\%$ 的同构对齐，数据流转变得极度丝滑，仿佛是操作系统中的“缺页中断”处理：

1.  **意图抛出 (Intent Throwing)**：当前 PCP 会话抛出完整的 Intent Prompt 寻求记忆支撑。
2.  **缺页中断 (Page Fault)**：触发对外部 P-Mem 接口的独立调用。
3.  **独立寻址 (Independent Routing)**：P-Mem 本地 Router 独立执行两级匹配（Keyword -> Summary），格式化符合 PCP 规范的 `<Node>` 数组。
4.  **注入与执行 (Injection)**：这些携带绝对 `timestamp` 且 `origin="Storage"` 的记忆页被植入 PCP 上下文。PCP 仅将其视为带有历史标签的同构 Page。
5.  **回写与沉淀 (Write-back)**：PCP 导出总结资产，P-Mem 直接接收入库。
6.  **后台代谢与纠错 (Metabolism)**：P-Mem Consolidator 执行树型合并与定向错误剔除。

## 5. 结论

通过引入 P-Mem，我们将传统 LLM 的 **“对话应用层 - 向量数据库层”** 这个异构、充满摩擦的二元体系，彻底升级为了基于 **PCP 协议** 的 **“热缓存 (Context) - 冷存储 (Memory)”** 的同构内存管理体系。LLM 由此真正跨越了会话的生死边界，实现了认知的一致性延长。
