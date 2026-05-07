# C++ TDD 指南

## 核心原则：接口隔离

**原则：不测第三方库本身，测你的封装层；用接口隔离硬依赖。**

```
IDecoder / IEncoder (接口)   ← 单元测试用 Mock
FFmpegDecoder (具体实现)     ← 集成测试用真实小媒体文件
业务逻辑层 (TranscodeService) ← TDD 主战场
```

---

## 红-绿-重构完整示例

### 红：写失败的测试

```cpp
// tests/test_discount.cpp
#include <gtest/gtest.h>
#include "discount_calculator.h"

TEST(DiscountCalculator, appliesNormalDiscount) {
    DiscountCalculator calc;
    EXPECT_DOUBLE_EQ(calc.calculate(100.0, 0.2), 80.0);
}

TEST(DiscountCalculator, throwsOnInvalidRate) {
    DiscountCalculator calc;
    EXPECT_THROW(calc.calculate(100.0, -0.1), std::invalid_argument);
}
```

运行，确认失败：
```bash
cmake --build build && ctest --output-on-failure
# 预期：FAIL - discount_calculator.h: No such file or directory
```

### 绿：写最小实现

```cpp
// src/discount_calculator.h
#pragma once
#include <stdexcept>

class DiscountCalculator {
public:
    double calculate(double price, double rate) {
        if (rate < 0 || rate > 1) {
            throw std::invalid_argument("rate must be between 0 and 1");
        }
        return price * (1 - rate);
    }
};
```

### 验证绿

```bash
cmake --build build && ctest --output-on-failure
# 预期：PASS
```

---

## Google Mock 示例（接口隔离）

```cpp
// src/ipayment_gateway.h
#pragma once

class IPaymentGateway {
public:
    virtual ~IPaymentGateway() = default;
    virtual bool charge(double amount) = 0;
};

// src/payment_processor.h
#pragma once
#include "ipayment_gateway.h"

class PaymentProcessor {
    IPaymentGateway* gateway_;
public:
    explicit PaymentProcessor(IPaymentGateway* g) : gateway_(g) {}
    bool process(double amount) {
        return gateway_->charge(amount);
    }
};
```

```cpp
// tests/test_payment_processor.cpp
#include <gtest/gtest.h>
#include <gmock/gmock.h>
#include "payment_processor.h"
#include "ipayment_gateway.h"

using ::testing::Return;

class MockPaymentGateway : public IPaymentGateway {
public:
    MOCK_METHOD(bool, charge, (double amount), (override));
};

TEST(PaymentProcessor, delegatesToGateway) {
    MockPaymentGateway gateway;
    PaymentProcessor processor(&gateway);
    EXPECT_CALL(gateway, charge(100.0)).WillOnce(Return(true));
    EXPECT_TRUE(processor.process(100.0));
}
```

---

## 项目结构

```
myproject/
├── src/
│   ├── discount_calculator.h
│   ├── payment_processor.h
│   └── ipayment_gateway.h
├── tests/
│   ├── test_discount.cpp
│   └── test_payment_processor.cpp
├── tests/fixtures/          # 集成测试用真实小文件
│   ├── test_1s.mp4
│   └── corrupt.mp4
├── CMakeLists.txt
└── ...
```

| 层 | 测试方式 |
|----|----------|
| 业务逻辑层 | TDD + Google Test / Google Mock |
| 第三方封装层 | 集成测试 + 真实 fixtures |
| UI 层 | 不测（或交给 `qwidget.md` / `qml.md`） |

---

## 常用命令

```bash
# 构建并运行全部测试
cmake --build build && ctest --output-on-failure

# 只运行单个测试
cmake --build build && ./tests/test_discount --gtest_filter=DiscountCalculator.appliesNormalDiscount

# 生成覆盖率报告（需 gcov + lcov）
cmake --build build && make coverage
```

---

## C++ 测试原则

1. **接口隔离**：业务逻辑依赖抽象接口（`IPaymentGateway`），测试时注入 Mock
2. **Google Mock**：用 `MOCK_METHOD` 替换外部依赖，不要 mock 被测类本身
3. **测试夹具**：用 `TEST_F` 共享重复 setup 代码
4. **异常路径**：用 `EXPECT_THROW` / `ASSERT_THROW` 测错误分支
5. **浮点比较**：用 `EXPECT_DOUBLE_EQ` 或 `EXPECT_NEAR`，不要直接 `==`
