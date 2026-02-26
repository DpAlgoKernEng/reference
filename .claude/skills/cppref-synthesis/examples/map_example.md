# std::map - 映射容器

## 概述

`std::map` 是 C++ 标准库中的关联容器（associative container），用于存储键值对（key-value pairs）。它维护按键排序的元素集合，并允许通过键快速查找、插入和删除元素。

map 定义在 `<map>` 头文件中，位于 `std` 命名空间下。

## 来源与演变

### 首次引入

`std::map` 首次在 **C++98** 标准中引入，是 STL 的一部分。它通常使用红黑树（Red-Black Tree）作为底层数据结构。

### 历史背景

在 map 出现之前，开发者需要：
- 手工实现哈希表或二叉搜索树
- 使用 C 语言风格的关联数组（如 `dict.h`）
- 依赖第三方库

map 提供了统一的、高效的键值存储接口。

### C++11 变化

- 新增 `emplace()` 和 `emplace_hint()` 方法
- 新增 `try_emplace()` 方法
- 支持移动语义
- 新增 `at()` 方法用于键访问
- 新增 `cbegin()`, `cend()` 常量迭代器

### C++17 变化

- `try_emplace()` 方式支持处理已存在键
- `extract()` 和 `merge()` 方法用于节点处理
- 新增 `contains()` 方法

### C++20 变化

- `erase()` 返回迭代器类型改进
- 更一致的容器接口

## 语法与声明

### 基本模板定义

```cpp
namespace std {
    template<
        class Key,
        class T,
        class Compare = std::less<Key>,
        class Allocator = std::allocator<std::pair<const Key, T>>
    > class map;
}
```

### 类型别名

```cpp
using key_type = Key;
using mapped_type = T;
using value_type = std::pair<const Key, T>;
using size_type = std::size_t;
using difference_type = std::ptrdiff_t;
using key_compare = Compare;
using allocator_type = Allocator;
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
| `map()` | 默认构造，空容器 |
| `explicit map(const Compare&, const Allocator& = Allocator())` | 使用指定比较器和分配器 |
| `explicit map(const Allocator&)` | 使用指定分配器 |
| `template<class InputIt> map(InputIt first, InputIt last, const Compare& = Compare(), const Allocator& = Allocator())` | 范围构造 |
| `map(const map& other)` | 复制构造 |
| `map(map&& other) noexcept` | 移动构造 |
| `map(std::initializer_list<value_type> init, const Compare& = Compare(), const Allocator& = Allocator())` | 列表初始化 |

## 参数说明

- **Key**: 键类型，必须可比较
- **T**: 值类型
- **Compare**: 键比较器，默认为 `std::less<Key>`
- **Allocator**: 内存分配器

### 键类型要求

- 必须是可复制构造的
- 必须是可移动构造的
- 必须是可析构的
- 必须支持 `<` 比较运算符或自定义比较器

## 底层原理

### 数据结构

`std::map` 通常使用**红黑树**作为底层数据结构。红黑树是一种自平衡二叉搜索树，具有以下特性：

1. 每个节点要么是红色，要么是黑色
2. 根节点是黑色
3. 所有叶子节点（NIL）是黑色
4. 红色节点的子节点都是黑色
5. 从任一节点到其每个叶子的所有路径都包含相同数量的黑色节点

### 时间复杂度

| 操作 | 时间复杂度 |
|------|-----------|
| 插入 (`insert`) | O(log n) |
| 查找 (`find`, `operator[]`) | O(log n) |
| 删除 (`erase`) | O(log n) |
| 容量查询 (`size`, `empty`) | O(1) |
| 迭代器遍历 | O(n) |
| 范围查找 (`lower_bound`, `upper_bound`) | O(log n) |

### 与 unordered_map 对比

| 特性 | `std::map` | `std::unordered_map` |
|------|-----------|---------------------|
| 底层结构 | 红黑树 | 哈希表 |
| 查找复杂度 | O(log n) | 平均 O(1)，最坏 O(n) |
| 有序性 | 按键排序 | 无序 |
| 内存开销 | 较高（每个节点 3-4 指针） | 较低 |
| 迭代顺序 | 键排序 | 无特定顺序 |
| 需要哈希函数 | 否 | 是 |

### 迭代器特性

- **双向迭代器**：支持 `++` 和 `--` 操作
- **弱异常保证**：插入操作失败可能使容器处于中间状态
- **关键点**：map 的迭代器始终指向有效的元素，直到该元素被删除

## 使用场景

### 适合使用 map 的场景

| 场景 | 原因 |
|------|------|
| 需要按键有序遍历 | map 维护排序顺序 |
| 需要范围查询 | 支持 lower_bound/upper_bound |
| 键类型不支持哈希 | 使用比较器即可 |
| 需要稳定的迭代顺序 | map 提供稳定的遍历顺序 |

### 不适合使用 map 的场景

| 场景 | 推荐替代 | 原因 |
|------|---------|------|
| 只需要快速查找 | `std::unordered_map` | 平均 O(1) 查找 |
| 不关心键的顺序 | `std::unordered_map` | 更低内存开销 |
| 键是简单的整数 | `std::vector` + 索引 | 数组访问更快 |

### 最佳实践

1. **使用 `try_emplace` 或 `operator[]`**：根据是否需要覆盖现有值选择
2. **考虑 `unordered_map`**：如果不需要有序性，unordered_map 通常更快
3. **使用 `contains()` (C++17)**：比 `find() != end()` 更清晰的语法
4. **自定义比较器**：默认使用 `<`，可自定义比较逻辑

## 代码示例

### 基础用法

```cpp
#include <map>
#include <string>
#include <iostream>

