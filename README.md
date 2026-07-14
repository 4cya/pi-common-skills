# pi-common-skills

日常使用的 Pi Coding Agent 通用技能合集。

## 安装

```bash
# 方法一：符号链接（推荐，方便更新）
# Windows (管理员 PowerShell)
New-Item -ItemType Junction -Path "$env:USERPROFILE\.pi\agent\skills\pi-common-skills" -Target "E:\1_develop\pi\pi-common-skills\skills"

# Linux / macOS
ln -s /path/to/pi-common-skills/skills ~/.pi/agent/skills/pi-common-skills
```

确保 `~/.pi/config.yaml` 或 `~/.pi/agent/config.yaml` 中的 `skills` 配置指向正确的路径。

## 技能目录

```
skills/
├── productivity/        # 开发效率类技能
│   ├── handoff/         # Copy conversation into a handoff document
│   ├── grilling/        # Relentless interview about a plan or design
│   └── caveman/         # Ultra-compressed communication mode
├── in-progress/         # 正在开发中的技能
```

## 技能清单

| Skill | Category | Description |
|-------|----------|-------------|
| **handoff** | productivity | Compact the current conversation into a handoff document for another agent to pick up |
| **grilling** | productivity | Grill the user relentlessly about a plan, decision, or idea until every branch of the decision tree is resolved |
| **caveman** | productivity | Ultra-compressed communication mode. Cuts token usage ~75% by dropping filler, articles, and pleasantries while keeping full technical accuracy |

## 贡献

1. 新技能放到 `skills/<category>/<skill-name>/SKILL.md`
2. 开发中的技能先放在 `in-progress/`，成熟后移入对应分类
3. 在 README 的技能清单中添加记录
