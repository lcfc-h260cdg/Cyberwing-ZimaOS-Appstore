# 为 ZimaOS AppStore 做贡献

本指南适用于向**本仓库**提交变更的贡献者。

如果你正在构建自己的外部商店，请参考：
- [docs/guides/third-party-store-guide.zh-CN.md](docs/guides/third-party-store-guide.zh-CN.md)

## 1. 提交流程

1. Fork 本仓库，并在你自己的分支中进行开发。
2. 在本地 ZimaOS 环境中测试应用。
3. 发起 Pull Request，并包含：
   - 新增应用或更新内容
   - 变更说明
   - 测试证明（安装成功 + WebUI 可访问）

## 2. 必需的应用文件

`Apps/` 下的每个应用目录至少应包含：
- `docker-compose.yml`（或 `docker-compose.yaml`）
- `icon.svg` 或 `icon.png`
- 至少一张截图（`screenshot-1.png` / `.jpg`）

## 3. Compose 规则（仓库策略）

- Compose 文件必须是有效的 YAML。
- 不要使用镜像标签 `latest`；请固定明确版本。
- 顶层 `name` 是应用 ID，且必须唯一。
- `services.<service>.container_name` 通常应与服务名保持一致。

支持的运行时变量包括 `$PUID`、`$PGID`、`$TZ`、`$AppID`。

## 4. 顶层 `x-casaos` 规则

顶层 `x-casaos` 应按照以下字段分组顺序编写：

1. 访问入口：`main`、`index`、`port_map`、`scheme`
2. 展示信息：`title`、`icon`、`thumbnail`、`screenshot_link`、`tagline`、`description`、`tips`
3. 元数据：`author`、`developer`、`category`、`architectures`
4. 版本信息：`version`、`update_at`、`release_notes`
5. 相关链接：`website`、`repo`、`support`、`docs`

字段含义：

| 字段 | 类型/格式 | 含义 |
|---|---|---|
| `app_id` | string，必填 | 应用源 ID，要求反向域名风格，如 `org.icewhale.teable`；构建时会转小写。 |
| `main` | string | 主服务名，也就是提供 Web UI 的 compose service。多服务应用里应指向前端/主应用服务。 |
| `index` | string | Web UI 入口路径，常见为 `/`。 |
| `port_map` | string | Web UI 对外端口，必须写成字符串，如 `"3200"`。 |
| `scheme` | `http` / `https` | Web UI 协议。 |
| `icon` | string URL/path | 应用图标。构建后会被改写为构建产物里的图标 URL，用于安装后 dashboard。 |
| `title` | i18n object/string | 应用名称。源文件中通常是多语言对象；构建后 compose 里保留目标语言的字符串。 |
| `tagline` | i18n object/string | 应用短描述，会被抽取到 `meta.json`。 |
| `description` | i18n object/string | 应用详细描述，支持 markdown 文本，会被抽取到 `meta.json`。 |
| `thumbnail` | string URL/path | 应用缩略图，会进入 `meta.json` / store 展示资源。 |
| `screenshot_link` | string[] | 应用截图列表。 |
| `tips` | object | 安装提示，例如 `before_install`，内部支持多语言。 |
| `author` | string | 打包/维护者名称。 |
| `developer` | string | 上游开发者。 |
| `category` | string | 应用分类，必须是官方分类之一：`Media`、`Productivity`、`Home`、`Networking`、`AI`、`Finance`、`Social`、`Developer`、`Others`。 |
| `architectures` | string[] | 支持的 CPU 架构，如 `amd64`、`arm64`。 |
| `version` | string | 应用版本，v2 新增可选字段，用于商店展示。 |
| `update_at` | string | 更新时间，推荐 `YYYY-MM-DD`，如 `2026-05-13`。 |
| `release_notes` | i18n object/string | 源 compose 中的发布说明；构建输出到 `meta.json` 时字段名会变成 `release_note`。 |
| `website` | string URL | 官方网站。 |
| `repo` | string URL | 源码仓库地址。 |
| `support` | string URL | 支持/帮助地址。 |
| `docs` | string URL | 文档地址。 |

完整字段参考和构建输出行为请查看：
- [docs/guides/third-party-store-guide.md](docs/guides/third-party-store-guide.md)
- [docs/internal-dev-guide.md](docs/internal-dev-guide.md)

## 5. i18n 规则

Locale key 必须使用 `ll_CC` 格式（例如 `en_US`、`zh_CN`、`vi_VN`）。

i18n 块至少应包含 `en_US`。

## 6. PR 前的构建/校验

提交前请运行本地构建校验：

```bash
python3 scripts/build_appstore.py \
  --source . \
  --output dist \
  --base-url "https://cdn.jsdelivr.net/gh/IceWhaleTech/CasaOS-AppStore@gh-pages"
```

预期结果：
- 构建成功且没有错误
- 已生成 `dist/index.json`
- 变更的应用已更新 `dist/apps/<app-id>/docker-compose.yml` 和 `meta.json`

## 7. CI 说明

仓库 CI 会在 PR 中自动校验 compose 和构建流程。

请保持变更兼容：
- `.github/workflows/validator.yml`
- `.github/workflows/release.yml`

## 8. 资源文件建议

对于希望进入推荐/精选位的应用：
- `icon`：透明背景，推荐 256x256 SVG/PNG
- `thumbnail`：推荐 784x442（发布流程会转换为 WebP）
- `screenshot`：推荐 1280x800（16:10，发布流程会转换为 WebP）

如果你对该流程有建议，请提交 Issue 或 PR。
