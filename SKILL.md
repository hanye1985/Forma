---
name: auto-docs-to-forma
description: |
  自动从对话上下文整理帮助手册内容，并通过Forma平台生成在线手册。
  触发关键词包括"生成手册"、"创建在线手册"、"整理文档并发布"、"生成帮助中心"、"发布用户手册"、"制作产品文档"等。
  工作流程：从对话历史提取准确内容 → 整理成结构化文档 → 上传到Forma → 生成并发布在线手册。
---

# 智能文档手册生成器

自动分析用户与Agent的对话历史，提取准确内容，整理成结构化文档，并通过Forma平台一键生成精美的在线手册。

## 触发条件

在以下情况下使用此技能：
- 用户说"生成手册"、"创建在线手册"、"发布文档"
- 用户希望"把对话整理成帮助文档并发布"
- 用户需要"产品文档中心"、"用户帮助中心"
- 用户表示"项目完成，需要交付文档"

## 工作流程

**对话 → 提取 → 文档 → Forma → 手册**

### Phase 1: 从对话提取准确内容

#### 步骤 1：分析对话历史

按时间顺序回顾整个对话，**严格依据对话上下文**提取以下信息：

**用户意图与需求**
- 初始请求：用户最初要求构建什么？
- 解决的问题：是什么痛点驱动了这次创造？
- 成功标准：如何衡量项目成功？

**设计与开发历程**
- 关键迭代：方向的重大转变
- 决策点：为什么选择某些方法
- 范围变更：添加或移除的功能

**技术实现**
- 架构：高层结构和组件
- 使用的技术：语言、框架、库
- 文件结构：关键文件及用途
- 依赖项：外部需求和配置

**功能与特性**
- 核心功能：创造物的主要能力
- 用户工作流：分步交互说明
- 边界情况：特殊场景处理

> ⚠️ **重要原则**：提取内容必须完全基于对话上下文，**禁止凭空发挥、擅自捏造**。如果某些信息在对话中未提及，标记为`[对话中未涉及]`，不得编造。

#### 步骤 2：构建文档结构

根据创造物类型选择模板：

**模板 A：软件/工具文档**
```markdown
# {项目名称}

> {一句话描述核心价值}

## 简介
{是什么、解决什么问题、适合谁}

## 功能特性
- 核心功能
- 特色亮点

## 快速开始
### 安装
### 基本使用

## 详细文档
## 技术栈
## 项目结构
## 常见问题
```

**模板 B：用户操作手册**
```markdown
# {产品} 用户手册

## 目录
1. 产品介绍
2. 开始使用
3. 核心功能
4. 使用场景
5. 常见问题

## 产品介绍
### 这是什么？
### 适合谁？
### 能解决什么问题？

## 开始使用
### 环境要求
### 安装步骤
### 首次配置

## 核心功能
{每个功能的用途、操作步骤、注意事项}

## 使用场景
{详细操作流}

## 常见问题
```

**模板 C：API/技术文档**
```markdown
# {API名称} 文档

## 概览
- 基础URL
- 认证方式

## 端点列表
{每个端点的URL、描述、参数、响应示例}

## 错误处理
## 限流规则
## 变更日志
```

#### 步骤 3：内容准确性自检

生成文档前，必须确认以下检查项：

- [ ] **准确性**：所有技术细节都与对话记录一致
- [ ] **零捏造**：没有添加对话中不存在的信息
- [ ] **完整性**：对话中提及的所有功能都有描述
- [ ] **可追溯性**：每个事实都能在对话中找到出处

### Phase 2: Forma发布流程

#### Step 1 — 登录：发送验证码

主动询问：
> 请输入你的邮箱地址，我来帮你发送登录验证码。

用户输入邮箱后，执行：
```bash
curl -s -X POST "https://www.forma123.com/api/bff/auth/code/init" \
  -H "Content-Type: application/json" \
  -d '{"channel": "email", "target": "用户邮箱"}' \
  -c /tmp/forma_cookies.txt
```
从响应中提取 `flow_id`，保存备用。

#### Step 2 — 登录：验证验证码

主动询问：
> 验证码已发送，请输入你收到的验证码。

用户输入验证码后，执行：
```bash
curl -s -X POST "https://www.forma123.com/api/bff/auth/code/verify" \
  -H "Content-Type: application/json" \
  -d '{"flow_id": "上一步的flow_id", "code": "用户输入的验证码"}' \
  -c /tmp/forma_cookies.txt -b /tmp/forma_cookies.txt
```
登录成功后 session cookie 自动保存到 `/tmp/forma_cookies.txt`，后续所有请求加 `-b /tmp/forma_cookies.txt`。

