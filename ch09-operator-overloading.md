# 第9章 运算符重载 · 预览版

> 本章你将学习如何让自定义类型像内置类型一样使用运算符——`+`、`-`、`[]`、`()`、`<<`、`>>` 甚至 `<=>`。运算符重载让代码更直观、更自然。  
> **完整版包含**：成员函数与非成员函数重载的区别、赋值运算符的深拷贝实现、下标运算符 `[]`、函数调用运算符 `()`、输入输出运算符 `<<`/`>>`、自增/自减的前缀后缀形式、C++20 三向比较运算符 `<=>`、类型转换运算符、可以重载的运算符列表、以及完整实战项目：**矩阵类（支持加减乘、深拷贝、下标访问）**。  
> 📖 文末有购买链接，解锁全部16章 + 实战项目。

---

## 9.1 为什么需要运算符重载？

假设你定义了一个 `Point` 类表示二维点，如果想让两个点相加（对应坐标相加），传统做法是写一个 `add` 函数：

```cpp
Point p1{2,3}, p2{4,5};
Point p3 = add(p1, p2);   // 不够直观
```

而通过重载 `+` 运算符，你可以直接写：

```cpp
Point p3 = p1 + p2;        // 清晰、自然
```

C++ 允许为自定义类型重载大多数运算符，包括算术、关系、下标、函数调用、输入输出等。每个运算符对应一个**运算符函数**，如 `operator+`、`operator[]`。

---

## 9.2 两种重载方式：成员函数 vs 外部函数

### 成员函数重载

```cpp
class Point {
    double x{}, y{};
public:
    Point(double x, double y) : x{x}, y{y} {}
    Point operator+(const Point& other) const {
        return Point(x + other.x, y + other.y);
    }
};
```

调用 `p1 + p2` 等价于 `p1.operator+(p2)`。**左操作数必须是本类对象**，不能是其他类型。

### 外部函数重载（通常需要友元）

```cpp
class Point {
    // ...
    friend Point operator+(const Point& a, const Point& b);
};

Point operator+(const Point& a, const Point& b) {
    return Point(a.x + b.x, a.y + b.y);
}
```

外部函数允许左右操作数**隐式转换**。例如，如果 `Point` 有一个 `Point(double)` 构造函数，那么 `2 + p1` 会自动转换为 `Point(2) + p1`，而成员函数版本不支持。

> 完整版会详细对比两种方式的适用场景，并给出最佳实践。

---

## 9.3 下标运算符 `[]` 与函数调用运算符 `()`

### 下标运算符 `[]`

让对象像数组一样通过下标访问成员。通常需要提供两个版本：`const` 版本（只读）和非 `const` 版本（可写）。

```cpp
class Point {
    double x{}, y{};
public:
    double& operator[](int i) {
        if (i == 0) return x;
        if (i == 1) return y;
        throw std::out_of_range("下标非法");
    }
    double operator[](int i) const {  // const 版本
        if (i == 0) return x;
        if (i == 1) return y;
        throw std::out_of_range("下标非法");
    }
};
```

### 函数调用运算符 `()`

让对象“像函数一样”被调用，称为**函数对象**（Functor）。

```cpp
class Point {
    double x{}, y{};
public:
    double operator()(int n = 2) const {
        if (n == 1) return std::abs(x) + std::abs(y);
        return std::pow(x, n) + std::pow(y, n);
    }
};
// 使用
Point p{2,3};
std::println("{}", p(3));   // 2^3 + 3^3 = 35
```

函数对象广泛用于标准库算法（如 `std::sort` 的自定义比较），比函数指针更灵活、高效。

---

## 9.4 输入输出运算符 `<<` 和 `>>`

它们必须重载为**外部函数**，因为左操作数是 `std::ostream` 或 `std::istream`，而不是自定义类。

```cpp
class Point {
    double x{}, y{};
public:
    friend std::ostream& operator<<(std::ostream& os, const Point& p);
    friend std::istream& operator>>(std::istream& is, Point& p);
};

std::ostream& operator<<(std::ostream& os, const Point& p) {
    os << '(' << p.x << ',' << p.y << ')';
    return os;
}

std::istream& operator>>(std::istream& is, Point& p) {
    is >> p.x >> p.y;
    return is;
}
```

现在可以像内置类型一样使用 `cin` 和 `cout`：

```cpp
Point p;
std::cin >> p;          // 输入 2 3
std::cout << p << '\n'; // 输出 (2,3)
```

---

## 9.5 自增/自减运算符：前缀与后缀

- **前缀** `++p`：先自增，返回**自身引用**。  
- **后缀** `p++`：先返回**旧值副本**，再自增。

```cpp
class Point {
    double x{}, y{};
public:
    Point& operator++() {      // 前缀
        ++x; ++y;
        return *this;
    }
    Point operator++(int) {    // 后缀（int 参数仅用于区分）
        Point temp = *this;
        ++(*this);
        return temp;
    }
};
```

> 前缀效率更高（无拷贝），应优先使用。

---

## 9.6 C++20 三向比较运算符 `<=>`（太空船）

