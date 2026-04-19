# 第8章 类与对象 · 预览版

> 本章你将学习面向对象编程（OOP）的核心思想，从定义类、创建对象到掌握构造函数、析构函数、拷贝控制、静态成员、友元等关键语法，并通过**智能家居系统**、**顺序表/链表**和**贪吃蛇游戏**等实战案例，体会OOP如何让代码更清晰、更易扩展。  
> **完整版包含**：类与对象的所有细节（封装、继承、多态基础）、`this`指针、成员初始化列表、深浅拷贝、`explicit`与`delete`、`const`成员函数、`mutable`、静态成员、友元、内联成员函数、对象数组、头文件分离，以及两个大型实战项目（线性表实现 + 贪吃蛇游戏引擎），还有大量习题与实验。  
> 📖 文末有购买链接，解锁全部16章 + 实战项目。

---

## 8.1 为什么需要面向对象？—— 智能家居对比

**过程式编程**将数据和操作分离，全局数据被所有函数共享。当系统变复杂时，这种“数据裸奔”会导致耦合高、扩展难。

**面向对象编程（OOP）** 将数据和操作封装在**对象**中，通过隐藏内部细节、暴露公共接口，使代码更易维护和扩展。

**智能家居案例**（完整版含完整代码）：
- 过程式：用全局数组和函数控制灯光、空调，新增设备需修改多处代码。
- OOP：定义 `Light`、`AirConditioner` 类，每个设备自己知道如何开关和调节；`Scene` 类批量控制设备；`SmartHomeHub` 协调一切。添加新设备只需写一个新类，无需改动原有逻辑。

> 完整版会详细对比两种范式的优缺点，并给出完整的OOP智能家居实现（含类图、序列图）。

---

## 8.2 类与对象基础

**类**是蓝图，**对象**是实例。用 `class` 或 `struct` 定义：

```cpp
class Date {
private:
    int year{2000}, month{1}, day{1};
public:
    void print() const {
        std::cout << year << '-' << month << '-' << day << '\n';
    }
};

int main() {
    Date today;               // 对象
    today.print();            // 输出 2000-1-1
}
```

**成员访问**：`.` 用于对象，`->` 用于指针。  
**`this`指针**：每个非静态成员函数内隐含指向当前对象的指针。

> 完整版会深入讲解 `this` 的用法、返回 `*this` 实现链式调用，以及类对象的内存布局（对齐与大小）。

---

## 8.3 构造函数：对象初始化

构造函数与类同名，无返回值，在创建对象时自动调用。

```cpp
class Date {
    int year, month, day;
public:
    // 带默认参数的构造函数（也是默认构造函数）
    Date(int y = 2000, int m = 1, int d = 1)
        : year(y), month(m), day(d) {}   // 初始化列表
};
```

**初始化列表**比在函数体内赋值更高效，且**必须**用于 `const` 成员和引用成员。

**委托构造函数**（C++11）：一个构造函数调用另一个构造函数，避免重复代码。

```cpp
Date(int y, int m, int d) : year(y), month(m), day(d) {}
Date(const int* arr) : Date(arr[0], arr[1], arr[2]) {}
```

> 完整版还会介绍拷贝构造函数、移动构造函数（后续章节）、`=default` 和 `=delete` 的用法。

---

## 8.4 拷贝控制：深拷贝 vs 浅拷贝

编译器默认生成的拷贝构造函数和赋值运算符执行**浅拷贝**——只复制指针值，不复制指向的资源。这会导致多个对象共享同一内存，析构时重复释放（double free）。

**深拷贝**：为指针成员分配独立内存，复制内容。

```cpp
class String {
    char* data;
public:
    // 深拷贝构造函数
    String(const String& other) {
        data = new char[other.len + 1];
        std::strcpy(data, other.data);
    }
    // 深拷贝赋值运算符
    String& operator=(const String& other) {
        if (this != &other) {
            delete[] data;
            data = new char[other.len + 1];
            std::strcpy(data, other.data);
        }
        return *this;
    }
    ~String() { delete[] data; }
};
```

**规则三/五**：如果类需要自定义析构函数，则通常也需要自定义拷贝构造和拷贝赋值（以及移动构造和移动赋值）。

> 完整版会详细演示 `String` 类的完整实现，并解释为什么浅拷贝危险。

---

## 8.5 访问控制与友元

- `private`：仅类内部可访问（数据隐藏）。
- `public`：对外接口。
- `protected`：供派生类访问（第10章）。

**友元**（`friend`）：允许外部函数或类访问私有成员，用于运算符重载或紧密协作。

```cpp
class A {
    friend void printA(const A& a);   // 全局函数友元
    friend class B;                    // 类友元
private:
    int secret;
};
```

> 友元破坏封装，应谨慎使用。完整版会给出典型场景（如 `<<` 运算符重载）。

---

## 8.6 静态成员：属于类本身

