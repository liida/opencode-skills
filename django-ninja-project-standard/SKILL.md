---
name: django-ninja-project-standard
description: 定义 Django + Django Ninja + Celery + Docker 默认技术栈的项目起步标准，包含目录组织、文件职责、边界约定与默认开发实践。
compatibility: opencode
role: standard
requires:
  - project-workflow
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

除非用户明确要求别的栈，否则默认这套初始化和约束。

## 强触发条件

当出现以下任一情况时，应优先加载本 skill：
- 用户要求"新建一个 Python 项目"
- 用户要求"给我一个 Django Ninja 项目标准"
- 用户要求"给我一个默认带 Celery、Docker 的 Python 后端骨架"
- 用户要求"参考 backend/apps/config 风格初始化项目"
- 用户没有明确指定技术栈，但希望直接采用仓库默认 Python 后端方案

## 默认参数

`project_name`、`python_package`、`backend_dir`（默认 `backend`）、`api_prefix`（默认 `/api`）、`use_uv`/`use_celery`/`use_docker`/`use_postgres`/`use_redis`（默认 `true`）、`allow_sqlite_fallback`（默认 `true`）、`apps`（默认 `common`、`example`）

## 参考目录结构

```
.
├── .dockerignore/.gitattributes/.gitignore
├── .env.local/.env.docker
├── logs/
├── README.md
├── docker-compose.yml
└── backend/
    ├── Dockerfile/manage.py/pyproject.toml/pytest.ini
    ├── apps/
    │   ├── common/ (api.py, schemas/, capabilities/, libs/, tests/)
    │   └── example/ (apps.py, api.py, models.py, schemas.py, services.py, tests/)
    └── config/ (settings.py, urls.py, asgi.py, wsgi.py, celery_app.py)
```

## 目录与文件职责

### 根目录

- `.dockerignore` / `.gitattributes` / `.gitignore`：Docker 和 Git 配置
- `.env.local` / `.env.docker`：本地和容器环境变量
- `logs/`：日志目录，运行日志不提交 Git
- `README.md`：项目说明
- `docker-compose.yml`：容器编排

### `backend/`

- `manage.py`：Django 管理入口
- `pyproject.toml`：依赖与工具配置
- `pytest.ini`：pytest 配置
- `Dockerfile`：镜像构建

### `backend/config/`

- `settings.py`：Django 配置（不承载业务逻辑）
- `urls.py`：路由聚合
- `asgi.py` / `wsgi.py`：ASGI/WSGI 入口
- `celery_app.py`：Celery 入口

### `backend/apps/`

业务 app 目录，每个 app 围绕单一业务域。

### `backend/apps/common/`

项目级公共 app，不承载业务域：
- `api.py`：Router 聚合
- `schemas/`：通用 schema
- `capabilities/`：公共能力（health/auth/permissions 等）
- `libs/`：底层支撑（api/cache/tasks/clients/storage/utils）

### `backend/apps/<app>/`

- `apps.py`：Django 注册
- `api.py`：接口层
- `schemas.py`：数据结构
- `services.py`：业务逻辑
- `models.py`：ORM 模型（需要持久化时）
- `tests/`：测试目录

#### Model 字段说明规范

```python
class User(models.Model):
    username = models.CharField(max_length=50, unique=True, verbose_name="用户名", help_text="...")
    email = models.EmailField(unique=True, verbose_name="邮箱", help_text="...")
    role = models.ForeignKey("Role", on_delete=models.PROTECT, verbose_name="角色", help_text="...")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="创建时间")
```

字段说明应包含：verbose_name（字段显示）、help_text（含义用途）、约束标记（unique/db_index/choices/default）、外键关系（on_delete/related_name）

#### Schema 字段说明规范

```python
class UserSchema(Schema):
    id: int = Field(description="用户唯一标识 ID")
    username: str = Field(description="用户名，用于登录")

class CreateUserRequest(Schema):
    username: str = Field(description="用户名，4-20 位字母数字下划线")
    email: EmailStr = Field(description="有效的邮箱地址")
```

字段说明应包含：字段含义、格式约束、取值范围、业务规则。避免模糊描述如"用户 ID"，应写清楚"用户唯一标识 ID"。

#### 文件拆分时机

文件膨胀时可拆分为 `api/`、`models/`、`schemas/`、`services/`，保持 `__init__.py` 入口。小 app 默认单文件。

## 文件边界规则

- `api.py`：接口定义、调用服务、返回响应
- `schemas.py`：数据结构与校验
- `services.py`：业务逻辑
- `models.py`：ORM 模型
- `settings.py`、`urls.py`（路由聚合）
- `celery_app.py`：Celery 初始化
- `common/`：公共能力（schemas/capabilities/libs）

## 仓库默认模板硬约束

固定使用：
- 目录：`backend/`、`backend/config/`、`backend/apps/`
- App：`common`（含 health）、`example`
- 路径：`/api/`、`/api/ops/health`、`/api/docs`、`/api/openapi.json`
- 日志：`logs/`
- Compose：`docker-compose.yml`，服务名 `app/db/redis/worker`
- 入口：`backend/config/celery_app.py`、`backend/pyproject.toml`

不要改成：`src/` 布局、非 backend、非 common/example、省略 worker/redis/db

## 项目分层

### 最小必需结构

`.gitignore`/`.gitattributes`/`.dockerignore`、`.env.local`/`.env.docker`、`logs/`、`README.md`、`docker-compose.yml`、`backend/Dockerfile`/`manage.py`/`pyproject.toml`/`pytest.ini`、`backend/config/settings.py`/`urls.py`/`asgi.py`/`wsgi.py`/`celery_app.py`、`backend/apps/common/`/`example/`、`/api/ops/health`

### 推荐默认结构

测试用例、PostgreSQL/Redis 配置、Celery worker、Compose 服务定义

### 可调整项

Docker、Celery、Redis、PostgreSQL，可裁剪

## 默认开发约定

### 配置

环境变量读取、数据库默认 PostgreSQL 可选 SQLite、缓存/broker 默认 Redis、settings.py 不承载业务逻辑

### 日志

Django LOGGING 配置，`logs/` 目录，运行日志不提交 Git

### 版本策略

优先稳定主流版本，仅用户指定、项目既有、明确兼容性边界时固定版本

### 路由

Django Ninja，挂载 `/api/`，`/api/ops/health` 健康检查

### API 文档

`/api/docs` 文档、`/api/openapi.json` JSON，`example` 包含完整示例

### app 组织

`common`（公共能力）+ `example`（参考实现），每个 app 含 `apps.py`/`api.py`/`schemas.py`/`services.py`/`tests/`，需要持久化加 `models.py`

### 运行体验

必须 `uv venv`，Docker 构建时需配置环境变量：
```dockerfile
ENV PYTHONDONTWRITEBYTECODE=1
ENV UV_PROJECT_ENVIRONMENT=/opt/venv
ENV PATH="/opt/venv/bin:$PATH"
```
Compose 启动 app/db/redis/worker，Celery/Docker 默认

## 项目起步完成标准

docker-compose 验证：配置检查 → 依赖安装 → 测试 → 静态检查 → 服务健康检查

## 禁止事项

不改成：通用 Python 模板、Celery/Docker 按需、目录树、移除 Ninja、backend 改名、省略 common/example、health 独立 app、common 承载业务、遗漏日志

## 维护要求

仅维护本文件，规则变化时直接更新
