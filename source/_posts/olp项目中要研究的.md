---
title: olp项目中要研究的
categories:
  - null
description: olp项目中要研究的
date: 2025-08-26 15:28:09
tags:
 - krjx
---

```ts
// 删除窗体
delConfirm(content?: string, tip?: string) {
  return ElMessageBox.confirm(
    content ? content : t('common.delMessage'),
    tip ? tip : t('common.confirmTitle'),
    {
      confirmButtonText: t('common.ok'),
      cancelButtonText: t('common.cancel'),
      type: 'warning'
    }
  )
},

const handleDelete = async (id: number) => {
  await message.delConfirm() // 直接使用异步函数来显示组件
  console.log('开始移除课件id', id)
  let resourceIds: string[] = [];
  resourceIds.push(id.toString())
  await ClassInfoApi.deleteMaterial(resourceIds)
  message.success(t('common.delSuccess'))
  await getList()
}
```
