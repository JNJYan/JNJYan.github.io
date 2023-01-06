---
title: gflag
date: 2021-08-14 18:12:50
tags:
- C++
categories:
- 源码阅读
---

## FlagRegistry

## AddFlagValidator
```c++
bool AddFlagValidator(const void* flag_ptr, ValidateFnProto validate_fn_proto) {
    FlagRegistry* const registry = FlagRegistry::GlobalRegistry();
    FlagRegistryLock frl(registry);
}
```