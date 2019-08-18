---
layout: post
title: "小区搜索"
comments: true
description: "通过小区搜索，UE可以接入小区，获取小区的信息，实现下行同步。"
keywords: "PSS, SSS, PBCH, DMRS, MIB, SIB1"
---

# 什么是小区搜索（Cell Search）？
> Cell search is the procedure by which a UE acquires time and frequency synchronization with a cell and detects the Cell ID of that cell. NR cell search is based on the primary and secondary synchronization signals, and PBCH DMRS, located on the synchronization raster.

**小区搜索程序**
1. UE调整到合适频率
2. UE检测PSS/SSS
3. UE检测PBCH
4. UE解MIB，根据获得的信息解SIB1和OSI

