# AI Project Scaffold

通用 AI 工程项目脚手架，内置 OpenSpec 工作流支持与六大开发规范约束。

## 包含内容

- **OpenSpec 工作流**：支持 Cursor 和 Claude Code 的斜杠命令（`/opsx:propose`、`/opsx:apply`、`/opsx:archive`）
- **CLAUDE.md**：六大开发规范约束（单一来源，Cursor 和 Claude Code 共用）
- **openspec/config.yaml**：OpenSpec per-artifact 规则

## 开发规范

规范详见 [CLAUDE.md](./CLAUDE.md)，包含以下六大约束（按优先级排序）：

1. **模块化约束** — DDD 四层架构（Interface / Application / Domain / Infrastructure）
2. **安全约束** — 对齐 OWASP Top 10:2025
3. **测试约束** — Domain 单测、契约测试、E2E 测试
4. **环境隔离约束** — 对齐 12-Factor App
5. **代码质量约束** — Clean Code + SOLID（S/O/L/I）
6. **可观测性约束** — 结构化日志、指标埋点、健康检查

## 推荐 Skills

以下 Skills 与本项目规范高度互补，建议按需安装。

### Claude Code

在 Claude Code 对话框中输入以下命令安装：

**Anthropic 官方 Skills**

```
/plugin marketplace add anthropics/skills
/plugin install example-skills@anthropic-agent-skills
```

安装后可单独使用 `webapp-testing`（Playwright 测试）和 `mcp-builder`（MCP Server 生成）。

**Trail of Bits 安全审计**

手动安装（见下方 Cursor 通用方式），或参考 [trailofbits/skills](https://github.com/trailofbits/skills)。

---

### Cursor

Cursor 通过 `.cursor/skills/` 目录加载 Skills，手动复制 SKILL.md 文件即可。

**1. webapp-testing（Playwright 测试）**

```bash
mkdir -p .cursor/skills/webapp-testing
curl -o .cursor/skills/webapp-testing/SKILL.md \
  https://raw.githubusercontent.com/anthropics/skills/main/skills/webapp-testing/SKILL.md
```

**2. mcp-builder（MCP Server 生成）**

```bash
mkdir -p .cursor/skills/mcp-builder
curl -o .cursor/skills/mcp-builder/SKILL.md \
  https://raw.githubusercontent.com/anthropics/skills/main/skills/mcp-builder/SKILL.md
```

**3. Trail of Bits 安全审计**

```bash
git clone --depth 1 https://github.com/trailofbits/skills /tmp/trailofbits-skills
cp -r /tmp/trailofbits-skills/.cursor/skills/* .cursor/skills/
```

---

## OpenSpec 工作流

初始化后，通过以下斜杠命令启动开发流程：

```
/opsx:propose "你的功能描述"   # 生成提案、设计、任务清单
/opsx:apply                   # 按任务清单实施变更
/opsx:archive                 # 归档变更，更新规范文档
```

## 初始化新项目

```bash
# 克隆模板
git clone https://github.com/mgeer/AI_Project_Scaffold.git my-project
cd my-project

# 重新初始化 git
rm -rf .git
git init
git add .
git commit -m "chore: init from AI_Project_Scaffold template"
```
