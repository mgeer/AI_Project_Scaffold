# 项目知识库初始化指南（仅首次执行）

生成以下文件，内容要求简洁精准：

## structure.md
- 有意义的目录层级，每个模块一句话说明职责
- 标注微服务 / SO 库 / 共享组件

## symbol-index.md
- 覆盖：Service 类、核心数据结构、SO 调用封装层、协议解析器
- 格式：`ClassName → src/path/file.cpp:行号`

## dependencies.md
- 微服务间调用关系
- 每个 SO 库的接口清单（函数签名 + 用途）
- 外部依赖（消息队列、数据库、协议格式）

## gotchas.md
- 不能修改的接口及原因
- 已知线程安全问题
- 历史已知坑

## patterns.md
- 编码规范和惯用写法
- 重构时应遵守的模式

## progress.md
- 初始化时留空，重构开始后持续更新
