---
name: django-ninja-project-standard
description: 定义 Django + Django Ninja + Celery + Docker 默认技术栈的项目起步标准，包含目录组织、文件职责、边界约定与默认开发实践。
compatibility: opencode
role: standard
---

# Django + Django Ninja 项目开发标准

## 目标

为 Python 项目开始开发时提供统一的目录组织、文件职责、边界约定和默认实践。

本 skill 的默认技术栈是：

- Django
- Django Ninja
- Celery
- Redis
- Docker / Docker Compose
- PostgreSQL
- pytest
- ruff
- `pyproject.toml + uv`（虚拟环境管理）

除非用户明确要求别的栈，否则默认按上面这套初始化和约束。

## 何时使用

- 用户要求“新建一个 Python 项目”
- 用户要求“给我一个 Django Ninja 项目标准”
- 用户要求“给我一个默认带 Celery、Docker 的 Python 后端骨架”
- 用户要求“参考 backend/apps/config 风格初始化项目”
- 用户没有明确指定技术栈，但希望直接采用仓库默认 Python 后端方案

## 默认理解的可调整项

开始新项目时，通常需要先明确这些项目级参数：

- `project_name`
- `python_package`
- `backend_dir`（默认 `backend`）
- `api_prefix`（默认 `/api`）
- `use_uv`（默认 `true`）
- `use_celery`（默认 `true`）
- `use_docker`（默认 `true`）
- `use_postgres`（默认 `true`）
- `use_redis`（默认 `true`）
- `allow_sqlite_fallback`（默认 `true`）
- `apps`（默认 `common`、`example`）

如用户未明确提供，可先按以下默认理解：

```yaml
project_name: my_service
python_package: my_service
backend_dir: backend
api_prefix: /api
use_uv: true
use_celery: true
use_docker: true
use_postgres: true
use_redis: true
allow_sqlite_fallback: true
apps:
  - common
  - example
```

## 参考目录结构

以下为默认目录结构：

```text
.
├── .dockerignore
├── .gitattributes
├── .env.docker
├── .env.local
├── .gitignore
├── logs/
├── README.md
├── docker-compose.yml
└── backend/
    ├── Dockerfile
    ├── manage.py
    ├── pyproject.toml
    ├── pytest.ini
    ├── apps/
    │   ├── common/
    │   │   ├── apps.py
    │   │   ├── api.py
    │   │   ├── schemas/
    │   │   │   ├── __init__.py
    │   │   │   ├── base.py
    │   │   │   ├── response.py
    │   │   │   └── health.py
    │   │   ├── capabilities/
    │   │   │   ├── health.py
    │   │   │   ├── auth.py
    │   │   │   ├── permissions.py
    │   │   │   ├── exceptions.py
    │   │   │   ├── responses.py
    │   │   │   └── logging.py
    │   │   ├── libs/
    │   │   │   ├── api/
    │   │   │   ├── cache/
    │   │   │   ├── tasks/
    │   │   │   ├── clients/
    │   │   │   ├── storage/
    │   │   │   └── utils/
    │   │   └── tests/
    │   └── example/
    │       ├── apps.py
    │       ├── api.py
    │       ├── models.py
    │       ├── schemas.py
    │       ├── services.py
    │       └── tests/
    └── config/
        ├── __init__.py
        ├── settings.py
        ├── urls.py
        ├── asgi.py
        ├── wsgi.py
        └── celery_app.py
```

## 目录与文件职责

### 根目录

- `.dockerignore`：Docker 构建时忽略无需进入镜像上下文的文件。
- `.gitattributes`：统一 Git 文本换行、二进制文件处理及特定文件属性。
- `.env.local`：本地开发环境变量文件。
- `.env.docker`：Docker / Compose 运行时环境变量文件。
- `.gitignore`：忽略虚拟环境（`uv venv` 默认创建的 `.venv` 目录）、缓存、环境变量文件、构建产物等无需提交的内容。
- `logs/`：本地日志输出目录。默认应存在，并通过 `.gitignore` 忽略目录内运行日志，仅保留占位文件（如 `.gitkeep`）或目录结构说明。
- `README.md`：项目说明、启动方式、开发命令、测试命令与常见注意事项。
- `docker-compose.yml`：本地容器编排文件，默认应包含 app、db、redis 等核心服务。

### `backend/`

- `manage.py`：Django 管理入口，用于 `runserver`、`migrate`、`createsuperuser` 等命令。
- `pyproject.toml`：项目依赖与工具配置入口，优先用于管理 Python 依赖、pytest、ruff 等配置。
- `pytest.ini`：pytest 运行配置。
- `Dockerfile`：后端镜像构建文件。

