---
slug: writing-cpp
categories: [languages]
priority: 1
description: C++17/20/23 style — RAII, smart pointers, move semantics, const-correctness, error handling, concepts, undefined-behavior footguns, testing.
applies_when:
  - writing C++ code
  - reviewing C++ code
  - designing C++ APIs
  - handling errors or resources in C++
related: [coding-style, writing-tests]
source: https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines
---

# C++

Target C++20 as the baseline (C++17 minimum); reach for C++23 (`std::expected`,
`std::print`, `ranges::to`, deducing-this) where the toolchain supports it. The
canonical reference is the C++ Core Guidelines (rule IDs like `C.35`, `F.16`,
`R.21`, `ES.20` are cited inline below). C++ has no runtime safety net -- correct
code depends on discipline plus tooling. Build CI with warnings-as-errors and
sanitizers; they catch most of the footguns in this guide mechanically.

## RAII & Resource Management

RAII (Resource Acquisition Is Initialization) is *the* foundational C++ idiom
(`R.1`): tie every resource -- memory, file, lock, socket -- to an object whose
constructor acquires and destructor releases. Cleanup then happens on every exit
path, including exceptions, for free. Smart pointers, `std::lock_guard`,
`std::fstream`, and the containers are all RAII.

### Rule of Zero (the default)

Design classes so the compiler-generated destructor, copy, and move are correct
(`C.20`). Delegate ownership to members that already manage themselves; then the
class needs *zero* user-declared special members.

```cpp
// CORRECT (Rule of Zero): every special member is correct and implicit
class Widget {
    std::string name_;
    std::vector<int> data_;
    std::unique_ptr<Impl> impl_;
};
```

### Rule of Five (when you must manage a resource directly)

If you declare or `=delete` any of {destructor, copy ctor, copy assign, move
ctor, move assign}, declare or `=delete` them all (`C.21`) -- they are
interrelated. A user destructor that frees, plus a compiler-default copy that
shallow-copies the pointer, is a double-free.

```cpp
// WRONG: user dtor frees, default copy shallow-copies the pointer -> double free
struct Buf { int* p; ~Buf() { delete[] p; } };

// CORRECT: prefer a self-managing member (Rule of Zero) and write none of them
struct Buf { std::vector<int> data; };
```

Key interactions (why Rule of Five exists): a user-declared destructor, copy
ctor, or copy assign *suppresses* the implicit move operations (you silently fall
back to copies); a user-declared move op *deletes* the copy operations.

Rules:
- A polymorphic base's destructor must be **public and virtual, or protected and
  non-virtual** (`C.35`). `delete base_ptr;` through a non-virtual destructor is
  undefined behavior -- the derived destructor never runs.
- Make move operations `noexcept` (`C.66`). `std::vector` reallocation only
  *moves* your elements if the move ctor is `noexcept`; otherwise it *copies* to
  preserve the strong exception guarantee. A throwing move is a silent
  pessimization.
- Destructors are implicitly `noexcept`; never let one throw -- a throw during
  stack unwinding calls `std::terminate` (`C.37`).
- Never call virtual functions from a constructor or destructor (`C.82`): the
  dynamic type is the class under construction, so the call does *not* dispatch
  to the derived override. Do post-construction work in a factory function.

## Ownership & Smart Pointers

Express ownership with smart pointers (`R.20`); use raw pointers and references
only for non-owning access.

- `std::unique_ptr<T>` -- exclusive ownership, move-only, zero overhead (size of a
  raw pointer). **The default owning pointer** (`R.21`).
- `std::shared_ptr<T>` -- shared ownership via an atomic reference count. Use only
  when ownership is genuinely shared -- it is not free.
- `std::weak_ptr<T>` -- non-owning observer of a `shared_ptr`; `.lock()` yields a
  `shared_ptr` if still alive. Use it to **break reference cycles** (`R.24`) --
  two objects holding `shared_ptr`s to each other never reach refcount zero.
- Raw `T*` / `T&` -- **non-owning** (`R.3`, `R.4`). Still the correct tool for
  observation and required (`T&`) vs optional (`T*`) arguments.

