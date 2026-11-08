# 基于 DSHA 的 Android AI App 改造与 Skills 共享平台

**产品需求文档（PRD）v0.1 · MVP 开发基线**  
**日期：2026-11-08**  
**状态：可进入技术预研与分阶段开发**  
**项目代号：AppForge（暂定，不作为最终品牌名）**

> **一句话定位：** 用户在 Android 手机上选择本机应用或 APK，使用 DeepSeek Harness + Skills 分析并执行授权修改；改造成功后可以将可复用的方法生成、安装和发布为 Skill。MVP 不建设自有服务端。  
> **产品边界：** 默认不要求 Root、ADB 或电脑；不能承诺任意第三方 App 都能改造、覆盖安装或者保留原有数据。所有执行均受 Android 实际权限、签名及设备兼容性限制。

## 1. 目标与原则

### 1.1 用户目标

1. 在手机上直接选择**已安装应用**，也允许通过系统文件选择器导入 APK/APKS。
2. 输入自然语言需求，优先支持**去除用户可见广告**，随后支持资源文本、界面布局等其他合法、获得授权的修改。
3. 使用本地 Skills 或从社区获取 Skills，按 Skill 运行、跟踪执行、查看结果。
4. 将一次成功改造提炼成通用 Skill 或特定 App 的 Recipe，并向其他人发布。
5. 其他用户可以基于包名、版本和特征自动发现、安装、验证适用方案。
6. 用户可以自主开启**完全访问模式**，而不是永久受到产品层面的权限限制。

### 1.2 工程原则

- **复用 DSHA**：保留 Android 宿主、Ubuntu arm64、DeepSeek Harness 和既有设备桥，不重新开发 Linux 运行时。
- **本地优先**：APK、补丁、构建产物、执行记录保存在手机本地；远程模型只发送完成任务必要的上下文。需要上传内容时展示范围。
- **确定性工具优先，AI 负责决策**：AI 分析和制定计划；反编译、资源处理、DEX 修改、构建、签名尽量使用可重复执行的工具。
- **默认受限，主动放开**：默认禁止未经确认的高风险行为；用户可明确授权完全访问模式。**不将 App 内的“完全访问”误称为 Android Root。**
- **不建设自有后端**：MVP 的 Skills 发现、获取、发布、更新，直接利用 ClawHub 和 GitHub；本机维护索引和缓存。
- **诚实报告结果**：构建成功、安装成功、运行成功、去广告验证成功是四个不同状态；禁止把前一状态说成后一状态。
- **用户数据优先**：涉及卸载原版、覆盖数据、改动系统设置，必须说明影响；不得默默卸载或清空数据。

### 1.3 成功标准（MVP 退出条件）

在**一台无 Root、不开启 ADB 调试的 Android 11+ arm64 真机**上，从应用选择开始，完成一次授权示例 App 的广告改造、重新签名、经用户确认安装、启动验证；能导出一个可复用的 Skill/Recipe，能从 ClawHub 获取一个 Skill 并运行，能通过 ClawHub 作者账号发布并由另一台设备获取。同一改造任务可查看全过程日志和失败原因。

不以“对所有商业 App 的去广告成功率”作为 MVP 验收条件。

## 2. 目标用户与场景

- **普通用户**：不懂反编译，通过选择 App 和自然语言完成有限范围内的改造；不愿开启开发者选项。
- **技术用户**：自行导入 APK、选择分析工具、打开日志、调整补丁规则、授予完整执行权限。
- **Skill 作者**：把可重复的操作发布给社区，维护适用版本并接收反馈。

**主路径 A（改造）**：选择应用 → 查看包信息与风险 → 输入“去除启动广告” → AI 生成方案 → 用户同意执行 → 构建与签名 → 安装前风险提示 → 用户确认 → 验证与保存结果。  
**主路径 B（复用）**：选择应用 → 自动搜索匹配 Skill/Recipe → 查看来源、权限和兼容性 → 安装 Skill → 执行并验证。  
**主路径 C（分享）**：打开已完成任务 → “生成 Skill” → AI 整理 SKILL.md + Recipe + 测试说明 → 校验与脱敏 → 用户预览 → 登录 ClawHub → 发布 → 返回公开地址。

