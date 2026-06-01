# 📋 spec.md - 聚合支付新模块骨架构建规范

## 🎯 1. 目标描述 (Context & Goal)
由于 `yudao-cloud` 自带的旧支付模块不适合 Stripe 模式的聚合支付业务，当前任务需要为整个大系统开辟一个**全新的、完全独立**的后端业务子模块，命名为 **`yudao-module-unionpay`**。

---

## 🔍 2. 问题与任务范围 (Scope of Work)

OpenCode 智能体需要进入 `./yudao-cloud` 子模块目录中，执行以下机械化动作：

1. **创建模块目录结构**：
   * 在 `./yudao-cloud/` 根目录下，新建文件夹 `yudao-module-unionpay`。
   * 在其内部建立标准的 Maven 多模块 Java 项目骨架：
     ```text
     yudao-module-unionpay/
     ├── pom.xml
     └── yudao-module-unionpay-biz/ (存放所有核心业务逻辑)
         ├── pom.xml
         └── src/main/java/cn/iocoder/yudao/module/unionpay/
     ```
2. **编写子模块 `pom.xml`**：
   * `yudao-module-unionpay` 作为聚合父 POM。
   * `yudao-module-unionpay-biz` 作为具体的业务实现子模块，必须引入 JDK 21 编译依赖，并依赖 `yudao-framework` 中的必要基础设施（如安全、MyBatis-Plus、Web 核心）。
3. **注册到主项目**：
   * 修改 `./yudao-cloud/pom.xml`，在 `<modules>` 节点中添加 `<module>yudao-module-unionpay</module>`，确保整个大项目在根目录编译时能够联动构建。

---

## 🛡️ 3. 验收标准与不变量 (Acceptance Criteria)

* [ ] **不变量 1**：在执行前，已自动在 `./.agent/history/2026-06-01_prompt.md` 记录 `req-20260601-0001`，在 `2026-06-01_task.md` 中记录了 `task-20260601-0001`。
* [ ] **不变量 2**：`yudao-module-unionpay` 内部的任何 Java 基础配置（如包路径）必须严格遵循 `cn.iocoder.yudao.module.unionpay` 命名空间。
* [ ] **门控校验**：进入 `./yudao-cloud` 目录运行 `./mvnw clean compile -pl yudao-module-unionpay -am`，终端返回 `BUILD SUCCESS`，且不触发任何原有模块的编译中断。

---

## 📥 4. 原始输入追溯 (Raw Prompt Input)
* **Request ID**: `req-20260601-0001`
* **人类指派命令**: "为 yudao-cloud 新增一个独立的 yudao-module-unionpay 模块作为未来 Stripe 聚合支付的纯净开发区，并将其注册到 yudao-cloud 根 pom.xml 的 modules 中，通过编译检查。"