```cpp
// WRONG: raw owning new/delete -- leaks on any early return or throw
X* q = new X{};  /* ... */  delete q;
std::shared_ptr<X> p(new X(a));   // 2 allocations; leak if another arg throws

// CORRECT
auto u = std::make_unique<X>();   // R.23: exception-safe, no raw new
auto p = std::make_shared<X>(a);  // R.22: single allocation for object + control block
```

Prefer stack/scoped objects over heap allocation (`R.5`); avoid explicit `new`
/`delete` (`R.11`) and `malloc`/`free` (`R.10`).

### Passing smart pointers

Only take a smart pointer as a parameter to express a **lifetime contract**. A
function that merely *uses* a widget should take `const T&` or `T*` (`F.7`), not
constrain every caller's ownership model.

```cpp
// WRONG: forces every caller into shared_ptr just to read the widget
void render(std::shared_ptr<Widget> w);
// CORRECT
void render(const Widget& w);   // required, non-owning
```

- `unique_ptr<T>` by value -> function **takes (sinks) ownership** (`R.32`, `F.26`).
- `shared_ptr<T>` by value -> function **becomes a shared owner** (`R.34`, `F.27`).
- `const shared_ptr<T>&` -> function **might retain** a reference count (`R.36`).

## Value & Move Semantics

Value semantics is the default: objects are copied or moved, not implicitly
shared. Prefer returning by value and let move + copy elision make it cheap.

`std::move` is a **cast, not a move** -- it unconditionally casts an lvalue to an
rvalue, *enabling* a move that the move constructor performs. `std::forward`
casts *conditionally*, preserving the original value category. Write `std::move`
only where you deliberately hand an object to another scope (`ES.56`); use
`std::forward` only on a forwarding reference in a perfect-forwarding wrapper.

```cpp
template <class... Args>
auto emplace(Args&&... args) {
    return T(std::forward<Args>(args)...);  // preserves each arg's value category
}
```

Common mistakes:
- **`return std::move(local)`** disables copy elision / NRVO -- a pessimization,
  and can dangle (`F.48`). A bare `return local;` already moves (or elides).

  ```cpp
  // WRONG: blocks NRVO
  std::vector<int> make() { std::vector<int> r; /* ... */ return std::move(r); }
  // CORRECT
  std::vector<int> make() { std::vector<int> r; /* ... */ return r; }
  ```
- **Moving a `const` object silently copies** -- `T&&` can't bind to `const T`, so
  overload resolution falls back to the copy ctor. Don't make things you intend
  to move `const`.
- **Use-after-move**: a moved-from object is *valid but unspecified*. You may
  reassign or destroy it, but must not read its value or call operations with
  preconditions. Assume moved-from containers are not empty.
- Don't assume moves are cheap or even present (`std::array` moves are O(n); a
  non-`noexcept` move gets copied by containers).

## Parameter Passing

Prefer simple, conventional passing (`F.15`). For **in** parameters, pass
cheap-to-copy types by value and everything else by `const&` (`F.16`): pass by
value if the type is trivially copyable and `sizeof(T) <= 2 * sizeof(void*)`
(e.g. `int`, `std::string_view`, `std::span`); otherwise `const T&`.

| Role | Signature |
|---|---|
| in (small/trivial) | `f(X)` |
| in (expensive to copy) | `f(const X&)` |
| in/out | `f(X&)` |
| out | `X f()` (return by value; `F.20`) |
| out (multiple) | `struct`/`tuple` return + structured bindings (`F.21`) |
| sink / consume | `f(X)` then `std::move`, or `f(X&&)` |
| forward | `template f(X&&)` + `std::forward` (`F.19`) |

Use a **reference** when the argument is required and never null; a **raw
pointer** when it is optional (nullable) or must be rebound. Neither implies
ownership.

For **sink** parameters absorbed into a member, prefer *pass-by-value-and-move*
as the default -- one function handles lvalues and rvalues, near-optimal:

```cpp
void set_name(std::string n) { name_ = std::move(n); }  // rvalue: 2 moves; lvalue: copy + move
```

Use two overloads (`const std::string&` + `std::string&&`) only when profiling
shows the extra move matters; use `std::string&&` for move-only sinks.

## const-correctness & constexpr

Make objects `const` by default and only mutable what must change (`Con.1`,
`ES.25`); make non-mutating member functions `const` (`Con.2`); pass pointers and
references to `const` (`Con.3`). Use `constexpr` for values and functions
computable at compile time (`Con.5`, `F.4`).

