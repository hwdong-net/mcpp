# 第10章 继承与派生 · 预览版

> 继承是面向对象编程的基石之一，它让你能够基于已有类创建新类，实现代码复用和类型层次化设计。本章将带你深入理解 C++ 的继承机制，包括访问控制、构造/析构顺序、虚函数与多态、抽象类、菱形继承与虚基类，以及两个大型实战项目：**公司员工管理系统** 和 **雷电战机游戏**。  
> **完整版包含**：所有继承方式对比、对象切割、向上/向下转换、`override`/`final` 关键字、运行时类型识别（RTTI）、多继承二义性解决方案，以及完整的代码实现与测试样例。  
> 📖 文末有购买链接，解锁全部16章 + 实战项目。

---

## 10.1 为什么需要继承？

假设你要描述普通汽车和跑车。如果不使用继承，你可能会定义两个独立的类，导致大量代码重复：

```cpp
class Car {
    double speed, fuel;
public:
    void drive() { /* ... */ }
    void showStatus() { /* ... */ }
};

class SportsCar {
    double speed, fuel, acceleration;
public:
    void drive() { /* ... */ }      // 重复
    void showStatus() { /* ... */ } // 重复
    void turboBoost() { /* ... */ }
};
```

继承允许你将公共部分提取到基类，派生类只需关注自己特有的内容：

```cpp
#include <print>

class Car {
protected:
    double speed{0.0}, fuel{100.0};
public:
    void drive() { std::println("行驶中..."); }
    void showStatus() const {
        std::println("速度: {} km/h, 燃油: {} L", speed, fuel);
    }
};

class SportsCar : public Car {
    double acceleration{5.0};
public:
    void turboBoost() {
        speed += acceleration * 2;
        std::println("涡轮增压！速度提升至 {} km/h", speed);
    }
};
```

**关键点**：`SportsCar` 自动获得了 `Car` 的成员，并可以添加自己的成员或重写基类方法。

> 完整版会详细讲解 `protected` 成员的作用、构造/析构顺序、以及如何正确初始化基类部分。

---

## 10.2 访问控制与继承方式

继承方式（`public`、`protected`、`private`）决定了基类成员在派生类中的可见性：

| 基类成员 | public 继承 | protected 继承 | private 继承 |
| :------- | :---------- | :------------- | :----------- |
| private  | 不可见      | 不可见         | 不可见       |
| protected| protected   | protected      | private      |
| public   | public      | protected      | private      |

- **public 继承**：表达 **is-a** 关系，派生类对象可被视为基类对象。
- **protected/private 继承**：主要用于实现重用，不表达接口继承。

> 完整版会通过示例演示不同继承方式对成员访问的影响，并讨论何时使用 `private` 继承。

---

## 10.3 虚函数与多态

当通过基类指针或引用调用一个普通函数时，调用的是基类的版本（静态绑定）。要实现根据对象实际类型调用正确的函数，需要将函数声明为 **虚函数**（`virtual`）。

```cpp
#include <print>

class Animal {
public:
    virtual void speak() const { std::println("动物叫"); }
};

class Dog : public Animal {
public:
    void speak() const override { std::println("汪汪"); }
};

class Cat : public Animal {
public:
    void speak() const override { std::println("喵喵"); }
};

int main() {
    Animal* zoo[] = { new Dog(), new Cat() };
    for (auto a : zoo) {
        a->speak();   // 输出：汪汪  喵喵（动态绑定）
        delete a;
    }
}
```

**虚函数的工作原理**：每个包含虚函数的类都有一个虚函数表（VTable），每个对象有一个指向该表的指针（VPtr）。调用时通过 VPtr 找到正确的函数地址。

> 完整版会深入讲解虚函数表、虚析构函数（防止内存泄漏）、`override`/`final` 关键字，以及 `dynamic_cast` 的安全向下转换。

---

## 10.4 抽象类与纯虚函数

包含纯虚函数（`= 0`）的类称为**抽象类**，不能实例化，只能作为基类。纯虚函数定义接口，强制派生类实现。

```cpp
class Shape {
public:
    virtual double area() const = 0;   // 纯虚函数
    virtual void draw() const = 0;
};

class Circle : public Shape {
    double radius;
public:
    Circle(double r) : radius(r) {}
    double area() const override { return 3.14159 * radius * radius; }
    void draw() const override { std::println("画圆"); }
};
```

> 完整版会介绍纯虚函数可以有定义（提供默认行为），以及纯虚析构函数必须提供定义的原因。

---

## 10.5 菱形继承与虚基类

当派生类通过多条路径继承同一个基类时，形成菱形继承，导致基类子对象重复。

```cpp
class A { public: int a; };
class B : public A {};
class C : public A {};
class D : public B, public C {};  // D 中包含两份 A
```