#### Step 3 — 创建项目

根据文档主题自动推断项目名称（如对话涉及"Python数据处理工具" → 项目名 `Python数据处理工具`），自动生成项目 code（如将名称转为拼音首字母或时间戳，例如 `python-data-tool-20260418`）。

主动询问：
> 项目名称将使用「{名称}」，项目访问码（code）默认为 `{code}`，你可以直接回车确认，或输入自定义 code（只允许字母、数字、短横线）。

用户确认或输入 code 后，执行：
```bash
curl -s -X POST "https://www.forma123.com/api/bff/projects" \
  -H "Content-Type: application/json" \
  -b /tmp/forma_cookies.txt \
  -d '{"name": "项目名称", "code": "项目code", "enable_kb": true}'
```
从响应中提取 `project_id` 和 `code`，**必须保存备用**（Step 6 返回访问链接时需要使用 `code` 构造 URL）。

#### Step 4 — 上传文档

将Phase 1生成的文档保存为临时文件，然后立即上传：

```bash
# 保存生成的文档内容为临时文件
cat > /tmp/manual.md << 'EOF'
生成的文档内容
EOF

# 上传文档
curl -s -X POST "https://www.forma123.com/api/bff/projects/{project_id}/documents" \
  -b /tmp/forma_cookies.txt \
  -F "file=@/tmp/manual.md"
```

保存返回的 `document_id`。

> 注意：不需要询问用户是否有其他文档，直接进行下一步。

#### Step 5 — 触发一键生成并轮询发布

**5.1 触发一键生成**

用 `document_id` 创建一键生成模块（该模块是触发器，生成后会被软删除，不需要跟踪它的状态）：
```bash
curl -s -X POST "https://www.forma123.com/api/bff/projects/{project_id}/modules/generate" \
  -H "Content-Type: application/json" \
  -b /tmp/forma_cookies.txt \
  -d '{
    "template_code": "one_click",
    "document_ids": ["document_id"],
    "params": {},
    "config_snapshot": {}
  }'
```

**5.2 轮询检查并发布页面**

一键生成触发后，约需 1 分钟才会出现第一个子页面；第一个页面出现后，后续每隔约 10 秒生成一个，共 5~15 个。

轮询方式：**不使用长时间 sleep 阻塞**，每隔 60 秒执行一次：

```bash
# 获取模块列表
curl -s "https://www.forma123.com/api/bff/projects/{project_id}/modules?include_hidden=true" \
  -b /tmp/forma_cookies.txt
```

检查逻辑：
1. 解析响应，获取所有模块（子页面）列表
2. 对每个模块检查：`data.runtime.artifacts.final` 是否为非空数组
3. 统计：总页面数、已发布数、待发布数（有final但未发布）、生成中数

向用户输出当前状态摘要：
```
[轮询 #3] 已发现 5 个页面 | 已发布 2 | 生成中 3 | 无新页面 8s
```

**发布条件**：
- `final` 非空 → 立即发布（无论 `ui_status` 是 `completed` 还是 `waiting_input`）
- `ui_status == "failed"` 且 `final` 为空 → 记录失败，跳过
- 其他 → 继续等待

发布接口：
```bash
curl -s -X PUT "https://www.forma123.com/api/bff/projects/{project_id}/modules/{module_id}" \
  -H "Content-Type: application/json" \
  -b /tmp/forma_cookies.txt \
  -d '{"is_published": true}'
```

每成功发布一个页面，立即输出：
```
✓ 已发布：「页面名称」
```

**5.3 完成确认（必须执行）**

继续轮询，直到满足**终止条件**：
- 连续 180 秒没有新模块出现，且所有已知页面均已发布或失败

如果发现有遗漏页面（有final但未发布），继续执行发布操作，直到所有页面都发布完成。

**重要**：满足终止条件后，**必须**进入 Step 6 返回访问链接。无论发布成功多少页面，都必须执行 Step 6。

#### Step 6 — 最终检查并返回访问链接（强制步骤）

> ⚠️ **此步骤为强制步骤，必须执行并返回访问链接，不能跳过。**

**最终检查步骤**：

在返回访问链接之前，执行一次最终检查：

```bash
# 最终检查：获取完整的模块列表
curl -s "https://www.forma123.com/api/bff/projects/{project_id}/modules?include_hidden=true" \
  -b /tmp/forma_cookies.txt
```