| Keyword | Applies to | Meaning |
|---|---|---|
| `const` | objects, member fns | Runtime immutability. Not necessarily a constant expression. |
| `constexpr` | objects & functions | *May* run at compile time; runs at runtime otherwise. A `constexpr` object is also `const`. |
| `consteval` (C++20) | functions | *Immediate*: every call **must** be evaluated at compile time. |
| `constinit` (C++20) | static/thread-local | Forces constant initialization (fixes the init-order fiasco). Does **not** imply `const`. |

```cpp
constexpr double area(double r) { return pi * r * r; }  // compile-time or runtime
consteval int sq(int n) { return n * n; }               // compile-time only
constinit int counter = compute();                       // constant-initialized, still mutable
```

## Error Handling

Pick a strategy early (`E.1`) and reserve exceptions for error handling, never
normal control flow (`E.3`).

| Mechanism | Use for |
|---|---|
| **Exceptions** | Truly exceptional / unrecoverable-at-call-site errors; constructor failures (no return channel); errors that must not be silently ignored. |
| **`std::expected<T,E>`** (C++23) | Expected, recoverable failures in normal flow / hot paths; codebases where exceptions are banned. |
| **`std::optional<T>`** (C++17) | Failure with no error detail needed ("not found", "absent"). |
| **`std::error_code`** | System / OS / library / ABI boundaries; non-throwing, categorized. |
| **`assert` / contracts** (C++26) | *Bugs* (broken preconditions/invariants), never expected runtime failures. |

A precondition violation is a **caller bug** -> assert/contract, not an
exception. A runtime disappointment (bad input, I/O failure) -> exception or
`expected`. Mark fallible functions `[[nodiscard]]` so ignored results warn.

```cpp
// C++23 expected with monadic chaining -- short-circuits on the error branch
std::expected<Png, Err> r = load(img)
    .and_then(find_cat)            // T -> expected<U,E>
    .transform(add_bow_tie)        // T -> U  (value branch only)
    .transform_error(to_png_err);  // E -> G  (error branch only)
```

### Exception safety

Provide, from strongest to weakest: **nothrow** (destructors, `swap`,
deallocation, move ops -- mark `noexcept`), **strong** (commit-or-rollback, via
copy-and-swap), **basic** (invariants preserved, no leaks). Basic is the minimum;
RAII gives it almost for free.

```cpp
Widget& operator=(Widget rhs) {   // copy (may throw) happens before any mutation
    swap(*this, rhs);             // noexcept commit -> strong guarantee, self-assign safe
    return *this;
}
```

Rules:
- Mark functions `noexcept` when they cannot or must not throw (`E.12`);
  destructors, `swap`, and move ops especially (`C.66`, `C.84`). A throw out of a
  `noexcept` function calls `std::terminate` -- it is a hard promise.
- Throw by value, catch by reference (`E.15`); order catch clauses most-derived to
  most-base (`E.31`); derive exception types from `std::exception` (`E.14`).
- Never throw while directly owning a raw resource -- wrap it in RAII first
  (`E.13`). Prefer RAII to `try`/`catch` (`E.18`); don't catch what you can't
  handle (`E.17`).
- For a C API with no RAII handle, use a scope guard: `auto g = gsl::finally([&]{
  fclose(fp); });`.

## Type Design

### Make invalid states unrepresentable

```cpp
// WRONG: fields can contradict each other
struct Order { std::string status; std::optional<Instant> shipped_at; std::string tracking; };

// CORRECT: each state carries exactly its own data
struct Pending { std::vector<Item> items; };
struct Shipped { std::vector<Item> items; Instant shipped_at; std::string tracking; };
using Order = std::variant<Pending, Shipped>;

std::visit([](auto&& state) { handle(state); }, order);  // exhaustive, compile-checked
```

### enum class over bool and plain enum

Scoped enums have no implicit `int` conversion and no scope leak (`Enum.3`).

```cpp
// WRONG: what does true mean?
void send(std::string_view msg, bool urgent);
// CORRECT
enum class Priority { Normal, Urgent };
void send(std::string_view msg, Priority priority);

enum class Color : std::uint8_t { Red, Green, Blue };  // explicit underlying type
```

