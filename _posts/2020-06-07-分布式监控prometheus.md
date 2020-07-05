---
layout:     post
title:      分布式监控Prometheus一
subtitle:   如何获取数据
date:       2020-06-07
author:     Link
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Prometheus
---

# Prometheus

## 简介
Prometheus的核心数据模型是时间序列，因此它非常适合记录时间序列数据，并根据记录的时间序列进行相关的聚合和分析操作。