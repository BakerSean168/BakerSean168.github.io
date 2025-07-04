---
title: 使用Nodejs的crypto模块实现用AES算法加解密密码
categories:
  - null
description: 使用Nodejs的crypto模块实现用AES算法加解密密码
date: 2025-06-10 15:52:29
tags:
---

希望能够  
在登录时选择记住密码的时候，记录下用户的密码  
login 服务就能直接从记录中读取密码进行认证，实现快速登录了。

所以在登录会话服务中采用 对称加密方法AES 来实现密码加密保存  

- iv 来生成随机向量，能够让相同的密码生成不同的结果
- hex格式字符串方便存储，二进制字符串方便处理

```ts
 /**
   * AES密码加密
   * 使用AES-256-CBC算法对密码进行可逆加密
   *
   * @param password 原始密码
   * @returns 加密后的密码（包含IV）
   * @private
   */
  private async encryptPassword(password: string): Promise<string> {
    try {
      // 生成随机初始化向量
      const iv = crypto.randomBytes(16);

      // 将密钥从hex转换为Buffer
      const key = Buffer.from(this.secretKey, "hex");

      // 创建加密器
      const cipher = crypto.createCipheriv(this.algorithm, key, iv);
      // cipher.setAutoPadding(true);

      // 加密密码
      let encrypted = cipher.update(password, "utf8", "hex");
      encrypted += cipher.final("hex");

      // 将IV和加密数据组合
      const result = iv.toString("hex") + ":" + encrypted;

      return result;
    } catch (error) {
      console.error("密码加密失败:", error);
      throw new Error("密码加密失败");
    }
  }

  /**
   * AES密码解密
   * 解密使用AES-256-CBC算法加密的密码
   *
   * @param encryptedData 加密的密码数据（包含IV）
   * @returns 解密后的原始密码
   * @private
   */
  private async decryptPassword(encryptedData: string): Promise<string> {
    try {
      // 分离IV和加密数据
      const parts = encryptedData.split(":");
      if (parts.length !== 2) {
        throw new Error("加密数据格式错误");
      }

      const iv = Buffer.from(parts[0], "hex");
      const encrypted = parts[1];

      // 将密钥从hex转换为Buffer
      const key = Buffer.from(this.secretKey, "hex");

      // 创建解密器
      const decipher = crypto.createDecipheriv(this.algorithm, key, iv);
      // decipher.setAutoPadding(true);

      // 解密密码
      let decrypted = decipher.update(encrypted, "hex", "utf8");
      decrypted += decipher.final("utf8");

      return decrypted;
    } catch (error) {
      console.error("密码解密失败:", error);
      throw new Error("密码解密失败");
    }
  }
```