## 3. MVP 范围与明确不做

### 3.1 必须实现（P0）

| 编号 | 功能 | 最低交付范围 |
|---|---|---|
| F01 | 本机应用选择 | 列表、搜索、包名/版本、用户安装应用优先；系统文件选择器导入 |
| F02 | APK 提取与识别 | base APK；识别并保留 split 信息；原始文件 SHA-256、签名摘要 |
| F03 | AI 任务 | 自然语言任务、预设“去广告”、生成执行计划、日志、取消、失败提示 |
| F04 | APK 工具链 | 静态分析、资源/Smali 修改、构建、zipalign、签名、校验 |
| F05 | 安装 | 系统确认安装，显示签名冲突/卸载与数据丢失风险 |
| F06 | Skills 管理 | 本地列表、查看 SKILL.md、安装、停用、卸载、版本锁定 |
| F07 | 在线获取 | ClawHub 搜索/详情/下载 + GitHub URL 导入 |
| F08 | 本地生成 Skill | 从成功任务生成规范 Skill/Recipe，支持编辑、预览、导出 ZIP |
| F09 | 发布 | ClawHub 登录、发布前校验/脱敏、dry-run、正式发布、查看结果 |
| F10 | 权限 | 默认受限、完全访问模式显式开启、关键 Android 权限分开请求 |
| F11 | 任务记录 | 本地保存任务状态、改造内容、产物位置、验证等级、失败原因 |

### 3.2 下一阶段（P1）

- App 兼容性搜索：在本机缓存/下载的 Skills 中，按包名、版本、签名摘要、DEX 特征精确匹配与推荐。
- 自动生成更通用的 Recipe；多个版本差异识别；规则合并、回归验证。
- 附加验证：用户授权无障碍后自动进入页面、检查广告视图与崩溃日志。
- Skill 更新通知、历史版本回退、用户反馈及失效记录；可选匿名成功率聚合（只有经同意且有可用接收平台时）。

### 3.3 MVP 明确不做

- 不开发自有用户账号、Skill 商店服务端、评价系统、支付系统。
- 不承诺静默安装、不使用 Root/ADB 绕过 Android 系统安装签名校验。
- 不自动删除原版应用、不自动迁移第三方 App 私有数据。
- 不分享修改后的商业 APK；默认仅分享 Skill、Recipe、脚本及不含版权资源的配置。
- 不提供破解付费许可、盗取账号/密钥、禁用安全防护等预置功能。
- 不以绕过 App 自身的完整性或服务端校验为 MVP 目标；遇到此类阻断时清楚报告。
- 不自动为所有安装 App 持续后台扫描、改造或上传信息。

## 4. 产品界面与交互

底部导航建议为：**应用｜任务｜Skills｜我的**。

### 4.1 应用页

显示名称、图标、包名、版本、来源（已安装 / 导入文件）、是否多 Split、最近一次改造状态。默认只展示可见的用户安装应用；系统应用需明确开关。支持搜索和手动导入。

点击应用进入详情，展示：APK 大小/结构、Android 版本、签名证书摘要、已发现的本地 Skills、最近改造记录。主按钮“开始改造”。未提取成功的应用提示原因并提供导入文件方式。

### 4.2 改造任务页

输入框支持自然语言及预设操作：去启动广告、去信息流广告、自定义修改。启动前 AI 生成**修改计划卡**，包括预计文件/组件、拟使用工具、可能受影响功能、操作权限、可验证结果、失败风险。用户可修改要求并确认开始。

执行页展示：当前阶段、可折叠日志、预计剩余工作步骤（不是承诺时间）、暂停/取消、失败原因、重新分析。允许查看每次补丁的差异摘要。禁止未经确认的卸载。

### 4.3 结果页

分别展示：**静态构建通过 / 签名校验通过 / 安装通过 / 启动通过 / 目标效果经验证**。未验证的层级用“未测试”，不能显示“成功”。显示输出文件位置、安装按钮、查看日志、重新改造、生成 Skill。

若原版仍在且签名冲突，显示明确警告：**无法直接覆盖；卸载原版可能永久丢失应用私有数据、登录状态及下载内容**，提供“仅保存 APK / 取消安装 / 用户确认后前往系统卸载”的独立选择。

