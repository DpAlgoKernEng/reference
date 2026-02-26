# std::vector - 向量容器

## 概述

`std::vector` 是 C++ 标准库中最重要的序列容器（sequence container）之一。它封装了动态数组（dynamic array），提供连续内存存储，支持快速随机访问。vector 可以自动管理内存，在容量不足时自动扩容，是 C++ 中最常用的容器类型。

vector 定义在 `<vector>` 头文件中，位于 `std` 命名空间下。

## 来源与演变

### 首次引入

`std::vector` 首次在 **C++98** 标准中引入，是 STL（Standard Template Library）的核心组件之一。STL 最初由 Alexander Stepanov 和 Meng Lee 设计，于 1994 年被纳入 C++ 标准库。

### 历史背景

在 vector 出现之前，C++ 开发者主要使用以下方式处理动态数组：

1. **C 风格数组**：需要手动管理内存，容易出错
2. **自定义数组类**：每个开发者需要实现自己的动态数组

vector 的出现解决了这些问题，提供了：
- 自动内存管理
- 与数组兼容的内存布局
- 一致的接口设计

### C++11 变化

- 新增 `emplace_back()` 方法，原地构造元素
- 新增 `shrink_to_fit()` 方法，请求缩减容量
- 新增 `data()` 方法，获取底层指针
- 移动语义支持，提高性能
- 新增 `cbegin()`, `cend()`, `crbegin()`, `crend()` 常量迭代器

### C++17 变化

- 新增 `try_emplace()` 相关方法（unordered_map 等）
- `std::vector<bool>` 优化

### C++20 变化

- `erase()` 返回迭代器类型改进
- `contains()` 方法（部分容器）

## 语法与声明

### 基本模板定义

```cpp
namespace std {
    template<
        class T,
        class Allocator = std::allocator<T>
    > class vector;
}
```

### 类型别名

```cpp
using value_type = T;
using allocator_type = Allocator;
using size_type = std::size_t;
using difference_type = std::ptrdiff_t;
using reference = value_type&;
using const_reference = const value_type&;
using pointer = std::allocator_traits<Allocator>::pointer;
using const_pointer = std::allocator_traits<Allocator>::const_pointer;
using iterator = /* implementation-defined */;
using const_iterator = /* implementation-defined */;
using reverse_iterator = std::reverse_iterator<iterator>;
using const_reverse_iterator = std::reverse_iterator<const_iterator>;
```

### 构造函数

| 声明 | 说明 |
|------|------|
| `vector() noexcept` | 默认构造，空容器 |
| `explicit vector(const Allocator&)` | 使用指定分配器构造 |
| `explicit vector(size_type n, const Allocator& = Allocator())` | 构造包含 n 个默认值元素的容器 |
| `vector(size_type n, const T& value, const Allocator& = Allocator())` | 构造包含 n 个 value 副本的容器 |
| `template<class InputIt> vector(InputIt first, InputIt last, const Allocator& = Allocator())` | 范围构造 |
| `vector(const vector& other)` | 复制构造 |
| `vector(vector&& other) noexcept` | 移动构造 |
| `vector(std::initializer_list<T> init, const Allocator& = Allocator())` | 列表初始化 |

## 参数说明

- **T**: 元素类型，必须满足 `Erasable` 和 `MoveInsertable` 要求
- **Allocator**: 内存分配器类型，默认使用 `std::allocator<T>`

### 元素类型要求

- 必须可复制构造或移动构造
- 必须可复制赋值或移动赋值
- 必须可析构

## 底层原理

### 内存布局

vector 使用**连续内存**存储元素，这与 C 风格数组相同：

```
内存地址:  0x1000  0x1004  0x1008  0x100C
元素:      [0]     [1]     [2]     [3]
```

这种布局带来的优势：
- **缓存友好**：连续内存访问充分利用 CPU 缓存
- **指针算术**：支持 `&v[i]` 和指针遍历
- **与 C 数组兼容**：`v.data()` 返回的指针可传递给 C 函数

