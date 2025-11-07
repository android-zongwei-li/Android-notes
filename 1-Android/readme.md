# Android 知识库（1-Android） — 概览与学习路线

本目录用于记录 Android 技术栈的笔记、实战、样例与工程化经验。目标是把“能解决实际问题的知识”组织成：文档 + 可运行示例 + 测试 + CI。将此目录作为展示你专业能力与工程实战经验的入口。

目录索引（建议学习顺序与说明）

- 0-Java/ — Java 基础与常见陷阱（保留关键概念笔记）
- 0-Kotlin/ — Kotlin 语言要点：协程、DSL、扩展、内联与泛型
- 0-基础知识/ — Android 基础（Activity/Service/Intent/生命周期等）
- 1-四大组件/ — 四大组件的原理与实践示例
- 2-UI体系/ — 布局、View、RecyclerView、自定义控件与动画
- 15-Jetpack/ — Architecture Components（ViewModel/LiveData/Room/Paging/WorkManager 等）
- 9-实战/ — 小型实战项目、示例应用与 Case Study（每个示例应包含可运行代码与 README）
- JNI/ — NDK/Native 与 JNI 实践笔记
- 4-性能优化/ — Profiler 使用、内存/CPU/网络优化、启动优化、LeakCanary 案例
- 4-计算机网络/ — OkHttp/Retrofit、缓存、断线重连、WebSocket、gRPC/GraphQL
- 5-IPC机制/ — Binder、AIDL、Messenger、跨进程调试方法
- 3-数据结构与算法/ — 工程常用算法与面试题（更多面向实战）
- 7-三方库/ — 常用库（Retrofit/OkHttp/Glide/Hilt/Coil 等）使用与陷阱
- 存储/ — 文件、SharedPreferences、Room 与 Scoped Storage 实践
- 操作系统/ — Android/Linux 原理（Binder、内存、进程、zygote 等）
- 其他/ — 零散笔记或临时资料
- 00-问答集/、000-Q&A/ — 常见问答与面试题（建议合并并索引）
- 0-模板.md — 笔记模板（问题/解决方案/代码/测试/参考）

如何使用本知识库

1. 学习路线：按目录索引从基础到进阶学习。每阅读一章，尽量把“理论”转换为“小样例”放入 9-实战/ 或 samples/。
2. 实战优先：对核心主题至少产出一个带有单元/集成测试的可运行示例。示例应包含 README（场景、复现步骤、测试方法）。
3. 贡献与维护：新增笔记或样例请遵循 0-模板.md 的格式；外部资料放入 references/ 并注明来源。
4. CI/自动化：为关键示例打开 GitHub Actions（lint + 单元测试），并在 README 显示 badge（构建/测试 状态）。

短期建议（优先级）

- 为每个 top-level 目录添加 README（说明目录目标与任务清单）。
- 合并并清理重复的问答目录（00-问答集 / 000-Q&A）。
- 将重要外部 HTML 转为 Markdown 并集中到 references/。
- 新建 samples/ 并完成至少 3 个可运行示例（含单元/集成测试）并配置 CI。

长期目标（3 个月）

- 每个核心主题至少保留 1 篇 Case Study + 1 个 samples（含 CI）。
- 增加 ADRs/（architecture-decisions/）记录关键架构决策。
- 建立自动化构建发布流程，能生成 signed AAB 并示例上传步骤。

贡献指南快速入口

- LICENSE（如计划开源请添加）、CONTRIBUTING.md、ISSUE_TEMPLATE、PULL_REQUEST_TEMPLATE。

联系方式

- 仓库维护者: android-zongwei-li

---

（此文件为自动生成的仓库索引与使用说明，后续可由脚本或 PR 更新。）