- **静态成员变量**：所有对象共享一份，必须在类外定义（C++17起可用 `inline` 在类内定义）。
- **静态成员函数**：只能访问静态成员，可通过类名直接调用。

```cpp
class Counter {
    static inline int count = 0;   // C++17
public:
    Counter() { ++count; }
    static int getCount() { return count; }
};
```

> 静态成员常用于对象计数、单例模式等。

---

## 8.7 const 与 mutable

- **`const` 对象**：只能调用 `const` 成员函数。
- **`const` 成员函数**：承诺不修改对象状态（非 `mutable` 成员）。
- **`mutable`**：允许在 `const` 成员函数中修改（如缓存、日志计数器）。

```cpp
class Cache {
    mutable int accessCount = 0;
    int data;
public:
    int get() const {
        ++accessCount;   // 允许修改 mutable 成员
        return data;
    }
};
```

> 完整版会讲解 `const` 重载（同一函数名提供 const 和非 const 版本）以及 `const_cast` 的危险用法。

---

## 8.8 实战：线性表（顺序表 & 链表）

**顺序表（Vector）**：基于动态数组，支持随机访问，中间插入/删除需移动元素。  
**链表（List）**：基于节点指针，插入/删除快，但随机访问慢。

**Vector 核心**（完整版给出完整代码）：
```cpp
class Vector {
    ElemType* data;
    int capacity, size;
    void expand() { /* 扩容 */ }
public:
    bool insert(int pos, const ElemType& e);
    bool erase(int pos);
    // ...
};
```

**List 核心**：
```cpp
class List {
    struct Node { ElemType data; Node* next; };
    Node* head;
    Node* locate(int i) const;  // 定位第 i 个节点
public:
    bool push_front(const ElemType& e);
    bool pop_front();
    // ...
};
```

基于这些容器，可实现**图书管理系统**（完整版提供可运行代码）。

> 通过自己实现容器，能深入理解 `std::vector` 和 `std::list` 的内部原理，为后续学习标准库打下坚实基础。

---

## 8.9 实战：贪吃蛇游戏（OOP 游戏引擎）

将游戏拆分为对象：
- `ChGL`：字符图形库（帧缓冲区、绘图）
- `InputHandler`：跨平台键盘输入
- `BackGround`：绘制边界
- `Snake`：用链表存储身体节点，支持移动、吃食物、增长
- `Egg`：随机生成食物
- `GameEngine`：主循环，处理事件、更新、碰撞、渲染

**Snake 移动核心**（链表头尾操作）：
```cpp
void Snake::move() {
    // 计算新蛇头位置
    Position newHead = ...;
    // 链表尾插入新节点（成为新蛇头）
    tail->next = new Node(newHead);
    tail = tail->next;
    if (!eating) {
        // 未吃到食物则删除链表头节点（蛇尾）
        Node* p = head->next;
        head->next = p->next;
        delete p;
    } else {
        eating = false;  // 吃到食物，蛇长增加
    }
}
```

完整版提供约300行完整代码，支持方向键控制、碰撞检测（边界+自身）、得分和难度加速。

---

## ✨ 为什么要购买完整版？

你在本预览版中已经学到了：
- 类与对象的基本概念、构造函数、初始化列表
- 深拷贝与浅拷贝的区别及实现
- 访问控制、友元、静态成员、`const`/`mutable`
- 顺序表和链表的核心设计思想
- 贪吃蛇游戏的OOP架构

**完整版第8章还将提供**：
- ✅ 拷贝构造函数、赋值运算符的完整实现与自赋值处理
- ✅ `explicit` 防止隐式转换、`delete` 禁用函数
- ✅ 委托构造函数、类内成员初始化器的对比
- ✅ 类对象数组、头文件与源文件分离的最佳实践
- ✅ 线性表（Vector + List）的完整实现（含扩容、定位、插入删除）
- ✅ **图书管理系统**完整代码（基于Vector/List）
- ✅ **贪吃蛇游戏**完整代码（含跨平台输入、帧动画、碰撞检测）
- ✅ **50+ 道习题**（选择、填空、简答、编程）
- ✅ **多个实验**：智能灯、快递包裹、String类深拷贝、学生成绩管理等

以及后续 **8 章** 的精彩内容：运算符重载、派生类、类模板、异常处理、移动语义、内存管理、标准库、元编程……每章都配有实战案例。

---

## 📖 立即购买《现代C++编程实战》电子版

- 一次性解锁全部 16 章 + 8 个完整实战项目
- 支持作者持续创作，获取最新更新（C++26 特性）
- 可在线阅读、离线下载（PDF/EPUB）

👉 [点击前往 Leanpub 购买](https://leanpub.com/c01)

**读者评价**：
> “这本书直接讲现代 C++，没有 C 语言的旧包袱，示例简洁易懂，适合自学。” —— 某位已购读者

---

**继续阅读**：点击侧边栏第9章预览，学习运算符重载（让自定义类型像内置类型一样运算）。  
**购买后**：你将收到完整版 PDF，并可以随时在线阅读全书。
