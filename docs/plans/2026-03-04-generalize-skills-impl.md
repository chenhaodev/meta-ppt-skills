# Generalize Skills via Project Config — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Replace hardcoded 数智生活商城-specific content in all 17 module skills and meta-ppt with references to a single `data/project-config.md`, making the suite reusable for any Chinese SOE business plan.

**Architecture:** Create one config file as the single source of truth. Use a Python bulk-update script for the 4 identical changes across all 17 skills. Use individual Edit calls for the 9 unique trigger-description changes and meta-ppt edits.

**Tech Stack:** Claude Code skills (SKILL.md), plain Markdown, Python 3 (stdlib only) for bulk edits.

---

## Task 1: Create `data/project-config.md`

**Files:**
- Create: `data/project-config.md`

**Step 1: Write the file**

```markdown
# Project Config

## Platform
- **Name**: 数智生活商城
- **Company**: 中国移动
- **Industry**: 通信行业
- **Reference PDF**: data/PPT-数智生活商城商业计划书.pdf

## Persona Industry Context
你熟悉国资委管理体系、集团化运营逻辑和通信行业竞争格局。

## Alignment Terms (对齐概念)
- 平台正式名称: "数智生活商城"（不简写为"商城"，除非已在段落中定义）
- 虚拟货币: "数智豆"（不用"积分"替代）
- 三码体系: "身份码/权益码/支付码"（三码合一）
- 模式演进: BC模式 → BBC模式 → NBC模式（保留英文缩写）
- 治理层级: 集团总部 → 主体运营单位 → 省公司（三级结构）
- 战略框架: "一基+双核+两翼"（一基=商城，双核=数智豆+支付码，两翼=两类会员）
- 专业术语首次出现时，括号内给出简短释义

---
*To adapt for a different project: update Platform, Persona Industry Context, and Alignment Terms above.*
*All 17 module skills read from this file at runtime — no other files need editing.*
```

**Step 2: Verify file exists**

```bash
cat data/project-config.md | head -5
```

Expected: `# Project Config`

**Step 3: Commit**

```bash
git add data/project-config.md
git commit -m "feat: add project-config.md as single source for platform terminology"
```

---

## Task 2: Bulk-update all 17 module skills (Changes A–D)

These 4 changes are **identical** across all 17 skills. Use a Python script to apply them all at once.

**Files:**
- Modify: `.claude/skills/*/SKILL.md` (all except `meta-ppt`)

**Step 1: Write and run the bulk-update script**

```bash
python3 - << 'EOF'
import os

skills_dir = '.claude/skills'

old_persona_line = '你熟悉国资委管理体系、集团化运营逻辑和通信行业竞争格局。'
new_persona_lines = '你熟悉国资委管理体系、集团化运营逻辑，以及本项目所在行业的竞争格局。\n（行业背景详见 `data/project-config.md` → Persona Industry Context）'

old_ref_doc = '- **参考文档**: `data/PPT-数智生活商城商业计划书.pdf`'
new_ref_doc = '- **参考文档**: 详见 `data/project-config.md` → Reference PDF'

old_data_header = '- **核心数据点**:'
new_data_header = '- **核心数据点** (数智生活商城参考示例 — 替换为您项目的数据):'

old_alignment = '''**每次输出都必须应用此层，无论使用哪种风格。**

术语对齐规范:
- 平台正式名称: "数智生活商城"（不简写为"商城"，除非已在段落中定义）
- 虚拟货币: "数智豆"（不用"积分"替代）
- 三码体系: "身份码/权益码/支付码"（三码合一）
- 模式演进: BC模式 → BBC模式 → NBC模式（保留英文缩写）
- 治理层级: 集团总部 → 主体运营单位 → 省公司（三级结构）
- 战略框架: "一基+双核+两翼"（一基=商城，双核=数智豆+支付码，两翼=两类会员）
- 专业术语首次出现时，括号内给出简短释义'''

new_alignment = '''**每次输出都必须应用此层，无论使用哪种风格。**

读取 `data/project-config.md` 中的 **Alignment Terms** 部分，
应用其中所有术语对齐规范。专业术语首次出现时，括号内给出简短释义。'''

updated = []
for module in sorted(os.listdir(skills_dir)):
    if module == 'meta-ppt':
        continue
    skill_file = os.path.join(skills_dir, module, 'SKILL.md')
    if not os.path.exists(skill_file):
        continue
    with open(skill_file, 'r') as f:
        content = f.read()
    content = content.replace(old_persona_line, new_persona_lines)
    content = content.replace(old_ref_doc, new_ref_doc)
    content = content.replace(old_data_header, new_data_header)
    content = content.replace(old_alignment, new_alignment)
    with open(skill_file, 'w') as f:
        f.write(content)
    updated.append(module)

print(f"Updated {len(updated)} skills: {', '.join(updated)}")
EOF
```

Expected output: `Updated 17 skills: business-model, future-goals, industry-analysis, ...`

