# 存储类说明符 (Storage Class Specifiers)

## 1. 概述 (Overview)

存储类说明符是名称声明语法中声明说明符序列的一部分。与名称的作用域一起，它们控制名称的两个独立属性：**存储期 (storage duration)** 和 **链接 (linkage)**。

### 核心概念

```
声明说明符序列
       ↓
存储类说明符 + 类型说明符
       ↓
控制两个属性：
├── 存储期：对象的生命周期
└── 链接：名称在其他作用域/翻译单元中的可见性
```

## 2. 来源与演变 (Origin and Evolution)

### 历史背景

C++ 的存储类说明符继承自 C 语言，但在后续标准中有重要变化。

### 版本演进

| C++ 标准版本 | 发布年份 | 相关改进 |
|-------------|---------|---------|
| C++98 | 1998 | 基本存储类说明符体系 |
| C++11 | 2011 | `thread_local` 存储；`auto` 改为类型推导 |
| C++14 | 2014 | 变量模板链接规则 |
| C++17 | 2017 | 移除 `register` 关键字 |
| C++20 | 2020 | 模块链接；翻译单元局部实体 |

### 关键变更

| 变更 | 说明 |
|-----|------|
| `auto` (C++11) | 不再是存储类说明符，改为类型推导 |
| `register` (C++17) | 移除，变量不能声明为 `register` |
| `thread_local` (C++11) | 新增线程存储期 |

### 缺陷报告

| 缺陷报告 | 问题 | 解决方案 |
|---------|------|---------|
| CWG 809 | `register` 功能很小 | 弃用 |
| CWG 1648 | `thread_local` 与 `extern` 组合时隐含 `static` | 仅当无其他说明符时才隐含 |
| CWG 2387 | `const` 变量模板的链接不明确 | `const` 不影响变量模板链接 |
| P2788R0 | 模块单元中的 `const` 变量仍有内部链接 | 不给予内部链接 |

## 3. 语法与参数 (Syntax and Parameters)

### 3.1 存储类说明符关键字

```cpp
auto          // C++11 前：自动存储期；C++11 起：类型推导
register      // C++17 前：寄存器提示
static        // 静态存储期或内部链接
thread_local  // C++11 起：线程存储期
extern        // 外部链接或声明
mutable       // 允许在 const 成员函数中修改
```

### 3.2 说明符组合规则

在声明说明符序列中，**最多只能有一个存储类说明符**，但 `thread_local` 可以与 `static` 或 `extern` 一起出现。

```cpp
static int a;                    // OK
thread_local int b;              // OK (C++11)
thread_local static int c;       // OK (C++11)
extern int d;                    // OK
thread_local extern int e;       // OK (C++11)

static extern int f;             // 错误！不能同时使用
```

### 3.3 说明符适用范围

| 说明符 | 非成员变量 | 函数参数 | 非静态成员 | 静态成员 | 非成员函数 | 成员函数 |
|-------|----------|---------|-----------|---------|-----------|---------|
| `auto` (旧) | 仅块作用域 | 是 | 否 | 否 | 否 | 否 |
| `register` | 仅块作用域 | 是 | 否 | 否 | 否 | 否 |
| `static` | 是 | 否 | 声明静态 | 是 | 命名空间作用域 | 声明静态 |
| `thread_local` | 是 | 否 | 否 | 是 | 否 | 否 |
| `extern` | 是 | 否 | 否 | 是 | 否 | 否 |

## 4. 底层原理 (Underlying Principles)

### 4.1 存储期类型

存储期是定义对象存储最小潜在生命周期的属性。

| 存储期 | 关键字 | 生命周期 | 存储位置 |
|-------|-------|---------|---------|
| **静态存储期** | `static` | 程序整个运行期 | 数据段/BSS |
| **线程存储期** | `thread_local` | 线程整个运行期 | 线程局部存储 |
| **自动存储期** | (默认) | 块作用域期间 | 栈 |
| **动态存储期** | `new` | 手动控制 | 堆 |

#### 静态存储期

满足以下条件的变量具有静态存储期：

1. 属于命名空间作用域，或首次声明时使用 `static` 或 `extern`
2. 没有线程存储期

