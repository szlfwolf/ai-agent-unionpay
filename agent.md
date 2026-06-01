# 🧭 agent.md - OpenCode 智能体控制地图 (UnionPay Map)

> **最高指令：**
> 你身处 `ai-agent-unionpay` 聚合支付主仓库。
> 本项目弃用 yudao 旧支付模块，完全使用全新的 `yudao-module-unionpay`。
> 接收任务必先读取 `spec.md` 任务书，严格执行以下地图指针、审计铁律与熔断约束。

---

## 🏗️ 1. 仓库即记录系统 (System of Record)

### 📂 目录结构指针 (Directory Roadmap)
*   `./spec.md` ──> **【当前任务书】** 存放当前阶段问题描述与边界要求。
*   `./docs/backend-guide.md` ──> **【后端技术规范】** 聚合支付后端核心技术规范（JDK 21 / Spring Boot 3）。
*   `./docs/frontend-guide.md` ──> **【前端技术规范】** 聚合支付前端核心技术规范（Vue 3 / TS / Vite）。
*   `./.agent/history/` ──> **【审计历史】** 存放按日期分割的 `prompt.md` 和 `task.md`。
*   `./yudao-cloud/yudao-module-unionpay/` ──> **【后端边界】** 核心代码完全收拢在此新模块，严禁污染旧 pay 模块。
*   `./yudao-ui-admin-vue3/src/views/unionpay/` ──> **【前端边界】** 所有管理前端视图完全收拢在此新目录。

---

## 💻 2. 技术栈不变量 (Technical Invariants)



| 领域 | 核心技术栈 | OpenCode 智能体约束边界 (AI Constraint) |
| :--- | :--- | :--- |
| **后端** | JDK 21 + Spring Boot 3 | 限制在 `yudao-module-unionpay` 编码；<br>必须用虚拟线程处理网关 I/O；<br>必须用 `switch` 模式匹配解析多渠道回调；<br>所有 API 传输对象（DTO）必须用 `record` 声明。 |
| **前端** | Vue 3 + TS + Vite | 限制在 `views/unionpay/` 编码；<br>必须采用 Composition API（`<script setup>`）；<br>严格禁止污染公共组件。 |

---

## 📝 3. 不可变审计铁律 (Strict Audit Ledger)

编码前必须执行以下**机械化审计双写**动作，严禁跳过：

*   **Prompt 记录**：提取 `spec.md` 原始输入，分配统一请求 ID `req-YYYYMMDD-XXXX`，追加至 `./.agent/history/YYYY-MM-DD_prompt.md`。
*   **原子 Task 拆解**：必须将当前任务打散为**不可再分的原子操作步骤**。
*   **原子 ID 映射**：必须为每一个原子步骤分配带连字符后缀的独立任务 ID `task-YYYYMMDD-XXXX-NN`（如 `task-20260601-0001-01`）。
*   **看板级追加**：将关联的 `req-id`、原子 Task ID、执行步骤和初始状态（Todo），以 Markdown 表格形式追加至 `./.agent/history/YYYY-MM-DD_task.md`。

---

## 🛠️ 4. 机械化门控与日志熔断 (Enforcement & Log Breakout)

防长日志引发 TPM 错误。必须严格执行以下动作：

*   **编译门控**：后端变更必先运行 `./mvnw clean compile -pl yudao-module-unionpay -am`。
*   **测试守卫**：新增通道或退款必须同步编写 JUnit 5 单元测试。
*   **日志落盘**：所有构建指令必须重定向至本地 `./build.log`。
*   **限制终端**：严禁在主终端直接打印任何全量构建输出。
*   **尾部断言**：仅允许用 `tail -n 30` 读取日志最后 30 行。
*   **状态判定**：依据尾部 `BUILD SUCCESS` 或 `FAILURE` 判定结果。
*   **精准抓取**：失败时仅允许 `grep` 过滤提取前 50 行 `[ERROR]` 或 `fatal` 错误。
*   **硬性熔断**：严禁向大模型思考回路输入超过 100 行的日志。
