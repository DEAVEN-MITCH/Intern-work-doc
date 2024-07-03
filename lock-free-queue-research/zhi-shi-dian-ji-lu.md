---
description: knowledge acquired during investigation
---

# 知识点记录

## atomic库

atomic库通过硬件原语自动保证了不同核上对应变量的缓存一致性，不需要手动实现。

compare\_exchange\_weak可能由于硬件问题在\*r=oldvalue时替换失败，而compare\_exchange\_strong则引入较大开销防止该因硬件而失败的可能性。

> 用#if strong #def cas compare\_exchange\_strong #else compare\_exchange\_weak #endif?



## memory库