### `backend/config/`

- `settings.py`：Django 配置入口，负责读取环境变量、注册 apps、中间件、数据库、缓存、Celery、日志等基础配置；不承载业务逻辑。
- `urls.py`：全局路由聚合入口，负责挂载 Django Ninja API、管理后台路由或其他顶层 URL。
- `asgi.py`：ASGI 部署入口。
- `wsgi.py`：WSGI 部署入口。
- `celery_app.py`：Celery 应用入口，默认需要存在。

### `backend/apps/`

- `apps/` 用于存放业务 app，每个 app 应围绕单一业务域组织。

### `backend/apps/common/`

- `common` 是项目级公共 app，用于承载跨 app 复用的公共能力。
- `health` 作为 `common` 下的能力模块维护，不单独创建 app。
- 推荐结构：
  - `api.py`：common Router 聚合入口
  - `schemas/`：通用 schema
  - `capabilities/`：公共能力模块
  - `libs/`：底层支撑代码
- `common` 适合放健康检查、通用响应、权限认证、异常处理、日志、基础模型与各类跨 app 复用封装。
- `common` 不承载具体业务域逻辑。

### `backend/apps/common/capabilities/`

- `capabilities/` 用于放项目直接使用的公共能力模块。
- 常见内容：`health.py`、`auth.py`、`permissions.py`、`exceptions.py`、`responses.py`、`logging.py`。

### `backend/apps/common/schemas/`

- `schemas/` 用于放 common 对外暴露的通用 schema。
- 常见内容：`base.py`、`response.py`、`health.py`。
- `schemas/__init__.py` 作为聚合导出入口。

### `backend/apps/common/libs/`

- `libs/` 用于放低耦合、可复用的支撑代码。
- 可按能力拆分为 `api/`、`cache/`、`tasks/`、`clients/`、`storage/`、`utils/`。

### `backend/apps/<app>/`

- `apps.py`：Django app 注册配置。
- `api.py`：接口层，定义 Django Ninja Router、路由与请求入口。
- `schemas.py`：接口输入输出的数据结构定义，如请求体、响应体、序列化对象。
- `services.py`：业务逻辑层，负责用例编排、领域逻辑和可复用服务函数。
- `models.py`：数据模型定义；该 app 需要持久化时创建。
- `tests/`：该 app 的测试目录，优先放置与当前 app 对应的接口、服务、模型测试。

#### Model 字段说明规范

Model 的每个字段必须包含清晰的 `help_text` 和必要的 `db_index`、`unique` 等约束：

```python
class User(models.Model):
    """用户表"""

    username = models.CharField(
        max_length=50,
        unique=True,
        db_index=True,
        verbose_name="用户名",
        help_text="用于登录的用户名，4-20 位字母数字下划线"
    )
    email = models.EmailField(
        unique=True,
        db_index=True,
        verbose_name="邮箱",
        help_text="有效的邮箱地址，用于接收通知"
    )
    password = models.CharField(
        max_length=128,
        verbose_name="密码",
        help_text="加密后的密码，至少 8 位"
    )
    status = models.SmallIntegerField(
        choices=UserStatus.choices,
        default=UserStatus.PENDING,
        verbose_name="账号状态",
        help_text="1-正常 2-禁用 3-待审核"
    )
    role = models.ForeignKey(
        "Role",
        on_delete=models.PROTECT,
        related_name="users",
        verbose_name="角色",
        help_text="关联角色表，删除时级联保护"
    )
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name="创建时间",
        help_text="记录创建时间，UTC 时区"
    )
    updated_at = models.DateTimeField(
        auto_now=True,
        verbose_name="更新时间",
        help_text="最后更新时间，UTC 时区"
    )

    class Meta:
        db_table = "users"
        verbose_name = "用户"
        verbose_name_plural = "用户列表"
        indexes = [
            models.Index(fields=["username", "status"]),
            models.Index(fields=["email", "status"]),
        ]
```

字段说明应包含：

- **verbose_name**：字段显示名称，用于 admin 和迁移
- **help_text**：字段含义、用途、格式约束
- **约束标记**：`unique`、`db_index`、`choices`、`default`
- **外键关系**：`on_delete`、`related_name`

#### Schema 字段说明规范

Schema 的每个字段必须包含清晰的字段说明，确保 API 可自解释：