```cpp
int global_var;           // 静态存储期
static int file_static;   // 静态存储期
void func() {
    static int local_static;  // 静态存储期
}
```

#### 线程存储期 (C++11)

所有声明为 `thread_local` 的变量具有线程存储期：

```cpp
thread_local int tl_var;  // 每个线程有独立副本

void func() {
    thread_local static int tls;  // 每个线程独立
}
```

#### 自动存储期

以下变量具有自动存储期：

1. 属于块作用域且未显式声明为 `static`、`thread_local` 或 `extern`
2. 函数参数

```cpp
void func(int param) {    // param: 自动存储期
    int local = 0;        // 自动存储期
    static int s = 0;     // 静态存储期
}
```

#### 动态存储期

以下方式创建的对象具有动态存储期：

1. `new` 表达式
2. 隐式创建的对象
3. 异常对象

```cpp
int* p = new int(42);     // 动态存储期
delete p;
```

### 4.2 链接类型

链接决定名称在不同作用域中的可见性。

| 链接类型 | 可见性 |
|---------|-------|
| **外部链接** | 可在其他翻译单元访问 |
| **模块链接** (C++20) | 可在同一模块的其他翻译单元访问 |
| **内部链接** | 仅在本翻译单元访问 |
| **无链接** | 仅在当前作用域访问 |

#### 无链接

块作用域中以下名称无链接：

- 未显式声明 `extern` 的变量
- 局部类及其成员函数
- 块作用域中的 typedef、枚举、枚举器

```cpp
void func() {
    int x;              // 无链接
    class Local {};     // 无链接
    typedef int Int;    // 无链接
}
```

#### 内部链接

命名空间作用域中以下名称具有内部链接：

- 声明为 `static` 的变量、函数、模板
- 非模板非 volatile 的 `const` 变量（除非 `inline`、`extern` 或在模块接口中）
- 匿名联合的成员
- 匿名命名空间中的所有名称

```cpp
static int internal_var;     // 内部链接
const int const_var = 42;    // 内部链接
namespace {
    int anon_var;            // 内部链接
}
```

#### 外部链接

命名空间作用域中以下名称具有外部链接：

- 非 `static` 函数和非 `const` 变量
- 枚举
- 类名、成员函数、静态数据成员、嵌套类
- 非 `static` 模板

```cpp
int global;              // 外部链接
void func();             // 外部链接
class MyClass {          // 外部链接
    static int s;        // 外部链接
};
```

#### 模块链接 (C++20)

附加到命名模块但不导出、也无内部链接的名称具有模块链接：

```cpp
export module my_module;

int module_var;          // 模块链接
export int exported_var; // 外部链接
```

### 4.3 静态块变量初始化

块作用域中具有静态或线程存储期的变量：

1. 首次通过声明时初始化
2. 如果初始化抛出异常，下次再尝试
3. 递归初始化是未定义行为
4. 多线程并发初始化只发生一次（C++11）

```cpp
class Widget {
public:
    Widget() { std::cout << "Constructed\n"; }
    ~Widget() { std::cout << "Destructed\n"; }
};

void func() {
    static Widget w;  // 首次调用时构造
    // 程序退出时析构
}
```

### 4.4 翻译单元局部实体 (C++20)

实体是**翻译单元局部 (TU-local)** 的，如果：

1. 具有内部链接的名称，或
2. 无链接但在 TU-local 实体定义中引入，或
3. 使用 TU-local 实体的模板

```cpp
// file.cpp
namespace {
    int internal;  // TU-local
}

// 不能在模块接口中暴露 TU-local 实体
export int* get_internal() {
    return &internal;  // 危险！
}
```

## 5. 使用场景 (Use Cases)

### 5.1 存储期选择指南

| 场景 | 推荐存储期 | 说明 |
|-----|-----------|------|
| 局部临时变量 | 自动 | 默认行为 |
| 跨调用持久状态 | 静态 | 函数内 `static` |
| 全局配置 | 静态 | 命名空间作用域 |
| 线程独立状态 | 线程 | `thread_local` |
| 动态大小数据 | 动态 | `new`/智能指针 |