检查清单：
- [ ] 所有模块都已出现在列表中
- [ ] 所有有final的模块 `is_published` 都为 `true`
- [ ] 没有处于生成中状态的模块

**检查结果处理**：

- **检查通过**：立即输出访问链接（见下方格式）
- **检查不通过**：继续轮询等待，每隔 30 秒重新检查，直到通过

**访问链接输出格式（必须严格遵循）**：

检查通过后，**必须**向用户输出以下内容：

```
页面已全部生成并发布！你的手册访问地址是：

https://www.forma123.com/@PROJECT_CODE
```

其中 `PROJECT_CODE` 替换为 Step 3 中保存的项目 code。

**备用输出机制**：

如果由于任何原因无法完成最终检查，**仍然必须**向用户输出项目访问链接：

```
手册生成流程已完成。你的项目访问地址是：

https://www.forma123.com/@PROJECT_CODE

注意：部分页面可能仍在生成中，请稍后刷新页面查看。
```

> 🔴 **关键要求**：Step 6 的访问链接输出是强制性的，**无论如何都必须返回链接给用户**，这是整个流程的最后一步，不能遗漏。

## Cookie 管理

- 所有 curl 命令统一使用 `-c /tmp/forma_cookies.txt -b /tmp/forma_cookies.txt` 管理 session
- 登录后 cookie 自动持久化，无需用户手动处理
- 如果中途出现 401，重新执行 Step 1-2 重新登录

## 示例场景

### 示例：Python工具 + 在线手册

**场景**：用户与Agent协作开发了一个数据处理脚本，希望生成文档并发布

**执行**：
1. 分析对话，提取功能说明、使用方法、技术栈（严格依据对话内容）
2. 生成完整的README风格文档
3. 保存为临时文件
4. 引导用户登录Forma
5. 创建项目（**保存 project_id 和 code**），上传文档
6. 一键生成手册
7. 轮询所有页面并逐一发布
8. 最终检查确认所有页面已发布
9. **返回访问链接（强制性步骤，不能遗漏）**

**流程检查清单**：
- [ ] Step 3 保存了 project_id 和 project_code
- [ ] Step 5 完成了文档上传
- [ ] Step 5.3 完成确认后进入了 Step 6
- [ ] **Step 6 返回了访问链接 `https://www.forma123.com/@{project_code}`**

## 附录：Forma API 参考

### 基础信息
- **Base URL**: `https://www.forma123.com`
- **BFF路径**: `/api/bff/...`
- **BFF流式路径**: `/api/bff-stream/...`（SSE接口）
- **鉴权**: Cookie（登录后自动携带）

### 响应结构约定

后端标准响应格式：
```json
{
  "code": 0,
  "message": "success",
  "data": { ... }
}
```

列表接口：
```json
{
  "data": { "list": [...], "total": 0 }
}
```

### 核心接口

