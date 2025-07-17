---
author: Axton
pubDatetime: 2025-07-17T15:22:00Z
modDatetime: 2025-07-17T15:22:00Z
title: React Hook Form + Zod使用示例
featured: false
draft: false
tags:
  - React Hook Form
  - Zod
description:
  "React Hook Form + Zod使用示例"
---

## ✅ 什么是 `resolver`？

`resolver` 是 React Hook Form 中用于**接入第三方校验库**（比如 Zod、Yup、Joi）的 **桥接器**。它可以让你把像 Zod 这样的 schema 用来做表单校验。

> 简单说：**resolver = 把表单数据送进 Zod/Yup，拿回校验结果的中间件**

---

## 🧩 使用场景举例：

- 你不想一个个用 `register('xxx', { required: true })` 去设置校验规则。
    
- 你希望用一个统一的 schema 文件定义校验逻辑，并且和 TS 类型一致。
    
- 想要表单、接口请求、接口响应用同一份 schema。
    

---

## 🚀 示例：React Hook Form + Zod

### 第一步：安装依赖

```bash
npm install react-hook-form zod @hookform/resolvers
```

---

### 第二步：写 schema（Zod）

```ts
// schemas/login.ts
import { z } from 'zod';

export const LoginSchema = z.object({
  email: z.string().email('请输入合法的邮箱'),
  password: z.string().min(6, '密码至少 6 位'),
});

export type LoginFormValues = z.infer<typeof LoginSchema>;
```

---

### 第三步：用 `useForm` + `resolver`

```tsx
// pages/LoginForm.tsx
import React from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { LoginSchema, LoginFormValues } from '@/schemas/login';

export default function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<LoginFormValues>({
    resolver: zodResolver(LoginSchema), // 👈 核心点
  });

  const onSubmit = (data: LoginFormValues) => {
    console.log('提交的数据:', data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input
          type="text"
          placeholder="邮箱"
          {...register('email')}
        />
        {errors.email && <p style={{ color: 'red' }}>{errors.email.message}</p>}
      </div>

      <div>
        <input
          type="password"
          placeholder="密码"
          {...register('password')}
        />
        {errors.password && <p style={{ color: 'red' }}>{errors.password.message}</p>}
      </div>

      <button type="submit">登录</button>
    </form>
  );
}
```

---

## 💡 说明下关键点：

|点|意义|
|---|---|
|`resolver: zodResolver(schema)`|告诉表单使用哪个 schema 做验证|
|`register('field')`|绑定字段|
|`errors.xxx.message`|拿到 Zod/Yup 返回的错误信息|
|`LoginFormValues`|由 schema 自动推导出的类型，确保类型安全|

---

## 🧠 支持哪些库？

官方 `@hookform/resolvers` 支持：

- ✅ Zod
    
- ✅ Yup
    
- ✅ Joi
    
- ✅ Superstruct
    
- ✅ Vest
    
- ✅ Ajv
    
- ✅ MyZod
    
- ... 等等
    

---

## 📝 总结一句话：

> `resolver` 让你能把 Zod/Yup 的运行时校验逻辑无缝接入 React Hook Form，统一表单校验与类型定义。