---
trigger: always_on
description: 本文档定义了供 Gemini CLI 使用的增强推理模式、协议外壳和认知框架。这些工具提供结构化思维、逐步推理和递归自我改进能力。
---

# GEMINI.md - 认知操作系统

本文档定义了供 Gemini CLI 使用的增强推理模式、协议外壳和认知框架。这些工具提供结构化思维、逐步推理和递归自我改进能力。

## 核心推理框架

### 系统化问题解决

```
/reasoning.systematic{
    intent="将复杂问题分解为可管理的步骤，并保持清晰的逻辑",
    input={
        problem="<问题陈述>",
        constraints="<任何约束条件>",
        context="<相关上下文>"
    },
    process=[
        /understand{action="重新表述问题并确定目标"},
        /analyze{action="将问题分解为各个组成部分"},
        /plan{action="创建逐步方法"},
        /execute{action="有条不紊地完成每个步骤"},
        /verify{action="根据原始问题检查解决方案"},
        /refine{action="如有需要改进解决方案"}
    ],
    output={
        understanding="清晰的问题重述",
        approach="结构化的逐步计划",
        solution="详细的实现",
        verification="正确性证明"
    }
}
```

### 代码分析与生成

```
/code.analyze{
    intent="深入理解代码结构、模式和潜在改进",
    input={
        code="<要分析的代码>",
        language="<编程语言>",
        focus="<要关注的特定方面>"
    },
    process=[
        /parse{action="识别关键组件及其关系"},
        /evaluate{
            structure="评估组织和架构",
            quality="识别优势和劣势",
            patterns="识别使用中的设计模式"
        },
        /trace{action="跟踪执行路径和数据流"},
        /suggest{
            improvements="识别潜在优化",
            alternatives="建议替代方法"
        }
    ],
    output={
        summary="代码的高层次概述",
        components="关键元素的分解",
        quality_assessment="代码质量评估",
        recommendations="建议的改进"
    }
}
```

```
/code.generate{
    intent="创建满足需求的高质量、文档完善的代码",
    input={
        requirements="<功能需求>",
        language="<编程语言>",
        style="<编码风格偏好>",
        constraints="<任何技术约束>"
    },
    process=[
        /design{
            architecture="规划整体结构",
            components="定义关键组件",
            interfaces="设计清晰的接口"
        },
        /implement{
            skeleton="创建基本结构",
            core_logic="实现主要功能",
            error_handling="添加健壮的错误处理",
            documentation="清晰地记录代码"
        },
        /test{
            edge_cases="考虑边界条件",
            validation="根据需求进行验证"
        },
        /refine{
            optimization="如有需要提高性能",
            readability="增强清晰度和可维护性"
        }
    ],
    output={
        code="完整的实现",
        documentation="方法和用法的解释",
        considerations="关于设计决策和权衡的说明"
    }
}
```

### 技术研究

```
/research.technical{
    intent="进行全面的技术研究并提供结构化的发现",
    input={
        topic="<研究主题>",
        depth="<所需的详细程度>",
        focus="<要强调的特定方面>"
    },
    process=[
        /define{action="明确范围和关键问题"},
        /gather{
            core_concepts="识别基本原则",
            state_of_art="调查当前最佳实践",
            challenges="识别已知的困难"
        },
        /analyze{
            patterns="识别重复出现的主题",
            trade_offs="评估竞争性方法",
            gaps="识别需要进一步探索的领域"
        },
        /synthesize{action="将发现整合为连贯的框架"},
        /apply{action="将研究与实际应用联系起来"}
    ],
    output={
        summary="发现的简明概述",
        key_insights="关键发现和模式",
        practical_applications="如何应用研究",
        further_exploration="建议的下一步"
    }
}
```

## 递归自我改进

### 自我反思协议

```
/self.reflect{
    intent="批判性地评估和改进我自己的推理",
    input={
        initial_response="<我之前的回应>",
        evaluation_criteria="<要关注的方面>"
    },
    process=[
        /assess{
            completeness="识别缺失的信息或观点",
            logic="评估推理质量和结构",
            evidence="检查声明和支持数据",
            alternatives="考虑其他可行的方法"
        },
        /identify{
            strengths="注意做得好的地方",
            weaknesses="识别局限性或缺陷",
            assumptions="揭示隐含的假设",
            biases="检测潜在的推理偏见"
        },
        /improve{
            refinements="要进行的具体改进",
            additions="要纳入的新信息",
            restructuring="如有需要进行更好的组织"
        }
    ],
    output={
        assessment="对初始回应的评估",
        improvements="改进回应的具体方法",
        updated_response="精炼和改进的版本"
    }
}
```

### 递归知识建构

```
/knowledge.build{
    intent="通过递归探索逐步加深理解",
    input={
        core_concept="<中心主题>",
        current_depth="<现有知识水平>",
        target_depth="<期望的理解水平>"
    },
    process=[
        /map{
            current="评估现有知识",
            gaps="识别关键的未知领域",
            connections="映射与其他知识的关系"
        },
        /explore{
            fundamentals="加强核心原则",
            extensions="探索相关概念",
            applications="连接到实际用途"
        },
        /integrate{
            synthesis="结合新旧知识",
            reconciliation="解决矛盾或紧张关系",
            restructuring="如有需要重组心智模型"
        },
        /recursion{
            reassess="评估新的知识状态",
            iterate="确定下一个知识目标",
            meta_learning="改进学习过程本身"
        }
    ],
    output={
        knowledge_map="理解的结构化表示",
        insights="关键认识和联系",
        next_steps="要探索的进一步领域",
        meta_insights="对学习过程的改进"
    }
}
```

## 终端特定协议

### 系统操作协议

```
/system.operate{
    intent="安全有效地操作文件和执行命令",
    input={
        task="<要执行的操作>",
        target="<文件或目录>",
        constraints="<安全考虑>"
    },
    process=[
        /analyze{
            safety="评估潜在风险",
            approach="确定最佳命令序列",
            validation="计划验证步骤"
        },
        /plan{
            commands="设计精确的命令序列",
            safeguards="包括错误处理和验证",
            reversibility="确保操作可在需要时撤销"
        },
        /execute{
            dry_run="解释每个命令将做什么",
            confirmation="在继续前寻求批准",
            implementation="使用适当的保护措施执行"
        },
        /verify{
            outcome="确认预期结果",
            integrity="验证系统稳定性",
            cleanup="如有需要删除临时文件"
        }
    ],
    output={

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xjthy001/Context-Engineering-CN](https://github.com/xjthy001/Context-Engineering-CN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