```python
class UserSchema(Schema):
    """用户信息 Schema"""

    id: int = Field(description="用户唯一标识 ID")
    username: str = Field(description="用户名，用于登录")
    email: str = Field(description="用户邮箱地址，用于接收通知")
    full_name: str | None = Field(default=None, description="用户真实姓名")
    status: int = Field(description="账号状态：1-正常 2-禁用 3-待审核")
    created_at: datetime = Field(description="账号创建时间，ISO 8601 格式")
    updated_at: datetime = Field(description="最后更新时间，ISO 8601 格式")


class CreateUserRequest(Schema):
    """创建用户请求 Schema"""

    username: str = Field(description="用户名，4-20 位字母数字下划线")
    email: EmailStr = Field(description="有效的邮箱地址")
    password: str = Field(min_length=8, max_length=32, description="密码，至少 8 位")
    role_id: int = Field(description="角色 ID，对应角色表主键")
```

字段说明应包含：

- **字段含义**：字段代表什么数据
- **格式约束**：如长度、格式、范围限制
- **取值范围**：枚举值含义，如状态字段需说明每个值的意义
- **业务规则**：如必填、可选、默认值、关联关系

避免使用模糊描述如"用户 ID"、"名称"、"状态"，应写清楚"用户唯一标识 ID"、"用于登录的用户名"、"账号状态：1-正常 2-禁用"。
- 当 `api.py`、`models.py`、`schemas.py`、`services.py` 任一文件持续膨胀、职责已明显分叉，允许改为同名目录。
- 可拆分为：
  - `api/`
  - `models/`
  - `schemas/`
  - `services/`
- 拆分后应保持清晰入口，例如通过 `api/__init__.py`、`schemas/__init__.py` 暴露聚合导出，避免让调用方依赖过深的内部路径。
- 小 app 默认保持单文件，复杂后再拆目录。

## 文件边界规则

- `api.py` 负责接口定义、参数接收、调用服务层和返回响应。
- `schemas.py` 或 `schemas/` 负责数据结构与校验规则。
- `services.py` 或 `services/` 负责业务逻辑。
- `models.py` 或 `models/` 负责 ORM 模型。
- `settings.py` 负责配置与环境变量读取。
- `urls.py` 负责顶层路由聚合。
- `celery_app.py` 负责 Celery 初始化与自动发现任务。
- `common/api.py` 负责 common 路由聚合，`common/schemas/` 负责通用 schema，`common/capabilities/` 负责公共能力，`common/libs/` 负责底层支撑代码。
- `README.md` 说明启动、测试、迁移与开发方式。

## 仓库默认模板硬约束

除非用户明确要求偏离，否则默认按以下约束落盘：

- 后端目录名固定使用 `backend`
- Django 配置目录固定使用 `backend/config`
- 业务 app 固定放在 `backend/apps`
- 默认创建 `common`、`example` 两个 app，`health` 放在 `common` 下
- 健康检查接口固定为 `/api/ops/health`
- API 文档访问地址固定为 `/api/docs`
- OpenAPI JSON 固定为 `/api/openapi.json`
- API 顶层前缀固定为 `/api`
- 本地日志目录固定为根目录 `logs/`
- Docker Compose 文件名固定为根目录 `docker-compose.yml`
- Docker Compose 默认服务名固定为 `app`、`db`、`redis`、`worker`
- Celery 应用入口固定为 `backend/config/celery_app.py`
- Python 依赖与工具配置固定放在 `backend/pyproject.toml`
- 框架通用方法与内部公共库放在 `backend/apps/common/` 或其 `libs/` 子目录

如果用户没有明确提出命名或结构差异，不要改成：

- `src/` 布局
- 非 `backend` 的后端目录名
- 非 `backend/apps` 的业务 app 目录
- 缺省省略 `worker`、`redis` 或 `db` 服务名的 Compose 结构
- 不带 `common` / `example` 的默认初始化骨架
- 把 `health` 建成独立 app

## 项目分层

### 最小必需结构

最小必需结构至少应包含：

- `.gitignore`
- `.gitattributes`
- `.dockerignore`
- `.env.local`
- `.env.docker`
- `logs/`
- `README.md`
- `docker-compose.yml`
- `backend/Dockerfile`
- `backend/manage.py`
- `backend/pyproject.toml`
- `backend/pytest.ini`
- `backend/config/settings.py`
- `backend/config/urls.py`
- `backend/config/asgi.py`
- `backend/config/wsgi.py`
- `backend/config/celery_app.py`
- `backend/apps/common/`
- `backend/apps/example/`
- `/api/ops/health` 健康检查接口

### 推荐默认结构

推荐再包含：

- 测试目录与基础测试用例
- PostgreSQL 配置
- Redis 配置
- Celery worker 启动说明
- `docker-compose.yml` 中的 `app` / `db` / `redis` / `worker` 服务定义

### 可调整项

- Docker
- Celery
- Redis
- PostgreSQL