解决方法：使用 **虚继承**（`virtual` 关键字）。

```cpp
class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {};  // D 中只有一份 A
```

虚基类的构造函数由最底层派生类直接调用，确保只初始化一次。

> 完整版会给出完整的菱形继承代码示例，并解释构造/析构顺序以及内存布局。

---

## 10.6 实战项目一：公司员工管理系统

本项目综合运用继承、多态、组合和自定义容器，实现一个完整的员工管理程序。

**类层次设计**：

- `Employee`（基类）：姓名、ID、雇佣日期、部门、基本工资，纯虚函数 `calculateSalary()` 和 `displayInfo()`。
- `Manager`（经理）：增加管理级别，工资 = 基本工资 + 级别 × 500。
- `Developer`（开发人员）：增加擅长语言，工资 = 基本工资 + 1000（项目奖金）。

**核心功能**：添加、删除、查找、修改员工，计算总薪资，利用多态显示不同类型员工的详细信息。

```cpp
class Employee {
    // ...
public:
    virtual double calculateSalary() const = 0;
    virtual void displayInfo() const = 0;
    virtual ~Employee() = default;
};

class Manager : public Employee {
    int level;
public:
    double calculateSalary() const override {
        return getBaseSalary() + level * 500;
    }
    void displayInfo() const override {
        std::println("经理：{}，工资：{:.2f}", getName(), calculateSalary());
    }
};
```

> 完整版提供完整代码（约300行），包括日期类、自定义动态数组 `Vector`，以及交互式菜单。读者可以编译运行，体验完整的增删改查流程。

---

## 10.7 实战项目二：雷电战机游戏（控制台版）

基于第8章开发的字符图形库 `ChGL`，利用继承和多态实现一个竖版射击游戏。

**精灵类层次**：

- `Sprite`（抽象基类）：位置、速度、尺寸、生命值，纯虚函数 `update()`、`draw()`。
- `Fighter`（战机）：继承 `Sprite`，增加射击能力。
- `Player`：玩家战机，可响应键盘移动和射击。
- `Enemy`：敌机，自动移动、随机射击。
- `Bullet`：子弹。

**多态核心**：`GameEngine` 管理 `Sprite*` 容器，每帧调用 `update()` 和 `draw()`，实现统一的更新和渲染。

```cpp
class Sprite {
public:
    virtual void update() = 0;
    virtual void draw() = 0;
    virtual ~Sprite() = default;
};

class Enemy : public Sprite {
    // 实现自动移动、射击计时
    void update() override {
        // 向下移动，每隔一段时间发射子弹
    }
};
```

> 完整版提供完整游戏代码（约400行），包含碰撞检测、子弹管理、计时器、随机移动等。读者可以亲手编译运行，体验控制台游戏的开发乐趣。

---

## ✨ 为什么要购买完整版？

你在本预览版中已经学到了：
- 继承的基本语法和 is-a 关系
- 访问控制与继承方式
- 虚函数实现运行时多态
- 抽象类与纯虚函数
- 菱形继承与虚基类的概念

**完整版第10章还将提供**：
- ✅ 构造/析构函数的详细调用顺序与初始化列表用法
- ✅ 拷贝构造函数在继承中的正确写法（显式调用基类拷贝构造）
- ✅ 对象切割现象及避免方法
- ✅ 向上类型转换（隐式安全）与向下类型转换（`static_cast` vs `dynamic_cast`）
- ✅ `override` 和 `final` 关键字的正确使用
- ✅ 多继承的二义性解决方案（作用域解析符）
- ✅ **公司员工管理系统**完整代码（约300行，含菜单、动态数组、日期类）
- ✅ **雷电战机游戏**完整代码（约400行，含图形库、精灵管理、碰撞检测）
- ✅ **50+ 道习题**（选择、填空、简答、程序分析、编程）
- ✅ **6 个实验**（银行账户、几何图形、工资计算、智能设备、交通工具、员工管理）

以及后续 **6 章** 的精彩内容：类模板、异常处理、移动语义、内存管理、标准库、元编程……每章都配有实战案例。

---

## 📖 立即购买《现代C++编程实战》电子版

- 一次性解锁全部 16 章 + 8 个完整实战项目
- 支持作者持续创作，获取最新更新（C++26 特性）
- 可在线阅读、离线下载（PDF/EPUB）

👉 [点击前往 Leanpub 购买](https://leanpub.com/c01)

**读者评价**：
> “这本书直接讲现代 C++，没有 C 语言的旧包袱，示例简洁易懂，适合自学。” —— 某位已购读者

---

**继续阅读**：点击侧边栏第11章预览，学习类模板（泛型编程的进阶）。  
**购买后**：你将收到完整版 PDF，并可以随时在线阅读全书。
