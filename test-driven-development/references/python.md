# Python TDD 指南

## 红-绿-重构示例

### 红

```python
# tests/test_discount.py
def test_calculate_discount():
    result = calculate_discount(100, 0.2)
    assert result == 80

def test_calculate_discount_invalid_rate():
    with pytest.raises(ValueError, match="rate must be between 0 and 1"):
        calculate_discount(100, -0.1)
```

```bash
pytest tests/test_discount.py -v
# 预期：FAIL - name 'calculate_discount' is not defined
```

### 绿

```python
# src/discount.py
def calculate_discount(price, rate):
    if rate < 0 or rate > 1:
        raise ValueError("rate must be between 0 and 1")
    return price * (1 - rate)
```

### 验证绿

```bash
pytest tests/test_discount.py -v
# 预期：PASS
```

## 项目结构

```
myproject/
├── src/
│   └── mypackage/
│       ├── __init__.py
│       └── discount.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py          # 共享 fixtures
│   ├── unit/
│   │   └── test_discount.py
│   └── integration/
│       └── test_api.py
├── pyproject.toml
└── pytest.ini
```

## 常用命令

```bash
# 运行全部测试
pytest

# 运行单个测试文件
pytest tests/unit/test_discount.py -v

# 运行单个测试
pytest tests/unit/test_discount.py::test_calculate_discount -v

# 带覆盖率
pytest --cov=mypackage --cov-report=term-missing --cov-report=html

# 只运行快速测试（排除标记为 slow 的）
pytest -m "not slow"

# 失败即停
pytest -x
```

## 关键 pytest 特性

- **fixture**：用 `@pytest.fixture` 共享测试数据
- **parametrize**：用 `@pytest.mark.parametrize` 批量测试边界值
- **raises**：用 `pytest.raises(Exception)` 测异常
- **mock**：用 `unittest.mock.patch` 隔离外部依赖
