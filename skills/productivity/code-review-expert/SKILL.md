---
name: code-review-expert
description: >
  Python代码审查专家。以高级工程师视角结构化审查git变更：SOLID原则、
  安全扫描（含Python特有漏洞：pickle/eval/subprocess/SSTI）、性能分析、
  错误处理、边界条件、PEP8规范。P0-P3严重性分级，review-first（先审查后修复）。
  当用户说"review"、"code review"、"审查代码"、"检查代码"、
  "帮我看看代码"或调用/code-review-expert时使用。
  > Expert Python code review. Structured review of git changes with senior
  > engineer lens: SOLID, security (Python-specific: pickle/eval/subprocess/SSTI),
  > performance, error handling, boundary conditions, PEP8. P0-P3 severity,
  > review-first workflow. Use when user says "review", "code review", or
  > invokes /code-review-expert.
---

# Python Code Review Expert

基于 [sanyuan0704/code-review-expert](https://github.com/sanyuan0704/sanyuan-skills) 工作流骨架 + [yennanliu/python-code-reviewer](https://github.com/yennanliu/ai_experiment) Python 适配 + Python 专有安全检查整合而成。

## 严重性等级 (Severity Levels)

| Level | Name | 描述 | 动作 |
|-------|------|------|------|
| **P0** | Critical | 安全漏洞、数据丢失、正确性bug | 阻断合并 |
| **P1** | High | 逻辑错误、严重SOLID违规、性能退化 | 合并前修复 |
| **P2** | Medium | 代码坏味道、可维护性问题 | 本PR修复或建follow-up |
| **P3** | Low | 风格、命名、PEP8、小建议 | 可选改进 |

## 工作流 (Workflow)

### 1) Preflight — 范围检测

- 运行 `git status -sb`、`git diff --stat`、`git diff` 确定变更范围
- 使用 `grep` 或 `rg` 查找关联模块、调用方、契约
- 识别入口点、auth路径、支付逻辑、数据写入路径

**边界情况处理：**
- **无变更**: `git diff` 为空 → 告知用户，询问是否审查 staged 或指定 commit range
- **大diff (>500行)**: 先按文件汇总，再按模块/功能区分批审查
- **混合关注点**: 按逻辑功能分组，而非文件顺序

### 2) SOLID + 架构审查

检查以下违规：

**SRP (单一职责):**
- 一个文件混合了HTTP + DB + 业务规则等无关职责
- 大类/模块内聚性低，存在多个变更理由
- God object：一个类知道系统太多细节
- **问**: "这个模块因为什么单一原因会变化？"

**OCP (开闭原则):**
- 新增行为需要修改大量 switch/if 块
- 功能增长靠修改核心逻辑而非扩展
- 缺乏插件/策略/钩子点
- **问**: "能否不修改已有代码就添加新变体？"

**LSP (里氏替换):**
- 子类检查具体类型或对基类方法抛异常
- 重写方法弱化了前置条件或强化了后置条件
- **问**: "能否替换任何子类而调用方无感知？"

**ISP (接口隔离):**
- 接口方法多但大部分实现者用不到
- 调用方依赖宽接口满足窄需求
- 接口方法的空实现/stub
- **问**: "所有实现者都用到了所有方法吗？"

**DIP (依赖反转):**
- 高层逻辑依赖具体IO/存储/网络类型
- 硬编码实现而非抽象/注入
- 导入链将业务逻辑耦合到基础设施
- **问**: "能否不修改业务逻辑就替换实现？"

**常见代码坏味道：**

| 坏味道 | 信号 |
|--------|------|
| 长方法 | >30行，多层级嵌套 |
| 特性依恋 | 方法使用其他类的数据多于自身 |
| 数据泥团 | 同一组参数反复一起传递 |
| 基本类型偏执 | 用str/int代替领域类型 |
| 霰弹式修改 | 一个改动需要编辑多个文件 |
| 发散式变化 | 一个文件因多种无关原因变化 |
| 死代码 | 不可达或从未调用的代码 |
| 投机通用性 | 为假设的未来需求建抽象 |
| 魔法数字/字符串 | 无命名常量的硬编码值 |

**重构启发：**
1. 按职责而非大小拆分 — 小文件仍可违反SRP
2. 仅在需要时引入抽象 — 等待第二个用例出现
3. 保持重构增量 — 隔离行为后再移动
4. 先保护行为 — 重构前补充测试
5. 按意图命名 — 命名困难说明抽象可能有误
6. 组合优于继承 — 继承制造紧耦合
7. 让非法状态不可表示 — 用类型强制执行不变量

### 3) 死代码 + 删除计划

- 识别未使用、冗余、或被 feature flag 关闭的代码
- 区分 **现在安全删除** vs **推迟并制定计划**
- 删除前检查清单：
  - 搜索代码库所有引用 (`grep`/`rg`)
  - 检查动态/反射调用
  - 验证无外部消费者（API、SDK、文档）
  - 确认 feature flag 遥测数据
  - 更新/删除测试
  - 更新文档
  - 通知团队（共享代码时）

**删除计划模板：**

```
### 安全删除项
| 位置 | 理由 | 证据 | 影响 | 步骤 |
| path:line | 为什么删 | 无引用/死feature flag | 无活跃消费者 | 1.删代码 2.删测试 3.删配置 |

### 推迟删除项
| 位置 | 推迟原因 | 前置条件 | 迁移计划 | 时间线 | 回滚方案 |
| path:line | 有消费者/需迁移 | flag关闭2周 | 消费者迁移步骤 | 目标日期 | 如何恢复 |
```

### 4) 安全扫描

#### 通用安全（所有语言适用）

**注入类：**
- SQL/NoSQL/命令注入：字符串拼接或模板字面量构造查询
- SSRF：用户可控URL到达内部服务，无白名单校验
- 路径穿越：文件路径中使用用户输入，未清洗 `../`
- GraphQL注入：未限制查询深度/复杂度

**认证/授权：**
- 缺少租户或所有权检查
- 新端点缺少auth守卫或RBAC
- 信任客户端提供的角色/标志/ID
- IDOR (不安全的直接对象引用)
- 会话固定或弱会话管理

**JWT & Token:**
- 算法混淆攻击（接受 `none` 或 `HS256` 当预期 `RS256`）
- 弱密钥或硬编码密钥
- 缺少过期时间 (`exp`) 或未校验
- JWT payload包含敏感数据（token是base64编码，非加密）
- 未校验 `iss` (签发者) 或 `aud` (受众)

**密钥 & PII:**
- API密钥/token/凭证在代码/配置/日志中
- Git历史中的密钥或暴露给客户端的环境变量
- 日志中过度输出PII或敏感payload
- 错误消息缺少数据脱敏

**加密:**
- 弱算法（MD5、SHA1 用于安全目的）
- 硬编码IV或盐值
- 使用无认证的加密（ECB模式、无HMAC）
- 密钥长度不足

**竞态条件：**
- 共享状态无同步访问
- Check-Then-Act (TOCTOU)：`if exists then use` 模式非原子操作
- 数据库缺少乐观锁（`version`列）或悲观锁（`SELECT FOR UPDATE`）
- 计数器递增非原子操作（应用层 `value += 1` 而非 `UPDATE SET count = count + 1`）
- 分布式系统缺少分布式锁
- 缓存失效竞态（写后读陈旧数据）
- **关键提问**: "两个请求同时命中这段代码会怎样？"

**数据完整性：**
- 缺少事务，部分写入，不一致状态
- 持久化前弱校验（类型强制问题）
- 重试操作缺少幂等性
- 并发修改导致丢失更新

**运行时风险：**
- 无界循环、递归或大内存缓冲区
- 外部调用缺少超时/重试/限流
- 同步I/O阻塞请求路径
- 资源耗尽（文件句柄、连接、内存）

#### Python 专有安全（必须检查）

**反序列化攻击：**
- `pickle.loads()` / `cPickle.loads()` — 任意代码执行，**永远不要对不可信数据使用**
- `yaml.load()` 而非 `yaml.safe_load()` — 可构造Python对象
- `marshal.loads()` / `shelve` 模块 — 同样不安全
- 应使用: `json.loads()`, `yaml.safe_load()`, 或带限制的 `ast.literal_eval()`

**代码执行：**
- `eval()` / `exec()` / `compile()` 接受用户输入 → P0 严重
- `__import__()` 动态导入用户可控字符串
- `getattr()` / `setattr()` 使用用户可控的属性名
- `subprocess.Popen(shell=True)` → 命令注入。始终用 `shell=False` + 列表参数
- `os.system()` / `os.popen()` → 被 `subprocess.run()` 替代但仍需注意

**模板注入 (SSTI):**
- Jinja2: `render_template_string()` 使用用户输入
- Django: 自定义模板标签中 `mark_safe()` 误用
- 用户输入直接拼接到模板字符串中

**路径处理：**
- `os.path.join()` 的绝对路径绕过: `os.path.join("/safe/dir", "/etc/passwd")` → `/etc/passwd`
- 用 `Path` (pathlib) 或手动校验

**依赖安全：**
- 检查 `requirements.txt` / `pyproject.toml` 中是否有已知CVE的版本
- 不固定版本号 (`==` vs `>=`) 允许恶意更新
- 建议工具: `pip-audit`, `safety`

**日志安全：**
- `logging` 中避免 `%s` 格式化用户输入（虽然Python logging不易注入，但可能泄露敏感信息）
- Traceback 不应返回给客户端

### 5) 代码质量扫描

#### 错误处理

**反模式：**
```python
# ❌ 吞异常
try:
    ...
except Exception:
    pass

# ❌ 日志后忽略
try:
    ...
except Exception as e:
    logging.error(e)  # 调用方不知道失败了

# ❌ 过度宽泛的捕获
try:
    ...
except Exception:  # 应该捕获具体异常
    ...

# ❌ 异步错误未传播
async def handler():
    task = asyncio.create_task(work())  # 异常可能丢失
```

**应检查：**
- 异常在适当边界被捕获
- 错误消息用户友好（不暴露内部细节）
- 错误日志包含足够调试上下文
- 异步错误正确传播或处理
- 可恢复错误定义了降级行为
- 关键错误触发告警/监控

#### Python 专有反模式

**可变默认参数 (经典陷阱):**
```python
# ❌ 危险
def add_item(item, items=[]):
    items.append(item)
    return items

# ✅ 安全
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

**闭包延迟绑定:**
```python
# ❌ 循环中lambda都是同一个值
funcs = [lambda: i for i in range(5)]  # 全部返回4

# ✅ 用默认参数冻结
funcs = [lambda i=i: i for i in range(5)]
```

**资源管理:**
```python
# ❌ 未用上下文管理器
f = open('file.txt')
data = f.read()
f.close()  # 异常时可能不执行

# ✅ 始终用 with
with open('file.txt') as f:
    data = f.read()
```

**类型注解 (PEP 484):**
- 公开函数缺少类型注解
- `Any` 过度使用 — 绕过类型检查
- 错误使用 `Optional` (应 `Optional[X]` 而非 `X | None` 在旧版)

**PEP 8 检查:**
- 行长度 > 120 字符
- 命名: `snake_case` 函数/变量, `PascalCase` 类, `UPPER_CASE` 常量
- 空白: 函数间2空行, 类方法间1空行, 逗号后空格
- import 顺序: 标准库 → 第三方 → 本地
- 未使用的 import

**asyncio 专有:**
- `async def` 函数内使用同步阻塞调用（如 `time.sleep()` 而非 `await asyncio.sleep()`）
- `await` 忘记 → 协程未执行
- 事件循环中CPU密集型操作阻塞
- `asyncio.create_task()` 未保存引用 → 任务可能被GC
- 生产者-消费者模式缺少 `Queue` 边界

**GIL 考量:**
- CPU密集型操作放在主线程 → 阻塞所有Python线程
- 应使用 `multiprocessing` 或 C扩展释放GIL
- I/O密集型用 `threading` 或 `asyncio`

#### 性能

**数据库:**
- N+1 查询: 循环中逐条查询而非批量
- 缺少索引: 未索引列上的查询
- 过度查询: `SELECT *` 只需少量列时
- 无分页: 全量加载到内存

**缓存:**
- 重复的昂贵操作缺少缓存
- 缓存缺少TTL → 数据永不过期
- 缓存无失效策略 → 数据更新但缓存未清理
- 缓存键冲突

**内存:**
- 无界集合（列表/字典无限增长）
- 大文件一次性加载 → 用流式/分块读取
- 循环中字符串拼接 → 用 `''.join()` 或 `io.StringIO`
- 大对象引用阻止GC

**边界条件:**
- `None` 未检查 → `AttributeError`
- 空列表/字典 → `IndexError`/`KeyError`
- 除零 → `ZeroDivisionError`
- 负数索引/计数 → 逻辑炸弹
- 字符串空/纯空白 → 真值检查通过但实际为空
- 数值溢出 → Python int 无上限但 float/Decimal 有限制

### 6) 输出格式

```
## Code Review Summary

**Files reviewed**: X files, Y lines changed
**Overall assessment**: [APPROVE / REQUEST_CHANGES / COMMENT]

---

## ✅ Strengths

1. [做得好的地方]
2. ...

---

## 🔴 Findings

### P0 - Critical
(none or list)

### P1 - High
1. **`file:line`** — Brief title
   - 问题描述
   - 建议修复
   ```python
   # 修复代码示例
   ```

### P2 - Medium
2. (继续跨节编号)
   - ...

### P3 - Low
...

---

## 🗑️ Removal/Iteration Plan
(如适用)

## 📊 Overall Rating: X/10

**Summary:** [1-2 sentence summary]

---

## 🛠️ 推荐工具检查

- `ruff check .` — Python linting
- `bandit -r .` — Python 安全扫描
- `mypy .` — 类型检查
- `pip-audit` — 依赖漏洞扫描
```

### 7) 确认步骤 (Review-First)

审查完成后，**必须**展示下一步选项，**不得**在用户确认前修改代码：

```
---

## Next Steps

发现 X 个问题 (P0: _, P1: _, P2: _, P3: _)。

**如何继续？**

1. **修复全部** — 实施所有建议修复
2. **仅修复 P0/P1** — 只处理关键和高优先级
3. **修复指定项** — 告知要修复哪些问题
4. **不修改** — 审查完成，无需实施

请选择或提供具体指示。
```

## 行为规则

1. **审查优先**: 默认只输出审查结果，不主动修改代码
2. **具体引用**: 始终使用 `file:line` 格式定位问题
3. **Python 示例**: 修复建议必须用 Python 语法
4. **先正面后负面**: 先列出 Strengths，再列 Issues
5. **优先级排序**: P0 → P1 → P2 → P3
6. **权衡范围**: 不过度工程化简单脚本，不低工程化生产代码
7. **工具推荐**: 审查结束时推荐相应的静态分析工具
