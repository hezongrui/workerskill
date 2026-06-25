# C++ TDD 日常指南

## 目标

优先沿用项目已有 C++ 测试框架和命令，建立快速、可重复、能保护设计的测试反馈。没有既有规范时，默认使用 GoogleTest / GoogleMock / CTest。

核心原则：
- 测公开行为，不测私有实现。
- 业务逻辑尽量放在可单测的普通 C++ 模块里。
- 对文件、网络、线程、时钟、进程、硬件、第三方 SDK 等边界做接口隔离。
- 第三方库本身不需要你测，测你的封装层和集成路径。
- 已有 Catch2、Boost.Test、doctest 或自研测试框架时，不为 TDD 额外引入第二套框架。

## 推荐项目结构

```text
project/
├── CMakeLists.txt
├── include/project/
│   └── command_parser.hpp
├── src/
│   └── command_parser.cpp
├── tests/
│   ├── command_parser_test.cpp
│   ├── command_sender_test.cpp
│   └── fixtures/
└── tools/
```

优先把核心逻辑做成 library target，再让生产程序和测试程序都链接它。

## CMake / CTest 基线

项目没有既有测试基线时，可以从下面结构开始；已有基线时只补 target、测试文件和用例。

```cmake
include(CTest)

add_library(project_core
    src/command_parser.cpp
)
target_include_directories(project_core PUBLIC include)
target_compile_features(project_core PUBLIC cxx_std_17)

if(BUILD_TESTING)
    find_package(GTest CONFIG REQUIRED)

    add_executable(project_core_tests
        tests/command_parser_test.cpp
    )
    target_link_libraries(project_core_tests
        PRIVATE project_core GTest::gtest_main
    )

    include(GoogleTest)
    gtest_discover_tests(project_core_tests)
endif()
```

如果项目暂时不用 `gtest_discover_tests`，也可以用：

```cmake
add_test(NAME project_core_tests COMMAND project_core_tests)
```

常用命令：

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug -DBUILD_TESTING=ON
cmake --build build
ctest --test-dir build --output-on-failure

# 只跑一个 GoogleTest 用例
./build/project_core_tests --gtest_filter=CommandParser.ParsesStartCommand
```

## CI 和质量增强

1. PR 阶段：编译 Debug、运行单元测试、开启 warning 策略。
2. 主干阶段：跑全量 `ctest`，必要时加覆盖率趋势。
3. 夜间阶段：跑慢集成测试、ASan/UBSan，涉及并发的模块定期跑 TSan。
4. 发布前：跑关键真实 fixture、性能回归、长稳测试。

示例 sanitizer 配置可以放在项目 preset 或 CI 里：

```bash
cmake -S . -B build-asan -DCMAKE_BUILD_TYPE=Debug -DBUILD_TESTING=ON \
  -DCMAKE_CXX_FLAGS="-fsanitize=address,undefined -fno-omit-frame-pointer"
cmake --build build-asan
ctest --test-dir build-asan --output-on-failure
```

如果项目禁用异常，测试不要使用 `EXPECT_THROW` 驱动设计，应测试错误码、`std::optional`、`std::variant` 或项目自定义 `expected` 风格结果。

## 适合先测的 C++ 代码

- 纯函数、解析器、校验器、格式转换
- 状态机、策略选择、重试/超时/取消规则
- 资源生命周期、RAII、错误码/异常路径
- 配置加载后的业务含义校验
- 对外部系统的封装层契约

不建议优先测：
- 简单 getter/setter
- 第三方库内部行为
- 私有函数
- 只为测试暴露的生产接口

## 纯逻辑示例

先写失败测试：

```cpp
#include <gtest/gtest.h>
#include "project/command_parser.hpp"

TEST(CommandParser, ParsesStartCommand) {
    auto command = parse_command("START motor=left speed=30");

    ASSERT_TRUE(command.has_value());
    EXPECT_EQ(command->name, "START");
    EXPECT_EQ(command->params.at("motor"), "left");
    EXPECT_EQ(command->params.at("speed"), "30");
}

TEST(CommandParser, RejectsMissingCommandName) {
    EXPECT_FALSE(parse_command("   ").has_value());
}
```

再写最小实现，让测试通过。实现通过后再补边界测试，例如空格、非法参数、重复参数、大小写规则。

## 外部边界示例

当业务逻辑需要调用外部系统时，依赖抽象边界，而不是在业务类里直接创建真实对象。

```cpp
class Transport {
public:
    virtual ~Transport() = default;
    virtual bool send(std::string_view payload) = 0;
};

class CommandSender {
public:
    explicit CommandSender(Transport& transport) : transport_(transport) {}

    bool send_start() {
        return transport_.send("START");
    }

private:
    Transport& transport_;
};
```

测试边界交互：

```cpp
#include <gmock/gmock.h>
#include <gtest/gtest.h>

using ::testing::Return;

class MockTransport : public Transport {
public:
    MOCK_METHOD(bool, send, (std::string_view payload), (override));
};

TEST(CommandSender, SendsStartPayload) {
    MockTransport transport;
    CommandSender sender(transport);

    EXPECT_CALL(transport, send("START")).WillOnce(Return(true));

    EXPECT_TRUE(sender.send_start());
}
```

这里验证交互是合理的，因为 `Transport` 是架构边界，发送内容是对外契约。

## 依赖注入建议

- 非拥有依赖优先用引用：`Transport&`。
- 共享生命周期用 `std::shared_ptr`。
- 可选依赖用指针或 `std::optional`，并明确空值语义。
- raw pointer 只在非拥有、可为空或对接 C API 时使用，并写清楚生命周期。
- 不要为了测试把私有方法改 public。

## 并发、时间和 IO

并发测试：
- 避免裸 `sleep`。
- 使用 fake clock、条件变量、future、barrier 或可控调度点。
- 给等待设置明确超时，失败信息要能定位卡在哪里。

时间逻辑：
- 抽象 `Clock` 或传入 `std::chrono` 时间点。
- 测“时间推进后的行为”，不要等真实时间流逝。

文件系统：
- 用临时目录。
- fixture 文件要小、可读、放在 `tests/fixtures/`。
- 测解析结果，不测文件读取库本身。

日志和指标：
- 不默认断言日志文案。
- 审计、安全、告警、结构化事件可以测。
- 优先注入 test logger / memory sink，断言 level、event code、稳定字段和次数。
- 不断言时间戳、行号、线程号、完整自然语言文案。

## GoogleTest 写法建议

- 用 `TEST` 测简单行为，用 `TEST_F` 共享昂贵 setup。
- 浮点比较用 `EXPECT_NEAR` 或 `EXPECT_DOUBLE_EQ`。
- 先用 `ASSERT_*` 阻止后续无意义断言，再用 `EXPECT_*` 收集多个差异。
- 参数组合多时用 parameterized tests，但不要把测试变成复杂程序。
- 测异常路径时遵守项目风格：项目禁用异常就测错误码或 `expected` 风格结果。

## 评审清单

- [ ] 测试名描述业务行为
- [ ] 测试可独立运行、可重复运行
- [ ] 被测对象不是 mock
- [ ] mock 只出现在外部边界
- [ ] 没有通过 private/protected hack 测内部实现
- [ ] 没有真实网络、真实睡眠、共享全局状态
- [ ] 项目异常风格、错误码风格保持一致
- [ ] 日志只在它是契约时被断言
- [ ] 失败信息足够定位问题
- [ ] `ctest --test-dir build --output-on-failure` 可通过
