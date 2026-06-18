---
trigger: always_on
description: > 本文档从 C++ 中文周刊（第1期~第196期）中提炼的编码规范、设计模式、性能优化和避坑指南。
---

# C++ 开发技能指南

> 本文档从 C++ 中文周刊（第1期~第196期）中提炼的编码规范、设计模式、性能优化和避坑指南。
> 适用于日常 C++ 开发参考。

---

## 一、现代 C++ 编码规范

### 1.1 优先使用 range-based for 和 Ranges 库

传统 for 循环"过于灵活"，容易引入 off-by-one、修改错误变量等 bug。编译器无法防止这些问题。

```cpp
// ❌ 经典错误
for (auto i = 0; i <= vec.size(); ++i)   // 应该是 <
  use(vec[i]);

for (auto i = vec.size() - 1; i >= 0; --i)  // 无符号数永远 >= 0，死循环！
  use(vec[i]);

// ✅ 现代写法
for (auto const& rec : records)
  use(rec);

// 反向迭代（C++20）
for (auto const& rec : std::views::reverse(records))
  use(rec);

// 带索引迭代（C++23）
for (auto [i, rec] : std::views::enumerate(records))
  use(i, rec);

// 多序列同时迭代（C++23）
for (auto [name, rec] : std::views::zip(names, records))
  use(name, rec);
```

*来源：第190期*

### 1.2 用 `std::source_location` 替代 `__FILE__` / `__LINE__` 宏

```cpp
// ❌ 传统宏方法
#define ASSERT(cond, msg) Assert(cond, msg, __FUNCTION__, __LINE__)

// ✅ C++20
void Assert(bool condition, std::string_view msg,
            std::source_location loc = std::source_location::current()) {
  if (!condition) {
    std::clog << loc.function_name() << ':' << loc.line() << ": " << msg << '\n';
  }
}
// 调用时不需要宏：
Assert(1 != 2, "Not met");
```

关键：`std::source_location::current()` 作为默认参数，在调用侧求值。

*来源：第190期*

### 1.3 `constexpr` + `consteval` 双路径设计

```cpp
consteval size_t strlen_ct(const char* s) {  // 纯编译期
    size_t n = 0;
    for (; s[n] != '\0'; ++n);
    return n;
}

size_t strlen(const char* s);  // 纯运行期

constexpr size_t strlen_dual(const char* s) {  // 双路径
    if consteval {
        return strlen_ct(s);  // 编译期路径
    } else {
        return strlen(s);     // 运行期路径
    }
}
```

`constexpr` 函数最好两种分支都实现，避免意外问题。

*来源：第150期*

### 1.4 用 `std::expected` 替代异常做错误处理（C++23）

```cpp
std::expected<int, std::string> convertToInt(const std::string& input) {
    int value{};
    auto [ptr, ec] = std::from_chars(input.data(), input.data() + input.size(), value);
    if (ec == std::errc())
        return value;
    if (ec == std::errc::invalid_argument)
        return std::unexpected("Invalid number format");
    if (ec == std::errc::result_out_of_range)
        return std::unexpected("Number out of range");
    return std::unexpected("Unknown conversion error");
}
```

*来源：第150期*

### 1.5 Concepts 和 `requires` 的正确用法

```cpp
// 基本 requires
template <typename T>
    requires std::integral<T>
auto debug_output(const T& t);

// requires requires（检测成员函数）
template <typename T>
    requires requires(const T& t) { t.debug_output(); }
auto debug_output(const T& t);

// if constexpr + requires（编译期检测能力）
template <typename Cont, typename Rng>
void cont_assign(Cont& cont, Rng&& rng) {
    cont.clear();
    if constexpr (requires { cont.reserve(std::ranges::size(rng)); }) {
        cont.reserve(std::ranges::size(rng));
    }
    for (auto&& elem : std::forward<Rng>(rng)) {
        cont.push_back(std::forward<decltype(elem)>(elem));
    }
}
```

*来源：第170期*

### 1.6 `deducing this` 消除成员函数重复（C++23）

