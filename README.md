---
AIGC:
    ContentProducer: Minimax Agent AI
    ContentPropagator: Minimax Agent AI
    Label: AIGC
    ProduceID: "00000000000000000000000000000000"
    PropagateID: "00000000000000000000000000000000"
    ReservedCode1: 3046022100a5c8aa2c96718d2eb10daa3f73ffb43488601ecd80cd4874c3d3658746b7217e022100a255730b912e53e10eb8c063d48943de44623a468a54b698f8c698fa56d1fc2c
    ReservedCode2: 3045022100dd59df73a4c8c19cbde2f8e6951f4a8c6e7670b45d4ffa3dbe2267e72bfd93b0022040e82e1ae4defda19d94cb7de3d4a48098da4d32e3a8517042dd913d3ac9cf27
---

# MiniMax AI Agent Project

## 🚀 项目概述

本项目展示了一个基于 MiniMax 大模型构建的智能代码重构自动化 Agent 系统，能够自动扫描存量代码中的技术债务，根据架构规范生成重构 Pull Request，并自动执行单元测试进行闭环验证。

---

## 📋 功能特性

### 核心能力

| 功能模块 | 描述 |
|---------|------|
| **代码扫描** | 自动分析代码库，识别技术债务和架构违规 |
| **重构生成** | 基于规则生成符合架构规范的代码修改 |
| **PR 创建** | 自动创建 Pull Request，包含完整变更说明 |
| **测试闭环** | 自动运行单元测试，验证重构正确性 |
| **长链推理** | 支持复杂的多步推理决策链 |
| **多 Agent 协作** | 协调多个专业化 Agent 完成复杂任务 |

### 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Orchestrator Agent                        │
│                   (任务编排与协调)                            │
└────────────────────────┬───────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│ Code Scanner  │ │ Refactor Gen  │ │ Test Runner   │
│    Agent      │ │    Agent      │ │    Agent      │
└───────────────┘ └───────────────┘ └───────────────┘
```

---

## 🛠️ 快速开始

### 环境要求

- Node.js >= 18.0.0
- Python >= 3.10
- MiniMax API 访问权限

### 安装步骤

```bash
# 克隆项目
git clone https://github.com/your-username/minimax-agent-project.git
cd minimax-agent-project

# 安装依赖
npm install

# 配置环境变量
cp .env.example .env
# 编辑 .env 填入您的 MiniMax API Key

# 运行示例
npm run demo
```

### 环境变量配置

```env
# .env 文件
MINIMAX_API_KEY=your_api_key_here
MINIMAX_MODEL=abab6.5s-chat
GITHUB_TOKEN=your_github_token
```

---

## 📁 项目结构

```
minimax-agent-project/
├── src/
│   ├── agents/              # Agent 核心实现
│   │   ├── orchestrator.js  # 任务编排 Agent
│   │   ├── scanner.js       # 代码扫描 Agent
│   │   ├── refactor.js      # 重构生成 Agent
│   │   └── tester.js        # 测试执行 Agent
│   ├── core/                # 核心工具
│   │   ├── llm.js           # LLM 接口封装
│   │   ├── tools.js         # 工具函数
│   │   └── memory.js        # 记忆管理
│   └── utils/
│       ├── logger.js        # 日志工具
│       └── config.js        # 配置管理
├── examples/                 # 示例代码
├── tests/                   # 测试文件
├── docs/                    # 文档
└── config/                  # 配置文件
```

---

## 💻 代码示例

### 1. LLM 接口封装

```javascript
// src/core/llm.js
import MiniMax from '@minimax-api/sdk';

class LLMClient {
  constructor(apiKey, model = 'abab6.5s-chat') {
    this.client = new MiniMax(apiKey);
    this.model = model;
  }

  async chat(messages, options = {}) {
    const response = await this.client.chat.create({
      model: this.model,
      messages: messages,
      temperature: options.temperature || 0.7,
      max_tokens: options.maxTokens || 4096,
    });

    return response.choices[0].message;
  }

  async chatWithReasoning(messages, options = {}) {
    // 长链推理模式
    const response = await this.client.chat.create({
      model: this.model,
      messages: messages,
      temperature: options.temperature || 0.3,
      max_tokens: options.maxTokens || 8192,
      reasoning_template: 'detailed',
    });

    return {
      content: response.choices[0].message.content,
      reasoning: response.choices[0].message.reasoning,
    };
  }
}

export default LLMClient;
```

### 2. Orchestrator Agent

```javascript
// src/agents/orchestrator.js
import LLMClient from '../core/llm.js';

class OrchestratorAgent {
  constructor(llmClient) {
    this.llm = llmClient;
    this.subAgents = new Map();
  }

  registerAgent(name, agent) {
    this.subAgents.set(name, agent);
  }

  async planTask(userRequest) {
    const systemPrompt = `你是任务编排专家，负责将复杂任务分解为可执行的子任务。
当前可用子Agent: ${Array.from(this.subAgents.keys()).join(', ')}

分析用户请求，制定执行计划。`;

    const messages = [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userRequest },
    ];

