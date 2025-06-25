# Auto-Coder 异步代理运行器

这是一个 Go 实现的启动器，用于简化 git worktree 和 auto-coder 的操作流程。

## 🚀 功能特性

- 🎯 **智能 Markdown 解析** - 使用成熟的 markdown 解析器按标题结构自动分割任务
- 📁 **多种分割模式** - 支持按 H1/H2/H3 标题分割，以及自定义分隔符
- 🔧 **自动 Git Worktree 管理** - 为每个任务创建独立的工作环境
- 📝 **语义化文件命名** - 基于标题内容生成友好的临时文件名
- ⏰ **时间戳标识** - 自动生成时间戳避免冲突
- 🧹 **完整的清理功能** - 支持 worktree 列表查看和批量清理
- 🔗 **无缝集成** - 与 auto-coder.run 完美配合

## 🛠️ 安装

### 从源码构建

```bash
cd async_agent_runner
make build
make install-user  # 安装到 ~/bin
# 或者
make install       # 安装到 /usr/local/bin (需要 sudo)
```

### 确保路径设置

如果使用 `install-user`，请确保 `~/bin` 在你的 PATH 中：

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

## 📖 使用方法

### 分割模式

| 模式 | 说明 | 示例 |
|------|------|------|
| `h1` | 按一级标题分割 (默认) | `# 任务一` |
| `h2` | 按一、二级标题分割 | `# 任务一`<br/>`## 子任务` |
| `h3` | 按一、二、三级标题分割 | `# 任务一`<br/>`## 子任务`<br/>`### 细分任务` |
| `any` | 按指定级别范围分割 | 自定义 min-level 和 max-level |
| `delimiter` | 按自定义分隔符分割 (兼容模式) | `===` 分隔 |

### 基本用法

```bash
# 按 H1 标题分割 (默认模式)
cat task.md | async_agent_runner --model cus/anthropic/claude-sonnet-4 --pr

# 按 H2 标题分割
cat task.md | async_agent_runner --model your-model --split h2 --pr

# 使用自定义分隔符 (兼容原有方式)
cat task.md | async_agent_runner --model your-model --split delimiter --delimiter "===" --pr

# 按指定标题级别范围分割
cat task.md | async_agent_runner --model your-model --split any --min-level 2 --max-level 3 --pr

# 指定不同的基础分支
cat task.md | async_agent_runner --model your-model --from develop --pr
```

### 管理 Worktree

```bash
# 列出所有 worktree
async_agent_runner list

# 只显示由 async_agent_runner 管理的 worktree
async_agent_runner list --only-managed

# 清理所有管理的 worktree
async_agent_runner cleanup

# 清理匹配模式的 worktree
async_agent_runner cleanup --pattern test
```

## 📝 Markdown 文件示例

### 按 H1 分割的示例

```markdown
# 任务一：创建基础组件

创建基本的 React 组件结构。

要求：
- 使用 TypeScript
- 包含基本样式

# 任务二：添加用户认证

实现用户登录和注册功能。

要求：
- JWT 认证
- 密码加密

# 任务三：数据库集成

集成数据库功能。

要求：
- 使用 PostgreSQL
- 包含数据迁移
```

### 按 H2 分割的示例

```markdown
# 主要功能开发

## 用户管理

实现用户的 CRUD 操作。

## 权限系统

实现基于角色的权限控制。

## 数据分析

实现数据统计和可视化。

# 测试和部署

## 单元测试

编写全面的单元测试。

## 部署配置

配置生产环境部署。
```

## 🔧 工作原理

1. **输入解析**: 从标准输入读取 markdown 内容
2. **智能分割**: 使用 markdown 解析器按标题层级分割任务
3. **Worktree 创建**: 为每个任务创建独立的 git worktree
4. **文件生成**: 生成语义化命名的临时 markdown 文件
5. **Auto-coder 执行**: 通过管道执行 `cat file.md | auto-coder.run`
6. **结果输出**: 显示执行状态和结果

## 📁 生成的目录结构

```
../async_agent_runner_workdir/
├── stdin_创建基础组件_20231201123456/     # H1: 创建基础组件
│   └── stdin_创建基础组件.md
├── stdin_添加用户认证_20231201123456/     # H1: 添加用户认证  
│   └── stdin_添加用户认证.md
└── stdin_数据库集成_20231201123456/       # H1: 数据库集成
    └── stdin_数据库集成.md
```

## 💡 新增功能：智能 Markdown 文档分割

### 🎯 核心特性

#### 1. 多文档结构自动检测
- **YAML Front Matter 支持**：自动识别和解析包含 YAML 元数据的多文档结构
- **智能检测算法**：基于文档结构和分隔符模式自动判断是否为多文档
- **元数据提取**：支持从 YAML front matter 中提取标题、优先级、标签等元数据

#### 2. 多种分割模式
```bash
# 按一级标题分割（默认）
cat tasks.md | async_agent_runner --model cus/anthropic/claude-sonnet-4 --split h1 --pr

# 按二级标题分割
cat tasks.md | async_agent_runner --model xxxx --split h2 --pr

# 按 YAML front matter 分割多文档
cat multi-doc.md | async_agent_runner --model xxxx --split frontmatter --pr

# 使用自定义正则模式分割
cat tasks.md | async_agent_runner --model xxxx --split custom --pattern "^## Task:" --pr

# 使用分隔符分割（兼容模式）
cat tasks.md | async_agent_runner --model xxxx --split delimiter --delimiter "===" --pr
```

