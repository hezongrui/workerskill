# 测试反模式指南

**必读场景：** 写测试、加 mock、忍不住想给生产代码加测试专用方法时。

## 核心原则

**测真实行为，不测 mock 行为。**  
mock 是隔离工具，不是被测对象。

---

## 反模式 1：测 mock 行为

? 错误：
```typescript
test('渲染侧边栏', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```
你在验证 mock 存在，不是组件行为。

? 正确：
```typescript
test('渲染侧边栏', () => {
  render(<Page />);
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});
```

### 决策门
```
在断言任何 mock 元素之前:
  问: "我在测真实组件行为，还是仅仅验证 mock 存在？"
  如果只是在验证 mock 存在:
    停止 - 删掉断言或取消 mock，改测真实行为
```

---

## 反模式 2：给生产类加测试专用方法

? 错误：
```typescript
class Session {
  async destroy() {  // 只有测试用！
    await this._workspaceManager?.destroyWorkspace(this.id);
  }
}
```

? 正确：把清理逻辑放到 test utilities
```typescript
// test-utils/cleanup.ts
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}
```

### 决策门
```
在往生产类加任何方法之前:
  问: "这个方法只有测试会用吗？"
  如果是:
    停止 - 不要加，放到 test utilities 里

  问: "这个类拥有这个资源的生命周期吗？"
  如果不是:
    停止 - 这个方法不该放在这个类里
```

---

## 反模式 3：不理解依赖就 mock

? 错误：
```typescript
test('检测重复服务器', () => {
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));
  // 这个 mock 阻止了配置写入，导致后续断言失效
});
```

? 正确：在正确层级 mock
```typescript
test('检测重复服务器', () => {
  vi.mock('MCPServerManager'); // 只 mock 慢的外部启动
  await addServer(config);     // 配置写入保留
  await expect(addServer(config)).rejects.toThrow('duplicate');
});
```

### 决策门
```
在 mock 任何方法之前:
  1. 问: "真实方法有什么副作用？"
  2. 问: "当前测试依赖这些副作用吗？"
  3. 问: "我完全理解这个测试需要什么吗？"

  如果依赖副作用:
    在更低层级 mock（实际慢的/外部的操作）
    或用保留必要行为的 test double
    不要 mock 测试依赖的高层方法

  如果不确定:
    先用真实实现跑一遍测试，观察实际需要什么
    然后再在正确层级加最小 mock
```

---

## 反模式 4：不完整 mock

? 错误：
```typescript
const mockResponse = {
  status: 'success',
  data: { userId: '123' }
  // 漏了 metadata，下游代码访问时崩溃
};
```

? 正确：完整镜像真实 API 结构
```typescript
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
};
```

**铁律：** mock 必须包含真实 API 的完整数据结构，不能只填当前测试用到的字段。

### 决策门
```
在创建 mock 响应之前:
  检查: "真实 API 响应包含哪些字段？"
  行动:
    1. 查看文档或示例中的真实 API 响应
    2. 包含下游可能消费的所有字段
    3. 验证 mock 结构与真实响应 schema 完全一致
```

---

## 反模式 5：把测试当事后工作

? 错误：
> "功能写完了，准备进入测试阶段。"

? 正确：测试是实现的组成部分。没有测试，不算完成。

### 决策门
```
在声称"实现完成"之前:
  问: "测试写了吗？所有测试都通过了吗？"
  如果答案为否:
    停止 - 实现尚未完成，回到 TDD 循环
```

---

## 何时 Mock 变得过复杂

**警告信号：**
- mock setup 代码比测试逻辑还长
- 必须 mock 一切才能让测试通过
- mock 一变测试就坏

**解决方案：** 用真实组件的集成测试通常比复杂的 mock 更简单、更可靠。

---

## TDD 如何防止这些反模式

1. **先写测试** → 强迫你思考"到底在测什么"
2. **看着它失败** → 确认测的是真实行为，不是 mock
3. **最小实现** → 不会偷偷加测试专用方法
4. **先用真实依赖** → 在 mock 之前先看到测试实际需要什么

**如果你在测 mock 行为，你就违反了 TDD** —— 你在没看到测试对真实代码失败之前就加了 mock。

---

## 快速参考

| 反模式 | 修复方式 |
|--------|----------|
| 断言 mock 元素 | 测真实组件或取消 mock |
| 生产类加测试专用方法 | 移到 test utilities |
| 不理解依赖就 mock | 先理解依赖，最小化 mock |
| 不完整 mock | 完整镜像真实 API |
| 测试当事后工作 | TDD — 测试先行 |
| mock 过于复杂 | 考虑集成测试 |

---

## 红灯信号

- 断言检查 `*-mock` test ID
- 生产类里有只在测试文件被调用的方法
- mock 代码占测试的 50% 以上
- 去掉 mock 后测试就失败
- 说不清为什么需要 mock
