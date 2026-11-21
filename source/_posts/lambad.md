---
title: cpp中lambda的用法
date: 2026-11-21 20:16:01
tags:
  - csp
categories:
  - 语法
---
# C++ Lambda 函数用法大全

## 1. 基本语法

```cpp
// [捕获列表](参数列表) 说明符 -> 返回类型 { 函数体 }
auto add = [](int a, int b) -> int { return a + b; };
std::cout << add(3, 4);  // 7
```

## 2. 捕获列表（Capture List）

```cpp
int x = 10, y = 20;

// []       不捕获任何外部变量
auto f1 = []() { return 1; };

// [=]      按值捕获所有外部变量
auto f2 = [=]() { return x + y; };

// [&]      按引用捕获所有外部变量
auto f3 = [&]() { x++; y++; };

// [x]      按值捕获指定变量
auto f4 = [x]() { return x; };

// [&x]     按引用捕获指定变量
auto f5 = [&x]() { x = 100; };

// [x, &y]  混合捕获
auto f6 = [x, &y]() { y += x; };

// [=, &y]  默认按值，y 按引用
auto f7 = [=, &y]() { y = x; };

// [&, x]   默认按引用，x 按值
auto f8 = [&, x]() { y = x; };

// [this]   捕获当前对象指针（成员函数中）
// [*this]  按值捕获当前对象副本（C++17）
```

## 3. mutable 关键字

```cpp
int n = 0;
// 按值捕获的变量在 lambda 内部默认是 const 的
auto f1 = [n]() { /* n = 1; 错误 */ };

// 使用 mutable 才能修改副本
auto f2 = [n]() mutable { n = 1; return n; };
f2();     // 返回 1
// 外部 n 仍为 0，因为是副本
```

## 4. 返回类型推导与显式指定

```cpp
// 单条 return 语句：自动推导
auto f1 = [](int x) { return x * 2.0; };  // 返回 double

// 多条语句需显式指定返回类型
auto f2 = [](int x) -> int {
    if (x > 0) return x;
    return -x;
};

// 显式指定 void
auto f3 = []() -> void { std::cout << "hi\n"; };
```

## 5. 初始化捕获（C++14）

```cpp
// 在捕获列表中创建新变量
int x = 5;
auto f1 = [y = x * 2]() { return y; };  // y = 10

// 捕获移动对象（避免拷贝）
auto ptr = std::make_unique<int>(42);
auto f2 = [p = std::move(ptr)]() { return *p; };

// 以引用方式初始化捕获
auto f3 = [&r = x]() { r = 100; };  // 修改 x
```

## 6. 泛型 Lambda（C++14 起）

```cpp
// auto 作为参数类型
auto print = [](auto const& v) { std::cout << v << '\n'; };
print(1);
print("hello");
print(3.14);

// 多个泛型参数
auto add = [](auto a, auto b) { return a + b; };
```

## 7. 模板 Lambda（C++20）

```cpp
// 显式模板参数（可用于约束、完美转发等）
auto f = []<typename T>(T a, T b) { return a + b; };

// 结合 concepts 使用
auto g = []<std::integral T>(T a, T b) { return a + b; };
```

## 8. constexpr Lambda（C++17）

```cpp
// 若 lambda 体满足 constexpr 条件，则自动成为 constexpr
constexpr auto square = [](int x) { return x * x; };
static_assert(square(5) == 25);
```

## 9. 作为函数参数（回调）

```cpp
// 标准库函数
template <typename Fn>
void call_it(Fn fn) { fn(); }

call_it([]() { std::cout << "called\n"; });

// std::function 包装（有类型擦除开销）
std::function<int(int)> f = [](int x) { return x + 1; };

// 作为函数指针（无捕获时）
void (*fp)(int) = [](int x) { std::cout << x; };
```

## 10. 在 STL 算法中的常见用法

```cpp
std::vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6};

// 排序
std::sort(v.begin(), v.end(), [](int a, int b) { return a > b; });

// for_each
std::for_each(v.begin(), v.end(), [](int x) { std::cout << x << ' '; });

// find_if
auto it = std::find_if(v.begin(), v.end(), [](int x) { return x > 4; });

// transform
std::transform(v.begin(), v.end(), v.begin(), [](int x) { return x * x; });

// count_if
int cnt = std::count_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; });

// accumulate 配合 lambda
int sum = std::accumulate(v.begin(), v.end(), 0,
                          [](int acc, int x) { return acc + x; });

// 按引用修改外部变量
int total = 0;
std::for_each(v.begin(), v.end(), [&total](int x) { total += x; });
```

## 11. IIFE（立即调用的 Lambda）

```cpp
// 就地执行，用于初始化复杂变量
const int value = []() {
    int tmp = compute();
    return tmp * 2;
}();
```

## 12. 可调用对象的转换 / 传参技巧

```cpp
// 传给 C 风格 API（无捕获的 lambda 可转函数指针）
c_function([](int x) { return x; });

// 使用 std::function 存储到容器
std::vector<std::function<void()>> tasks;
tasks.push_back([]() { std::cout << "A\n"; });
tasks.push_back([&]() { std::cout << "B\n"; });

// 互相递归：借助 std::function
std::function<int(int)> fact = [&fact](int n) {
    return n <= 1 ? 1 : n * fact(n - 1);
};
```

## 13. 常见坑与注意事项

```cpp
// 悬空引用：lambda 生命周期超过被捕获变量
auto make_lambda() {
    int local = 10;
    return [&local]() { return local; };  // 危险！local 已销毁
}

// 按值捕获的 this 相关（C++17 的 *this）
struct S {
    int v = 1;
    auto f() {
        return [*this]() { return v; };   // 安全捕获对象副本
    }
};

// mutable 修改的是副本，不影响外部
int a = 1;
auto f = [a]() mutable { a = 100; };
f();
// 外部 a 仍为 1
```

## 14. 各版本特性速查

| 特性 | 版本 |
|------|------|
| 基本 lambda、捕获列表、`mutable` | C++11 |
| 泛型 lambda（`auto` 参数）、初始化捕获 | C++14 |
| `constexpr` lambda、`[*this]` 捕获 | C++17 |
| 模板 lambda、无捕获 lambda 在未求值上下文使用 | C++20 |