int main() {
    // 构造 map
    std::map<std::string, int> scores;

    // 插入元素
    scores["Alice"] = 95;
    scores["Bob"] = 87;
    scores["Charlie"] = 92;

    // 查找元素
    std::cout << "Alice's score: " << scores["Alice"] << std::endl;

    // 检查键是否存在 (C++17)
    if (scores.contains("Bob")) {
        std::cout << "Bob is in the map" << std::endl;
    }

    // 遍历元素（按键排序）
    for (const auto& [name, score] : scores) {
        std::cout << name << ": " << score << std::endl;
    }

    return 0;
}
```

### 高级用法

```cpp
#include <map>
#include <string>
#include <vector>

int main() {
    // 自定义比较器（降序）
    auto cmp = [](const std::string& a, const std::string& b) {
        return a > b;  // 降序
    };
    std::map<std::string, int, decltype(cmp)> scores(cmp);

    scores["Alice"] = 95;
    scores["Bob"] = 87;

    // 使用 try_emplace (C++17)
    auto [it, inserted] = scores.try_emplace("Charlie", 100);
    if (inserted) {
        std::cout << "Charlie inserted" << std::endl;
    } else {
        std::cout << "Charlie already exists" << std::endl;
    }

    // 范围查找
    auto lower = scores.lower_bound("B");
    auto upper = scores.upper_bound("D");
    for (auto it = lower; it != upper; ++it) {
        std::cout << it->first << std::endl;
    }

    return 0;
}
```

### 常见错误及修正

#### 错误 1：忽略 operator[] 的插入行为

```cpp
// ❌ 错误：operator[] 会插入新键
std::map<std::string, int> counts;
std::cout << counts["nonexistent"];  // 插入键 "nonexistent"，值为 0

// ✅ 修正：使用 at() 或 find()
std::map<std::string, int> counts;

// 方法 1: at() 抛出异常
try {
    std::cout << counts.at("nonexistent");  // 抛出 std::out_of_range
} catch (...) { }

// 方法 2: find() 返回迭代器
auto it = counts.find("nonexistent");
if (it != counts.end()) {
    std::cout << it->second;
}
```

#### 错误 2：修改键值

```cpp
// ❌ 错误：键是 const 的，不能修改
std::map<std::string, int> m;
m["key"] = 42;
// auto& key = m.begin()->first;  // const std::string&
// key = "newkey";  // 编译错误

// ✅ 修正：需要修改键时，先删除再插入
std::map<std::string, int> m;
auto node = m.extract("oldkey");
node.key() = "newkey";
m.insert(std::move(node));  // C++17 extract
```

#### 错误 3：误用引用

```cpp
// ❌ 错误：迭代器失效后使用引用
std::map<int, std::string> m = {{1, "one"}};
auto& ref = m[1];
m[2] = "two";  // 可能导致树重组
// std::cout << ref;  // ref 可能已失效

// ✅ 修正：使用键查找
std::map<int, std::string> m = {{1, "one"}};
m[2] = "two";
auto& ref = m[1];  // 重新获取引用
std::cout << ref;
```

## 注意事项

1. **键不可修改**：map 中的键是 `const` 的
2. **迭代器顺序**：按键的升序排列
3. **operator[] 行为**：不存在键时插入默认值
4. **内存开销**：每个元素需要额外存储树结构指针
5. **异常安全**：插入操作提供弱异常保证

## 相关概念

| 概念 | 关系 |
|------|------|
| `std::unordered_map` | 无序版本，哈希表实现，更快查找 |
| `std::set` | 只存储键，不存储值 |
| `std::multimap` | 允许重复键 |
| `std::flat_map` (C++23) | 有序向量实现，更好的缓存性能 |

## 总结

`std::map` 是 C++ 中最常用的关联容器之一，提供：

- **O(log n) 查找**：平衡的红黑树结构
- **有序遍历**：按键自动排序
- **范围查询**：支持 lower_bound/upper_bound
- **稳定的迭代顺序**：便于调试和预测行为

选择建议：
- 需要有序性 → 使用 `map`
- 只需快速查找 → 考虑 `unordered_map`
- 简单整数键 → 考虑 `vector` + 索引

## 参考资料

- cppreference.com: https://en.cppreference.com/w/cpp/container/map
- C++ Standard: [associative.reqmts]
- Effective STL, Scott Meyers, Item 22
