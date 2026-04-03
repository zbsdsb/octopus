<div align="center">

<img src="web/public/logo.svg" alt="Octopus Logo" width="120" height="120">

### Octopus Fork

**zbsdsb 维护的 Octopus 分支，聚焦评分路由、健康度可视化与 Docker/GHCR 部署体验**

简体中文 | [English](README.md)

</div>

## 上游项目

这个仓库基于上游项目 `bestruirui/octopus` 维护，不重复搬运上游完整文档。

- 上游仓库：https://github.com/bestruirui/octopus
- 上游中文 README：https://github.com/bestruirui/octopus/blob/dev/README_zh.md
- 上游英文 README：https://github.com/bestruirui/octopus/blob/dev/README.md

如果你想看原始功能介绍、配置说明、完整 API 文档和基础使用方式，请直接以上游 README 为准。

## 这个 Fork 的差异

这个 fork 不改变 Octopus 作为 LLM API 聚合服务的基础定位，主要补的是运维和调度体验：

- 增强分组调度策略，不再只靠固定优先级或权重
- 增强管理后台，方便定位“哪个渠道更稳、哪个模型更健康、哪些日志更值得看”
- 补齐适合 fork 自己部署的 GHCR / Docker 文档和 workflow

## 新增功能

### 1. 分组评分优先策略

新增 `Scored` 分组模式，支持按多维评分优先选择渠道/模型/key。

- 支持按健康度、priority、weight 等维度综合排序
- key 选择会考虑最近使用、429 限流和累计成本
- 评分权重已做成可配置项，不再写死在代码里

### 2. 渠道健康度页面

新增健康度页面，可查看：

- 每个渠道的综合健康度
- 每个渠道下各模型的健康度明细
- 搜索、状态筛选、排序
- 渠道模型展开 / 收起

### 3. 日志页增强

日志页面新增：

- 搜索
- 状态筛选
- 排序
- 实时刷新开关
- API 格式展示与筛选

### 4. 分组编辑器增强

右侧“已选模型”区域新增：

- 搜索
- 排序
- 同源重复模型识别
- `Base URL` 展示

适合处理“同一个 Base URL 下多 key 重复挂同一模型”的调整场景。

### 5. 分组默认值配置

设置页新增分组默认值：

- 默认首字超时
- 默认会话保持

规则是：

- 分组里填了自定义值时，自定义优先
- 分组里留空或填 `0` 时，继承全局默认值

### 6. 稳定性修复

这个 fork 还顺手修了一些会影响实际使用的问题：

- locale 非法标签导致的设置页报错
- 动画基础组件的 lint / build 问题
- 分组成员元数据缺失导致的生产构建失败

## 部署

### 推荐镜像

如果你直接使用这个 fork，推荐优先拉取 GHCR 镜像：

- `ghcr.io/zbsdsb/octopus:dev`
- `ghcr.io/zbsdsb/octopus:dev-alpine`

其中：

- `dev`：跟随 fork 的 `dev` 分支自动发布，适合日常测试和部署
- `latest`：保留给正式 release / `master` 流程

### Docker 直接运行

```bash
docker run -d \
  --name octopus \
  -v /path/to/data:/app/data \
  -p 8080:8080 \
  ghcr.io/zbsdsb/octopus:dev
```

### Docker Compose

仓库里提供了可直接复用的 compose 示例：

```bash
cp docker-compose.ghcr.yml docker-compose.yml
docker compose up -d
```

默认会拉取：

```bash
ghcr.io/zbsdsb/octopus:dev
```

如果你想覆盖镜像、数据目录或端口：

```bash
export OCTOPUS_IMAGE=ghcr.io/zbsdsb/octopus:dev
export OCTOPUS_DATA_DIR=./data
export OCTOPUS_PORT=8080
docker compose -f docker-compose.ghcr.yml up -d
```

### 从官方镜像迁移到这个 Fork

如果你当前线上跑的是官方镜像，例如：

```yaml
image: bestrui/octopus
```

并且你希望无损切换到这个 fork，最稳的做法是：

1. 保持原来的数据挂载目录不变
2. 先备份数据库和配置
3. 只替换镜像地址
4. 再重建容器

以当前已经验证过的线上形态为例：

```yaml
services:
  octopus:
    image: ghcr.io/zbsdsb/octopus:dev
    ports:
      - "8080:8080"
    volumes:
      - "/path/to/data:/app/data"
    container_name: octopus
    restart: unless-stopped
```

切换步骤：

```bash
# 1. 备份数据
sudo cp -a /path/to/data/data.db /path/to/data/data.db.bak-$(date +%Y%m%d-%H%M%S)
sudo cp -a /path/to/data/config.json /path/to/data/config.json.bak-$(date +%Y%m%d-%H%M%S)

# 2. 修改 compose 中的镜像
# image: bestrui/octopus
# -> image: ghcr.io/zbsdsb/octopus:dev

# 3. 拉取并重建
docker compose pull octopus
docker compose up -d octopus
```

迁移时不要把挂载目录改成新的空目录，否则会导致程序起一套全新的空数据。

如果你当前使用的是 SQLite，也不建议让“老容器”和“新容器”同时写同一个 `data.db` 文件。更稳的做法是先复制一份数据库副本，在新端口做预演，通过后再切正式端口。

### GHCR 自动发布链路

这个 fork 现在有两条镜像发布路径：

1. `dev` 分支
   - workflow：`.github/workflows/docker-dev.yaml`
   - 触发条件：`push` 到 `dev` 或手动触发
   - 产出标签：
     - `ghcr.io/zbsdsb/octopus:dev`
     - `ghcr.io/zbsdsb/octopus:dev-alpine`

2. `master` 分支
   - workflow：`.github/workflows/release.yaml`
   - 触发条件：`push` 到 `master`
   - 产出标签：
     - `ghcr.io/zbsdsb/octopus:latest`
     - `ghcr.io/zbsdsb/octopus:latest-alpine`

### 部署注意事项

- 容器升级时继续保留 `/app/data` 挂载即可，已有数据不会丢
- 新增的 setting 项会在启动时自动补齐，不需要你手改 `data/config.json`
- 如果要让其他人直接拉你的 GHCR 镜像，请到 GitHub Packages 页面确认 `octopus` 容器包是 `public`

## 当前分支状态

当前 fork 默认分支是 `dev`，并已包含这批增强功能。  
如果你只想使用这些增强功能，直接部署 `ghcr.io/zbsdsb/octopus:dev` 即可。