#### 3. 智能长度管理
```bash
# 设置文档长度约束
cat large-doc.md | async_agent_runner \
  --model xxxx \
  --min-length 100 \
  --max-length 5000 \
  --overlap 200 \
  --pr
```

**功能说明：**
- `--min-length`: 过滤过短的文档片段
- `--max-length`: 自动分割过长的文档
- `--overlap`: 设置文档间重叠内容，保持上下文连续性

#### 4. 高级配置选项

| 参数 | 说明 | 默认值 | 示例 |
|------|------|--------|------|
| `--split` | 分割模式 | `h1` | `h1`, `h2`, `h3`, `any`, `delimiter`, `frontmatter`, `custom` |
| `--delimiter` | 自定义分隔符 | `===` | `---`, `***`, `<!-- split -->` |
| `--min-level` | 最小标题级别 | `1` | 用于 `any` 模式 |
| `--max-level` | 最大标题级别 | `6` | 用于 `any` 模式 |
| `--pattern` | 自定义正则模式 | - | `"^## Task:"`, `"^# Chapter"` |
| `--min-length` | 最小文档长度 | `50` | 字符数 |
| `--max-length` | 最大文档长度 | `10000` | 字符数 |
| `--overlap` | 重叠大小 | `100` | 字符数 |

### 📄 支持的文档格式

#### 1. YAML Front Matter 多文档
```markdown
---
title: 后端开发任务
priority: high
team: backend
tags: [api, database]
---

# 后端开发

开发用户管理 API...

---
title: 前端开发任务
priority: medium
team: frontend
tags: [react, ui]
---

# 前端开发

开发用户界面...
```

#### 2. 标题结构分割
```markdown
# 任务一：数据库设计
设计用户表和订单表...

# 任务二：API 开发
实现 REST API 接口...

## 子任务 2.1：用户认证
实现 JWT 认证...

## 子任务 2.2：权限管理
实现角色权限控制...
```

#### 3. 自定义模式分割
```markdown
介绍内容...

TASK: 创建数据库
设计数据库表结构...

TASK: 实现 API
开发 REST API...

TASK: 编写测试
编写单元测试...
```

### 🔧 使用示例

#### 基础用法
```bash
# 简单的 H1 标题分割
echo "# Task 1\nContent 1\n\n# Task 2\nContent 2" | \
  async_agent_runner --model cus/anthropic/claude-sonnet-4 --pr
```

#### 高级用法
```bash
# 复杂的多文档处理
cat examples/multi_document_example.md | \
  async_agent_runner \
    --model cus/anthropic/claude-sonnet-4 \
    --split frontmatter \
    --min-length 200 \
    --max-length 8000 \
    --overlap 150 \
    --pr
```

#### 自定义分割器
```bash
# 按自定义模式分割
cat project-tasks.md | \
  async_agent_runner \
    --model xxxx \
    --split custom \
    --pattern "^### (Task|Story|Feature):" \
    --pr
```

### 🎯 应用场景

1. **项目管理**：将包含多个任务的文档自动分割为独立的工作项
2. **技术文档**：按章节或主题分割大型技术文档
3. **需求文档**：按功能模块分割产品需求文档
4. **代码审查**：将大型代码审查任务分解为多个小任务
5. **培训材料**：按课程单元分割培训内容

### 📊 性能特性

- **智能检测**：自动识别文档结构，无需手动指定
- **内容保持**：智能分割点选择，保持内容完整性
- **上下文保留**：支持重叠内容，保持上下文连续性
- **元数据保留**：保留并传递文档元数据信息
- **错误处理**：分割失败时自动回退到兼容模式

### 🧪 测试用例

运行测试验证功能：
```bash
cd async_agent_runner
go test ./pkg/markdown -v
```

测试覆盖的场景：
- ✅ 多文档结构检测
- ✅ YAML front matter 解析
- ✅ 各种分割模式
- ✅ 长度约束应用
- ✅ 自定义分割器
- ✅ 重叠内容处理
- ✅ 元数据提取和保留

## 💡 高级功能

### 自定义工作目录

```bash
cat task.md | async_agent_runner --model your-model --workdir ./my_workspace --pr
```

### 组合多种选项

```bash
cat complex_task.md | async_agent_runner \
  --model cus/anthropic/claude-sonnet-4 \
  --split h2 \
  --from develop \
  --workdir ../custom_workdir \
  --pr
```

## 🧪 测试

```bash
# 运行单元测试
go test ./...

# 运行演示
./scripts/demo.sh

# 验证构建
make test
```

## 📊 与原始方式对比

### 原来的繁琐操作

```bash
git worktree add ../ac004 -b ac004 master
cd ../ac004 && cat /path/to/task.md | auto-coder.run --model your-model --pr
```

### 现在的简化操作

```bash
cat /path/to/task.md | ac --model your-model --pr
```

**简化程度：从 2 行复杂命令 → 1 行简单命令**

## 🔍 故障排除

### 常见问题

1. **命令未找到**: 确保二进制文件在 PATH 中
2. **git worktree 失败**: 确保在 git 仓库目录中运行
3. **auto-coder.run 未找到**: 确保 auto-coder.run 已正确安装

### 调试

```bash
# 查看生成的 worktree
ac list

# 查看详细的 worktree 信息
git worktree list

# 手动清理 worktree
ac cleanup
```

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📄 许可证

MIT License

---

**项目状态**: ✅ 生产就绪

> 这个工具将大幅提升你的 auto-coder 工作效率！🚀