| 功能 | 方法 | 路径 |
|------|------|------|
| **认证** |
| 发送验证码 | POST | `/api/bff/auth/code/init` |
| 验证登录 | POST | `/api/bff/auth/code/verify` |
| 获取会话 | GET | `/api/bff/auth/session` |
| 登出 | POST | `/api/bff/auth/logout` |
| **项目管理** |
| 获取项目列表 | GET | `/api/bff/projects` |
| 获取项目详情 | GET | `/api/bff/projects/detail?project_id=xxx` 或 `?code=xxx` |
| 创建项目 | POST | `/api/bff/projects` |
| 更新项目 | PUT | `/api/bff/projects/{project_id}` |
| 删除项目 | DELETE | `/api/bff/projects/{project_id}` |
| **模块管理** |
| 获取模块列表 | GET | `/api/bff/projects/{project_id}/modules?include_hidden=true` |
| 获取模块详情 | GET | `/api/bff/projects/{project_id}/modules/{module_id}` |
| 创建模块（触发生成） | POST | `/api/bff/projects/{project_id}/modules/generate` |
| 更新模块 | PUT | `/api/bff/projects/{project_id}/modules/{module_id}` |
| 删除模块 | DELETE | `/api/bff/projects/{project_id}/modules/{module_id}` |
| 模块SSE流 | GET | `/api/bff-stream/projects/{project_id}/modules/{module_id}/stream` |
| **Artifact** |
| 获取Artifact列表 | GET | `/api/bff/projects/{project_id}/modules/{module_id}/artifacts` |
| 获取单个Artifact | GET | `/api/bff/projects/{project_id}/artifacts/{artifact_id}` |
| 更新Artifact | PUT | `/api/bff/projects/{project_id}/artifacts/{artifact_id}` |
| 确认Artifact | POST | `/api/bff/projects/{project_id}/artifacts/{artifact_id}/confirm` |
| **素材（文档）** |
| 获取文档列表 | GET | `/api/bff/projects/{project_id}/documents` |
| 上传文档 | POST | `/api/bff/projects/{project_id}/documents` |
| 获取文档详情 | GET | `/api/bff/documents/{document_id}` |
| 更新文档可见性 | PUT | `/api/bff/documents/{document_id}` |
| 删除文档 | DELETE | `/api/bff/documents/{document_id}` |
| 下载文档 | GET | `/api/bff/documents/{document_id}/download?inline=true/false` |
| **Human Input** |
| 获取当前输入请求 | GET | `/api/bff/projects/{project_id}/human-inputs/current?module_id=xxx` |
| 提交输入 | POST | `/api/bff/projects/{project_id}/human-inputs/{request_id}/submit` |
| **流水线运行** |
| 获取Run详情 | GET | `/api/bff/projects/{project_id}/runs/{run_id}` |
| 停止Run | POST | `/api/bff/projects/{project_id}/runs/{run_id}/stop` |
| **知识库** |
| 知识检索 | POST | `/api/bff/knowledge/search` |
| 刷新知识库状态 | POST | `/api/bff/projects/{project_id}/knowledge/refresh-status` |
| **Copilot** |
| 获取Copilot配置 | GET | `/api/bff/projects/{project_id}/copilot` |
| 更新Copilot配置 | PUT | `/api/bff/projects/{project_id}/copilot` |
| 获取会话列表 | GET | `/api/bff/projects/{project_id}/copilot/sessions` |
| 创建会话 | POST | `/api/bff/projects/{project_id}/copilot/sessions` |
| 获取消息列表 | GET | `/api/bff/projects/{project_id}/copilot/sessions/{session_id}/messages` |
| 流式发送消息 | POST | `/api/bff-stream/projects/{project_id}/copilot/sessions/{session_id}/messages/stream` |
| 归档会话 | DELETE | `/api/bff/projects/{project_id}/copilot/sessions/{session_id}` |
| **Share页（公开访问）** |
| 获取Share项目数据 | GET | `/api/bff/share/projects/{project_code}` |
| 创建公开Copilot会话 | POST | `/api/bff/share/projects/{project_code}/copilot/sessions` |
| 获取公开会话消息 | GET | `/api/bff/share/projects/{project_code}/copilot/sessions/{session_id}/messages` |
| 公开Copilot流式消息 | POST | `/api/bff-stream/share/projects/{project_code}/copilot/sessions/{session_id}/messages/stream` |
| 下载公开文档 | GET | `/api/bff/share/projects/{project_code}/documents/{document_id}/download` |
| **模块模板** |
| 获取模板定义列表 | GET | `/api/bff/module-template-definitions?is_enabled=true` |
| **Agent定义** |
| 获取Agent定义列表 | GET | `/api/bff/project-agent-definitions` |

### 模块状态说明

| 状态 | 说明 | 处理策略 |
|------|------|----------|
| `pending` | 等待执行 | 继续等待 |
| `running` | 执行中 | 继续等待 |
| `waiting_input` | 等待人工输入 | 如果有final则发布，否则需提交Human Input |
| `completed` | 已完成 | 可以发布 |
| `failed` | 失败 | 跳过 |

### Human Input 处理

当模块状态为 `waiting_input` 且 `final` 为空时：

1. 获取当前输入请求：
```bash
curl -s "https://www.forma123.com/api/bff/projects/{project_id}/human-inputs/current?module_id={module_id}" \
  -b /tmp/forma_cookies.txt
```

2. 提交用户输入：
```bash
curl -s -X POST "https://www.forma123.com/api/bff/projects/{project_id}/human-inputs/{request_id}/submit" \
  -H "Content-Type: application/json" \
  -b /tmp/forma_cookies.txt \
  -d '{
    "module_id": "module_id",
    "payload": {
      "inputs": { "query": "用户输入内容" },
      "action": "continue"
    },
    "submitted_by": "forma-live-user"
  }'
```

> **注意**：`current` 接口 **必须传 `module_id`**，否则同项目多个 `waiting_input` 模块并发时会拿错请求。

---

*版本：v2.1 - 补充完整API文档和状态处理*
