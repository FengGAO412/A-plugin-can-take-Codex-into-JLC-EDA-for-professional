# 嘉立创EDA Codex Bridge

通过本地 MCP 服务与嘉立创EDA专业版扩展 API，让 Codex 能够读取工程信息、检查原理图与 PCB、检索 API 文档，并执行经过确认的设计操作。

本项目面向希望使用自然语言辅助电子设计的开发者，也可作为学习 MCP、EDA 扩展 API 和 AI 工具集成的参考。

> 社区集成项目，基于嘉立创公开的扩展 API 和官方开源网关实现，与嘉立创及 OpenAI 不存在官方隶属关系。

## 功能

- **工程上下文**：读取当前工程、活动文档、板和原理图页信息。
- **原理图检查**：统计元件、导线和网络，辅助发现重复标记、标记与网络不一致等问题。
- **PCB 检查**：统计元件、走线、过孔、铺铜、区域和网络，并按需运行设计规则检查。
- **API 检索**：搜索随包提供的 API 参考、类型定义、开发指南和文件格式资料。
- **设计操作**：通过扩展 API 执行查询与修改，支持工程和文档目标校验。
- **多窗口支持**：列出已连接的 EDA 窗口，并选择操作目标。
- **紧凑输出**：限制结果长度、数组规模和展开深度，减少不必要的上下文占用。
- **超时处理**：在支持的桥接模式下查询超时请求状态，辅助判断操作是否已经完成。

能够执行哪些设计操作，取决于当前 EDA 版本、开放的扩展 API、文档状态及权限。本项目不承诺自动完成整板设计或替代人工工程审查。

## 工作原理

```text
Codex
  │
  │ MCP / stdio
  ▼
本地 MCP 服务
  │
  │ HTTP / WebSocket
  │ 127.0.0.1:49620–49629
  ▼
Run API Gateway 扩展
  │
  │ 嘉立创EDA扩展 API
  ▼
当前工程、原理图与 PCB
```

MCP 服务会发现或创建本机桥接服务。导入网关扩展后，嘉立创EDA窗口通过 WebSocket 与桥接连接。

本项目提供工具与连接能力。AI 模型、账号和模型服务由用户使用的 Codex 环境提供。

## 环境要求

| 项目 | 要求或说明 |
| --- | --- |
| 操作系统 | 当前安装流程面向 Windows，其他平台尚未系统验证 |
| Codex | 支持本地插件和 MCP 服务的版本 |
| 嘉立创EDA | 支持扩展功能的专业版桌面客户端 |
| Node.js | 18 或更高版本；源码构建目标为 Node 18 |
| npm | 从源码安装依赖、构建和运行测试时需要 |
| 网关扩展 | 随包提供 `run-api-gateway_v1.0.5.eext` |

现有版本的开发环境记录为：

- 嘉立创EDA专业版：`3.2.149.88089769`
- 扩展 API：`0.2.53`
- Run API Gateway：`1.0.5`

这些版本记录不代表对其他版本的兼容性保证。更换 EDA 或网关版本后，应重新验证主要功能。

## 安装

### 1. 获取项目

克隆本仓库，或下载并解压仓库 ZIP。

后续安装命令均在仓库根目录执行，即包含以下文件的目录：

```text
.agents/
plugins/
install-codex-plugin.ps1
```

请保留隐藏目录 `.agents`，其中包含本地插件市场配置。

### 2. 检查 Node.js

在 PowerShell 中执行：

```powershell
node --version
npm --version
```

确保 Codex 启动的环境也能够找到 `node`。安装 Node.js 或修改环境变量后，请重新启动 Codex。

### 3. 构建 MCP 服务

在仓库根目录执行：

```powershell
Push-Location .\plugins\jlceda-codex-bridge
npm ci
npm run build
Pop-Location
```

如果发行包已经包含构建后的 `server/server.mjs`，运行时通常无需重新构建。

### 4. 安装 Codex 插件

添加本地插件市场：

```powershell
codex plugin marketplace add .
```

重新启动桌面客户端，在插件目录中选择 `jlceda-local` 来源，找到并安装 **嘉立创EDA Codex Bridge**。

安装后新建一个聊天，使技能与 MCP 工具加载。

对于支持 `codex plugin add` 的 CLI 版本，也可以执行：

```powershell
codex plugin add jlceda-codex-bridge@jlceda-local
```

