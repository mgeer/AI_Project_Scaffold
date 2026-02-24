# AI Project Scaffold

面向内部微服务的 AI 辅助开发脚手架，内置 OpenSpec 结构化开发工作流与 DDD 六大开发规范约束。

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

## 包含内容

- **OpenSpec 工作流**：支持 Cursor 和 Claude Code 的斜杠命令（`/opsx:propose`、`/opsx:apply`、`/opsx:archive`）
- **CLAUDE.md**：六大开发规范约束（单一来源，Cursor 和 Claude Code 共用）
- **openspec/config.yaml**：OpenSpec per-artifact 规则

## 开发规范

规范详见 [CLAUDE.md](./CLAUDE.md)，包含以下六大约束（按优先级排序）：

1. **安全约束** — SQL/命令注入防护、敏感字段加密、依赖审计
2. **模块化约束** — DDD 四层架构（Interface / Application / Domain / Infrastructure），含各层判断规则与异常跨层转换规范
3. **测试约束** — 禁止 Mock 框架，统一使用 In-memory Fake；Domain 单测覆盖率 ≥ 80%
4. **环境隔离约束** — 对齐 12-Factor App，Fail Fast + 优雅关闭
5. **代码质量约束** — Clean Code + SOLID（S/O/I）
6. **可观测性约束** — 结构化日志、Trace ID 跨服务传播、指标埋点、健康检查

## OpenSpec 工作流

初始化后，通过以下斜杠命令启动开发流程：

```
/opsx:propose "你的功能描述"   # 生成提案、设计、任务清单
/opsx:apply                   # 按任务清单实施变更
/opsx:archive                 # 归档变更，更新规范文档
```

---

## 附录：推荐 Skills

以下 Skills 与本项目规范互补，按需安装。Cursor 和 Claude Code 均通过 `.claude/skills/` 目录加载。

**1. webapp-testing（Playwright 测试）**

```bash
mkdir -p .claude/skills/webapp-testing
curl -o .claude/skills/webapp-testing/SKILL.md \
  https://raw.githubusercontent.com/anthropics/skills/main/skills/webapp-testing/SKILL.md
```

**2. mcp-builder（MCP Server 生成）**

```bash
mkdir -p .claude/skills/mcp-builder
curl -o .claude/skills/mcp-builder/SKILL.md \
  https://raw.githubusercontent.com/anthropics/skills/main/skills/mcp-builder/SKILL.md
```

**3. Trail of Bits 安全审计**

参考 [trailofbits/skills](https://github.com/trailofbits/skills)，将 SKILL.md 复制至 `.claude/skills/` 对应目录。
