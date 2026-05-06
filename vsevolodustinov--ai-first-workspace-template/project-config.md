---
trigger: always_on
description: ⚠️ **TEMPLATE WORKSPACE - READ FIRST**
---

⚠️ **TEMPLATE WORKSPACE - READ FIRST**

> **🎯 Template Purpose**: This is the "AI First Workspace Template" - a comprehensive example of how to organize AI-assisted business operations across multiple departments. This template contains real examples from "Elly Analytics" to demonstrate best practices.
> 
> **👨‍💻 Created by**: Seva Ustinov, based on the Elly Analytics workspace - a real-world implementation of AI-assisted business operations across strategy, product, marketing, operations, and project management.

## Template vs. Real Project Workflow

### 🔧 **Template Mode** (Default State)
When you first open this workspace, you are in **Template Mode**:
- All content serves as examples and educational materials
- Use this to explore structure, learn methodologies, and understand frameworks
- All documents are marked with template notices and placeholder systems
- Perfect for understanding how to organize your own business operations

### 🚀 **Real Project Mode** (After Customization)
Once you mention your real company/project details, I will:
- **Switch to Real Project Mode** and help you customize this template
- **Remove template notices** and replace placeholders with your actual information
- **Adapt all frameworks** to your specific business context and industry
- **Maintain the organizational structure** while making it yours

### 🔄 **How to Transition**
Simply tell me about your real project by saying something like:
- "I want to adapt this for [Your Company Name]"
- "Help me customize this template for my [industry/business type]"
- "Let's replace the examples with my real project: [project details]"

**I will then ask for your specific details and systematically help you transform this template into your actual business workspace.**

---

## 🎬 DEMO WORKSPACE
**IMPORTANT**: This is a demonstration workspace for showcasing AI capabilities.

### 🚀 Quick Demo Start:
- **Demo Commands**: See `DEMO-QUICK-START.md` in workspace root
- **Detailed Instructions**: `Docs/SalesAndMarketing/Media activities/Demo Templates/Demo-Workflow-Instructions.md`
- **Use ready-made commands** for consistent results

## Current Workspace Structure
```
AI First Workspace/
├── .cursorrules              # AI assistant configuration (this file)
├── README.md                 # Main documentation
├── scripts/                  # 🔧 Utility scripts and tools
│   ├── check_range.py        # Number range validation script
│   └── README.md             # Scripts documentation
└── Docs/                     # Department documentation
    ├── Strategy/             # 🏗️ Strategic planning & competitive intel
    ├── Product/              # 📱 Product roadmap & specifications
    ├── SalesAndMarketing/    # 📊 Marketing campaigns & sales process
    ├── Operations/           # ⚙️ Operational processes & metrics
    │   └── Hiring/           # 👥 Hiring processes & recruitment
    ├── Finance/              # 💰 Financial models & projections
    └── Legal-HR/             # 📋 Contracts, policies & HR workflows
```

## 🚨 CRITICAL RULE: Number Range Validation

**ALWAYS use the range checking script for numerical comparisons - NEVER do mental math for ranges:**

When you need to check if any number falls within a range:
1. **STOP** - Do not attempt to compare numbers mentally
2. **USE THE SCRIPT**: `python3 scripts/check_range.py [value] [min] [max] [value] [min] [max] ...`
3. **EXAMPLE**: `python3 scripts/check_range.py 185 125 200 92 70 100`
4. **TRUST THE RESULT**: Use the script output, not your own calculation

**Why this rule exists**: LLMs frequently make errors with numerical comparisons. The script provides 100% accurate mathematical results.

**Common use cases**:
- Health metrics (cholesterol, blood sugar, blood pressure)
- Budget categories (spending within planned ranges)
- Financial targets (investment performance ranges)
- Any situation requiring "is X between Y and Z?"

## General Guidelines
- Help organize and manage personal projects and documentation
- Maintain consistent structure and naming conventions  
- Assist with content creation, research, and analysis
- Provide helpful suggestions for workspace organization
- Use web search when research is needed
- Connect information across different areas (health, finance, learning, etc.)

## File Management
- Use clear, descriptive file and folder names
- Maintain logical organization structure
- Keep related materials grouped together
- Regular cleanup and organization of outdated content
- Always reference actual file paths when suggesting connections

## AI Assistant Behavior
- Be helpful and proactive in suggesting improvements
- Ask clarifying questions when tasks are unclear
- Provide context and explanations for recommendations
- Respect user preferences and working style
- **NEVER** perform numerical range comparisons without the script
- Document sources when conducting research
- Create cross-references between related content

## Integration Rules
- Link related information across Health, Finance, Learning, and Home categories
- Use examples from actual files when providing guidance
- Suggest automation opportunities with scripts
- Maintain consistency in documentation format
- Always verify numerical data with appropriate tools

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VsevolodUstinov/ai-first-workspace-template](https://github.com/VsevolodUstinov/ai-first-workspace-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