### 4.4 Skills 页

三个页签：**已安装 / 发现 / 我发布的**。Skills 详情显示名称、作者、来源、版本、许可、描述、依赖工具、请求的能力、适用 App/版本、更新日期、风险提示。支持安装、启停、卸载、查看内容、锁定版本。

### 4.5 发布页

从任务结果进入“生成 Skill”；展示内容预览、兼容性、修改清单、测试证据、敏感信息扫描结果。用户登录 ClawHub；在发布前必须明确提示其公开发布许可和不可逆的公开传播风险（ClawHub 当前技能发布按 MIT-0 开放共享）。用户确认后 dry-run，再正式发布；失败可保存草稿重试。

## 5. 详细功能需求与验收标准

### F01 / F02：应用清单、导入和提取

1. 原生 Android 层使用 `PackageManager` 枚举当前可见应用；尊重 Android 11+ 包可见性约束。若确需 `QUERY_ALL_PACKAGES`，独立评估分发渠道及 Play 申报；不因拿不到应用清单而要求开启 ADB。
2. 支持通过 Android Storage Access Framework 导入 `.apk`，并针对 `.apks` / split 集合提供解析接口。应用图标和包信息尽量读取，不支持时给出原因。
3. 提取主 APK 与 `splitSourceDirs` 指向的拆分包；**先校验复制完整性，再开始任何改造**。因权限或 ROM 限制失败时不假装成功。
4. 保存原始 APK 文件的摘要和只读副本；识别包名、`versionCode`、`versionName`、SDK 范围、ABI、证书摘要和 split 列表。
5. 默认把工作目录放在应用私有存储；向共享目录导出由用户主动选择。

**验收**：能够选择并提取至少一个单 APK 示例应用和一个 Split APK 示例应用；提取摘要一致；失败时保留原始 App 不受影响。

### F03 / F04：AI 分析与 APK 改造流水线

流水线采用如下状态：

`CREATED → EXTRACTED → ANALYZING → PLAN_READY → APPROVED → PATCHING → BUILDING → SIGNING → VERIFIED → READY_TO_INSTALL → [INSTALLED] → [RUNTIME_VERIFIED]`

任何阶段可转入 `FAILED` / `CANCELLED`，已生成的原始副本保持不变。

1. 先运行确定性扫描：Manifest、资源、DEX、常见广告 SDK 特征；使用 JADX 提供可读分析，Apktool/smali 或 dexlib2 执行修改。工具链版本固定并留存。
2. AI 生成**结构化补丁计划**，默认展示后才执行。每一步包含输入、预期匹配条件、修改操作、结果校验条件、是否需要网络或额外授权。
3. 优先应用已有 Recipe；若匹配不成功，进入 AI 深度分析。禁止用“命中关键词”直接删除整个 SDK 作为通用策略。
4. 修改时复制工作副本，不直接覆盖原始 APK；逐步记录修改差异、脚本退出码和摘要。
5. 支持有边界的自动修复：对同一失败最多自动重新规划 2 次（可配置）；防止模型无限循环耗费 Token。
6. 重打包后执行 zipalign、apksigner、APK 结构校验；若资源编译/AAPT2 在特定 ARM64 设备不兼容，要清晰记录错误并退回导出，不得报成功。
7. Split APK 必须同包名、版本号、签名证书并保持必须的拆分结构。若当前版本尚未支持某类动态拆分组合，**显式阻止安装并提示不支持**。
8. “其他修改”在 MVP 中至少覆盖一种可验证的资源文字替换或简单 UI 资源变更；非去广告请求仍复用同一流水线。

**验收**：自有/已授权的广告测试 App 在无 PC 情况下完成改造，输出可安装的文件；错误可定位到具体阶段；取消后不会修改原始 APK。

### F05：安装、签名与数据风险

