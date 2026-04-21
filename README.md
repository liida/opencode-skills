# 全局 Skills 说明

用于说明当前全局 skills 的职责边界、默认加载策略与维护原则，避免多个 skill 同时承担同一类规则。

## 当前保留的 skills

### 1. `project-workflow`
负责项目级任务流程：
- 新任务开始前读取稳定上下文
- 编码前检索现有实现与复用模式
- 任务中决定何时连续推进、何时暂停确认
- 改动后进行本地验证
- 按需回写长期有效的项目记忆

适合：
- 中大型仓库任务
- 多文件修改
- 需要统一验证
- 需要项目记忆或协作规则

通常不必强制加载：
- 单文件、小范围、低风险修改
- 不涉及项目上下文、记忆或统一流程的快速任务

### 2. `coding-rule`
负责通用编码行为：
- 不假设，不隐藏困惑
- 简单优先
- 精确修改
- 用可验证目标驱动实现

适合：
- 编写、修改、审查、重构代码
- 需要防止过度设计或无关改动

### 3. `django-ninja-project-standard`
负责定义 Python + Django + Django Ninja 新项目开始开发时的目录组织、文件职责与默认约定。

适合：
- 初始化新服务
- 明确 Django Ninja 新项目开发标准
- 参考 `backend/apps/` 与 `backend/config/` 风格搭建项目

不适合：
- 普通仓库任务
- 与 Django Ninja 新项目标准无关的问题

## 默认加载策略

- 仓库内中大型开发任务：加载 `project-workflow`，涉及编码时同时加载 `coding-rule`
- 小范围代码修改：优先只加载 `coding-rule`
- 新建 Django Ninja 项目并确定起步标准：加载 `django-ninja-project-standard`；若还需要遵循仓库级流程，再补 `project-workflow`
- 非编码问答或简单命令：通常不需要加载 skill

## 默认加载决策树

```text
这是一个 Django Ninja 新项目起步标准任务吗？
├─ 是 → 加载 django-ninja-project-standard
│        └─ 如果还需要遵循仓库级流程，再补 project-workflow
└─ 否 → 这是仓库内的开发任务吗？
         ├─ 否 → 通常不需要加载 skill
         └─ 是 → 是否涉及中大型改动、多文件修改、统一验证、项目记忆？
                  ├─ 是 → 加载 project-workflow
                  │        └─ 如果涉及编码，再补 coding-rule
                  └─ 否 → 是否是小范围代码修改或代码审查？
                           ├─ 是 → 加载 coding-rule
                           └─ 否 → 视任务复杂度决定，默认从不加载开始
```

## 常见组合

### 仓库内正常开发任务
- `project-workflow`
- `coding-rule`

### 小范围代码修改
- `coding-rule`

### 新建 Django Ninja 项目
- `django-ninja-project-standard`
- 如需同时遵循仓库级工作流，再补 `project-workflow`

## 分工原则

- `project-workflow` 管流程，不负责具体编码细则
- `coding-rule` 管编码行为，不负责仓库记忆、任务连续推进、上下文读取策略
- 专项项目标准 skill 只在对应技术场景下启用，不应作为默认全局规则
- 已删除的旧 skill 不参与新工作流设计；旧引用应迁移到当前保留的 skill

## 维护建议

- 新增 skill 时，先明确它属于“流程层”“行为层”“专项能力层”还是“兼容层”
- 新增 skill 的 frontmatter 应包含 `role` 字段，建议取值包括 `workflow`、`coding`、`standard`、`research`、`review`
- 如果一个规则已经存在于流程层，就不要在行为层再重复一遍
- 如果一个 skill 只适用于特定框架或技术栈，不要把它包装成通用全局规则
