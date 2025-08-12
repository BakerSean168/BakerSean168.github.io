---
title: 「date-fns」库的使用
categories:
  - null
description: 「date-fns」库的使用
date: 2025-07-25 20:21:40
tags:
---

[中文文档](https://date-fns.p6p.net/general/getting-started.html)

`pnpm add date-fns`

### 显示时间数据

format(date, formatStr)
格式化日期为指定字符串，如：format(new Date(), 'yyyy-MM-dd HH:mm')

formatDistance(date, baseDate, options?)
显示两个日期之间的距离（如“3天前”、“2个月后”）

formatRelative(date, baseDate, options?)
显示相对时间（如“昨天”，“明天”）
{
    "_uuid": "05bd7999-f0fe-4746-a263-f3eab8d41e47",
    "_domainEvents": [],
    "_name": "123123",
    "_description": null,
    "_color": "#FF5733",
    "_dirUuid": "97d150ee-7e7d-4ea3-aa62-1c603d6e8ce7",
    "_startTime": "2025-07-26T15:43:56.689Z",
    "_endTime": "2025-08-25T15:43:56.689Z",
    "_note": null,
    "_keyResults": [
        {
            "_uuid": "ca8fee17-6fe3-4dc8-b799-7ed8d70fb10e",
            "_name": "22222",
            "_startValue": 0,
            "_targetValue": 10,
            "_currentValue": 0,
            "_calculationMethod": "sum",
            "_weight": 4,
            "_lifecycle": {
                "createdAt": "2025-07-26T15:44:03.687Z",
                "updatedAt": "2025-07-26T15:44:05.310Z",
                "status": "active"
            }
        }
    ],
    "_records": [],
    "_reviews": [],
    "_analysis": {
        "motive": "",
        "feasibility": ""
    },
    "_lifecycle": {
        "createdAt": "2025-07-26T15:43:56.689Z",
        "updatedAt": "2025-07-26T15:44:06.922Z",
        "status": "active"
    },
    "_analytics": {
        "overallProgress": 0,
        "weightedProgress": 0,
        "completedKeyResults": 0,
        "totalKeyResults": 0
    },
    "_version": 9
}