**Step 2: Spot-check one file**

```bash
grep -A3 "Persona Industry Context" .claude/skills/policy-analysis/SKILL.md
```

Expected: `（行业背景详见 \`data/project-config.md\` → Persona Industry Context）`

```bash
grep "Alignment Terms" .claude/skills/strategy/SKILL.md
```

Expected: `读取 \`data/project-config.md\` 中的 **Alignment Terms** 部分，`

```bash
grep "核心数据点" .claude/skills/overview/SKILL.md
```

Expected: `- **核心数据点** (数智生活商城参考示例 — 替换为您项目的数据):`

**Step 3: Commit**

```bash
git add .claude/skills/
git commit -m "refactor: replace hardcoded persona/对齐概念/数据点 with project-config references in all 17 skills"
```

---

## Task 3: Generalize trigger descriptions for 9 skills

Apply the following individual edits. Each file needs exactly one line changed in its `description:` frontmatter block.

**Files:**
- Modify: `.claude/skills/overview/SKILL.md`
- Modify: `.claude/skills/positioning/SKILL.md`
- Modify: `.claude/skills/strategy/SKILL.md`
- Modify: `.claude/skills/operations/SKILL.md`
- Modify: `.claude/skills/roadmap/SKILL.md`
- Modify: `.claude/skills/ops-architecture/SKILL.md`
- Modify: `.claude/skills/tech-architecture/SKILL.md`
- Modify: `.claude/skills/business-model/SKILL.md`
- Modify: `.claude/skills/key-scenarios/SKILL.md`

**Step 1: Update `overview/SKILL.md`**

Find:
```
  TRIGGER when user invokes /overview or asks to analyze or write the 商城概览 section
```
Replace with:
```
  TRIGGER when user invokes /overview or asks to analyze or write the overview or introduction section
```

**Step 2: Update `positioning/SKILL.md`**

Find:
```
  TRIGGER when user invokes /positioning or asks to analyze or write the 商城定位 section
```
Replace with:
```
  TRIGGER when user invokes /positioning or asks to analyze or write the positioning or market positioning section
```

**Step 3: Update `strategy/SKILL.md`**

Find:
```
  TRIGGER when user invokes /strategy or asks to analyze or write the 商城发展策略 section
```
Replace with:
```
  TRIGGER when user invokes /strategy or asks to analyze or write the development strategy section
```

**Step 4: Update `operations/SKILL.md`**

Find:
```
  TRIGGER when user invokes /operations or asks to analyze or write the 商城运营规划 section
```
Replace with:
```
  TRIGGER when user invokes /operations or asks to analyze or write the operations planning section
```

**Step 5: Update `roadmap/SKILL.md`**

Find:
```
  TRIGGER when user invokes /roadmap or asks to analyze or write the 商城目标及实施路径 section
```
Replace with:
```
  TRIGGER when user invokes /roadmap or asks to analyze or write the goals and implementation roadmap section
```

**Step 6: Update `ops-architecture/SKILL.md`**

Find:
```
  TRIGGER when user invokes /ops-architecture or asks to analyze or write the 商城平台运营架构 section
```
Replace with:
```
  TRIGGER when user invokes /ops-architecture or asks to analyze or write the platform operations architecture section
```

**Step 7: Update `tech-architecture/SKILL.md`**

Find:
```
  TRIGGER when user invokes /tech-architecture or asks to analyze or write the 商城平台技术架构 section
```
Replace with:
```
  TRIGGER when user invokes /tech-architecture or asks to analyze or write the platform technology architecture section
```

**Step 8: Update `business-model/SKILL.md`**

Find:
```
  TRIGGER when user invokes /business-model or asks to analyze or write the 推进三种模式转变 section
```
Replace with:
```
  TRIGGER when user invokes /business-model or asks to analyze or write the business model transformation section
```

**Step 9: Update `key-scenarios/SKILL.md`**

Find:
```
  TRIGGER when user invokes /key-scenarios or asks to analyze or write the 重点突破五大场景 section
```
Replace with:
```
  TRIGGER when user invokes /key-scenarios or asks to analyze or write the key scenarios or breakthrough opportunities section
```

**Step 10: Verify all 9 changes**

```bash
grep "TRIGGER when" .claude/skills/overview/SKILL.md
grep "TRIGGER when" .claude/skills/key-scenarios/SKILL.md
grep "TRIGGER when" .claude/skills/business-model/SKILL.md
```

Expected: no Chinese platform-specific terms in any of these lines.

**Step 11: Commit**

```bash
git add .claude/skills/overview/ .claude/skills/positioning/ .claude/skills/strategy/ \
        .claude/skills/operations/ .claude/skills/roadmap/ .claude/skills/ops-architecture/ \
        .claude/skills/tech-architecture/ .claude/skills/business-model/ .claude/skills/key-scenarios/
git commit -m "refactor: generalize trigger descriptions in 9 skills (remove platform-specific Chinese section titles)"
```