C++20 引入了 `<=>` 运算符，**只需重载一次，编译器自动生成 `<`、`<=`、`>`、`>=`、`==` 和 `!=`**，极大减少重复代码。

### 基本用法：`= default`

```cpp
#include <compare>

class Point {
    double x{}, y{};
public:
    auto operator<=>(const Point&) const = default;   // 按成员顺序比较
};
```

`= default` 会按成员声明顺序依次比较，并自动推导合适的返回类型：
- 如果所有成员都是整数（无 `NaN`），返回 `std::strong_ordering`（全序）。
- 如果包含 `float`/`double`，返回 `std::partial_ordering`（支持 `NaN` 不可比较）。

### 手动实现（例如大小写不敏感字符串）

```cpp
class CaseInsensitiveString {
    std::string s;
public:
    std::weak_ordering operator<=>(const CaseInsensitiveString& other) const {
        // 自定义比较逻辑（忽略大小写）
        // ...
    }
    bool operator==(const CaseInsensitiveString& other) const = default;
};
```

> 完整版会详细讲解三种比较类别（`strong_ordering`、`weak_ordering`、`partial_ordering`）的区别以及何时使用 `= default`。

---

## 9.7 类型转换运算符

通过 `operator type()` 可以将类对象隐式转换为其他类型。

```cpp
class Point {
    double x{}, y{};
public:
    operator double() const {   // 转换为原点到该点的距离平方
        return x*x + y*y;
    }
};

Point p{3,4};
double d = p;   // d = 25
```

如果希望禁止隐式转换，可以加 `explicit`：

```cpp
explicit operator double() const { ... }
// 使用时必须 static_cast<double>(p)
```

> 注意：隐式转换可能导致歧义（例如同时定义了 `int` 转换和 `double` 转换），应谨慎使用。

---

## 9.8 实战：矩阵类（部分展示）

矩阵类是运算符重载的经典案例。下面展示核心结构，完整版包含深拷贝、加减乘、下标访问等全部实现。

```cpp
#include <cassert>

class Matrix {
    double* data{nullptr};
    int rows{}, cols{};
public:
    Matrix(int m, int n) : rows(m), cols(n) {
        data = new double[rows * cols]{};
    }
    // 深拷贝构造、赋值、析构（完整版提供）
    
    double& operator()(int i, int j) {        // 二维下标访问
        assert(i >= 0 && i < rows && j >= 0 && j < cols);
        return data[i * cols + j];
    }
    double operator()(int i, int j) const {   // const 版本
        assert(i >= 0 && i < rows && j >= 0 && j < cols);
        return data[i * cols + j];
    }
    
    Matrix& operator+=(const Matrix& other);
    friend Matrix operator+(const Matrix& a, const Matrix& b);
    // ... 其他运算符（-、*、==等）
};
```

使用示例：

```cpp
Matrix A(2,3), B(2,3);
// 初始化 A、B
Matrix C = A + B;       // 矩阵加法
C(0,0) = 99;            // 修改元素
std::println("{}", C(0,0));
```

> 完整版还会实现矩阵乘法 `operator*`、单位矩阵静态方法 `identity()`，并详细讲解如何用 `copy-and-swap` 实现异常安全的赋值运算符。

---

## ✨ 为什么要购买完整版？

你在本预览版中已经学到了：
- 运算符重载的两种方式及其区别
- 下标 `[]` 和函数调用 `()` 运算符
- 输入输出 `<<`/`>>` 运算符
- 前缀/后缀自增自减
- C++20 `<=>` 三向比较（极大简化比较运算符）
- 类型转换运算符
- 矩阵类的核心设计

**完整版第9章还将提供**：
- ✅ 赋值运算符的深拷贝实现与 `copy-and-swap` 惯用法
- ✅ 所有运算符的详细列表与设计原则（哪些必须成员、哪些可以外部）
- ✅ `<=>` 三种返回类型的深入对比（`strong_ordering`、`weak_ordering`、`partial_ordering`）
- ✅ 避免隐式转换歧义的最佳实践（`explicit` 关键字）
- ✅ **矩阵类完整实现**：加法、减法、乘法、深拷贝、下标访问、单位矩阵
- ✅ **50+ 道习题**（选择、填空、简答、编程）
- ✅ **多个实验**：复数类、字符串类、大整数类等

以及后续 **7 章** 的精彩内容：派生类、类模板、异常处理、移动语义、内存管理、标准库、元编程……每章都配有实战案例。

---

## 📖 立即购买《现代C++编程实战》电子版

- 一次性解锁全部 16 章 + 8 个完整实战项目
- 支持作者持续创作，获取最新更新（C++26 特性）
- 可在线阅读、离线下载（PDF/EPUB）

👉 [点击前往 Leanpub 购买](https://leanpub.com/c01)

**读者评价**：
> “这本书直接讲现代 C++，没有 C 语言的旧包袱，示例简洁易懂，适合自学。” —— 某位已购读者

---

**继续阅读**：点击侧边栏第10章预览，学习继承与多态（面向对象的另一核心）。  
**购买后**：你将收到完整版 PDF，并可以随时在线阅读全书。
