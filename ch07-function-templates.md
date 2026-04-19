# 第7章 函数模板 · 预览版

> 本章你将学习如何用一份代码处理任意数据类型——这就是泛型编程的魅力。从基础的函数模板到高级的可变参数模板，你将掌握编写通用、高效、可复用代码的核心技术。  
> **完整版包含**：函数模板定义与实例化、非类型模板参数、返回类型推导、模板重载与专门化、可变参数模板（参数包展开、折叠表达式、递归处理）、模板模板参数、实战案例（通用数据处理函数库），以及大量习题与实验。  
> 📖 文末有购买链接，解锁全部16章 + 实战项目。

---

## 7.1 为什么需要函数模板？

假设你要写一个交换两个变量值的函数。如果不用模板，你需要为每种数据类型重复编写几乎完全相同的代码：

```cpp
void swap(int& a, int& b) { int t = a; a = b; b = t; }
void swap(double& a, double& b) { double t = a; a = b; b = t; }
void swap(std::string& a, std::string& b) { std::string t = a; a = b; b = t; }
// ... 每增加一种类型就要多写一份
```

这不仅冗长，而且难以维护。**函数模板** 允许你只写一次，让编译器根据实际类型自动生成对应版本：

```cpp
#include <print>

template <typename T>
void swap(T& a, T& b) {
    T temp = a;
    a = b;
    b = temp;
}

int main() {
    int x = 1, y = 2;
    swap(x, y);              // 编译器生成 int 版本
    double a = 1.5, b = 2.5;
    swap(a, b);              // 编译器生成 double 版本
    std::println("x = {}, y = {}", x, y);
    std::println("a = {}, b = {}", a, b);
}
```

**优点**：代码复用、类型安全、易于维护。

> 完整版会详细讲解模板的实例化过程、自动推导与显式指定、以及非类型模板参数（如编译期常量）。

---

## 7.2 非类型模板参数：让数值也成为模板参数

除了类型，模板还可以接受编译期常量值。例如，编写一个函数，重复打印某个值 `N` 次：

```cpp
#include <print>

template <int N, typename T>
void repeat_print(T value) {
    for (int i = 0; i < N; ++i)
        std::print("{} ", value);
    std::println("");
}

int main() {
    repeat_print<3>("hello");   // 输出 hello hello hello
    repeat_print<5>(42);        // 输出 42 42 42 42 42
}
```

`N` 在编译期就确定了，不会有运行时开销。你甚至可以让编译器自动推导数组大小：

```cpp
#include <print>

template <typename T, int N>
T sum(const T (&arr)[N]) {      // 数组引用，N 自动推导为数组长度
    T total{};
    for (int i = 0; i < N; ++i) total += arr[i];
    return total;
}

int main() {
    int arr[] = {1, 2, 3, 4, 5};
    std::println("{}", sum(arr));      // 输出 15，N 自动推导为 5
}
```

> 完整版会深入讲解非类型模板参数的允许类型、C++17的 `auto` 非类型参数、以及如何利用它实现编译期计算。

---

## 7.3 可变参数模板：处理任意数量、任意类型的参数

传统方法（函数重载、`initializer_list`）要么代码爆炸，要么只能处理同类型参数。**可变参数模板** 可以接受任意数量、任意类型的参数。

**示例：打印任意多个任意类型的值（C++17 折叠表达式）**

```cpp
#include <print>

template <typename... Args>
void print_all(Args... args) {
    ((std::print("{} ", args)), ...);   // 逗号折叠
    std::println("");
}

int main() {
    print_all(1, 2.5, "hello", 'x');     // 输出 1 2.5 hello x
}
```

**示例：求和（折叠表达式）**

```cpp
#include <print>

template <typename... Args>
auto sum(Args... args) {
    return (args + ... + 0);   // 二元右折叠，空包时返回 0
}

int main() {
    std::println("{}", sum(1, 2, 3, 4));        // 10
    std::println("{}", sum(1, 2.5, 3));         // 6.5（混合类型自动推导）
}
```

**示例：递归模板处理（C++11 经典方式）**

```cpp
#include <print>

// 基情形
void print() {}

// 递归情形：处理第一个参数，再递归处理剩余
template <typename T, typename... Rest>
void print(T first, Rest... rest) {
    std::print("{} ", first);
    print(rest...);
}

int main() {
    print(1, 2.5, "hello", 'x');
    std::println("");
}
```

> 完整版会对比三种处理方式（参数包展开、折叠表达式、递归模板）的优缺点，并给出实战案例：通用的最大值、最小值、平均值、打印函数库。

---

## 7.4 模板专门化：为特定类型定制行为

通用模板适用于大多数类型，但有时某些类型需要特殊处理。例如，比较 C 风格字符串时，我们希望比较内容而不是指针地址：