Rules:
- Always mark intended overrides `override` -- a signature mismatch otherwise
  creates a *new* function and silently breaks polymorphism. Use `final` to
  forbid further overriding and enable devirtualization.
- Wrap primitives in strong types (a `struct UserId { std::uint64_t v; };`) so the
  compiler rejects `get_order(order_id, user_id)` argument swaps.
- Prevent object slicing: give polymorphic bases a protected/deleted copy, or make
  them abstract; pass and store by reference or (smart) pointer, never by base
  value.

## Standard Library Idioms

- **Prefer algorithms and ranges to raw loops** (`ES.1`) -- they express intent and
  optimize well. C++20 `std::ranges` algorithms take a whole range and compose
  lazily; C++23 adds `ranges::to<Container>()`, `views::enumerate`, `views::zip`.

  ```cpp
  auto evens_sq = v | std::views::filter([](int x) { return x % 2 == 0; })
                    | std::views::transform([](int x) { return x * x; });
  ```
- **`std::string_view`** (C++17) for read-only string params -- take **by value**;
  it accepts `std::string`, literals, and `char*` with zero copy. It is
  non-owning: never store it or return one into a temporary (see Footguns).
- **`std::span<T>`** (C++20) replaces `(T* ptr, size_t len)` parameter pairs with a
  single bounds-carrying view; `span<const T>` for read-only.
- **`std::optional` / `std::variant`** for maybe-a-value and closed sum types;
  prefer `variant` + `std::visit` over `std::any`.
- **`std::array` / `std::vector` over C arrays** (`SL.con.1`) -- C arrays decay to
  pointers and lose their size. `std::vector` is the **default** container
  (`SL.con.2`); `reserve(n)` before bulk `push_back`. Prefer `unordered_map` for
  associative lookup, `map` only when you need ordering. Avoid `std::list`.
- **`std::format`** (C++20) / **`std::println`** (C++23) over iostreams and
  `printf` -- type-safe and compile-time-checked. Avoid `std::endl` (it flushes);
  use `'\n'`.

## auto, Structured Bindings & Range-for

Use `auto` to avoid redundant type names and accidental conversions (`ES.11`), but
force the type when `auto` would latch onto a proxy (`std::vector<bool>::reference`).
Always initialize objects (`ES.20`); prefer `{}` init, which rejects narrowing --
but beware its `initializer_list` preference (see Footguns).

```cpp
auto it = m.begin();                     // avoid spelling the iterator type
int  x{7};                               // {} rejects narrowing: int x{7.9}; is ill-formed

for (const auto& s : strings) use(s);    // read-only, no copy
for (auto& s : strings) s.clear();       // mutate in place
for (const auto& [key, val] : my_map) {  // structured bindings, no copy
    use(key, val);
}
if (auto [it, inserted] = m.try_emplace(k, v); inserted) { /* ... */ }
```

## Templates & Generics

- **Concepts (C++20) replace SFINAE.** Constrain template parameters so intent is
  visible and errors fire at the call site, not deep in instantiation (`T.10`).
  Prefer standard concepts (`std::integral`, `std::ranges::range`) over hand-rolled
  ones (`T.11`).

  ```cpp
  template <std::integral T> void f(T x);        // constrained
  void g(std::ranges::range auto&& r);           // abbreviated
  ```
- **`if constexpr` (C++17)** for compile-time branching -- the discarded branch is
  not instantiated, collapsing tag-dispatch / `enable_if` pairs into one body.
- **Static vs dynamic polymorphism**: templates dispatch at compile time
  (inlinable, but cause code bloat and slower builds) and suit homogeneous
  compile-time-known types; `virtual` suits heterogeneous collections and stable
  ABIs. CRTP gives static polymorphism without a vtable; C++23 **deducing this**
  (`void f(this auto&& self)`) removes the CRTP boilerplate.
- Don't over-genericize (`T.61`); prefer a non-template core for ABI stability and
  less bloat (`T.84`); `extern template` suppresses redundant instantiation.

## Naming & Organization

There is **no single universal C++ naming standard** (`NL.10`) -- adopt one house
style (Google, LLVM, or "follow the standard library" snake_case) and apply it
consistently; leave imported libraries in their own style. Reserve `ALL_CAPS` for
macros only (`NL.9`); no Hungarian / type-encoding (`NL.5`); digit separators for
large literals (`1'000'000`).

