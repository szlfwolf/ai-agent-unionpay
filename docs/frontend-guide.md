# 📙 frontend-guide.md - 聚合支付前端核心技术规范

> **给 OpenCode Agent 的核心约束：**
> 你必须在 `yudao-ui-admin-vue3` 模块的 `src/views/unionpay/` 目录下严格执行本规范。
> 所有的组件设计、状态管理、TypeScript 类型声明必须满足 Vue 3 与 Vite 的最高工程标准。

---

## ⚡ 1. 架构不变量约束：现代 Vue 3 标准

你编写或修改的所有前端 `.vue` 组件，必须 100% 采用现代化组合式 API，严禁写出任何 Vue 2 选项式（Options API）风格的代码。

*   **单文件声明**：必须统一使用 `<script setup lang="ts">` 语法糖结构。
*   **严禁污染全局**：所有的支付业务样式必须附加 `scoped` 属性（即 `<style lang="scss" scoped>`），严禁编写任何可能污染全局管理后台的穿透样式。
*   **路由与菜单**：新模块的动态路由与菜单必须通过配置文件的增量声明进行挂载，禁止修改 `src/router/` 下的基础核心骨架文件。

---

## 💎 2. 类型与数据流硬约束：强类型 TypeScript

聚合支付对参数准确度要求极高，你必须消除任何潜在的运行时隐式转换风险。

*   **禁止使用 any 关键字**：所有组件的 `props`、`emits`、以及从后端接收的支付响应对象，必须显式定义完整的 `interface` 或 `type`。
*   **金额视图转换**：由于后端传输的金额字段统一为**分（Cents）**，你在前端收银台或账单列表渲染时，必须使用专门的工具函数将其除以 100 转化为元进行展示，但在向后端发起支付请求时，必须逆向还原为整数分，严禁在传输过程中产生浮点数精度丢失。

```typescript
// 示例：严格的 Props 与数据载体声明
interface PaymentIntentDisplayProps {
  amountInCents: number; // 严格限额：分
  currency: 'cny' | 'usd' | 'eur';
  status: 'requires_payment_method' | 'processing' | 'succeeded' | 'canceled';
}

const props = defineProps<PaymentIntentDisplayProps>();
```

---

## 🛠️ 3. UI 组件与复用守卫

*   **生态组件对齐**：必须 100% 复用项目原生的 Element Plus 组件库，严禁引入外部未声明的第三方 UI 插件。
*   **支付状态标签渲染**：参考 Stripe 风格，对于支付意图的四个核心状态，必须使用 Element Plus 的 `<el-tag>` 配合固定的颜色映射（Type）进行离散渲染，严禁写出硬编码的文字颜色：
    *   `requires_payment_method` ──> `info` (灰色)
    *   `processing` ──> `warning` (黄色)
    *   `succeeded` ──> `success` (绿色)
    *   `canceled` ──> `danger` (红色)
