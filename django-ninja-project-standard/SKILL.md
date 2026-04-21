---
name: django-ninja-project-standard
description: 定义 Django + Django Ninja + Celery + Docker 默认技术栈的项目起步标准，包含目录组织、文件职责、边界约定与默认开发实践。
compatibility: opencode
---

# Django + Django Ninja 项目开发标准

> 本技能**必须**依赖 `@project-workflow`。
> 在开始任何编码工作前，**必须先加载并读取** `@project-workflow` 的内容。
> 所有的编码行为、验证步骤和文件修改必须严格遵守 `@project-workflow` 中定义的“编码行为准则”和“项目任务流程”。
> **不要**仅凭记忆执行，**必须**加载该文件以确保上下文完整。

## 目标与触发条件

**目标**：为 Python 项目开始开发时提供统一的目录组织、文件职责、边界约定和默认实践。

**触发条件**：用户要求"新建一个 Python 项目"、"Django Ninja 项目标准"、"带 Celery/Docker 的后端骨架"等，或没有明确指定技术栈但希望采用仓库默认方案。

**默认参数**：
- `project_name`、`python_package`、`backend_dir`（默认 `backend`）、`api_prefix`（默认 `/api`）
- `use_uv`/`use_celery`/`use_docker`/`use_postgres`/`use_redis`（默认 `true`）
- `allow_sqlite_fallback`（默认 `true`）、`apps`（默认 `common`、`example`）

## 技术栈

Django + Django Ninja + Celery + Redis + Docker Compose + PostgreSQL + pytest + ruff + pyproject.toml + uv

除非用户明确要求别的栈，否则默认这套。

## 目录结构

```
.
├── .dockerignore / .gitattributes / .gitignore
├── .env.local / .env.docker
├── logs/
├── README.md
├── docker-compose.yml
└── backend/
    ├── Dockerfile / manage.py / pyproject.toml / pytest.ini
    ├── apps/
    │   ├── common/ (api.py, schemas/, capabilities/, libs/, tests/)
    │   └── example/ (apps.py, api.py, models.py, schemas.py, services.py, tests/)
    └── config/ (settings.py, urls.py, asgi.py, wsgi.py, celery_app.py)
```

## 文件职责

### 根目录

- `.dockerignore` / `.gitattributes` / `.gitignore`：Docker 和 Git 配置
- `.env.local`：本地开发环境变量
- `.env.docker`：容器环境变量
- `logs/`：日志目录（运行日志不提交 Git）
- `README.md`：项目说明
- `docker-compose.yml`：容器编排

### backend/

- `manage.py`：Django 管理入口（runserver/migrate/createsuperuser）
- `pyproject.toml`：依赖与工具配置
- `pytest.ini`：pytest 配置
- `Dockerfile`：镜像构建

### backend/config/

- `settings.py`：Django 配置（读取环境变量、注册 apps、中间件、数据库、缓存、Celery、日志），不承载业务逻辑
- `urls.py`：路由聚合
- `asgi.py` / `wsgi.py`：ASGI/WSGI 入口
- `celery_app.py`：Celery 入口

### backend/apps/common/

项目级公共 app，不承载业务域：
- `api.py`：Router 聚合
- `schemas/`：通用 schema（base.py, response.py, health.py）
- `capabilities/`：公共能力（health.py, auth.py, permissions.py, exceptions.py, responses.py, logging.py）
- `libs/`：底层支撑（api/, cache/, tasks/, clients/, storage/, utils/）

### backend/apps/<app>/

业务 app，围绕单一业务域：
- `apps.py`：Django 注册
- `api.py`：接口层（定义 Router、路由、请求入口）
- `schemas.py`：数据结构（请求体、响应体）
- `services.py`：业务逻辑（用例编排、领域逻辑）
- `models.py`：ORM 模型（需要持久化时）
- `tests/`：测试目录

## 编码规范

### Model 字段规范

```python
class User(models.Model):
    username = models.CharField(max_length=50, unique=True, verbose_name="用户名", help_text="...")
    email = models.EmailField(unique=True, verbose_name="邮箱", help_text="...")
    role = models.ForeignKey("Role", on_delete=models.PROTECT, verbose_name="角色", help_text="...")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="创建时间")
```

字段必须包含：verbose_name（显示名称）、help_text（含义用途）、约束标记（unique/db_index/choices/default）、外键关系（on_delete/related_name）

### Schema 字段规范

**所有 Schema（请求、响应、内部使用）必须遵守此规范**

```python
class UserSchema(Schema):
    id: int = Field(description="用户唯一标识 ID")
    username: str = Field(description="用户名，用于登录")

class CreateUserRequest(Schema):
    username: str = Field(description="用户名，4-20 位字母数字下划线")
    email: EmailStr = Field(description="有效的邮箱地址")
```

字段说明应包含：字段含义、格式约束、取值范围、业务规则。避免模糊描述如"用户 ID"，应写清楚"用户唯一标识 ID"。

### 文件拆分时机

文件膨胀时可拆分为 `api/`、`models/`、`schemas/`、`services/`，保持 `__init__.py` 入口。小 app 默认单文件。

## 硬约束

**固定使用**：
- 目录：`backend/`、`backend/config/`、`backend/apps/`
- App：`common`（含 health）、`example`
- 路径：`/api/`、`/api/ops/health`、`/api/docs`、`/api/openapi.json`
- 日志：`logs/`
- Compose：`docker-compose.yml`，服务名 `app/db/redis/worker`
- 入口：`backend/config/celery_app.py`、`backend/pyproject.toml`

**不要改成**：`src/` 布局、非 backend、非 common/example、省略 worker/redis/db

## 开发约定

### 配置

- 全部从环境变量读取
- 数据库默认 PostgreSQL，可选 SQLite
- 缓存/broker 默认 Redis
- settings.py 不承载业务逻辑

### 日志

Django LOGGING 配置，`logs/` 目录，运行日志不提交 Git

### 版本策略

优先稳定主流版本，仅用户指定、项目既有、明确兼容性边界时固定版本

### 路由

Django Ninja，挂载 `/api/`，`/api/ops/health` 健康检查

### API 文档

`/api/docs` 文档、`/api/openapi.json` JSON，`example` 包含完整示例

## 运行与验证

### Dockerfile

```dockerfile
FROM python:3.10-slim
WORKDIR /app
RUN pip install uv
ENV PYTHONDONTWRITEBYTECODE=1
ENV UV_PROJECT_ENVIRONMENT=/opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY backend/pyproject.toml /app/
RUN uv sync
COPY backend/ /app/
EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### 本地运行

`.env.local` 配置数据库/Redis，使用 `uv venv` 创建虚拟环境

### 容器运行

`.env.docker` 配置容器环境，`docker-compose up` 启动 app/db/redis/worker 服务

### 验证步骤

**优先本地验证**（遵循 project-workflow 规则：优先当前环境可完成的最小验证）：

1. `ruff check .` - 静态检查
2. `pytest` - 本地测试
3. `python manage.py check` - Django 检查

**Docker 验证**（补充验证，确保容器环境一致）：

4. `docker-compose config` - 配置检查
5. `docker-compose run --rm app uv sync` - 依赖安装
6. `docker-compose run --rm app pytest` - 容器测试
7. `/api/ops/health` - 健康检查

## 禁止事项

不改成：通用 Python 模板、Celery/Docker 按需、目录树、移除 Ninja、backend 改名、省略 common/example、health 独立 app、common 承载业务、遗漏日志

## 维护要求

仅维护本文件，规则变化时直接更新
