# Python TDD 日常指南

## 目标

优先沿用项目已有 Python 测试框架和命令，写快、清楚、稳定的测试，让代码在小步反馈中演进。没有既有规范时，默认使用 pytest。

核心原则：
- 测公开行为，不测私有实现。
- 优先用普通函数和小模块承载核心逻辑。
- 文件、网络、时间、环境变量、数据库、第三方 API 放在边界层。
- monkeypatch 只补不可直接注入的边界，不要随手 patch 内部实现。
- 已有 `unittest`、pytest 插件或项目自定义 fixture 时，先按项目约定写，不为了 TDD 改换测试栈。

## 推荐结构

```text
project/
├── pyproject.toml
├── src/
│   └── app/
│       ├── command_parser.py
│       └── command_sender.py
└── tests/
    ├── test_command_parser.py
    ├── test_command_sender.py
    └── fixtures/
```

常用命令。已有项目优先使用 README、CI 或 `pyproject.toml` 中定义的命令：

```bash
pytest
pytest tests/test_command_parser.py -q
pytest tests/test_command_parser.py::test_parses_start_command -q
pytest -x -q
pytest --cov=app
```

## CI 和质量增强

1. PR 阶段：跑相关 pytest、格式化、静态检查。
2. 主干阶段：跑全量 pytest 和覆盖率趋势。
3. 夜间阶段：跑慢集成测试、真实 fixture、外部服务 contract fake。
4. 发布前：跑关键端到端路径和性能回归。

覆盖率用于发现风险，不用于替代测试质量判断。

## 纯逻辑示例

先写失败测试：

```python
from app.command_parser import parse_command


def test_parses_start_command():
    command = parse_command("START motor=left speed=30")

    assert command.name == "START"
    assert command.params == {"motor": "left", "speed": "30"}


def test_rejects_missing_command_name():
    assert parse_command("   ") is None
```

实现通过后，再补边界测试。

参数组合多时使用 `pytest.mark.parametrize`：

```python
import pytest

from app.command_parser import parse_command


@pytest.mark.parametrize("text", ["", " ", "\n\t"])
def test_rejects_blank_commands(text):
    assert parse_command(text) is None
```

## 文件系统

使用 `tmp_path`，不要写真实项目目录或用户目录：

```python
from app.config_loader import load_config


def test_loads_config_from_file(tmp_path):
    config_file = tmp_path / "app.toml"
    config_file.write_text("timeout = 3\n", encoding="utf-8")

    config = load_config(config_file)

    assert config.timeout == 3
```

## 环境变量和全局状态

使用 `monkeypatch` 修改环境或不可注入的全局边界，pytest 会在测试结束后恢复：

```python
from app.settings import load_settings


def test_reads_timeout_from_environment(monkeypatch):
    monkeypatch.setenv("APP_TIMEOUT", "5")

    settings = load_settings()

    assert settings.timeout == 5
```

优先选择依赖注入：

```python
from app.command_sender import CommandSender


class FakeTransport:
    def __init__(self):
        self.sent = []

    def send(self, payload: str) -> bool:
        self.sent.append(payload)
        return True


def test_sender_sends_start_payload():
    transport = FakeTransport()
    sender = CommandSender(transport)

    assert sender.send_start() is True
    assert transport.sent == ["START"]
```

只有当依赖无法自然注入时，再用 monkeypatch。

## unittest.mock 和异步测试

优先用 fake 或 monkeypatch。`unittest.mock` 适合下面情况：

1. 第三方对象创建成本高，且无法自然注入。
2. 需要验证边界交互，例如外部 client 是否收到指定 payload。
3. 需要阻止真实网络、真实进程或真实时间。

避免 patch 被测函数内部的每一步。mock 越多，越说明边界可能需要重新设计。

异步代码：
1. 项目已有 `pytest-asyncio` 时，按项目约定写 async test。
2. 不要用真实 sleep 等待异步结果。
3. 优先注入 fake clock、fake client 或可控队列。

## 异常、日志和时间

异常：

```python
import pytest


def test_rejects_negative_timeout():
    with pytest.raises(ValueError, match="timeout"):
        parse_timeout("-1")
```

日志只在它是契约时断言。优先断言稳定字段，不断言完整自然语言文案：

```python
import logging

from app.command_handler import handle_command


def test_records_rejected_command_event(caplog):
    with caplog.at_level(logging.WARNING):
        handle_command("???", request_id="r1")

    record = next(
        r for r in caplog.records
        if getattr(r, "event", None) == "command_rejected"
    )
    assert record.levelno == logging.WARNING
    assert record.error_code == "INVALID_COMMAND"
    assert record.request_id == "r1"
```

如果项目只有普通文本日志，也只断言稳定关键词或错误码，不断言时间戳、行号和整句文案。

时间逻辑：
- 优先把当前时间作为参数传入。
- 或封装 clock 函数并在边界处替换。
- 不要在测试中等待真实时间流逝。

## Test Double 建议

优先 fake，其次 stub，最后 mock。

适合 fake：
- 内存仓库
- 临时目录
- 假 transport/client
- 假 clock

适合 monkeypatch：
- 环境变量
- 当前工作目录
- 不可注入的第三方调用
- 禁止真实网络访问

避免：
- patch 被测函数内部的每一步
- patch 路径写错后测试仍然通过
- mock 返回结构只包含当前断言字段
- 为了测试给生产代码加“测试专用开关”

## 遗留代码策略

1. 先找外部入口，例如 CLI 函数、服务函数、模块 API。
2. 用 characterization tests 锁住当前行为。
3. 抽出纯逻辑函数。
4. 给新函数补更细的单元测试。
5. 再减少粗粒度测试里的重复和慢路径。

## 评审清单

- [ ] 测试名说明行为
- [ ] 使用 pytest fixture 管理临时资源
- [ ] 没有真实网络、真实用户目录、共享全局状态
- [ ] monkeypatch 只作用在边界
- [ ] mock 不替代被测逻辑本身
- [ ] 日志只在它是契约时被断言
- [ ] 异常和边界条件有覆盖
- [ ] 测试失败信息清楚
- [ ] `pytest -q` 或相关测试命令可通过