### 容量管理

vector 维护三个关键指标：

| 指标 | 说明 |
|------|------|
| `size()` | 当前元素数量 |
| `capacity()` | 当前分配的存储空间 |
| `max_size()` | 理论最大容量 |

**扩容机制**：当 `size() == capacity()` 时，插入元素会触发扩容：
1. 分配新的更大内存块
2. 将现有元素移动到新位置
3. 释放旧内存
4. 扩容倍数通常是 1.5x 或 2x（实现相关）

### 时间复杂度

| 操作 | 时间复杂度 |
|------|-----------|
| 随机访问 (`operator[]`, `at`) | O(1) |
| 尾部插入/删除 (`push_back`, `pop_back`) | 均摊 O(1)，扩容时 O(n) |
| 中间插入/删除 (`insert`, `erase`) | O(n) |
| 容量查询 (`size`, `empty`, `capacity`) | O(1) |
| 迭代器遍历 | O(n) |

### 异常安全保证

- **强异常安全**：`push_back`, `emplace_back`, `insert` 等操作失败时保持容器状态不变
- **基本异常安全**：`reserve` 可能分配内存失败但不破坏容器状态
- **不抛出异常**：`swap`, `clear`, `pop_back`, `operator[]` 等操作保证不抛出异常

## 使用场景

### 适合使用 vector 的场景

| 场景 | 原因 |
|------|------|
| 需要频繁随机访问 | 连续内存，O(1) 随机访问 |
| 尾部插入/删除频繁 | 均摊 O(1) 操作 |
| 需要与 C 接口交互 | 内存布局与 C 数组兼容 |
| 需要缓存性能 | 连续内存缓存友好 |

### 不适合使用 vector 的场景

| 场景 | 推荐替代 | 原因 |
|------|---------|------|
| 频繁中间插入/删除 | `std::list` 或 `std::deque` | vector 需要 O(n) 移动元素 |
| 需要稳定的迭代器 | `std::list` | vector 扩容会使所有迭代器失效 |
| 需要键值查找 | `std::map` 或 `std::unordered_map` | vector 查找需要 O(n) |

### 最佳实践

1. **预分配空间**：如果知道大致元素数量，使用 `reserve()` 避免多次扩容
2. **使用 `emplace_back` 代替 `push_back`**：避免不必要的临时对象构造
3. **优先使用 `at()` 进行边界检查的访问**：调试时使用，生产环境可用 `operator[]`
4. **使用 `shrink_to_fit()` 减少内存占用**：在大量删除操作后

### 性能考虑

```cpp
// ❌ 不推荐：多次扩容
std::vector<int> v;
for (int i = 0; i < 1000; ++i) {
    v.push_back(i);  // 可能触发多次扩容
}

// ✅ 推荐：预分配
std::vector<int> v;
v.reserve(1000);    // 一次性分配足够空间
for (int i = 0; i < 1000; ++i) {
    v.push_back(i);
}
```

### 线程安全性

vector 本身**不是线程安全**的。如果多个线程同时访问同一个 vector：
- 只读操作是安全的
- 读写或写写操作需要外部同步（如互斥锁）

## 代码示例

### 基础用法

```cpp
#include <vector>
#include <iostream>

int main() {
    // 构造 vector
    std::vector<int> v1;              // 空向量
    std::vector<int> v2(5);          // 5 个 0
    std::vector<int> v3(5, 10);      // 5 个 10
    std::vector<int> v4 = {1, 2, 3, 4, 5};  // 列表初始化

    // 访问元素
    std::cout << "First element: " << v4[0] << std::endl;      // 1
    std::cout << "Last element: " << v4.back() << std::endl;   // 5

    // 修改元素
    v4[2] = 100;

    // 添加元素
    v4.push_back(6);

    // 遍历
    for (int x : v4) {
        std::cout << x << " ";
    }
    // 输出: 1 2 100 4 5 6

    return 0;
}
```

### 高级用法

