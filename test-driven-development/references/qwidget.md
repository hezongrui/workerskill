# Qt Widget TDD 指南

## 原则：MVP 分层测试

```
View (QWidget)   → 只负责显示和事件转发，不测复杂逻辑
Presenter        → 纯 C++ 逻辑，处理用户输入和状态变更，TDD 重点
Model / Service  → 纯业务核心，TDD 重点
```

**测试边界**：
- **Presenter**：用 Google Test / QTest 直接测，不依赖 QWidget
- **View (QWidget)**：不测（或只做极少的手动/自动化 UI 验证）
- **Model**：按 `references/cpp.md` 进行 TDD

---

## Presenter TDD（重点）

### 红

```cpp
// tests/test_loginpresenter.cpp
#include <gtest/gtest.h>
#include "loginpresenter.h"
#include "mockloginview.h"

TEST(LoginPresenter, emptyPasswordShowsError) {
    MockLoginView view;
    LoginPresenter presenter(&view);
    presenter.onLoginClicked("user", "");
    EXPECT_EQ(view.lastError(), "密码不能为空");
    EXPECT_FALSE(view.navigatedToDashboard());
}

TEST(LoginPresenter, validCredentialsNavigates) {
    MockLoginView view;
    LoginPresenter presenter(&view);
    presenter.onLoginClicked("user", "123456");
    EXPECT_TRUE(view.navigatedToDashboard());
}
```

### 绿

```cpp
// src/presenters/loginpresenter.h
#pragma once
#include <QString>
#include "iloginview.h"

class LoginPresenter {
public:
    explicit LoginPresenter(ILoginView *view) : m_view(view) {}
    void onLoginClicked(const QString &username, const QString &password) {
        if (password.isEmpty()) {
            m_view->showError("密码不能为空");
            return;
        }
        m_view->navigateToDashboard();
    }
private:
    ILoginView *m_view;
};
```

```cpp
// src/views/iloginview.h
#pragma once
#include <QString>

class ILoginView {
public:
    virtual ~ILoginView() = default;
    virtual void showError(const QString &msg) = 0;
    virtual void navigateToDashboard() = 0;
};
```

### 测信号槽

```cpp
#include <QSignalSpy>

TEST(LoginPresenter, startLoadingEmitsProgress) {
    MockLoginView view;
    LoginPresenter presenter(&view);
    QSignalSpy spy(&presenter, &LoginPresenter::progressChanged);
    presenter.startLogin("user", "123456");
    EXPECT_EQ(spy.count(), 1);
}
```

---

## QWidget 视图层

**不直接测 QWidget**。View 只负责：
1. 把用户事件转发给 Presenter
2. 按 Presenter 的指令更新界面

```cpp
// src/views/loginwidget.cpp
void LoginWidget::on_btnLogin_clicked() {
    m_presenter->onLoginClicked(ui->editUser->text(), ui->editPass->text());
}

void LoginWidget::showError(const QString &msg) {
    QMessageBox::warning(this, "错误", msg);
}
```

---

## 项目结构

```
myproject/
├── src/
│   ├── main.cpp
│   ├── models/
│   ├── presenters/
│   │   ├── loginpresenter.cpp
│   │   ├── loginpresenter.h
│   │   └── iloginview.h
│   └── views/
│       ├── loginwidget.cpp
│       └── loginwidget.h
├── tests/
│   ├── cpp/
│   │   ├── test_loginpresenter.cpp
│   │   └── mockloginview.h
│   └── integration/
└── CMakeLists.txt
```

## 运行命令

```bash
# C++ 单元测试（Presenter + Model）
cmake --build build && ctest --output-on-failure

# 或 QTest
qmake && make check
```
