# Halo Agent Skills

面向 [Halo](https://github.com/halo-dev/halo) 开发者的 Agent Skills 合集，帮助 AI Agent 辅助开发主题、插件及其他 Halo 生态组件。

## 什么是 Agent Skills？

Agent Skills 为 AI Agent（如 Cursor、Codex 等）提供特定任务工作流。这两个 skill 会识别目标 Halo 版本、按需获取相关的官方 Markdown 文档，并仅在本地保留不易替代的开发约束。skill 可以包含：

- `SKILL.md` 入口文件，包含文档路由和开发工作流指南
- 可选的 `assets/` 目录，提供开箱即用的起始模板

## 安装

通过 [Skills CLI](https://skills.sh/) 安装：

```bash
# 全局安装（在所有项目中可用）
npx skills add halo-dev/dev-skills@halo-theme-dev -g
npx skills add halo-dev/dev-skills@halo-plugin-dev -g

# 或仅安装到当前项目
npx skills add halo-dev/dev-skills@halo-theme-dev
npx skills add halo-dev/dev-skills@halo-plugin-dev
```

## Skills 列表

### [`halo-theme-dev`](skills/halo-theme-dev/)

用于创建、修改、调试和打包 Halo 主题的版本感知工作流。执行任务时按需获取[主题 Markdown 文档索引](https://docs.halo.run/developer-guide/theme/index.md)中的最新内容。

**涵盖内容：**

- 主题目录结构及 `theme.yaml` / `settings.yaml` 配置
- Thymeleaf layout fragment 与模板路由
- 模板变量与 Finder API 使用
- 静态资源管理及通过 [`vite-plugin-halo-theme`](https://github.com/halo-sigs/vite-plugin-halo-theme) 进行 Vite 集成
- Halo 2.26+ 页面布局集成及可选的 Console/UC 主题 UI provider
- 使用 FormKit Schema 构建主题设置表单
- 模型元数据（Annotations）——定义与读取自定义字段
- 自定义模板标签（`halo:comment`、`halo:footer`）

**起始模板（位于 `assets/` 目录）：**

| 模板             | 说明                                                                               |
| ---------------- | ---------------------------------------------------------------------------------- |
| `theme-minimal/` | 无需构建工具，使用 Thymeleaf layout fragment 的最小主题                            |
| `theme-vite/`    | 完整 Vite 工程，集成 `vite-plugin-halo-theme`、partials 布局复用，支持 TailwindCSS |

### [`halo-plugin-dev`](skills/halo-plugin-dev/)

用于创建、修改、迁移和调试 Halo 插件的版本感知工作流。执行任务时按需获取[插件 Markdown 文档索引](https://docs.halo.run/developer-guide/plugin/index.md)中的最新内容。

**涵盖内容：**

- 插件目录结构及 `plugin.yaml` 清单配置
- Java 后端：自定义扩展（`AbstractExtension`、`@GVK`）、自定义 API（`CustomEndpoint`、`@Controller`）
- 插件生命周期（`BasePlugin` start/stop/delete）及 `SchemeManager` 注册与清理
- RBAC 角色模板，包含 view/manage 角色及匿名用户聚合
- Vue 3 前端：`definePlugin`、Console 和 User Center 路由注册
- 通过 `@halo-dev/ui-plugin-bundler-kit`（Vite/Rsbuild）构建和打包 IIFE/ESM UI provider
- 通过 `@Finder` 注解实现主题端 Thymeleaf 模板变量集成
- DevTools 工作流（`haloServer`、`reload`、`watch`）及 OpenAPI 客户端生成

## 目录结构

```
skills/
└── <skill-name>/
    ├── SKILL.md          # 入口文件，优先阅读
    ├── agents/           # 可选的 UI 元信息和调用策略
    └── assets/           # 可选的起始模板与输出资源
```

## 贡献

如需新增 skill，可参考现有 skill 的目录结构进行创建。

## 相关链接

- [Halo](https://github.com/halo-dev/halo) — 开源 CMS
- [Halo 开发者文档](https://docs.halo.run/developer-guide/index.md) — 官方文档
- [Halo LLM 文档索引](https://docs.halo.run/llms.txt) — Markdown 文档路由
- [theme-starter](https://github.com/halo-dev/theme-starter) — 最小主题模板
- [theme-vite-starter](https://github.com/halo-dev/theme-vite-starter) — 基于 Vite 的主题模板
- [plugin-starter](https://github.com/halo-dev/plugin-starter) — 最小插件模板
