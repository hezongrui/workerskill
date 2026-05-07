# Go TDD 指南

## 红-绿-重构示例

### 红

```go
// discount_test.go
package main

import "testing"

func TestCalculateDiscount(t *testing.T) {
	result := CalculateDiscount(100, 0.2)
	if result != 80 {
		t.Errorf("CalculateDiscount(100, 0.2) = %v; want 80", result)
	}
}

func TestCalculateDiscountInvalidRate(t *testing.T) {
	_, err := CalculateDiscount(100, -0.1)
	if err == nil {
		t.Error("expected error for negative rate")
	}
}
```

```bash
go test ./... -v
# 预期：FAIL - undefined: CalculateDiscount
```

### 绿

```go
// discount.go
package main

import "fmt"

func CalculateDiscount(price float64, rate float64) (float64, error) {
	if rate < 0 || rate > 1 {
		return 0, fmt.Errorf("rate must be between 0 and 1")
	}
	return price * (1 - rate), nil
}
```

### 验证绿

```bash
go test ./... -v
# 预期：PASS
```

---

## 表格驱动测试（推荐）

```go
func TestCalculateDiscountTable(t *testing.T) {
	tests := []struct {
		name     string
		price    float64
		rate     float64
		expected float64
		wantErr  bool
	}{
		{"normal", 100, 0.2, 80, false},
		{"free", 100, 1.0, 0, false},
		{"zero price", 0, 0.2, 0, false},
		{"invalid rate", 100, -0.1, 0, true},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got, err := CalculateDiscount(tt.price, tt.rate)
			if (err != nil) != tt.wantErr {
				t.Fatalf("CalculateDiscount() error = %v, wantErr %v", err, tt.wantErr)
			}
			if got != tt.expected {
				t.Errorf("CalculateDiscount() = %v, want %v", got, tt.expected)
			}
		})
	}
}
```

---

## 项目结构

```
myproject/
├── cmd/
│   └── app/
│       └── main.go
├── internal/
│   └── discount/
│       ├── discount.go
│       └── discount_test.go
├── pkg/
│   └── utils/
│       ├── utils.go
│       └── utils_test.go
├── go.mod
└── go.sum
```

**原则**：
- 测试文件与被测文件同包（`package discount`）或 `package discount_test`（黑盒测试）
- `_test.go` 后缀只在测试时编译

## 常用命令

```bash
# 运行当前包测试
go test ./...

# 运行单个测试函数
go test ./... -run TestCalculateDiscount -v

# 带覆盖率
go test ./... -cover

# 生成覆盖率报告
go test ./... -coverprofile=coverage.out
go tool cover -html=coverage.out

# 失败即停（Go 1.20+）
go test ./... -failfast
```

## Go 测试原则

1. **表格驱动**：一个测试函数覆盖多组输入输出
2. **t.Parallel()**：无状态测试加 `t.Parallel()` 加速执行
3. **testify 可选**：`github.com/stretchr/testify/assert` 可简化断言
4. **接口隔离**：通过 interface 替换外部依赖，便于 mock