如用户明确要求极简模式，可裁剪其中部分基础设施。

## 默认开发约定

### 配置

- 配置全部从环境变量读取
- 数据库默认 PostgreSQL，可选回退 SQLite
- 缓存 / broker 默认 Redis
- 不把业务逻辑写进 `settings.py`

### 日志

- 默认提供 Django 日志配置，并在 `settings.py` 中集中声明 `LOGGING`。
- 本地文件日志默认写入根目录 `logs/`，例如：
  - `logs/app.log`
  - `logs/error.log`
  - `logs/celery.log`
- 控制台日志用于容器与开发调试，文件日志用于本地排障与持久化查看。
- `logs/` 目录应默认创建，但运行日志文件不应提交到 Git。
- 如接入集中式日志平台，仍保留本地基础日志配置。

### 版本策略

- 默认不要把 Python、Django、Django Ninja、Celery、pytest、ruff 等依赖钉死到过旧或过窄的单一版本。
- 如无用户明确要求，优先选择当前稳定、主流、彼此兼容的版本范围。
- 只有在以下情况才应显式固定版本：
  - 用户明确指定版本
  - 项目已有既定版本约束
  - 某个依赖存在明确兼容性边界，需要避免歧义

### 路由

- 使用 Django Ninja
- 默认统一挂载在 `/api/`
- 健康检查接口默认使用 `/api/ops/health`

### API 文档

- 默认启用 OpenAPI 文档生成，访问地址固定为 `/api/docs`
- OpenAPI JSON 固定为 `/api/openapi.json`
- 文档页面应包含所有已注册 Router 的接口说明
- 如有公开 API 与内部 API 区分需求，可在 `common/capabilities/` 下维护统一的 OpenAPI 配置
- `example` app 应包含至少一个带完整 schema 说明的接口示例，供后续开发参考

### app 组织

- 默认创建 `common`、`example` 两个 app，`health` 作为 `common` 下的健康检查模块。
- `common` 负责公共能力、健康检查、基础探针、框架通用封装与跨 app 复用代码。
- `example` 负责提供参考实现，帮助后续 app 按既有模式扩展。
- 框架通用方法、跨 app 可复用库、基础设施适配层放在 `backend/apps/common/`。

每个业务 app 推荐至少包含：

- `apps.py`
- `api.py`
- `schemas.py`
- `services.py`
- `tests/`

如该 app 需要持久化数据，再加入 `models.py`。

### 运行体验

- **必须使用 `uv` 创建和管理虚拟环境**（`uv venv`），而非 `python -m venv` 或其他工具
- 优先提供 `uv sync` / `uv run` 命令
- 默认支持 Docker Compose 启动 `app`、`db`、`redis`、`worker`
- 可保留 SQLite 作为本地回退方案
- Celery 和 Docker 是默认工程组成，不应被写成可有可无的附属项

## 自包含要求

- 本 skill 应尽量自包含，不依赖额外模板清单、示例输入文件或目录树说明文件。
- 如需说明项目起步标准，应直接给出目录结构、关键文件职责、默认约定与验证方式。
- 不要求依赖同目录下的额外模板文件才能完成任务。

## 项目起步完成标准

一个符合该标准的新项目起步阶段，至少应满足：

在 `use_uv=true` 的默认场景下，**必须先使用 `uv venv` 创建虚拟环境**，然后满足：

1. `uv sync`
2. `uv run python manage.py check`
3. `uv run python manage.py migrate`
4. `uv run pytest`
5. `uv run ruff check .`
6. `docker compose config` 可通过
7. Celery worker 可按 README 中命令启动
8. `/api/ops/health` 可访问

若项目未使用 `uv`，应替换为等价的依赖安装、Django 检查、迁移、测试与静态检查命令。

## 禁止事项

- 不要把该 skill 改写成通用 Python 模板
- 不要把 Celery、Docker、Redis 默认降为纯按需组件
- 不要只输出目录树
- 不要默认移除 Django Ninja
- 不要默认把 `backend` 改成别的目录名
- 不要默认省略 `common`、`example`
- 不要把 `health` 做成独立 app
- 不要默认修改 Compose 服务名 `app`、`db`、`redis`、`worker`
- 不要让 `common` 承载具体业务域逻辑
- 不要遗漏日志配置和 `logs/` 目录约定
- 不要在文件尚小且职责清晰时预防性拆出过多子目录
- 不要强制所有接口堆在一个文件里

## 维护要求

- 除 `SKILL.md` 外，不应继续维护额外说明文件、示例输入文件或模板索引文件。
- 若规则发生变化，应直接更新本文件，使其保持为单文件、自解释的 skill。
