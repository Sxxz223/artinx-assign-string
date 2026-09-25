# Assignment 2 · 自定义 String 类
password


本仓库是本次作业的**起始框架**：文档、工程结构与自动测试已经就绪，
实现部分（`include/my_string.h` 的私有成员 + `src/my_string.cpp`）留给你来完成。

本次作业**只需要实现 `String` 类**，没有其他类、也没有附加部分。目标是用
`char` 数组手写一个简化版的 `std::string`，练习：

- 动态内存管理（`new[]` / `delete[]`）与 RAII；
- Rule of Five：拷贝/移动构造、拷贝/移动赋值、析构；
- 运算符重载：`+`、`[]` 与 `const char*` 转换（流运算符 `<<` / `>>` 为选做 bonus）；
- 深拷贝值语义，以及自赋值、自移动、自插入、自交换等边界情况；
- 强异常安全（分配失败时原对象不被破坏）与内存安全（ASan/UBSan 零报告）。

> 没学过异常或 `noexcept` 也不影响开始：本作业只用到很少一点。
> 建议先花 10 分钟浏览 [`docs/guide.md`](docs/guide.md)
> （先修概念 + 关键函数的实现骨架，含新手常见报错）。

## 里程碑：一步一步变绿

测试按难度拆成 5 个里程碑 + 1 个选做 bonus。**每个里程碑是独立的测试程序**，
只引用「本里程碑及更早里程碑」的函数——只实现了 M1 时，`m1_basics` 就能链接并通过，
它的链接错误（`undefined reference to ...`）就是 M1 的待实现清单。

| 里程碑 | 内容 | 测试程序 | 分级 |
| --- | --- | --- | --- |
| M1 | 封装与不变量：构造/析构、`size`/`capacity`、`[]`、`at`、`c_str`、`push_back` | `tests/m1_basics.cpp` | **必做** |
| M2 | 值语义：拷贝构造 / 复制赋值 / 自赋值 / `operator+` / `insert`（含越界异常） | `tests/m2_value_semantics.cpp` | **必做** |
| M3 | 移动语义：移动构造 / 移动赋值、`noexcept`、被移动后对象的有效性 | `tests/m3_move.cpp` | 进阶 |
| M4 | 边界与自操作：自插入、自交换、容量边界、空串自操作、深拷贝压力测试 | `tests/m4_edge_cases.cpp` | 进阶 |
| M5 | 强异常安全：注入 `std::bad_alloc`，验证失败时原对象不变、不泄漏 | `tests/m5_strong_safety.cpp` | 进阶（M2 之后即可做） |
| bonus | 流运算符 `<<` / `>>` | `tests/stream_tests.cpp` | 选做 |

公开接口签名（函数名、参数、返回类型、`const` / `noexcept`）由
`tests/test_util.h` 里的 `static_assert` 在编译期强制检查，不需要人工比对。

预计工作量：只做必做（M1 + M2）约 6-10 小时；做到进阶（M3 + M4 + M5）再加 5-8 小时。
评分构成见 [`TASKS.md`](TASKS.md) 第 5 节。

## 仓库结构

```text
assignment2-string/
├── CMakeLists.txt              # 构建脚本（默认开启 ASan+UBSan，可关闭）
├── README.md                   # 本文件：概览、快速开始与验收
├── TASKS.md                    # 作业要求（接口、语义、约束、评分构成）
├── .vscode/                    # VS Code 预置配置（clangd + CodeLLDB）
│   ├── extensions.json         #   推荐的扩展
│   ├── settings.json           #   clangd 指向 build/compile_commands.json
│   ├── tasks.json              #   配置 / 构建 / 运行测试
│   └── launch.json             #   调试 m1~m5 与 bonus
├── docs/
│   ├── build-and-test.md       # 构建 / 测试 / ASan+UBSan 详解与 FAQ
│   ├── guide.md                # 教学指南：先修概念 + 实现骨架
│   └── vscode.md               # VS Code 配置说明与排错
├── include/
│   └── my_string.h             # String 的公开接口（需要你补私有数据成员）
├── src/
│   └── my_string.cpp           # 实现文件（目前为空，需要你填写）
└── tests/                      # 自动测试（请勿修改）
    ├── test_util.h             #   共用断言 + 公开接口 static_assert
    ├── m1_basics.cpp           #   里程碑 M1（必做）
    ├── m2_value_semantics.cpp  #   里程碑 M2（必做）
    ├── m3_move.cpp             #   里程碑 M3（进阶）
    ├── m4_edge_cases.cpp       #   里程碑 M4（进阶）
    ├── m5_strong_safety.cpp    #   里程碑 M5：强异常安全（进阶）
    └── stream_tests.cpp        #   流运算符 bonus（选做）
```

## 环境要求