仓库还提供安装脚本：

```powershell
.\install-codex-plugin.ps1
```

该脚本使用上述 CLI 安装命令。如果当前 CLI 不支持相应子命令，请使用插件目录完成安装。

### 5. 安装嘉立创EDA网关扩展

1. 启动嘉立创EDA专业版。
2. 打开 **高级 → 扩展管理器 → 导入**。
3. 选择仓库中的扩展文件：

   ```text
   plugins/jlceda-codex-bridge/assets/run-api-gateway_v1.0.5.eext
   ```

4. 根据客户端提示启用扩展，并允许所需的“外部交互”权限。
5. 重新启动嘉立创EDA。
6. 打开需要操作的工程及原理图或 PCB 文档。

Codex 插件和 EDA 网关扩展都需要安装，才能建立完整连接。

### 6. 配置 EDA 程序路径

服务通过环境变量 `JLCEDA_PRO_EXE` 定位嘉立创EDA可执行文件。当前实现保留了开发环境默认路径，其他电脑应显式配置。

下面仅为示例，请替换为实际安装路径：

```powershell
[Environment]::SetEnvironmentVariable(
    "JLCEDA_PRO_EXE",
    "C:\Program Files\LCEDA-Pro\lceda-pro.exe",
    "User"
)
```

设置完成后，重新启动 Codex，使 MCP 服务继承新的环境变量。

该变量用于程序路径检查和启动 EDA，不会代替网关扩展的安装。

### 7. 验证连接

在新的 Codex 聊天中输入：

> 检查嘉立创EDA桥接状态，列出已连接窗口，并读取当前工程和活动文档，不要修改设计。

连接成功后，应能获取已连接窗口及当前文档信息。

## 使用示例

### 查看工程

> 告诉我当前打开的嘉立创EDA工程名称、活动文档类型，以及有哪些原理图页和 PCB。

### 检查原理图

> 检查当前原理图的网络端口、重复标记和网络连接问题，先给出检查结果，不要修改。

### 检查 PCB

> 统计当前 PCB 的元件、走线、过孔和铺铜，运行 DRC，并解释主要问题。

### 检索 API

> 查找创建网络端口的 API，确认参数与返回值，并给出调用示例。

### 执行修改

> 先读取当前选中的走线，确认所属网络和原宽度，再将宽度改为 10 mil，最后回读验证。

### 选择窗口

> 列出已连接的嘉立创EDA窗口，让我选择本次要操作的窗口。

## MCP 工具

| 工具 | 用途 |
| --- | --- |
| `status` | 检查或启动桥接，报告连接状态、程序路径和扩展包状态 |
| `launch` | 启动嘉立创EDA程序 |
| `list_windows` | 列出已连接的 EDA 窗口 |
| `select_window` | 选择后续操作使用的窗口 |
| `get_context` | 读取当前工程和文档上下文 |
| `inspect_design` | 执行结构化设计检查，并按需运行规则检查 |
| `search_api` | 检索随包 API 文档、指南及类型定义 |
| `execute` | 在 EDA 扩展环境中执行 API 代码 |
| `request_status` | 在支持的桥接模式下查询请求状态及迟到结果 |

`execute` 的代码作为以下异步函数的函数体运行：

```javascript
async function (eda) {
  // 使用已确认的扩展 API
  // 返回可 JSON 序列化的结果
}
```

执行代码前应先通过 `search_api` 确认方法、参数、返回值与坐标单位。

## 操作与数据边界

- **修改前保存工程**：API 操作可能直接修改当前设计，目标校验不提供事务或自动回滚。
- **核对操作目标**：修改时应传入当前工程 UUID、文档 UUID 和文档类型守卫，并在完成后回读验证。
- **保留客户端审批**：随包配置为启动程序、选择窗口和执行代码设置了审批提示；实际审批行为由客户端版本与设置决定。
- **仅在可信本机环境使用**：桥接仅监听 `127.0.0.1`，但当前没有独立的身份认证或 Origin 校验，不能阻止其他本机进程访问。
- **不要暴露桥接端口**：不要通过端口转发或反向代理将服务暴露到局域网或互联网。
- **本地桥接不代表模型离线运行**：工具结果会交给 Codex 处理，使用前请确认设计资料适合进入所选 AI 服务。

请求超时只表示未在期限内收到结果，**不代表 EDA 操作已经取消**。

