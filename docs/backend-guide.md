# 📘 ai-agent-guide.md - 聚合支付后端核心技术规范

> **给 OpenCode Agent 的核心约束：**
> 你必须在 `yudao-module-unionpay` 模块内严格执行本规范。
> 所有的接口、状态机设计与并发处理必须满足 JDK 21 与 Spring Boot 3 的原生最高标准，严禁引入历史债务代码。

---

## 🏎️ 1. 高并发吞吐约束：虚拟线程 (Virtual Threads)

聚合支付涉及大量对第三方支付网关（Stripe, PayPal, 微信, 支付宝）的 HTTP I/O 阻塞调用。你必须遵循以下虚拟线程专有约束：

*   **禁止手动创建线程池**：严禁显式声明 `ThreadPoolTaskExecutor` 或原生 `ExecutorService` 线程池。
*   **原生拦截**：所有异步通知（Webhook）派发与耗时对账任务，必须直接依赖由 Spring Boot 3 配置好的 Tomcat/TaskExecutor 虚拟线程池进行自动调度。
*   **禁止使用 synchronized 关键字**：为防止虚拟线程在执行 I/O 阻塞时发生线程固定（Pinning）导致性能塌陷，任何涉及锁的场景**必须**使用 `java.util.concurrent.locks.ReentrantLock` 替代。

---

## 💎 2. 数据载体约束：不可变 Records (Immutable Data)

为了确保资金流数据的安全性和线程安全，所有新定义的传输对象必须使用不可变的 `record` 声明：

*   **API 门控 DTO**：所有前端入参（Request DTO）与出参（Response VO）必须 100% 采用 `record`。
*   **禁止使用 @Data 属性污染**：严禁在 `record` 上附加 Lombok 的 `@Data` 或 `@Setter` 注解。
*   **金额严禁使用浮点数**：参考 Stripe 规范，所有涉及金额（Amount）的字段必须使用 `Long` 类型，单位一律为**分 (Cents)**，严禁写出 `double` 或 `float`。

```java
// 正确示例规范
public record PaymentIntentCreateReqVO(
    @NotNull(message = "金额不能为空") Long amount, // 单位：分
    @NotBlank(message = "币种不能为空") String currency, // 示例: "cny", "usd"
    @NotBlank(message = "支付渠道不能为空") String channelId
) {}
```

---

## 🔄 3. 核心业务约束：Stripe 风格支付意图状态机

你必须实现基于 `PaymentIntent`（支付意图）的对象生命周期管理，取代传统的多表混乱订单。

### 📊 状态不变量 (State Invariants)
支付意图的演进状态必须严格限定为以下四个：
1. `requires_payment_method` (等待用户选择支付方式)
2. `processing` (网关扣款处理中)
3. `succeeded` (支付成功 - 终态)
4. `canceled` (已取消 - 终态)

### 🔀 状态分发硬约束（必须使用 JDK 21 模式匹配）
在解析各个支付渠道（如支付宝、微信、Stripe）返回的异步通知或同步轮询结果时，你**必须**使用 JDK 21 的 `switch` 模式匹配进行强类型智能分发：

```java
public class PaymentNotifyResolver {
    
    public void resolveChannelNotify(ChannelNotifyData rawData) {
        // 必须使用 JDK 21 强类型模式匹配分发，严禁使用一系列的 if-else instanceof
        switch (rawData) {
            case AliPayNotifyData ali -> processAliPayStatus(ali);
            case WeChatNotifyData wx -> processWeChatStatus(wx);
            case StripeNotifyData stripe -> processStripeStatus(stripe);
            default -> throw new IllegalArgumentException("未知的底层支付渠道通知: " + rawData);
        }
    }
}
```

---

## 🛡️ 4. 零资损断言与自愈门控 (Test & Self-Healing Guard)

*   **Mock 断言机制**：你编写的每一个支付路由业务类，必须在 `src/test/java` 下同步输出对应的 JUnit 5 单元测试。必须结合 `Mockito` 对第三方远程 Gateway 的 HTTP 响应进行 100% 成功与失败分支的双向模拟（Mock）。
*   **核心断言不变量**：测试用例中必须包含对 `PaymentIntent` 状态机流转不变量的 `assertEquals` 断言，状态不匹配严禁推入 Git 仓库。
