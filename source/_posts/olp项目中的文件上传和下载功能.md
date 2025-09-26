---
title: olp项目中的文件功能
categories:
  - null
description: olp项目中的文件上传和下载功能
date: 2025-08-26 14:45:20
tags:
  - krjx
---



### 上传课堂资源

#### 组件

uploadMaterial 组件  
抽屉页面，包含 CourseResource 组件 和 保存（触发create api， 取消 按钮。  

CourseResource 组件  
用于上传课堂资源，包含 LargeFileUpload 组件  

LargeFileUpload 组件  

- 用于上传大文件
- 传入设置参数（文件数量、多选、文件类型、正在上传的文件列表
- 传出成功事件、删除事件、初始化事件

#### 上传流程

1. 点击后选择上传
2. 文件直接开始分片上传到服务器，上传成功后后端进行 merge 返回资源的 id、path、url  
3. 调用 addResource api 来将上传的资源进行保存？？？
4. 再调用 createMaterial api 来将保存的资源添加到 courseResource
  调用 createMaterial 需要一个资源文件的 id（需要 addResource api 返回）


### 资源上传又失败了

资源信息如下：  
[
    {
        "origin": 0,
        "format": "application/pdf",
        "mediaType": 3,
        "lang": [
            "1"
        ],
        "duration": 300,
        "fileId": 3585,
        "size": 22907,
        "url": "https://resource.credat.com.cn/sas/20250825/265b2c2d-7b15-4291-b2a8-8ad3116e9142.pdf",
        "name": "新建 DOCX 文档.pdf",
        "title": "新建 DOCX 文档.pdf"
    }
]

resource 接口返回的数据为 null  

使用了 pnpm test 测试环境 + `.env.test`中的 请求路径 为 空，能够正常上传数据了

### 删除

### 下载

```ts
// 更可靠的下载方法
const response = await fetch(url)
const blob = await response.blob()
const link = document.createElement('a')
link.href = URL.createObjectURL(blob)  // 使用 blob URL
link.download = fileName
document.body.appendChild(link)
link.click()
document.body.removeChild(link)
URL.revokeObjectURL(link.href)  // 清理内存
```

## 总结

先上传到服务器，再合并，再添加到服务器资源（此时前端应该传入 address 作为文件地址，方便后续下载），再链接到课程，