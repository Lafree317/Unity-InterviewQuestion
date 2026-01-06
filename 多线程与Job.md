# 多线程与 Job/Burst

## 何时使用 Job System / Burst
- **CPU 密集型**：大批量数据计算（碰撞预筛、路径采样、粒子更新）用 Job + Burst 可显著提升吞吐。
- **数据布局**：使用 `NativeArray/NativeSlice`，避免托管对象引用；结构体应为 blittable，方便 Burst 编译。

## Job 调度注意事项
- 避免在 Job 内访问 UnityEngine 对象或主线程 API；通过 `ComponentData`/纯数据传入，结果回主线程再应用。
- 依赖链使用 `JobHandle` 合并，避免主线程 `Complete` 过早阻塞；可将多 Job 合并到 `JobHandle.CombineDependencies`。

## 与主线程通信
- 生产者 Job 写入 `NativeQueue`/`NativeList`；主线程 `Complete` 后读取并应用到 GameObject。
- UI 或物理修改必须回到主线程，可在 `Update`/`LateUpdate` 轮询 Job 完成后再执行。

## 背景线程与 await 协作
- 后台线程执行耗时 IO/CPU，用 `Task.Run`；完成后通过 `SynchronizationContext.Post`/`UniTask.SwitchToMainThread` 回主线程。
- 避免在非主线程直接操作 Unity 对象，防止崩溃或未定义行为。
