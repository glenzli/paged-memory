# Paged-Memory (P-Mem)

**Paged-Memory (P-Mem)** 是一个同构的跨上下文记忆系统，旨在抹平大语言模型（LLM）单次任务上下文（Context）与全局持久化记忆（Memory）之间的物理差异。

## 与 PCP 的关系

P-Mem 是 [Paged-Context-Protocol (PCP)](https://github.com/glenzli/paged-context-protocol) 在时间维度上的持久化映射层：

- **数据同构**：P-Mem 存储的实体完全遵循 PCP 定义的 **Logical Page (逻辑页)** 规范。
- **架构同构**：P-Mem 内部复刻了 PCP 的 **Router (寻址器)** 和 **Consolidator (整理器)** 机制，实现两级匹配和自动化的记忆演化。
- **无缝对接**：两者共享同一套数据结构和逻辑法则，使记忆的调取如同操作系统的“缺页中断”一样自然。

## 项目状态

🚀 **开发中 (Under Development)**

本仓库目前正处于活跃开发阶段，目标是实现完整的 P-Mem 系统。我们正在将 [DESIGN.md](./DESIGN.md) 中的构想转化为实际的代码实现，构建一个能够跨越会话边界、实现认知一致性延长的同构内存管理体系。

---

License: [MIT](./LICENSE)
