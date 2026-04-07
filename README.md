# kddRepo
## 常见 Agent 设计模式

现代语言模型驱动的 Agent 系统通常基于以下几种经典设计模式。每种模式在推理效率、任务分解能力、令牌消耗和可扩展性上各有侧重。

### 1. ReAct（Reasoning + Acting）
**核心思想**：将推理（Reasoning）与行动（Acting）交替进行，让模型在生成每一步动作的同时解释其思考过程。  
**工作流程**：
- 模型输出「思考 → 行动 → 观察」的循环。
- 思考：分析当前状态，规划下一步。
- 行动：调用工具或输出最终答案。
- 观察：接收外部环境返回的结果。  
**优势**：可解释性强，能动态调整计划，适合需要多步交互的任务。  
**局限**：长序列任务可能产生冗余步骤，令牌消耗较高。

### 2. Plan-and-Solve（PS）
**核心思想**：将任务分解为长期计划再执行，减少 ReAct 中因逐步决策导致的遗漏或错误。  
**两个主要阶段**：
- **制定计划**：将复杂任务拆解为一系列有序的子任务。
- **执行计划**：按顺序完成每个子任务，必要时可引入细化提示（如“Let’s think step by step”）。  
**优势**：提升推理质量，降低计算错误，适合有明确分解路径的任务（如数学题、多跳问答）。  
**变体**：PS+（Plan-and-Solve+）增加了更细致的规划指令。

### 3. ReWOO（Reasoning Without Observation）
**核心思想**：将推理与外部观察分离，通过一次计划生成完整的工具调用蓝图，大幅减少令牌消耗。  
**三个角色**：
- **Planner（计划者）**：生成一个包含多个相互依赖步骤的计划蓝图，每个步骤指明需要调用的工具和参数。
- **Worker（工作者）**：批量执行计划中的工具调用，收集真实证据（不进行推理）。
- **Solver（求解器）**：整合所有计划与证据，一次性生成最终答案。  
**优势**：令牌效率极高（减少重复推理），适合高成本工具调用或大规模并行任务。  
**注意**：要求模型具备较强的预先规划能力。

### 4. Reflection（反射 / 言语强化学习）
**核心思想**：通过语言反馈让 Agent 进行自我反思，并将反思结果存入情景记忆缓冲区，用于改进后续决策 —— 无需更新模型权重。  
**工作流程**：
- 执行轨迹：Agent 尝试完成任务并得到结果（成功或失败）。
- 生成反思：基于失败原因，模型生成自然语言的「反思」文本。
- 记忆存储：反思内容被保存到记忆模块中。
- 重试或迁移：在相同或相似任务中，Agent 先读取反思，避免重复错误。  
**优势**：类似强化学习但无需训练，可解释性强，适合长期交互场景。  
**经典应用**：代码调试、游戏求解、对话策略优化。

---

### 设计模式对比表

| Agentic Method | 核心特点 | 适用场景 | 令牌效率 | 链接 | 论文 |
|----------------|-----------|-----------|-----------|-------|------|
| **ReAct** | 推理与行动交替，动态交互 | 多步工具调用、Web 导航、游戏 | 中低 | [文章](https://mp.weixin.qq.com/s/jTnCCpWy_QU5w4FR32aC3Q) | [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) |
| **Plan-and-Solve (PS)** | 先规划后执行，减少错误 | 数学推理、长文本问答、任务分解 | 中 | [文章](https://mp.weixin.qq.com/s/cIcODBeVXzE7DwEfvJ5_jA) | [Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning](https://arxiv.org/abs/2305.04091) |
| **ReWOO** | 计划-执行-求解分离，极低令牌 | 大批量工具调用、API 密集型任务 | 高 | [文章](https://mp.weixin.qq.com/s/F-xLzmukmuqCdWNWEB8VFA) | [ReWOO: Decoupling Reasoning from Observations for Efficient Augmented Language Models](https://arxiv.org/abs/2305.18323) |
| **Reflection** | 自我反思 + 情景记忆，无需训练 | 持续学习、代码调试、对话策略优化 | 中高 | [文章](https://mp.weixin.qq.com/s/Ik9sIDcwaorurBsoeVFVRg) | [Reflexion: Language Agents with Verbal Reinforcement Learning](https://arxiv.org/abs/2303.11366) |

> **注意**：ReWOO 的微信文章链接暂缺，建议自行搜索或替换为官方博客/教程。论文链接已提供。

---

### 扩展阅读与选型建议

- **如果你需要强可解释性 + 动态调整** → 选择 **ReAct**  
- **如果你的任务可以提前规划为线性子任务** → 选择 **Plan-and-Solve**  
- **如果你追求极低令牌成本 + 大批量工具调用** → 选择 **ReWOO**  
- **如果你希望 Agent 能从错误中持续改进而不重新训练** → 选择 **Reflection**

这些模式可以混合使用（例如 ReAct + Reflection 记忆），根据实际场景灵活组合。