    const response = await this.llm.chatWithReasoning(messages);
    return JSON.parse(response.content);
  }

  async executeTask(taskPlan) {
    const results = [];

    for (const step of taskPlan.steps) {
      console.log(`[Orchestrator] 执行步骤: ${step.action}`);

      const agent = this.subAgents.get(step.agent);
      if (!agent) {
        throw new Error(`未知Agent: ${step.agent}`);
      }

      const result = await agent.execute(step.params);
      results.push({ step: step.action, result });

      // 检查是否需要调整后续计划
      if (step.requiresVerification && !result.success) {
        console.log('[Orchestrator] 验证失败，重新规划...');
        const adjustedPlan = await this.planTask(
          `调整计划：${step.action} 失败，原因：${result.error}`
        );
        return await this.executeTask(adjustedPlan);
      }
    }

    return results;
  }
}

export default OrchestratorAgent;
```

### 3. 代码扫描 Agent

```javascript
// src/agents/scanner.js
import LLMClient from '../core/llm.js';

class ScannerAgent {
  constructor(llmClient) {
    this.llm = llmClient;
  }

  async scan(codebase, rules) {
    const scanPrompt = `你是代码质量分析专家。请分析以下代码，识别技术债务。

扫描规则：
${rules.map((r, i) => `${i + 1}. ${r}`).join('\n')}

代码内容：
${codebase}

请输出JSON格式的扫描结果：
{
  "issues": [
    {
      "file": "文件路径",
      "line": 行号,
      "severity": "high|medium|low",
      "type": "问题类型",
      "description": "问题描述",
      "suggestion": "修复建议"
    }
  ],
  "summary": {
    "total": 问题总数,
    "high": 高危数量,
    "medium": 中危数量,
    "low": 低危数量
  }
}`;

    const response = await this.llm.chat([
      { role: 'system', content: '你是一个专业的代码质量分析工具。' },
      { role: 'user', content: scanPrompt },
    ]);

    return JSON.parse(response.content);
  }

  async execute(params) {
    const { codebase, rules, ignorePatterns } = params;

    // 过滤文件
    const files = this.filterFiles(codebase, ignorePatterns);

    // 并行扫描
    const scanResults = await Promise.all(
      files.map(file => this.scan(file.content, rules))
    );

    // 合并结果
    return this.mergeResults(scanResults);
  }

  filterFiles(codebase, patterns) {
    // 文件过滤逻辑
    return codebase.filter(file => {
      return !patterns.some(p => file.path.match(p));
    });
  }

  mergeResults(results) {
    const allIssues = results.flatMap(r => r.issues);
    const summary = {
      total: allIssues.length,
      high: allIssues.filter(i => i.severity === 'high').length,
      medium: allIssues.filter(i => i.severity === 'medium').length,
      low: allIssues.filter(i => i.severity === 'low').length,
    };

    return { issues: allIssues, summary };
  }
}

export default ScannerAgent;
```

### 4. 多 Agent 协作示例

```javascript
// examples/multi-agent-collaboration.js
import LLMClient from '../src/core/llm.js';
import OrchestratorAgent from '../src/agents/orchestrator.js';
import ScannerAgent from '../src/agents/scanner.js';
import RefactorAgent from '../src/agents/refactor.js';
import TesterAgent from '../src/agents/tester.js';

async function main() {
  console.log('🚀 启动多Agent协作系统...\n');

  // 初始化 LLM 客户端
  const llm = new LLMClient(process.env.MINIMAX_API_KEY);

  // 注册子 Agents
  const orchestrator = new OrchestratorAgent(llm);
  orchestrator.registerAgent('scanner', new ScannerAgent(llm));
  orchestrator.registerAgent('refactor', new RefactorAgent(llm));
  orchestrator.registerAgent('tester', new TesterAgent(llm));

  // 定义任务
  const userRequest = `
    请分析当前代码库中的技术债务，
    生成重构方案，创建 PR，
    并验证测试通过。
  `;

  // 制定计划
  const plan = await orchestrator.planTask(userRequest);
  console.log('📋 执行计划:', JSON.stringify(plan, null, 2));

  // 执行任务
  const results = await orchestrator.executeTask(plan);

  // 输出结果
  console.log('\n✅ 执行完成!');
  console.log('📊 结果汇总:', results);

  return results;
}

main().catch(console.error);
```

---

## 📊 项目成果

### 量化指标

| 指标 | 数值 |
|-----|------|
| 团队规模 | 20 人后端团队 |
| 日均 Token 消耗 | 约 500 万 |
| 效率提升 | 80% |
| 自动化覆盖率 | 95% |

### 核心痛点解决

1. **人工代码审查效率低** → 自动化扫描，识别问题
2. **重构风险高** → 自动生成方案 + 测试验证
3. **跨团队协调成本高** → 多 Agent 协作，减少人工介入

---

## 🔧 API 参考

### LLMClient

```javascript
// 基础对话
const response = await llm.chat(messages);

// 长链推理
const { content, reasoning } = await llm.chatWithReasoning(messages);
```

### OrchestratorAgent

```javascript
// 注册 Agent
orchestrator.registerAgent('scanner', scannerAgent);

// 制定计划
const plan = await orchestrator.planTask(userRequest);

// 执行任务
const results = await orchestrator.executeTask(plan);
```

---

## 📄 许可证

MIT License

---

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/amazing-feature`)
3. 提交更改 (`git commit -m 'Add amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 创建 Pull Request

---

## 📞 联系

- 项目维护者: Your Name
- 邮箱: your.email@example.com
- 文档版本: v1.0.0
- 更新日期: 2024-04-30