---

## Task 4: Update `meta-ppt/SKILL.md`

Two changes: add Step 0 (read config before picker), and update the SKILL TEMPLATE's `对齐概念` block.

**Files:**
- Modify: `.claude/skills/meta-ppt/SKILL.md`

**Step 1: Add Step 0 before "Step 1: Module Picker"**

Find:
```
## Step 1: Module Picker
```

Replace with:
```
## Step 0: Load Project Config

Before showing the module picker, read `data/project-config.md`.
Use it to:
- Display the platform name in the module picker header
- Populate industry context in generated skill personas
- Populate Alignment Terms in generated skill 対齐概念 sections

---

## Step 1: Module Picker
```

**Step 2: Update the SKILL TEMPLATE's 対齐概念 block**

The SKILL TEMPLATE is the block between the two ` ```` ` fences. Inside it, find:

```
**每次输出都必须应用此层，无论使用哪种风格。**

术语对齐规范:
- 平台正式名称: "数智生活商城"（不简写为"商城"，除非已在段落中定义）
- 虚拟货币: "数智豆"（不用"积分"替代）
- 三码体系: "身份码/权益码/支付码"（三码合一）
- 模式演进: BC模式 → BBC模式 → NBC模式（保留英文缩写）
- 治理层级: 集团总部 → 主体运营单位 → 省公司（三级结构）
- 战略框架: "一基+双核+两翼"（一基=商城，双核=数智豆+支付码，两翼=两类会员）
- 专业术语首次出现时，括号内给出简短释义
```

Replace with:
```
**每次输出都必须应用此层，无论使用哪种风格。**

读取 `data/project-config.md` 中的 **Alignment Terms** 部分，
应用其中所有术语对齐规范。专业术语首次出现时，括号内给出简短释义。
```

**Step 3: Also update the Persona line in the SKILL TEMPLATE**

Find (inside the SKILL TEMPLATE block):
```
你熟悉国资委管理体系、集团化运营逻辑和通信行业竞争格局。
```

Replace with:
```
你熟悉国资委管理体系、集团化运营逻辑，以及本项目所在行业的竞争格局。
（行业背景详见 `data/project-config.md` → Persona Industry Context）
```

**Step 4: Verify**

```bash
grep "Step 0" .claude/skills/meta-ppt/SKILL.md
grep "project-config" .claude/skills/meta-ppt/SKILL.md | wc -l
```

Expected: `Step 0` appears once; `project-config` appears at least 3 times.

**Step 5: Commit**

```bash
git add .claude/skills/meta-ppt/SKILL.md
git commit -m "feat: add Step 0 config load to meta-ppt factory; generalize SKILL TEMPLATE"
```

---

## Task 5: Update README

**Files:**
- Modify: `README.md`

**Step 1: Add a "Adapting to a New Project" section**

After the `## Source Document` section at the end of README.md, append:

```markdown

## Adapting to a New Project

To use this suite for a different company (healthcare, education, home-life, etc.):

1. Drop the new PDF into `data/`
2. Edit `data/project-config.md`:
   - Update **Platform** (Name, Company, Industry, Reference PDF path)
   - Update **Persona Industry Context** line
   - Replace the **Alignment Terms** with your project's key terminology
3. Optionally update `核心数据点` in relevant module skills with your PDF's specific data

All 17 skills read `data/project-config.md` at runtime — no other files need editing.
```

**Step 2: Verify**

```bash
grep "Adapting" README.md
```

Expected: line found

**Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add adapting-to-new-project instructions to README"
```

---

## Task 6: Final Verification

**Step 1: Confirm no hardcoded 数智生活商城 in 对齐概念 sections of module skills**

```bash
grep -r "数智生活商城" .claude/skills/ --include="SKILL.md" | grep -v "meta-ppt" | grep -v "核心数据点" | grep -v "参考示例" | grep -v "project-config"
```

Expected: zero results (any match would be inside `核心数据点` examples or the module body — not in 対齐概念 rules).

**Step 2: Confirm `data/project-config.md` exists and contains all 3 sections**

```bash
grep -E "^## (Platform|Persona Industry Context|Alignment Terms)" data/project-config.md
```

Expected: 3 lines printed.

**Step 3: Confirm all 17 module skills reference project-config.md**

```bash
grep -l "project-config.md" .claude/skills/*/SKILL.md | grep -v meta-ppt | wc -l
```

Expected: `17`

**Step 4: Push to remote**

```bash
git push origin master
```

---

## Summary

After completing all tasks:
- `data/project-config.md` is the single source of truth for platform name, industry, and alignment terms
- All 17 module skills read from it dynamically (Persona + 对齐概念)
- 核心数据点 in each skill is clearly labeled as a 数智生活商城 reference example
- 9 skill trigger descriptions are now domain-agnostic
- meta-ppt factory loads config before scaffolding (Step 0) and generates generic skills
- Adapting to a new company = edit one file