```cpp
#include <vector>
#include <string>
#include <algorithm>

class Person {
public:
    Person(std::string name, int age) : name(std::move(name)), age(age) {}

    // 原地构造（C++11）
    Person(std::string&& name, int age) : name(std::move(name)), age(age) {}

    std::string name;
    int age;
};

int main() {
    // 使用 emplace_back 原地构造
    std::vector<Person> people;
    people.reserve(10);  // 预分配

    // ❌ 传统方式：先构造 Person，再复制
    people.push_back(Person("Alice", 25));

    // ✅ C++11 方式：原地构造，避免临时对象
    people.emplace_back("Bob", 30);  // 直接在容器内构造

    // 使用迭代器
    std::vector<int> nums = {5, 2, 8, 1, 9};
    std::sort(nums.begin(), nums.end());  // 排序

    // Lambda 表达式
    int sum = std::accumulate(nums.begin(), nums.end(), 0);

    return 0;
}
```

### 常见错误及修正

#### 错误 1：迭代器失效

```cpp
// ❌ 错误：扩容导致迭代器失效
std::vector<int> v = {1, 2, 3};
auto it = v.begin();
v.push_back(4);      // 可能扩容，it 失效！
// std::cout << *it;  // 未定义行为

// ✅ 修正：每次使用前重新获取迭代器
std::vector<int> v = {1, 2, 3};
v.push_back(4);
auto it = v.begin();
std::cout << *it;  // 安全

// ✅ 更好的修正：使用索引
std::vector<int> v = {1, 2, 3};
v.push_back(4);
std::cout << v[0];  // 安全
```

#### 错误 2：越界访问

```cpp
// ❌ 错误：未检查边界
std::vector<int> v = {1, 2, 3};
std::cout << v[5];  // 未定义行为

// ✅ 修正：使用 at() 进行边界检查
std::vector<int> v = {1, 2, 3};
try {
    std::cout << v.at(5);  // 抛出 std::out_of_range
} catch (const std::out_of_range& e) {
    std::cout << "Index out of range: " << e.what();
}
```

#### 错误 3：存储指针导致内存泄漏

```cpp
// ❌ 错误：vector 析构不会 delete 指针
{
    std::vector<int*> v;
    v.push_back(new int(42));
}  // 内存泄漏！int* 未被 delete

// ✅ 修正：使用智能指针
{
    std::vector<std::unique_ptr<int>> v;
    v.emplace_back(std::make_unique<int>(42));
}  // 自动清理，无内存泄漏
```

## 注意事项

1. **迭代器失效**：扩容时所有迭代器和引用失效
2. **移动后状态**：被移动的 vector 处于有效但未指定状态
3. **bool 特化**：`std::vector<bool>` 不是真正的容器，存在特殊行为
4. **异常安全**：push_back 操作保证强异常安全
5. **平台差异**：扩容倍数（1.5x vs 2x）在不同实现中不同

## 相关概念

| 概念 | 关系 |
|------|------|
| `std::array` | 固定大小版本，无动态分配 |
| `std::deque` | 双端队列，支持头部 O(1) 插入 |
| `std::list` | 双向链表，任意位置插入 O(1) |
| `std::string` | char 的 vector 特化 |
| `std::span` (C++20) | 对连续内存的非拥有视图 |

## 总结

`std::vector` 是 C++ 中最通用、最高效的容器之一。它提供了：

- **O(1) 随机访问**：连续内存布局
- **均摊 O(1) 尾部插入**：智能扩容机制
- **自动内存管理**：无需手动分配/释放
- **与 C 兼容**：可传递底层指针给 C 函数

核心使用建议：
1. 默认首选 vector，除非有特殊需求
2. 大致预估元素数量时使用 `reserve()`
3. 优先使用 `emplace_back` 构造元素
4. 注意迭代器失效问题

## 参考资料

- cppreference.com: https://en.cppreference.com/w/cpp/container/vector
- C++ Standard: [container.requirements]
- Effective STL, Scott Meyers, Item 14