- Split interface (`.h`/`.hpp`) from implementation (`.cpp`) (`SF.1`). Every header
  must be **self-contained** -- compile alone, include what it needs, carry an
  include guard (`SF.11`).
- Put no non-inline function or object definitions in headers (ODR) (`SF.2`).
- **Include what you use** (`SF.10`): include or forward-declare every symbol you
  name; never rely on transitive includes. The module's own header goes first
  (surfaces missing includes) (`SF.4`).
- `#pragma once` (concise, supported by all major compilers) or `#ifndef` guards
  (100% standard, symlink-safe) are both acceptable (`SF.8`) -- pick one.
- **Never `using namespace` (including `std`) at file or namespace scope in a
  header** (`SF.7`) -- it pollutes every includer. A narrow `using std::string;`
  inside a function body is fine (`SF.6`).

## Concurrency

- A **data race** (two threads access one location, one writes, unsynchronized and
  not both atomic) is undefined behavior. Guard shared mutable state with a
  `std::mutex` (`std::scoped_lock` / `std::lock_guard`) or make it `std::atomic`.
- **`volatile` is not for threading** -- it gives no atomicity, ordering, or
  synchronization (it is for memory-mapped I/O). Use `std::atomic<bool>`, never
  `volatile bool`, for a cross-thread flag.
- A `std::thread` still `joinable()` at destruction calls `std::terminate`. Prefer
  **`std::jthread`** (C++20), which auto-joins and carries a `stop_token`.
- Don't capture locals **by reference** in a detached/async lambda that may outlive
  them -- capture by value or move ownership in.
- The `std::future` from `std::async(std::launch::async, ...)` **blocks in its
  destructor**; a temporary future runs synchronously. Bind it to a named variable.

## Undefined Behavior & Footguns

UB means the compiler may assume it never happens and optimize accordingly -- code
"works" until `-O2` or under load. Build test/CI with sanitizers
(`-fsanitize=address,undefined`; `-fsanitize=thread` separately) to catch most of
these mechanically.

- **Dangling references / views.** Returning a reference or pointer to a local
  dangles. `std::string_view` and `std::span` are non-owning and do **not** extend
  a temporary's lifetime -- treat them as *parameter-only* types; never store one
  as a member or return one built from local storage.

  ```cpp
  std::string_view v = std::string("hi");  // WRONG: temporary destroyed; v dangles
  ```
- **Range-for over a temporary subobject** iterates freed memory before C++23:
  `for (char c : get_config().name())` where `name()` returns a reference into the
  temporary `Config`. Bind the temporary to a named variable first (fixed in C++23).
- **Iterator/reference invalidation.** `push_back` past `capacity()` reallocates and
  invalidates *all* iterators, pointers, and references. `erase(it)` invalidates
  `it`; capture its return value. Use `std::erase_if(v, pred)` (C++20) instead of
  the error-prone erase-remove idiom.
- **Object slicing.** Copying a `Derived` into a `Base` by value drops the derived
  part and resets the vtable. Pass/store by reference or pointer.
- **Integer pitfalls.** Signed overflow is UB (not wraparound). Mixed signed/
  unsigned comparison converts the signed operand to unsigned -- `int i = -1;
  unsigned n = 5; i < n` is `false`; use `std::cmp_less(i, n)` (C++20). `size()` is
  unsigned, so `v.size() - 1` on an empty vector is `SIZE_MAX`. Brace-init rejects
  narrowing; `()`/`=` init silently truncates.
- **Most vexing parse.** `Widget w();` declares a *function*, not an object. Use
  `Widget w;` or `Widget w{};`.
- **`{}` and `initializer_list` hijacking.** `std::vector<int> v{5};` is a
  one-element vector `{5}`, not five zeros -- braces strongly prefer an
  `initializer_list` ctor. Use `std::vector<int> v(5);` for a size.
- **Static initialization order fiasco.** The relative init order of namespace-scope
  objects across translation units is unspecified. Use construct-on-first-use: wrap
  the global in a function with a local `static`.
- **Strict aliasing.** Reinterpreting an object through an unrelated pointer type is
  UB; use `std::bit_cast` (C++20) or `std::memcpy`, not a `reinterpret_cast`.