### 5.2 链接选择指南

| 场景 | 推荐链接 | 说明 |
|-----|---------|------|
| API 导出函数 | 外部 | 默认行为 |
| 内部实现函数 | 内部 | `static` 或匿名命名空间 |
| 常量定义 | 内部 | `const` 或 `constexpr` |
| 头文件常量 | 外部 | `inline constexpr` |

### 5.3 注意事项

1. **`mutable` 不影响存储期**：仅允许在 const 成员函数中修改

2. **`const` 变量的链接**：
   - 命名空间作用域的 `const` 默认内部链接
   - C++14 变量模板 `const` 不影响链接

3. **`thread_local` 初始化**：每个线程独立初始化

4. **模块中的 `const`** (C++20)：模块接口中的 `const` 有外部链接

## 6. 代码示例 (Examples)

### 6.1 存储期演示

```cpp
#include <iostream>

int global = 0;  // 静态存储期

void demonstrate_storage() {
    int automatic = 0;         // 自动存储期
    static int static_var = 0; // 静态存储期

    automatic++;
    static_var++;

    std::cout << "automatic: " << automatic
              << ", static_var: " << static_var << '\n';
}

int main() {
    demonstrate_storage();  // automatic: 1, static_var: 1
    demonstrate_storage();  // automatic: 1, static_var: 2
    demonstrate_storage();  // automatic: 1, static_var: 3

    return 0;
}
```

### 6.2 线程存储期 (C++11)

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <string>

thread_local unsigned int rage = 1;
std::mutex cout_mutex;

void increase_rage(const std::string& thread_name) {
    ++rage;  // 每个线程独立的 rage
    std::lock_guard<std::mutex> lock(cout_mutex);
    std::cout << "Rage counter for " << thread_name
              << ": " << rage << '\n';
}

int main() {
    std::thread a(increase_rage, "a");
    std::thread b(increase_rage, "b");

    {
        std::lock_guard<std::mutex> lock(cout_mutex);
        std::cout << "Rage counter for main: " << rage << '\n';
    }

    a.join();
    b.join();

    return 0;
}

// 可能输出：
// Rage counter for a: 2
// Rage counter for main: 1
// Rage counter for b: 2
```

### 6.3 链接演示

```cpp
// header.hpp
#ifndef HEADER_HPP
#define HEADER_HPP

// 外部链接
extern int external_var;

// 内联变量 (C++17) - 外部链接
inline int inline_var = 0;

// constexpr 默认内部链接 (C++17 前有变化)
constexpr int constexpr_var = 42;

// 函数声明
void external_func();

#endif

// source1.cpp
#include "header.hpp"

int external_var = 100;  // 定义

static void internal_func() {  // 内部链接
    // 仅在本文件可见
}

namespace {
    int anon_var = 0;  // 内部链接
}

void external_func() {
    internal_func();  // 可以调用
}
```

### 6.4 静态局部变量

```cpp
#include <iostream>
#include <mutex>

class Logger {
public:
    static Logger& instance() {
        static Logger logger;  // 惰性初始化，线程安全 (C++11)
        return logger;
    }

    void log(const std::string& msg) {
        std::cout << "[LOG] " << msg << '\n';
    }

private:
    Logger() { std::cout << "Logger created\n"; }
    ~Logger() { std::cout << "Logger destroyed\n"; }
};

int main() {
    Logger::instance().log("First message");
    Logger::instance().log("Second message");
    // Logger 只创建一次

    return 0;
    // 程序退出时 Logger 析构
}
```

### 6.5 匿名命名空间

```cpp
// 使用匿名命名空间替代 static
namespace {
    int internal_var = 42;

    void internal_func() {
        std::cout << "Internal function\n";
    }

    class InternalClass {
        // ...
    };
}

// 等价于：
// static int internal_var = 42;
// static void internal_func() { ... }
// 但匿名命名空间可以包含类定义
```

### 6.6 常见错误及修正

#### 错误示例 1：误用 `auto` (C++11 前)

```cpp
// C++98 风格（已过时）
int main() {
    auto int x = 10;  // C++11 起：错误！auto 现在是类型推导
    return 0;
}
```

**正确做法**：

```cpp
int main() {
    int x = 10;  // 直接声明
    auto y = 10; // C++11 起：类型推导为 int
    return 0;
}
```

#### 错误示例 2：递归初始化静态变量

```cpp
// 错误：递归初始化
int func();