1. 为用户建立**本地改造签名身份**，签名私钥加密保存，支持同一改造包后续更新时复用；签名材料不上传、不写入 Skill；支持用户自行备份密钥（明确风险）。
2. 采用系统 PackageInstaller / 安装界面，请求用户安装授权；支持单 APK 和由多个拆分包构成的合法安装会话。
3. 安装前比较原始 App 与改造 App 证书、包名、版本，判断能否覆盖。不同证书不提供“强制覆盖”虚假选项。
4. 安装动作、卸载原版动作都必须由用户显式确认；安装失败展示系统状态码和可理解原因。
5. 不承诺保留第三方 App 私有数据；可备份的仅是本产品生成的工程、日志、Skill 和 APK 文件。

**验收**：不同签名覆盖安装被正确阻止并展示数据风险；用户取消安装无副作用；在干净环境能通过系统安装器安装。

### F06 / F07：Skills 获取、安装、管理

1. Skill 采用 Agent Skills 的 `SKILL.md` 规范，支持 `scripts/`、`references/`、`assets/`；安装目录按实际 Harness 版本的 Skills Provider 查明，不把路径写死在业务代码中。
2. **首选 ClawHub 作为注册中心**：搜索、浏览、查询详情、选择版本、下载、安装、更新、发布。对接方式优先 CLI（适配 Node.js 运行环境），必要时增加薄 API 适配层。
3. **GitHub 为补充来源**：用户粘贴仓库 URL 和 ref/tag/commit，选择其中 Skill 目录下载、校验并安装；不可执行来源不明的自动安装脚本。
4. 安装过程进行归档检查：路径穿越、软链接越界、解压炸弹、可执行文件与依赖声明、敏感权限请求。安装不等于立刻执行。
5. 记录来源、owner、版本、内容 SHA-256、安装时间、是否锁定版本、启用状态和审核提示。更新须展示差异与新增权限。
6. 用户可以导入本地 Skill ZIP；提供本地禁用/卸载/回退。不得修改 DSHA 内置受管核心组件。

**验收**：在线下载、安装、停用、卸载一个公开 Skill；再次启动仍被 Harness 正确加载；恶意 zip 路径不能逃出目标目录；错误更新可回退。

### F08 / F09：从改造结果生成并发布 Skill

1. 用户只能对自己完成的任务主动点击“生成 Skill”；AI 从任务计划、补丁、工具版本及验证记录中提取方法，不自动公开全部历史对话/日志。
2. 生成两种产物：**通用 Skill**（流程知识、工具调度）和 **Recipe**（针对 App/版本的确定性修改规则）。两者可以单独发布，也可以共同打包。
3. Recipe 必须包含目标包名、可兼容版本、原始签名摘要（可选但推荐）、精确代码/资源匹配条件、预期修改结果、验证说明；不匹配则停止而非盲打补丁。
4. 发布前脱敏检测：禁止包含原始商业 APK、个人路径、API Key、Cookie、账号、密钥、用户文件、实际私有日志。给出逐文件预览、人工确认和导出清单。
5. 通过 ClawHub 作者账户登录与发布（CLI device-code 或经用户提供的 Token）；凭据由安全存储管理。支持草稿、dry-run、发布结果链接、失败原因、版本更新。
6. 发布前明确显示目标注册中心许可证与内容公开性。对于 ClawHub，当前发布条款要求按 MIT-0 共享，用户不接受时允许仅导出 ZIP，而不强制发布。
7. 发布“通过”只代表平台接受上传，**不代表补丁已被其他设备验证**。本地发布页应展示平台审核中、已公开或拒绝等状态（以平台响应为准）。

**验收**：从一条成功任务生成可安装 Skill/Recipe；两台设备分别发布与下载；敏感文件被排除；未授权不能自动发布。

### F10：权限与高级模式（用户决策：允许主动授予完全访问）

必须将**Agent 工具执行授权**与**Android 系统授予的设备权限**分离，不能混为一谈。

| 模式 | 默认 | Agent 能力 | 用户交互 |
|---|---|---|---|
| 受限模式 | **是** | 使用产品封装的白名单分析/构建工具；只允许指定工作目录；社区脚本不自动执行 | 高风险操作单独确认 |
| 监督模式 | 否 | 允许逐条批准扩展命令和第三方脚本 | 每个风险步骤显示命令、访问范围、目标 |
| **完全访问模式** | 否 | 允许 Harness 在当前 **Android 应用 UID 可访问范围内**运行任意 Shell/脚本与工具，取消产品级逐命令审批 | 用户主动开启，展示后果、确认作用范围，可随时关闭 |

