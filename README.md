# SearXNG — 自部署隐私搜索引擎

基于 Docker Compose 部署的 [SearXNG](https://docs.searxng.org/) 实例，搭配 Valkey 缓存，提供隐私友好的元搜索服务。

## 项目结构

```
searxng/
├── docker-compose.yml      # Docker Compose 编排文件
├── .env                    # 环境变量配置
├── .env.example            # 环境变量示例
└── core-config/
    └── settings.yml        # SearXNG 核心配置
```

## 快速开始

### 前置要求

- [Docker](https://docs.docker.com/get-docker/) & [Docker Compose](https://docs.docker.com/compose/install/)

### 1. 配置环境变量

```bash
cp .env.example .env
```

按需编辑 `.env` 文件：

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `SEARXNG_VERSION` | 镜像版本标签 | `latest` |
| `SEARXNG_HOST` | 监听地址 | `[::]` |
| `SEARXNG_PORT` | 监听端口 | `8080` |

### 2. 启动服务

```bash
docker compose up -d
```

### 3. 访问

打开浏览器访问 `http://localhost:8080`

## 管理命令

```bash
# 启动
docker compose up -d

# 停止
docker compose stop

# 重启
docker compose restart

# 查看日志
docker compose logs -f core

# 停止并删除容器
docker compose down
```

## 配置说明

`core-config/settings.yml` 目前启用了以下功能：

- **`use_default_settings: true`** — 继承上游默认配置
- **`image_proxy: true`** — 启用图片代理，保护用户隐私
- **`search.formats`** — 同时支持 HTML 页面和 JSON API 响应
- **`limiter`** — 访问限流已注释，适合内网/本地使用；公网部署建议取消注释开启

完整配置项请参考 [SearXNG 官方文档](https://docs.searxng.org/admin/settings/)。

## 容器说明

| 服务 | 镜像 | 用途 |
|------|------|------|
| `core` | `searxng/searxng` | SearXNG 搜索引擎主体 |
| `valkey` | `valkey/valkey:9-alpine` | 缓存服务（Redis 替代） |

## 数据持久化

- 配置文件挂载：`./core-config/` → `/etc/searxng/`
- 缓存数据卷：`core-data` → `/var/cache/searxng/`
- Valkey 数据卷：`valkey-data` → `/data/`

## API 请求

SearXNG 支持 `format=json` 返回结构化数据，方便程序调用与二次开发。

### 请求格式

```bash
# 标准API请求，返回结构化JSON
curl "http://localhost:8080/search?q=你好&language=zh-CN&safesearch=0&categories=general&format=json"
```

### 常用参数

| 参数 | 说明 | 可选值 |
|------|------|--------|
| `q` | 搜索关键词 | 任意字符串 |
| `format` | 返回格式 | `html` / `json` |
| `language` | 搜索语言 | `zh-CN`, `en-US`, `auto` 等 |
| `safesearch` | 安全搜索 | `0`（关闭）/ `1`（中等）/ `2`（严格） |
| `categories` | 搜索分类 | `general`, `images`, `videos`, `news`, `files`, `science`, `social_media` 等 |
| `engines` | 指定搜索引擎 | 逗号分隔，如 `google,bing,duckduckgo` |
| `pageno` | 页码 | 从 `1` 开始 |

更多参数请参考 [SearXNG Search API](https://docs.searxng.org/dev/search_api.html)。

## 参考文档

- [SearXNG 官方文档](https://docs.searxng.org/)
- [SearXNG Docker 安装指南](https://docs.searxng.org/admin/installation-docker.html)
- [SearXNG 配置参考](https://docs.searxng.org/admin/settings/)
