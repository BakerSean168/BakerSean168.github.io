---
title: olp中的文件预览功能
categories:
  - null
description: olp中的文件预览功能
date: 2025-08-28 11:38:06
tags:
  - krjx
---

1. 下载文件
2. 把文件展示

```ts
// 获取文件下载进度的代码
const loadFileContent = async (file: any) => {
  try {
    fileLoading.value = true
    downloadProgress.value = 0
    isDownloading.value = false

    fileType.value = file.fileType

    if (isPdfFormat(fileType.value)) {
      showVideo.value = false
      if (file.fileUrl.includes('https')) {
        isDownloading.value = true
        try {
          const response = await axios.request({
            baseURL: file.fileUrl,
            responseType: 'blob',
            onDownloadProgress: (progressEvent) => {
              if (progressEvent.total) {
                downloadProgress.value = Math.round((progressEvent.loaded * 100) / progressEvent.total)
              }
            }
          })
          const blob = new Blob([response.data], { type: 'application/pdf' })
          const url = window.URL.createObjectURL(blob)
          resqUrl.value = url
        } catch (error) {
          console.error('Failed to download file:', error)
          // fallback to direct URL
          resqUrl.value = file.fileUrl
        } finally {
          isDownloading.value = false
        }
      } else {
        resqUrl.value = formatImgUrl(file.fileUrl)
      }
    } else if (isVideoFormat(fileType.value)) {
      resqUrl.value = file.fileUrl
      showVideo.value = true
    } else {
      // 其他格式文件，默认使用iframe显示
      resqUrl.value = file.fileUrl
      showVideo.value = false
    }
  } finally {
    fileLoading.value = false
  }
}
```