**系统权限独立开关**：存储、无障碍、安装未知应用、ADB、Shizuku、Root 分别申请。完全访问模式**不自动申请或获得**其中任何一项，更不会让 proot 的 root 变成 Android UID 0。若用户另行授予 Root/ADB/Shizuku，则实际设备能力以相应授权和系统规则为准。

实现要求：

- 上游 DSHA README 说明 Harness `danger-full-access` 可能是默认模式；**Fork 首次启动必须改为受限模式**，并验证真实生效，不能只改 UI 文字。
- 受限模式应禁用或替换可绕过权限经纪器的通用执行入口，仅暴露受管工具接口。由于 DSHA/Ubuntu/插件与宿主共享 Android 应用 UID，且缺少内核隔离，**受限模式不能宣称具有强沙箱隔离**。
- 用户开启完全访问前，显示风险说明：第三方 Skill 可以运行命令、删除/读取本 App 可访问文件、发起网络请求、泄漏凭据；授权不会解除 Android 系统权限边界。
- 完全访问可设为“仅本次任务”或“保持开启”；保持开启时主界面必须持续显示状态，随时一键关闭。关闭只作用于后续操作；无法自动撤销已执行的操作。
- 设备级破坏性操作（卸载 App、删除用户文件、发表公开内容、变更系统权限）仍要求 Android 实际许可及产品层面的明确用户意图；可选择取消重复确认，但不能假装绕过系统许可。
- 记录模式变更和高风险命令的**脱敏摘要**。恢复出厂类操作、盗取隐私等不属于预设产品功能。

**验收**：新用户默认处于受限模式；用户能主动进入并退出完全访问；应用 UID 权限之外的操作仍失败；模式状态重启后符合用户选择；未经用户同意不发生卸载、发布或数据上传。

### F11：任务记录及离线能力

1. 本地 SQLite 或 JSON 索引保存任务 ID、来源 APK、版本、Skill 版本、当前状态、阶段耗时、错误码、产物路径、签名摘要、验证记录。
2. 日志默认脱敏，支持导出，用户可清理。若任务中断，重启后显示“未完成/可恢复”，而不是静默重试不可逆步骤。
3. 本地工具链、已下载 Skill 和已有模型上下文可离线查看；**AI 使用远程 DeepSeek API、在线搜索 Skill 和发布 Skill 都需要网络**，不能宣称全离线智能改造。

**验收**：退出应用后可查看历史记录；断网可查看已有文件和记录；模型不可达时展示可恢复错误。

## 6. 模块设计与接口契约

### 6.1 架构

```text
Android App（Fork DSHA）
 ├─ AppInventory          列表、包信息、APK 与 split 提取
 ├─ ModificationUI        自然语言输入、计划、进度、结果
 ├─ InstallationGateway   签名冲突分析、PackageInstaller、用户确认
 ├─ SkillManagerUI        浏览、安装、启停、版本与发布
 ├─ PermissionManager     受限/监督/完全访问 + Android 设备授权
 ├─ LocalRepository       SQLite、产物、密钥及任务日志
 └─ AuthenticatedBridge   授权本地 API（复用 DSHA 桥的身份校验模式）
       │
       ▼
 Ubuntu ARM64（DSHA proot / proroot）
 ├─ DeepSeek Harness      Agent 规划、工具选择、Skills 加载
 ├─ ApkToolchainAdapter   JADX / Apktool / dexlib2 / aapt2 / zipalign / apksigner
 ├─ PatchEngine          确定性修改与 diff、验证
 ├─ RecipeMatcher        版本 / 签名 / 特征匹配
 ├─ SkillComposer        从任务生成 SKILL.md + Recipe
 └─ SkillRegistryAdapter ClawHub / GitHub 下载、安装、更新、发布
```

### 6.2 推荐接口（命名和 JSON 为本项目设计，不代表 DSHA 现有接口）

