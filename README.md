# meta-ppt-skills

A suite of Claude Code expert skills for Chinese SOE business plan writing.
Built around the 数智生活商城 business plan as a reference document.

## Quick Start

```bash
# Generate and run selected skills interactively
/meta-ppt

# Or invoke any skill directly
/policy-analysis
/industry-analysis --style=浓缩要义
/future-goals
```

## Skills

| Command | Module | Default Style | PDF Pages |
|---------|--------|---------------|-----------|
| `/meta-ppt` | Factory — scaffolds all skills | — | — |
| `/policy-analysis` | 看政策 | 喊出口号 | 3 |
| `/industry-analysis` | 看行业 | 总结亮点 | 4–5 |
| `/self-analysis` | 看自身 | 总结亮点 | 6–8 |
| `/overview` | 商城概览 | 浓缩要义 | 10 |
| `/positioning` | 商城定位 | 浓缩要义 | 11–12 |
| `/strategy` | 商城发展策略 | 整体闭环 | 13 |
| `/operations` | 商城运营规划 | 整体闭环 | 14–17 |
| `/roadmap` | 商城目标及实施路径 | 浓缩要义 | 18 |
| `/org-structure` | 组织架构 | 总结亮点 | 19–22 |
| `/kpi` | 计收政策、考核政策 | 总结亮点 | 23 |
| `/settlement` | 结算政策 | 总结亮点 | 24 |
| `/resources` | 资源保障 | 总结亮点 | 24 |
| `/ops-architecture` | 商城平台运营架构 | 整体闭环 | 25 |
| `/tech-architecture` | 商城平台技术架构 | 整体闭环 | 26 |
| `/business-model` | 推进三种模式转变 | 整体闭环 | 27 |
| `/key-scenarios` | 重点突破五大场景 | 总结亮点 | 28–30 |
| `/future-goals` | 2025-2027年发展目标 | 喊出口号 | 31 |

## Style Override

Any skill supports `--style=<name>`:

```bash
/policy-analysis --style=浓缩要义
/strategy --style=喊出口号
```

Supported styles: `喊出口号`, `整体闭环`, `浓缩要义`, `总结亮点`

Note: `对齐概念` (concept alignment) is applied as a universal baseline in ALL outputs.

## Self-Improvement via news/

Drop `.md` files into `news/` then run `/learn-<skill>`:

```bash
# Example
echo "# 2026年消费政策新动向\n..." > news/2026-03-policy.md
/learn-policy-analysis
```

## Output Language

All PPT output is in **Chinese (中文)**. Skill names and commands are in English.

## Source Document

`data/PPT-数智生活商城商业计划书.pdf` — China Mobile 数智生活商城 Business Plan, August 2025
