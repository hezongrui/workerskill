# Qt QML TDD 指南

## 原则：MVVM 分层测试

```
View (QML)      → 只负责显示和交互，不测复杂逻辑
ViewModel (C++) → 暴露 Q_PROPERTY 和 Q_INVOKABLE，TDD 重点
Model (C++)     → 纯业务逻辑，TDD 重点
```

**测试边界**：
- **ViewModel**：用 Google Test / QTest 直接测，不依赖 QML 引擎
- **View (QML)**：用 `qmltestrunner` 测绑定、用户交互和状态流转
- **Model**：按 `references/cpp.md` 进行 TDD

---

## ViewModel TDD（重点）

### 红

```cpp
// tests/test_counterviewmodel.cpp
#include <gtest/gtest.h>
#include "counterviewmodel.h"

TEST(CounterViewModel, initialValueIsZero) {
    CounterViewModel vm;
    EXPECT_EQ(vm.value(), 0);
}

TEST(CounterViewModel, incrementEmitsSignal) {
    CounterViewModel vm;
    QSignalSpy spy(&vm, &CounterViewModel::valueChanged);
    vm.increment();
    EXPECT_EQ(vm.value(), 1);
    EXPECT_EQ(spy.count(), 1);
}
```

### 绿

```cpp
// src/viewmodels/counterviewmodel.h
#pragma once
#include <QObject>

class CounterViewModel : public QObject {
    Q_OBJECT
    Q_PROPERTY(int value READ value NOTIFY valueChanged)
public:
    explicit CounterViewModel(QObject *parent = nullptr) : QObject(parent), m_value(0) {}
    int value() const { return m_value; }
    Q_INVOKABLE void increment() {
        m_value++;
        emit valueChanged(m_value);
    }
signals:
    void valueChanged(int newValue);
private:
    int m_value;
};
```

---

## QML 视图层测试

QML 只测**绑定生效**和**用户交互触发 ViewModel**：

```qml
// tests/qml/tst_Counter.qml
import QtQuick 2.15
import QtTest 1.15
import CounterViewModel 1.0

TestCase {
    name: "CounterViewTest"

    CounterViewModel { id: vm }

    Rectangle {
        id: root
        Text {
            id: label
            text: vm.value
        }
        MouseArea {
            id: clickArea
            onClicked: vm.increment()
        }
    }

    function test_bindingUpdatesLabel() {
        compare(label.text, "0");
        vm.increment();
        compare(label.text, "1");
    }

    function test_clickTriggersIncrement() {
        mouseClick(clickArea);
        compare(vm.value, 1);
    }
}
```

---

## 项目结构

```
myproject/
├── src/
│   ├── main.cpp
│   ├── models/
│   ├── viewmodels/
│   │   ├── counterviewmodel.cpp
│   │   └── counterviewmodel.h
│   └── qml/
│       └── Main.qml
├── tests/
│   ├── cpp/
│   │   └── test_counterviewmodel.cpp
│   └── qml/
│       └── tst_Counter.qml
└── CMakeLists.txt
```

## 运行命令

```bash
# C++ 单元测试（ViewModel + Model）
cmake --build build && ctest --output-on-failure

# QML 视图测试
qmltestrunner -input tests/qml
```
