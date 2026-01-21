---
title: C 语言进阶：巧用 Switch 击穿特性解决累进提成问题
published: 2026-01-21
pinned: false
description: 通过一道经典的企业奖金计算题，深入解析 C 语言中 switch-case 的 fall-through（击穿）机制及其实战应用。
tags: [C语言, 算法, 编程基础]
category: 技术教程
draft: false
---

在 C 语言编程中，`switch` 语句的“击穿”（Fall-through）特性通常被视为初学者的易错点。然而，在处理**阶梯式累进计算**（如税率、奖金提成）时，它反而能提供比 `if-else` 更优雅、更简洁的解法。

## 题目描述

企业发放的奖金根据利润 $I$ 阶梯式提成：

* $I \le 10W$：提成 **10%**
* $10W < I \le 20W$：$10W$ 以内按 10%，超过部分按 **7.5%**
* $20W < I \le 40W$：超过 20W 部分按 **5%**
* $40W < I \le 60W$：超过 40W 部分按 **3%**
* $60W < I \le 100W$：超过 60W 部分按 **1.5%**
* $I > 100W$：超过 100W 部分按 **1%**

## 解题思路：分段映射与逻辑击穿

核心逻辑是将连续的利润值映射为离散的 `case` 标签。我们以 10 万为一个基数单位：
1.  计算 `flag = (int)(利润 / 100000)`。
2.  **从高向低** 匹配 `case`。
3.  **故意不写 `break`**：计算完当前区间的差额后，让逻辑直接进入下一个 `case`，逐层“剥开”利润。



## 代码实现

### 完整解法

```cpp title="bonus_calc.c" {12-14, 38} ins="// 不使用 break 触发击穿"
#include <stdio.h>

int main() {
    double d;
    int money = 100000;
    float res = 0.0;
    int flag;

    printf("请输入当月利润: ");
    if (scanf("%lf", &d) != 1) return 1;

    // 逻辑转换：将利润分段映射到 0-10 的索引上
    flag = (int)(d / money);
    if (flag > 10) flag = 10;

    switch(flag) {
        case 10:
            res += (d - 10 * money) * 0.01;
            d = 10 * money;
        case 9:
        case 8:
        case 7:
        case 6:
            res += (d - 6 * money) * 0.015;
            d = 6 * money;
        case 5:
        case 4:
            res += (d - 4 * money) * 0.03;
            d = 4 * money;
        case 3:
        case 2:
            res += (d - 2 * money) * 0.05;
            d = 2 * money;
        case 1:
            res += (d - money) * 0.075;
            d = money;
        case 0:
            res += d * 0.1;
    }

    printf("应发放奖金总数：%.2f\n", res);
    return 0;
}