| 接口 | 输入 | 输出/约束 |
|---|---|---|
| `app.list` | 搜索条件、是否含系统 App | 可见应用及提取能力 |
| `app.extract` | 包名、用户授权 | 工作区 URI、base/split、摘要；原始文件只读 |
| `task.create` | APK 引用、用户意图、模式 | taskId、状态 |
| `task.plan` | taskId、可用 Skills | 结构化操作计划、风险和兼容性 |
| `task.approve` | taskId、批准的计划摘要 | 只执行该计划版本 |
| `task.run/cancel` | taskId | 流式阶段事件、日志和结果 |
| `task.artifacts` | taskId | APK/APKS、签名、校验和验证结果 |
| `skill.search/install` | Registry、query、version | 元信息、下载与本地安装结果 |
| `skill.compose` | taskId | SKILL.md、Recipe、脚本、测试、脱敏报告 |
| `skill.publish` | 草稿ID、注册中心、用户确认 | dry-run/发布回执、公开链接或审核状态 |
| `permission.set` | 模式、作用范围、确认令牌 | 生效配置及变更审计 |

**桥实现约束**：仅绑定 `127.0.0.1`，复用/扩展 DSHA 的 token 授权与生命周期校验；不可把设备安装、文件导出和发布能力暴露为无鉴权 HTTP 接口。所有高风险原生操作在 Android 宿主侧核验任务 ID、当前用户授权和目标包名。

### 6.3 推荐本地目录

```text
appforge/
 ├─ tasks/<task-id>/
 │   ├─ input/           # 原始 APK、split；不修改
 │   ├─ analysis/        # 识别结果与计划
 │   ├─ work/            # 反编译及修改副本
 │   ├─ output/          # 构建、对齐、签名后的 APK/APKS
 │   ├─ logs/            # 脱敏操作日志
 │   └─ manifest.json    # 任务状态、版本、校验值
 ├─ skills/              # 本地安装技能，路径由 Provider 适配
 ├─ recipes/             # 单独的版本化 Recipe
 ├─ toolchains/          # 固定版本与校验信息
 └─ registry-cache/      # ClawHub/GitHub 元数据与版本缓存
```

## 7. Skill 与 Recipe 数据格式

### 7.1 Skill 结构

```text
appforge-ad-removal/
 ├─ SKILL.md             # Agent Skills 标准入口
 ├─ recipes/
 │   └─ com.example.reader.json
 ├─ scripts/
 │   └─ apply.py
 ├─ tests/
 │   └─ smoke.yaml
 └─ README.md
```

`SKILL.md` 必须遵守 Agent Skills 规范所规定的 `name`、`description` 等 Frontmatter 格式。`recipes/`、`tests/` 是**本产品扩展**，并非通用 Agent Skills 标准规定的必需目录。

### 7.2 Recipe 示例（拟定 Schema v1）

```json
{
  "schemaVersion": 1,
  "id": "example-reader-launch-ad-1",
  "title": "移除示例阅读器启动广告",
  "target": {
    "packageName": "com.example.reader",
    "versionCodes": [30201],
    "originalSignerSha256": "<cert-sha256-placeholder>",
    "apkStructure": "base-plus-optional-splits"
  },
  "requiredTools": ["apktool", "apksigner"],
  "preconditions": [
    {"type": "resource-exists", "path": "res/layout/activity_splash.xml"}
  ],
  "operations": [
    {"type": "resource-replace", "resource": "@layout/activity_splash", "patchRef": "patches/splash.patch"}
  ],
  "postconditions": [
    {"type": "apk-signature-valid"},
    {"type": "launch-smoke-test", "manualAllowed": true}
  ],
  "validation": {"testedDevices": 1, "adGoneVerified": false},
  "author": "example-author"
}
```

其中 `originalSignerSha256` 是用于识别原包身份的**公开证书摘要**，不是签名私钥。`adGoneVerified: false` 表示仅完成构建或基础运行检查，不能宣称广告已消失。

### 7.3 兼容性算法（按可信度降序）

1. 包名一致、`versionCode` 一致、签名摘要一致、目标资源/DEX 特征一致 → **精确匹配**。
2. 包名一致、版本相近且特征匹配 → **候选方案**，必须重新分析和确认，不自动执行。
3. 包名一致但特征不匹配 → **拒绝套用旧补丁**，进入 AI 分析。
4. 包名未知 → 可执行通用 Skill，但不得当成已验证 Recipe。