```cpp
// ❌ 传统写法：4个重载
struct Foo {
    void bar() &;
    void bar() &&;
    void bar() const &;
    void bar() const &&;
};

// ✅ C++23
template <typename T>
class Optional {
    template <typename Self>
    constexpr auto operator->(this Self&& self) {
        return addressof(self.m_value);
    }
};
```

*来源：第50期*

### 1.7 inline namespace 做版本控制

```cpp
namespace gem {
    inline namespace v1 {
        struct Point { int x; int y; };
    }
    namespace v2 {
        struct Point { int y; int x; };  // v2 改了布局
    }
}
// 默认用 v1，需要时显式 gem::v2::Point
```

*来源：第160期*

### 1.8 小对象直接传值，不要 const T&

`string_view`、`span`、`int`、`chrono::duration` 等小对象，直接传值比传引用更高效。

*来源：第50期*

---

## 二、性能优化指南

### 2.1 火焰图驱动优化

RocksDB 优化案例（180s → 7.8s，23倍加速），每步用火焰图验证：

1. **Transaction Put → SST Writer**（180s → 19.5s）：消除锁和排序开销
2. **关掉导入阶段不需要的过滤器和压缩**（19.5s → 14.3s）
3. **fast_float 替换 sscanf**（14.3s → 12s，16%提速）
4. **std::string → vector\<char\>**（12s → 10.6s）：消除 null terminator 维护开销
5. **去掉热路径的运行期检查**（10.6s → 8.7s）
6. **消除 key 的隐藏拷贝**（8.7s → 7.8s）

**Key takeaways：**
- 避免热路径中的虚函数
- 别不必要地拷贝字符串
- 运行期检查能改 assert 就改 assert

*来源：第196期*

### 2.2 编译器比你聪明，不要手动"优化"

编译器将代码转换为 IR，相同操作的不同实现会被转换为规范形式。多种计算加法的方式（循环、递归、复杂逻辑）都会编译为单个 ARM 指令 `add w0, w1, w0`。

常量乘法也一样：手动优化 `522` 为 `(x << 9) + (x << 3) + (x << 1)`，编译器仍然会恢复为 `imul`。

**优先考虑代码清晰性而不牺牲性能。**

*来源：第190期*

### 2.3 `__builtin_unreachable()` 消除分支

```cpp
uint8_t sum_with_constraints(const uint8_t *data, size_t len) {
    if (len % 32 != 0) __builtin_unreachable();  // len 一定是32的倍数
    if (len == 0) __builtin_unreachable();         // len 一定非零
    return std::accumulate(data, data + len, uint8_t(0));
}
```

注意：把 `data*` 换成 `vector`，gcc 下可能不能优化。

*来源：第180期*

### 2.4 TLS 性能优化清单

thread_local 对象在有类构造函数 + `-fPIC` 共享库时需要额外调用 `__tls_get_addr`，成为性能瓶颈。

优化指南：
- TLS 对象尽可能合并
- 不要为 TLS 写构造函数（用 trivial 类型）
- 频繁访问的对象用 `__attribute__((visibility("hidden")))`
- 关键变量用 `__attribute__((tls_model("initial-exec")))`
- 非共享库不要用 `-fPIC`
- 考虑 `-mtls-dialect=gnu2`

*来源：第180期*

### 2.5 `-O3 -flto` 是免费午餐

Redis 测试：`-O3 -flto` 性能至少提升 5%。PGO（Profile-Guided Optimization）值得进一步研究。

*来源：第100期*

### 2.6 低延迟编程：减少分支和跳转

- 勤用 `&&` `||` 利用短路特性
- 关注能生成 `cmov` 的写法（三元表达式、简单 if 赋值）
- 减少虚函数使用（但 `variant` + `visit` 某些场景比虚函数好）
- 善用 `[[gnu::always_inline]]` / `__builtin_expect`
- 字符串比较的 if-else 链改 switch

*来源：第150期*

### 2.7 查表法替代除法/取模

除法指令很慢。整数转字符串（itoa）用查表法替代循环除 10：

```cpp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wanghenshui/cppweeklynews](https://github.com/wanghenshui/cppweeklynews) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