## Testing

Use **GoogleTest** (pairs with gMock for mocking; has death tests and rich
matchers) or **Catch2** (minimal boilerplate, natural `REQUIRE(a == b)`
assertions, `SECTION`-based setup, first-class BDD). The need for mocking usually
decides. Pull the framework via CMake `FetchContent` (pin a tag) and register
tests with `gtest_discover_tests`.

```cpp
TEST_F(QueueTest, PopReturnsFront) {
    ASSERT_FALSE(q_.empty());    // fatal guard: continuing would crash
    EXPECT_EQ(q_.front(), 1);    // non-fatal: records and continues
}
```

Rules:
- Prefer `EXPECT_*` (non-fatal); use `ASSERT_*` only when continuing would crash.
- Compare C-strings with `EXPECT_STREQ` (`EXPECT_EQ` on `char*` compares
  pointers); floats with `EXPECT_NEAR` / `EXPECT_DOUBLE_EQ`, never `EXPECT_EQ`.
- Use `EXPECT_DEATH(stmt, regex)` for `assert` / `abort` / contract violations.
- Parametrize with `TEST_P` + `INSTANTIATE_TEST_SUITE_P` (gtest) or `GENERATE` /
  `TEMPLATE_TEST_CASE` (Catch2) instead of duplicating tests.
- Arrange-Act-Assert; one behavior per test; no shared mutable global state so
  tests pass in any order. Run the suite under sanitizers in CI.

## Build & Tooling

- **CMake** is the de-facto standard. Model everything as **targets** with
  `target_*` commands and explicit `PRIVATE` / `PUBLIC` / `INTERFACE` visibility;
  never use directory-wide `include_directories` / `link_libraries`. Declare the
  standard as a usage requirement: `target_compile_features(tgt PUBLIC
  cxx_std_20)`. Avoid `file(GLOB)`. Set `CMAKE_EXPORT_COMPILE_COMMANDS ON` for the
  analyzers; share settings via `CMakePresets.json`.
- **Warnings**: `-Wall -Wextra -Wpedantic` baseline plus `-Wshadow -Wconversion
  -Wsign-conversion -Wnon-virtual-dtor`; MSVC `/W4 /permissive-`. Add `-Werror` in
  CI only.
- **Sanitizers** (compile *and* link): ASan `-fsanitize=address` (memory errors),
  UBSan `-fsanitize=undefined`, TSan `-fsanitize=thread` (data races), MSan
  (uninitialized reads, Clang). ASan+UBSan combine; ASan/TSan/MSan need separate CI
  jobs.
- **Static analysis**: `clang-tidy` (`bugprone-*`, `cppcoreguidelines-*`,
  `modernize-*`, `performance-*`) fed `compile_commands.json`; `cppcheck` as a
  complement. Enforce formatting with a committed `.clang-format` in CI check-mode.
- **Dependencies**: `FetchContent` for a few CMake-native libs; **vcpkg** (manifest
  mode) for broad needs with low ceremony; **Conan** for binary caching and
  enterprise control.
- **Modules (C++20)**: viable on MSVC and recent Clang 18+/GCC 15+, but the
  cross-platform build/tooling story is still maturing (CMake 3.28+, 3.30+ for
  experimental `import std`). For portable multi-compiler libraries, stay on
  headers through 2026; adopt modules only behind a pinned toolchain.

## Anti-Patterns to Avoid

- Raw owning `new`/`delete` and naked pointers with ownership -- use `unique_ptr` /
  `make_unique` and the Rule of Zero.
- `return std::move(local)` -- defeats copy elision.
- `std::move` on a `const` object -- silently copies.
- Non-virtual destructor on a polymorphic base -- UB on `delete` through the base.
- Passing `std::shared_ptr` where `const T&` or `T*` would do.
- `using namespace std;` in a header.
- C arrays, `std::endl`, `printf`/iostreams for new code -- use `std::array`/
  `std::vector`, `'\n'`, `std::format`/`std::println`.
- `volatile` for threading; raw `std::thread` you must remember to join.
- Plain `enum` and boolean parameters -- use `enum class`.
- Storing or returning a `std::string_view` / `std::span` into storage it doesn't
  own.
- Premature abstraction and template over-genericization -- add generality only
  with 2+ concrete uses.