任何网络目录返回的“兼容”标签只作为线索，本机必须重新计算校验值。

## 8. 性能、可靠性与测试

### 8.1 测试设备基线

优先真机：Android 11+、arm64-v8a、8 GB RAM、至少 5 GB 可用空间；验证 Android 13/15/16 不同 ROM。6 GB RAM 和低空间场景列为兼容性专项，不能先承诺支持所有设备。DSHA 当前 Standard 环境包含 Ubuntu，APK 解包会额外占用较多空间。

### 8.2 关键质量门禁

- **完整性**：每个改造任务保留原始副本，任务失败和取消时原包 SHA-256 不变。
- **幂等与恢复**：重复点击“执行”不能创建重复安装/发布动作；任务重启不自动重放未知结果的设备操作。
- **权限一致性**：高权限模式只能由用户可见交互启用，不能由 Skill 自己提升；Android 系统权限始终独立。
- **透明性**：运行计划/真实修改/输出产物可追踪；修改成功但实际广告未验证时不得显示“去广告成功”。
- **社区安全**：安装后不自动运行；更新显示内容摘要与风险变化；发布不上传 APK、用户日志或私钥。

### 8.3 必备端到端用例

| Case | 场景 | 预期结果 |
|---|---|---|
| E01 | 单 APK、自有广告示例、无 ADB/Root | 提取 → 计划 → 修改 → 签名 → 系统安装成功 |
| E02 | Split APK 示例 | 正确提取/识别；支持则合法安装，不支持则明确阻止 |
| E03 | 原版签名与改造版不同 | 明确提示不能覆盖与卸载数据风险 |
| E04 | 资源匹配失败 | 不执行不确定补丁；给出阶段与错误 |
| E05 | 执行期间取消、杀进程、断网 | 原包不变，任务状态一致，可恢复或终止 |
| E06 | 从 ClawHub 安装 Skill | 来源、版本、hash 可追踪，Harness 可加载 |
| E07 | 生成并发布 Skill | 脱敏→dry-run→确认→发布→跨设备获取 |
| E08 | 恶意 ZIP 包含 `../` 或软链接越界 | 阻止安装，不覆盖本地文件 |
| E09 | 新安装后尝试危险 Shell | 默认阻止或请求授权；用户明确完全访问后在 UID 范围内运行 |
| E10 | 高权限模式开启但无 Root | 不得假称获得 Root；系统禁止操作返回失败 |

## 9. 研发阶段与任务拆分

### Sprint 0：基座核验（技术风险先行）

- Fork DSHA，锁定上游 commit，阅读 `AGENTS.md`、`docs/security-model.md`、`docs/plugins.md`、`BUILD.md`。
- 构建并安装 DSHA Standard；确认 Ubuntu、Harness、终端、Skills Provider 和桥接服务均正常。
- **调整上游默认 `danger-full-access`**：验证受限模式真实生效及用户切换完整权限。
- 真机测试 Apktool、JADX、aapt2、zipalign、apksigner 的 arm64 可用性与内存压力。

**退出条件**：无 ADB/Root 的手机可以完成指定测试 APK 的一次反编译、重构、签名及手动安装。

### Sprint 1：改造闭环

- AppInventory、APK Extractor、Split 检测、原始文件存储；改造任务状态机及数据库。
- ToolchainAdapter + PatchEngine + 构建签名与 PackageInstaller；结果页和日志。
- 接入 DeepSeek Harness 的任务编排与第一个 `apk-ad-removal` Skill。

**退出条件**：用户从应用列表选中示例 App 并走完整流程，出现错误时不损害原版。

### Sprint 2：Skills 获取与运行

- ClawHub CLI 集成：搜索、详情、下载、安装、更新；GitHub URL 导入。
- 技能安装校验、版本锁定、启用/禁用、权限展示。
- Recipe 匹配器 v1；“应用详情 → 发现适用 Skill”。

**退出条件**：用户可通过手机 UI 获取并执行第三方或自建 Skill。

### Sprint 3：生成与发布