| 项目 | 要求 |
| --- | --- |
| 操作系统 | Linux 或 macOS（Windows 建议使用 WSL2） |
| 编译器 | 支持 C++17 的 GCC / Clang |
| 构建工具 | CMake ≥ 3.14 |
| 内存检查 | AddressSanitizer + UndefinedBehaviorSanitizer（GCC/Clang 自带，无需安装） |
| 编辑器 | 推荐 VS Code + clangd + CodeLLDB，配置已放在 `.vscode/`，见 [`docs/vscode.md`](docs/vscode.md) |

## 快速开始

在仓库根目录执行：

```bash
# 1. 配置 + 构建
cmake -S . -B build
cmake --build build -j

# 2. 运行全部里程碑测试
ctest --test-dir build --output-on-failure
```

也可以只跑某一个里程碑（先用它定位问题，输出更短）：

```bash
ctest --test-dir build -R m1_basics --output-on-failure
# 或直接运行某个测试程序
./build/m1_basics
```

每个测试程序全部通过时输出 `ALL TESTS PASSED`；失败信息会给出
**文件:行号、失败的表达式、实际值与期望值**。

> **默认构建就开启了 ASan + UBSan**（配置阶段会探测编译器支持）。
> 如果想做一次不带 sanitizer 的普通构建，加 `-DENABLE_SANITIZERS=OFF`：
>
> ```bash
> cmake -S . -B build-plain -DENABLE_SANITIZERS=OFF
> cmake --build build-plain -j
> ctest --test-dir build-plain --output-on-failure
> ```
>
> **刚开始构建失败是预期的**：`src/my_string.cpp` 目前为空，链接阶段会报
> `undefined reference to 'String::...'`。先看 `m1_basics` 报的那几条，
> 把它们实现掉，M1 就会变绿，再按 M2 → M3 → M4 往下做。
>
> 只想构建/运行更方便的话，还有两个便捷目标：
> `cmake --build build --target all_tests`（只构建全部测试）与
> `cmake --build build --target check`（构建并运行全部测试）。

### 选做：流运算符测试（bonus）

流运算符 `<<` / `>>` 是选做内容，测试单独放在 `tests/stream_tests.cpp`，
默认不构建（所以不做 bonus 也能全绿）：

```bash
cmake -S . -B build -DENABLE_BONUS_TESTS=ON
cmake --build build -j
ctest --test-dir build --output-on-failure   # m1~m5 与 bonus_stream 一起运行
```

### 内存检查（ASan + UBSan）

ASan + UBSan 已默认开启，所以上文 `build/` 里的测试就是内存检查版本：
配置阶段会先做一次真实的编译 + 链接探测，支持 `-fsanitize=address,undefined`
才继续；不支持（或使用 MSVC）时 CMake 直接报错，并提示改用 GCC/Clang 或加
`-DENABLE_SANITIZERS=OFF` 关闭——不会静默退化成普通构建。

显式写出开关（与默认行为等价）或需要独立目录时：

```bash
cmake -S . -B build-asan -DENABLE_SANITIZERS=ON
cmake --build build-asan -j
ctest --test-dir build-asan --output-on-failure
```

开启后编译与链接都会加上
`-fsanitize=address,undefined -fno-omit-frame-pointer -fno-sanitize-recover=all`。
手动指定标志、各平台注意事项、常见报错解读见
[`docs/build-and-test.md`](docs/build-and-test.md)。

## 验收标准

1. **必做**：`m1_basics`、`m2_value_semantics` 全部通过；
   **进阶**：`m3_move`、`m4_edge_cases`、`m5_strong_safety` 全部通过。
   以上在默认的 ASan/UBSan 构建与 `-DENABLE_SANITIZERS=OFF` 的普通构建下都要通过；
   （选做）开启 `-DENABLE_BONUS_TESTS=ON` 后，`bonus_stream` 也全部通过；
2. 无编译警告（工程统一开启 `-Wall -Wextra -Wpedantic`）；
3. 未使用 `std::string`、`std::string_view` 或任何 STL 容器；
4. 语义与边界正确：空串（默认构造的容量不低于 16）、`capacity()` 不含结尾 `'\0'`、
   长串与多次扩容、自赋值、自移动、自插入、自交换，非法 `insert` / `at` 位置抛出
   `std::out_of_range`；
5. 不得修改 `tests/` 与公开接口签名（后者由测试里的 `static_assert` 强制）；
   私有成员可自由添加；
6. 代码可读，命名与注释清晰；评分可能抽查实现细节与边界情况，请勿针对测试硬编码。

## 遇到问题？

先看 [`docs/build-and-test.md`](docs/build-and-test.md) 的 FAQ 与
[`docs/vscode.md`](docs/vscode.md)（编辑器配置问题）。仍然无法解决时再联系助教，
并在提问时附上完整命令、完整报错信息以及操作系统与编译器版本。
