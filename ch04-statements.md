# 第4章 语句 · 预览版

> 本章你将学习如何让程序根据条件做出选择、重复执行代码块，以及改变正常执行流程。控制语句是程序设计的核心，掌握了它们，你就能写出真正“智能”的程序。  
> **完整版包含**：`if` / `switch` 条件分支、`while` / `do-while` / `for` / 范围 `for` 循环、`break` / `continue` / `goto` 跳转、C++17 的初始化语句、两个完整项目实战（猜数字游戏 + 控制台Pong游戏）、50+ 道习题和 8 个实验。  
> 📖 文末有购买链接，解锁全部 16 章 + 实战项目。

---

## 4.1 条件语句：让程序“思考”

### 4.1.1 `if` / `else if` / `else`：多路分支

最常用的条件语句，根据布尔表达式选择执行路径。

```cpp
int score;
std::cin >> score;
if (score >= 90)
    std::println("优秀");
else if (score >= 80)
    std::println("良好");
else if (score >= 60)
    std::println("及格");
else
    std::println("不及格");
```

**注意**：`else` 总是与最近的未匹配 `if` 配对，使用花括号 `{}` 可以避免歧义。

### 4.1.2 `switch`：多选一

适合处理多个固定整数值（如菜单选项、状态机）。

```cpp
int choice;
std::cin >> choice;
switch (choice) {
    case 1:
        std::println("查询余额");
        break;
    case 2:
        std::println("存款");
        break;
    case 3:
        std::println("取款");
        break;
    default:
        std::println("无效选项");
}
```

- 每个 `case` 后通常需要 `break`，否则会“贯穿”到下一个 `case`。
- 表达式必须是整型或枚举类型，不能是浮点数或字符串。

### 4.1.3 C++17 的 `if` 初始化

在 `if` 的条件前直接定义变量，变量作用域仅限于 `if` / `else` 块内，代码更简洁安全。

```cpp
if (auto it = s.find("hello"); it != std::string::npos) {
    std::println("找到了，位置：{}", it);
} else {
    std::println("未找到");
}
// 这里 it 已失效，不会污染外部作用域
```

> 完整版会详细讲解 `switch` 的注意事项、`case` 内定义变量的坑、以及如何利用贯穿特性。

---

## 4.2 循环语句：重复执行代码块

### 4.2.1 `while` 和 `do-while`

- `while`：先判断条件，可能一次都不执行。
- `do-while`：至少执行一次循环体。

```cpp
// while 计算 1 到 100 的和
int sum = 0, i = 1;
while (i <= 100) {
    sum += i;
    ++i;
}

// do-while 至少执行一次（例如输入验证）
int password;
do {
    std::print("请输入密码：");
    std::cin >> password;
} while (password != 12345);
```

### 4.2.2 `for` 循环

适合计数循环，三个表达式（初始化、条件、增量）清晰明了。

```cpp
// 输出 1 到 10
for (int i = 1; i <= 10; ++i) {
    std::println("{}", i);
}
```

三个表达式都可以省略，例如 `for (;;)` 是无限循环。

### 4.2.3 范围 `for`（C++11）

最简洁的遍历方式，用于数组或容器。

```cpp
int arr[] = {10, 20, 30, 40, 50};
for (int x : arr) {
    std::print("{} ", x);
}
// 输出：10 20 30 40 50
```

### 4.2.4 `break` 与 `continue`

- `break`：立即终止循环。
- `continue`：跳过本次循环的剩余语句，进入下一次迭代。

```cpp
// 输出 1~100 中所有不能被 3 整除的数
for (int i = 1; i <= 100; ++i) {
    if (i % 3 == 0) continue;
    std::print("{} ", i);
}
```

> 完整版会对比 `for` 与 `while` 的等价性，讲解 `do-while` 的适用场景，以及如何用 `break` 跳出多层循环（或用 `goto` 作为备选）。

---

## 4.3 项目实战：猜数字游戏

下面是一个完整的猜数字游戏，展示了 `while` 循环、`if` 分支、`break` 和输入验证的综合运用。

```cpp
#include <print>
#include <random>
#include <iostream>

int main() {
    // 现代 C++ 随机数生成（1~100）
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<int> dist(1, 100);
    int target = dist(gen);

    int guess = 0, attempts = 0;
    std::println("欢迎来到猜数字游戏！数字范围 1~100。");

    while (true) {
        std::print("请输入你的猜测：");
        std::cin >> guess;

        // 输入验证：检查是否为整数
        if (std::cin.fail()) {
            std::cin.clear();
            std::cin.ignore(10000, '\n');
            std::println("输入无效，请输入一个整数！");
            continue;
        }

        // 范围验证
        if (guess < 1 || guess > 100) {
            std::println("数字必须在 1~100 之间！");
            continue;
        }

        attempts++;
        if (guess < target)
            std::println("太小了！");
        else if (guess > target)
            std::println("太大了！");
        else {
            std::println("恭喜你猜对了！共猜了 {} 次。", attempts);
            break;
        }
    }
    return 0;
}
```

**运行示例**（假设目标数字为 42）：
```
欢迎来到猜数字游戏！数字范围 1~100。
请输入你的猜测：50
太大了！
请输入你的猜测：30
太小了！
请输入你的猜测：42
恭喜你猜对了！共猜了 3 次。
```

> 完整版第4章还包含另一个大型项目：**控制台Pong游戏**（双人乒乓球），涉及键盘输入、帧缓冲区、跨平台适配等高级技术，适合学有余力的读者挑战。

---

## ✨ 为什么要购买完整版？

你在本预览版中已经学到了：
- `if` / `else if` / `else` 多路分支
- `switch` 语句与 `break` 的重要性
- C++17 的 `if` 初始化特性
- `while`、`do-while`、`for`、范围 `for` 循环
- `break` 与 `continue` 控制循环流程
- 一个完整的猜数字游戏（含随机数生成和输入验证）

**完整版第4章还将提供**：
- ✅ `goto` 语句的慎用场景与替代方案
- ✅ 嵌套循环与跳出多层循环的技巧
- ✅ 循环性能优化（如提前计算循环不变量）
- ✅ **项目实战：控制台Pong游戏**（双人对战，含帧缓冲区优化、跨平台适配）
- ✅ **50+ 道习题**（选择、填空、简答、代码分析、编程）
- ✅ **8 个实验**（计算器、ATM模拟、猜拳、杨辉三角等客观编程题）

以及后续 **12 章** 的精彩内容：数组指针、函数、模板、类与对象、继承、多态、异常、移动语义、内存管理、标准库、元编程……每章都配有实战案例。

---

## 📖 立即购买《现代C++编程实战》电子版

- 一次性解锁全部 16 章 + 8 个完整实战项目
- 支持作者持续创作，获取最新更新（C++26 特性）
- 可在线阅读、离线下载（PDF/EPUB）

👉 [点击前往 Leanpub 购买](https://leanpub.com/c01)

**读者评价**：
> “这本书直接讲现代 C++，没有 C 语言的旧包袱，示例简洁易懂，适合自学。” —— 某位已购读者

---

**继续阅读**：点击侧边栏第5章预览，了解复合类型（数组、指针、引用）。  
**购买后**：你将收到完整版 PDF，并可以随时在线阅读全书。