- SkillComposer、Recipe schema、敏感数据扫描、发布草稿。
- ClawHub device-code 登录、dry-run、发布结果回执和链接。
- 两台真机跨设备发布/获取/复用验收。

**退出条件**：完整覆盖“改造 → 沉淀 → 发布 → 获取 → 再执行”。

## 10. 已知风险与决策记录

| 风险 | 处理决策 |
|---|---|
| 改造版重新签名后无法覆盖原版 | 系统签名规则不能绕过；安装前警告，保留原包 |
| 应用数据可能丢失 | 不自动卸载；绝不承诺数据迁移 |
| Split/App Bundle 差异 | 显式识别；兼容性矩阵与受控失败 |
| 加固、混淆、Native/Flutter/Unity、服务端广告 | 允许“不支持/需人工分析”，不强称成功 |
| DSHA 同 UID 无强沙箱 | UI/工具级默认最小化、受控执行；高级用户明确承担风险 |
| 社区 Skill 恶意脚本 | 静态校验、来源记录、差异预览、默认不自动执行、可关闭完全访问 |
| API 模型可能接收代码片段 | 显示数据发送范围、允许用户选择不同模型服务，不默认上传完整 APK |
| ClawHub 平台策略/许可证变化 | 适配器抽象；GitHub 和 ZIP 作为备选出口 |
| 发布内容可能涉及版权或条款 | 用户只分享自己的操作方法与许可内容；不托管未经授权 APK |
| Google Play 权限/分发限制 | `QUERY_ALL_PACKAGES`、`REQUEST_INSTALL_PACKAGES` 按官方政策评估；MVP 可先非 Play 分发 |

## 11. DoD（可开始公开内测的交付清单）

- [ ] 无 Root、无 ADB 的 Android 11+ arm64 真机完成 E01 流程。
- [ ] Single APK / Split APK 的支持范围明确；不支持时失败可理解。
- [ ] App 选择、任务、结果、Skills、我的四个主页面可操作。
- [ ] 原包保全；计划审批；构建、安装与效果验证状态分离。
- [ ] 不同签名不能覆盖时清楚告知风险，绝不自动卸载。
- [ ] 默认受限、监督、完全访问三模式能切换且行为与 UI 一致。
- [ ] ClawHub 获取、安装、管理至少一个 Skill，并被 Harness 加载。
- [ ] 成功任务能生成 Skill/Recipe、导出、完成 ClawHub 发布及跨设备获取。
- [ ] 发布前扫描私密信息，用户能审查所有待发布文件。
- [ ] 全量日志可导出且默认脱敏；失败和取消不损坏输入。
- [ ] 有至少 10 个端到端用例测试记录和一份设备兼容性矩阵。

## 12. 参考资料（开发时应核对最新版本）

- DSHA：<https://github.com/DSH-APP/DSHA>
- DSHA 开发约束：<https://github.com/DSH-APP/DSHA/blob/main/AGENTS.md>
- DSHA 安全模型：<https://github.com/DSH-APP/DSHA/blob/main/docs/security-model.md>
- DeepSeek Harness：<https://github.com/deepseek-ai/deepseek-harness>
- Agent Skills 标准：<https://agentskills.io/specification>
- ClawHub CLI：<https://github.com/openclaw/clawhub/blob/main/docs/cli.md>
- ClawHub 发布：<https://github.com/openclaw/clawhub/blob/main/docs/publishing.md>
- ClawHub 认证：<https://docs.openclaw.ai/clawhub/auth>
- Skills.sh：<https://www.skills.sh/docs>
- Android PackageInstaller：<https://developer.android.com/reference/android/content/pm/PackageInstaller>
- Android 包可见性：<https://developer.android.com/training/package-visibility>
- Android 应用更新签名：<https://developer.android.com/google/play/app-updates>

---

**结论：** 本版本的首要交付不是“所有 APK 都能一键去广告”，而是一个**手机端可持续工作的改造闭环**：用户选 App → AI 执行可追踪修改 → 生成安装产物 → 用户验证 → 形成 Skill/Recipe → 对外发布并可被他人复用。允许用户主动授权完全访问，但不得混淆产品内执行权限与 Android 系统特权。