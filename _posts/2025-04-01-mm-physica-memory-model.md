---
layout: post
title: 物理内存模型
subtitle: linux physical memory model.
tags: [linux, mm]
---

原文：[Physical Memory Model](https://docs.kernel.org/mm/memory-model.html)

linux 支持3中物理内存模型：FLATMEM、SPARSEMEM、SPARSEMEM_VMEMMAP，不同的模型 page 到页号的转换实现是不一样的，每一种模型都需要定义 ==pfn_to_page== 和 ==page_to_pfn== 实现。

FLATMEM 适用于非 NUMA 系统，物理内存地址连续，mem_map 数组用于表示每个物理 page，FLATMEM 物理内存也可能有空洞（holes），但 holes 页面也有对应的 page 结构，只是这个 page 不能初始化。mem_map 通过 free_area_init 初始化，但直到 memblock_free_all()  执行后才能从 mem_map 分配内存。

SPARSEMEM_VMEMMAP 模型把物理上不连续的内存 page，映射到连续的虚拟地址空间，全局变量 vmemmap 就指向这片空间，该变量是通过 vmemmap_populate 建立起来的。

x86_64 模式使用 SPARSEMEM_VMEMMAP 模型。
