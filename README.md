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
│   ├── handoff/         # 对话交接
│   └── grill-me/        # 计划/设计压力测试
├── in-progress/         # 正在开发中的技能
```

## 技能清单

| 技能 | 分类 | 描述 |
|------|------|------|
| **handoff** | productivity | 将当前对话压缩为交接文档，供另一个 agent 接续工作 |
| **grill-me** | productivity | 对计划或设计进行面试式追问，逐个解决决策树分支 |

## 贡献

1. 新技能放到 `skills/<category>/<skill-name>/SKILL.md`
2. 开发中的技能先放在 `in-progress/`，成熟后移入对应分类
3. 在 README 的技能清单中添加记录