```cpp
#include <print>
#include <cstring>

// 通用模板
template <typename T>
bool is_equal(T a, T b) {
    return a == b;
}

// 针对 const char* 的全特化
template <>
bool is_equal<const char*>(const char* a, const char* b) {
    return std::strcmp(a, b) == 0;
}

int main() {
    const char* s1 = "hello";
    const char* s2 = "hello";
    std::println("{}", is_equal(s1, s2));   // 调用特化版本，输出 true
}
```

> 完整版会讲解全特化与偏特化的区别（函数模板只有全特化），以及何时使用重载而非特化。

---

## 7.5 模板模板参数：将模板作为参数

当你需要编写一个函数，它能够保持容器类型但改变元素类型时，就需要模板模板参数。例如，将 `vector<int>` 转换为 `vector<double>`，同时保持容器类型为 `vector`：

```cpp
#include <vector>
#include <print>

template <template <typename...> class Container, typename T>
Container<double> convertToDouble(const Container<T>& src) {
    Container<double> dst;
    for (const auto& elem : src)
        dst.push_back(static_cast<double>(elem));
    return dst;
}

int main() {
    std::vector<int> vi = {1, 2, 3};
    auto vd = convertToDouble(vi);   // vd 类型为 std::vector<double>
    for (double d : vd) std::print("{} ", d);
    std::println("");
}
```

> 完整版会详细解释模板模板参数的语法、匹配标准容器的技巧（使用 `typename...` 接受任意数量参数），以及它的典型应用场景。

---

## 7.6 实战：通用的数据处理函数库

综合运用本章知识，编写一组实用函数：

- `max_value`：求任意多个数的最大值（递归模板）
- `min_value`：求最小值（递归模板）
- `average`：求平均值（折叠表达式）
- `print_all`：打印任意多个任意类型的值（逗号折叠）

```cpp
#include <print>
#include <algorithm>

// 最大值（递归）
template <typename T>
T max_value(T t) { return t; }

template <typename T, typename... Rest>
auto max_value(T first, Rest... rest) {
    return std::max(first, max_value(rest...));
}

// 平均值（折叠表达式）
template <typename... Args>
double average(Args... args) {
    static_assert(sizeof...(args) > 0, "需要至少一个参数");
    auto sum = (args + ... + 0.0);
    return sum / sizeof...(args);
}

// 打印（逗号折叠）
template <typename... Args>
void print_all(Args... args) {
    ((std::print("{} ", args)), ...);
    std::println("");
}

int main() {
    std::println("max = {}", max_value(3, 7, 2, 9));    // 9
    std::println("avg = {}", average(1, 2, 3, 4));      // 2.5
    print_all(1, 2.5, "hello", 'x');                    // 1 2.5 hello x
}
```

> 完整版会给出完整的代码和运行结果，并讨论扩展（如处理空参数包、类型转换等）。

---

## ✨ 为什么要购买完整版？

你在本预览版中已经学到了：
- 函数模板解决代码重复问题
- 非类型模板参数（编译期常量）
- 可变参数模板（折叠表达式、递归处理）
- 模板专门化（为特定类型定制行为）
- 模板模板参数（将模板作为参数）
- 一个实用的通用函数库雏形

**完整版第7章还将提供**：
- ✅ 函数模板的实例化机制（隐式 vs 显式）
- ✅ 返回类型推导（`auto`、`decltype(auto)`、尾置返回类型）
- ✅ 模板重载的解析规则与最佳实践
- ✅ 可变参数模板的三种处理方式详细对比
- ✅ 折叠表达式的所有操作符示例（`+`、`*`、`&&`、`||`、`,` 等）
- ✅ 模板专门化的注意事项（与重载的区别）
- ✅ **50+ 道习题**（选择、填空、简答、编程）
- ✅ **实战项目**：通用数据处理函数库的完整扩展（支持空包、混合类型等）

以及后续 **9 章** 的精彩内容：类与对象、运算符重载、派生类、类模板、异常处理、移动语义、内存管理、标准库、元编程……每章都配有实战案例。

---

## 📖 立即购买《现代C++编程实战》电子版

- 一次性解锁全部 16 章 + 8 个完整实战项目
- 支持作者持续创作，获取最新更新（C++26 特性）
- 可在线阅读、离线下载（PDF/EPUB）

👉 [点击前往 Leanpub 购买](https://leanpub.com/c01)

**读者评价**：
> “这本书直接讲现代 C++，没有 C 语言的旧包袱，示例简洁易懂，适合自学。” —— 某位已购读者

---

**继续阅读**：点击侧边栏第8章预览，学习类与对象（面向对象编程的核心）。  
**购买后**：你将收到完整版 PDF，并可以随时在线阅读全书。
