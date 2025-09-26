---
title: 前端HTTP状态码的处理
categories:
  - null
description: 前端HTTP状态码的处理
date: 2025-08-25 16:14:47
tags:
  - 前端
  - HTTP 
---

下面给出简洁的最佳实践要点 + 推荐的 axios 拦截器模板，便于在你当前项目中统一处理 HTTP status 与业务 code。

要点（简短）
- 区分「HTTP 状态码」(network、401/403/500 等) 和「业务返回码」(res.data.code)；两者分别处理。  
- 在请求拦截器统一注入 token/语言/通用 headers。  
- 在响应拦截器统一处理：
  - HTTP 非 2xx（网络错误、超时、CORS） -> 友好提示/重试或上报。
  - HTTP 401 -> 触发刷新 token 流程（队列化待重试请求），刷新失败跳转登录。 
  - 2xx 中根据业务 code 判断成功与否（config.result_code），若不成功返回统一错误提示或抛出错误给上层处理。  
- 文件下载要返回原始 response（responseType: 'blob'）。  
- 将错误信息对用户友好化（不直接展示后台原始 msg），同时对错误做结构化上报（Sentry/日志）。  
- 对关键接口做限流/重试（指数退避），避免重复提交。  
- 若只需要 ID，提交 IDs；若需要完整对象，提前补齐（避免跨页丢失）。  

示例：推荐在 service.ts 放一个 axios 实例 + 拦截器（可根据你项目结构改名）

````typescript
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios'
import { config } from './config'
import { useCache } from '@/utils/cache' // 假设你有的工具
import { message } from '@/utils/message' // 统一提示封装
import router from '@/router'

const { wsCache } = useCache()

const instance: AxiosInstance = axios.create({
  baseURL: config.base_url,
  timeout: config.request_timeout,
  headers: { 'Content-Type': String(config.default_headers) }
})

// 请求拦截器：注入 token / lang / tag
instance.interceptors.request.use((req: AxiosRequestConfig) => {
  const token = wsCache.get('TOKEN')
  if (token) req.headers = { ...req.headers, Authorization: `Bearer ${token}` }
  const lang = wsCache.get('LANG')
  if (lang) req.headers = { ...req.headers, Lang: lang }
  req.headers = { ...req.headers, tag: 'constantine' }
  return req
}, (error) => {
  return Promise.reject(error)
})

// 刷新 token 的简单队列实现（示意）
let isRefreshing = false
let requestsQueue: Array<(token: string | null) => void> = []
const pushQueue = (cb: (token: string | null) => void) => requestsQueue.push(cb)
const runQueue = (token: string | null) => {
  requestsQueue.forEach(cb => cb(token))
  requestsQueue = []
}

// 响应拦截器：统一处理 HTTP + 业务 code
instance.interceptors.response.use(
  (response: AxiosResponse) => {
    // 文件下载直接返回原始 response
    if (response.config.responseType === 'blob' || response.headers['content-type']?.includes('application/octet-stream')) {
      return response
    }

    const res = response.data
    // 统一判定业务成功（以 config.result_code 为准）
    if (res && res.code !== undefined) {
      if (res.code === config.result_code) {
        return res.data === undefined ? res : res // 视项目约定可直接返回 res.data
      }
      // 业务错误：统一提示并抛出
      message.error(res.message || '操作失败')
      return Promise.reject({ isBusinessError: true, ...res })
    }

    // 无业务 code，直接返回 response.data
    return res
  },
  async (error) => {
    const { response, config: reqConfig } = error || {}
    // 网络/超时
    if (!response) {
      message.error('网络错误或请求超时')
      return Promise.reject(error)
    }

    const status = response.status
    if (status === 401) {
      // 刷新 token 流程
      if (!isRefreshing) {
        isRefreshing = true
        try {
          // 假设 refreshToken 接口在 /auth/refresh，返回新 token
          const refreshRes = await axios.post(`${config.base_url}/auth/refresh`, { /*...*/ })
          const newToken = refreshRes.data?.token
          wsCache.set('TOKEN', newToken)
          runQueue(newToken)
        } catch (e) {
          runQueue(null)
          wsCache.remove('TOKEN')
          router.push({ name: 'Login' })
        } finally {
          isRefreshing = false
        }
      }

      return new Promise((resolve, reject) => {
        pushQueue((token) => {
          if (!token) {
            reject(error)
            return
          }
          // 重试原请求
          reqConfig.headers = { ...reqConfig.headers, Authorization: `Bearer ${token}` }
          resolve(axios(reqConfig))
        })
      })
    }

    if (status >= 500) {
      message.error('服务端异常，请稍后重试')
    } else if (status >= 400) {
      const msg = response.data?.message || `请求错误 ${status}`
      message.error(msg)
    }

    return Promise.reject(error)
  }
)

export default instance
````

如何在你现有 index.ts 使用（简短）
- index.ts 的 request 应该调用上面 instance，并返回合适的值（res.data 或完整 response），并处理 headersType / baseURL 覆盖等。确保 get/post 等方法把结果 return 回去（以前代码片段中缺少 return）。

结语（一句话）
- 统一拦截器 + 区分 HTTP 与业务 code + token 刷新队列 + 用户友好提示，是前端 HTTP 错误处理的关键；提交时若会丢失对象，优先传 IDs 或在提交前补齐对象。