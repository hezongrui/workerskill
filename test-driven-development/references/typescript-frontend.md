# TypeScript 前端 TDD 指南

## 红-绿-重构示例（工具函数）

### 红

```typescript
// src/utils/formatPrice.test.ts
import { describe, it, expect } from 'vitest'
import { formatPrice } from './formatPrice'

describe('formatPrice', () => {
  it('formats positive price with currency', () => {
    expect(formatPrice(100)).toBe('$100.00')
  })

  it('throws on negative price', () => {
    expect(() => formatPrice(-1)).toThrow('price must be non-negative')
  })
})
```

```bash
npm test src/utils/formatPrice.test.ts
# 预期：FAIL - formatPrice is not defined
```

### 绿

```typescript
// src/utils/formatPrice.ts
export function formatPrice(price: number): string {
  if (price < 0) throw new Error('price must be non-negative')
  return `$${price.toFixed(2)}`
}
```

---

## React 组件 TDD 示例

### 红

```tsx
// src/components/Counter.test.tsx
import { render, screen, fireEvent } from '@testing-library/react'
import { Counter } from './Counter'

test('increments count when clicked', () => {
  render(<Counter initial={0} />)
  const button = screen.getByRole('button', { name: /increment/i })
  fireEvent.click(button)
  expect(screen.getByText('Count: 1')).toBeInTheDocument()
})
```

### 绿

```tsx
// src/components/Counter.tsx
import { useState } from 'react'

export function Counter({ initial }: { initial: number }) {
  const [count, setCount] = useState(initial)
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  )
}
```

**原则**：用 `@testing-library/react` 测**用户可见行为**，不测内部 state 或 DOM 结构细节。

---

## API 层 TDD（MSW 隔离）

### 红

```typescript
// src/services/userApi.test.ts
import { http, HttpResponse } from 'msw'
import { server } from '../mocks/server'
import { fetchUser } from './userApi'

test('returns user data on success', async () => {
  const user = await fetchUser('1')
  expect(user.name).toBe('Alice')
})

test('throws on 404', async () => {
  server.use(
    http.get('/api/users/1', () => {
      return new HttpResponse(null, { status: 404 })
    })
  )
  await expect(fetchUser('1')).rejects.toThrow('User not found')
})
```

### 绿

```typescript
// src/services/userApi.ts
export async function fetchUser(id: string) {
  const res = await fetch(`/api/users/${id}`)
  if (res.status === 404) {
    throw new Error('User not found')
  }
  return res.json()
}
```

---

## 项目结构

```
src/
├── components/
│   ├── Counter.tsx
│   └── Counter.test.tsx
├── utils/
│   ├── formatPrice.ts
│   └── formatPrice.test.ts
├── services/
│   ├── userApi.ts
│   └── userApi.test.ts
├── mocks/
│   ├── handlers.ts
│   └── server.ts
└── e2e/
    └── checkout.spec.ts      # Playwright
```

## 常用命令

```bash
# 运行全部测试
npm test

# 运行单个文件
npm test src/components/Counter.test.tsx

# 带覆盖率
npm test -- --coverage

# 只运行单元测试（排除 e2e）
npm test -- --testPathIgnorePatterns=e2e

# E2E（Playwright）
npx playwright test
```

## Mock 最佳实践对比

**? 坏的 mock：测 mock 本身**
```typescript
test('retry 能用', async () => {
  const mock = jest.fn().mockRejectedValueOnce(new Error()).mockResolvedValueOnce('success');
  await retryOperation(mock);
  expect(mock).toHaveBeenCalledTimes(2);
});
```

**? 好的 mock：用真实行为验证**
```typescript
test('retries failed operations 3 times', async () => {
  let attempts = 0;
  const operation = () => {
    attempts++;
    if (attempts < 3) throw new Error('fail');
    return 'success';
  };
  const result = await retryOperation(operation);
  expect(result).toBe('success');
  expect(attempts).toBe(3);
});
```

**? 好的 mock：MSW 拦截网络层**
```typescript
import { http, HttpResponse } from 'msw'
import { server } from '../mocks/server'

test('throws on 404', async () => {
  server.use(
    http.get('/api/users/1', () => {
      return new HttpResponse(null, { status: 404 })
    })
  )
  await expect(fetchUser('1')).rejects.toThrow('User not found')
})
```

## 前端测试原则

1. **测行为，不测实现**：不要断言 `component.state.count`，要断言 `screen.getByText('Count: 1')`
2. **语义化选择器**：优先用 `getByRole`、`getByText`，避免 `.css-class-xyz`
3. **MSW 隔离 API**：用 Mock Service Worker 替代直接 `fetch` mock
4. **E2E 覆盖关键路径**：只测最核心的用户流程