int& get_static() {
    static int value = func();  // 如果 func 调用 get_static...
    return value;
}

int func() {
    return get_static();  // 未定义行为！
}
```

**正确做法**：

```cpp
int func() {
    return 0;  // 不依赖静态变量
}

int& get_static() {
    static int value = func();  // 安全
    return value;
}
```

#### 错误示例 3：暴露内部链接实体

```cpp
// 错误：返回内部链接变量的指针
namespace {
    int internal = 42;
}

int* get_internal() {
    return &internal;  // 危险：其他翻译单元可能访问
}
```

**正确做法**：

```cpp
// 方案1：不暴露内部实体
namespace {
    int internal = 42;
}

int get_internal_value() {
    return internal;  // 返回值，不暴露地址
}

// 方案2：使用外部链接
int external = 42;  // 明确的外部链接
```

#### 错误示例 4：`const` 变量在头文件中

```cpp
// header.hpp - 错误方式
const int MAX_SIZE = 100;  // 每个翻译单元有独立副本！

// 如果需要取地址或跨翻译单元使用：
// file1.cpp
const int* p1 = &MAX_SIZE;
// file2.cpp
const int* p2 = &MAX_SIZE;
// p1 != p2  可能成立！
```

**正确做法**：

```cpp
// header.hpp - C++17 方式
inline constexpr int MAX_SIZE = 100;  // 单一定义

// 或传统方式
// header.hpp
extern const int MAX_SIZE;

// source.cpp
const int MAX_SIZE = 100;
```

## 7. 总结 (Summary)

### 7.1 核心要点

1. **存储期决定生命周期**：
   - 静态：程序整个运行期
   - 线程：线程整个运行期
   - 自动：块作用域期间
   - 动态：手动控制

2. **链接决定可见性**：
   - 外部：跨翻译单元可见
   - 模块 (C++20)：同模块跨翻译单元可见
   - 内部：仅本翻译单元可见
   - 无：仅当前作用域可见

3. **关键关键字**：
   - `static`：静态存储期或内部链接
   - `thread_local`：线程存储期
   - `extern`：外部链接
   - `mutable`：允许 const 函数修改

### 7.2 存储期与链接关系

| 声明位置 | 存储期 | 默认链接 |
|---------|-------|---------|
| 命名空间作用域变量 | 静态 | 外部 |
| 命名空间作用域 `const` 变量 | 静态 | 内部 |
| 命名空间作用域 `static` 变量 | 静态 | 内部 |
| 块作用域变量 | 自动 | 无 |
| 块作用域 `static` 变量 | 静态 | 无 |
| `thread_local` 变量 | 线程 | 同 static |
| `new` 创建的对象 | 动态 | N/A |

### 7.3 与 C 的差异

| 特性 | C++ | C |
|-----|-----|---|
| `auto` | 类型推导 (C++11) | 存储类说明符 |
| `register` | 移除 (C++17) | 保留 |
| 顶层 `const` | 内部链接 | 外部链接 |
| `thread_local` | 原生支持 | C11 `_Thread_local` |

### 7.4 学习建议

1. **理解存储期概念**：
   - 知道对象何时创建和销毁
   - 理解四种存储期的区别

2. **掌握链接规则**：
   - 默认链接行为
   - 如何改变链接

3. **现代 C++ 最佳实践**：
   - 避免全局变量，使用函数内静态
   - 使用匿名命名空间替代 `static` 函数
   - C++17 起：`inline constexpr` 用于头文件常量

4. **注意线程安全**：
   - 静态局部变量初始化是线程安全的 (C++11)
   - `thread_local` 用于线程独立状态

---

**参考资源**：
- ISO/IEC 14882:2024 (C++23 标准) 6.7.5
- `<thread>` - 线程支持
- `std::call_once` - 一次性初始化
- 模块 (Modules) - C++20