发生修改超时后，应先查询请求状态或检查设计，再决定是否重试，避免重复创建或修改图元。复用其他进程桥接的代理模式不支持查询本进程之外的请求历史。

## 项目结构

```text
.
├── .agents/
│   └── plugins/
│       └── marketplace.json          # 本地插件市场配置
├── plugins/
│   └── jlceda-codex-bridge/
│       ├── .codex-plugin/
│       │   └── plugin.json           # 插件元数据
│       ├── .mcp.json                 # MCP 启动与审批配置
│       ├── assets/                   # 网关扩展及相关许可
│       ├── scripts/                  # 开发与连接检查脚本
│       ├── server/                   # 构建后的 MCP 服务
│       ├── skills/
│       │   └── easyeda-api/          # 操作技能和 API 资料
│       ├── src/                      # MCP 服务源码
│       ├── test/                     # 自动化测试
│       ├── package.json
│       └── package-lock.json
├── install-codex-plugin.ps1
├── SHA256SUMS.txt
└── THIRD_PARTY_NOTICES.md
```

## 开发与测试

进入插件目录：

```powershell
Set-Location .\plugins\jlceda-codex-bridge
```

安装依赖、构建并运行测试：

```powershell
npm ci
npm run build
npm test
```

普通测试覆盖核心逻辑及 MCP 服务基本调用，不要求连接真实 EDA。

验证真实 EDA 连接：

```powershell
npm run test:live
```

运行前需要：

- 在嘉立创EDA中启用网关扩展。
- 保持 EDA 窗口打开并已连接。
- 打开一个受支持的活动设计文档。

如果没有窗口连接，脚本会报告跳过；跳过不能视为真实连接验证通过。

修改本地源码后，需要重新构建，并刷新或重新安装插件，使客户端加载更新后的安装副本。

## 常见问题

### Codex 找不到插件工具

确认插件已安装并启用，随后新建聊天。若仍未加载，请检查 MCP 启动日志，以及 Codex 环境能否找到 `node`。

### 桥接运行，但连接窗口数为 0

确认网关扩展已导入、启用并获得外部交互权限，然后重新启动嘉立创EDA。

### 提示找不到嘉立创EDA程序

检查 `JLCEDA_PRO_EXE` 是否指向实际存在的可执行文件。修改环境变量后，需要重新启动 Codex。

### 同时打开多个 EDA 窗口

先调用 `list_windows`，再通过 `select_window` 明确选择目标，随后重新读取上下文。

### API 不存在或参数不匹配

使用 `search_api` 核对当前接口，并确认 EDA 与参考文档的版本。不要直接假定不同版本的接口完全一致。

### 修改请求超时

先检查设计是否已经发生变化。支持时使用 `request_status` 查询结果，确认状态后再决定是否重试。

### 规则检查没有报错，是否就可以投板

仍需人工核对原理图、封装、连接、叠层及制造要求。自动检查结果取决于当前规则配置和检查范围。

## 贡献

欢迎通过 Issue 和 Pull Request 提交问题、改进安装兼容性、完善文档或补充测试。

报告问题时，请提供：

- 操作系统、Codex、Node.js 和 EDA 版本。
- 网关扩展版本。
- 可复现步骤、预期结果与实际结果。
- 已脱敏的相关日志。

请勿上传账号凭据、访问令牌或未经授权的设计文件。

## 许可证与第三方资料

本项目原创代码采用 **Apache License 2.0**，完整条款见仓库的 `LICENSE` 文件。

第三方代码、文档、扩展和类型定义分别遵循其原始许可，不因本项目采用 Apache-2.0 而变更。

本项目使用或参考了：

- [easyeda/easyeda-api-skill](https://github.com/easyeda/easyeda-api-skill)：API 参考、开发指南和格式资料，上游技能声明采用 MIT 许可证。
- [easyeda/eext-run-api-gateway](https://github.com/easyeda/eext-run-api-gateway)：官方 API 网关扩展，采用 Apache License 2.0。
- [嘉立创EDA扩展 API 文档](https://prodocs.lceda.cn/cn/api/guide/)。

第三方来源与许可说明见 `THIRD_PARTY_NOTICES.md`。复制或再分发相关材料时，请保留适用的版权声明和许可证。

## 致谢

感谢嘉立创EDA团队公开扩展 API、API 资料和 Run API Gateway，以及 MCP 生态相关开源项目的贡献者。
