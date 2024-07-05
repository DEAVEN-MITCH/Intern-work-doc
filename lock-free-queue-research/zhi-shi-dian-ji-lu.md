---
description: knowledge acquired during investigation
---

# 🤗 知识点记录

## atomic库

atomic库通过硬件原语自动保证了不同核上对应变量的缓存一致性，不需要手动实现。

compare\_exchange\_weak可能由于硬件问题在\*r=oldvalue时替换失败，而compare\_exchange\_strong则引入较大开销防止该因硬件而失败的可能性。

> 用#if strong #def cas compare\_exchange\_strong #else compare\_exchange\_weak #endif?

atomic\<shared\_ptr>在C++20才支持。

* `std::memory_order_relaxed`: 不强制任何顺序。
* `std::memory_order_consume`: 确保当前线程的依赖性读操作不会被重排。
* `std::memory_order_acquire`: 确保当前线程的后续读写操作不会被重排。用于读
* `std::memory_order_release`: 确保当前线程的先前读写操作不会被重排。用于写
* `std::memory_order_acq_rel`: 同时具有 acquire 和 release 的效果。用于读写
* `std::memory_order_seq_cst`: 最严格的顺序，确保全局顺序性。



## memory库

## compiler相关

* `__attribute__((aligned(64)))`是一个属性，它指示编译器将变量或数据结构的内存对齐设置为64字节。这通常用于性能优化，因为某些硬件平台上对齐的内存访问比非对齐的内存访问更快。

## 宏定义

* \#x表示将x字符串化，a##b表示将a和b两个宏拼接为一个token
* assert(expression&\&message)可以显示message信息
* `AE_NO_TSAN` 可能是一个宏定义，通常用于禁用 ThreadSanitizer (TSan) 工具对特定代码块的检测。
* `AE_TSAN_ANNOTATE_ACQUIRE()` 和 `AE_TSAN_ANNOTATE_RELEASE()` 可能是与线程安全分析（ThreadSanitizer, TSAN）相关的注释宏。它们通常用于注释和标记程序中的内存访问模式，以便静态或动态分析工具可以更好地检测和报告数据竞争和其他并发问题。
*

## type\_traits相关

* `std::alignment_of<Block>::value` 返回的是一个 `size_t` 类型的常量表达式，表示类型 `Block` 的对齐要求的字节数。
* sizeof()返回size\_t类型

## cstdint相关

* `std::uintptr_t` 是 C++ 标准库中的一种整数类型，专门用于存储指针的整数表示。这种类型定义在 `<cstdint>` 头文件中，通常用于需要以整数形式操作指针的场景。
*
