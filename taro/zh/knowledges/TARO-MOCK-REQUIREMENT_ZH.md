
### mock 文件的写法规则说明

在项目的**根目录下**创建一个名为 `mock` 的文件夹：

#### 1. 创建 mock 目录

```txt
your-taro-project/
├── mock/
├── src/
├── config/
└── package.json
```

#### 2. 编写接口配置文件
在 `mock` 目录中可以创建多个`.ts` 文件来定义接口，支持 ES6 模块语法和 TypeScript。

例如：创建文件 `mock/api.ts`

```ts
// mock/api.ts
export default {
  // GET 请求：获取用户信息
  'GET /mock/user/1': {
    name: 'luckyadam',
    age: 18,
    gender: 'male'
  },

  // POST 请求：上传文件
  'POST /mock/upload': {
    code: 0,
    message: '上传成功',
    file: 'xxxx'
  }
}
```


#### 4. 多文件组织（可选）
你可以将不同模块的接口拆分到不同的文件中，并统一导出。比如：

```txt
mock/
  ├── user.ts
  ├── order.ts
  └── index.ts
```

然后在 `index.ts` 中合并：

```ts
// mock/index.ts
import user from './user';
import order from './order';

export default {
  ...user,
  ...order
};
```

### 总结

| 特性 | 说明 |
|------|------|
| ✅ 目录位置 | 项目根目录下的 `mock/` 文件夹 |
| ✅ 文件格式 | 支持 `.js` 和 `.ts` 文件 |
| ✅ 语法支持 | 支持 ES6 模块、TypeScript、async/await 等现代语法 |
| ✅ 动态数据 | 可集成 `mockjs` 实现随机数据生成 |
| ✅ 请求方法 | 使用 `'METHOD /path'` 格式定义路由 |
| ✅ 多文件管理 | 可拆分多个文件并通过 `export default` 组织 |

### 注意

- 接口统一 `/mock` 前缀
- 每个接口通过 `'METHOD /url'` 的格式作为键名
- 方法包含:`GET`, `POST`, `PUT`, `DELETE`, `HEAD`, `OPTIONS`, `PATCH`, `TRACE`, `CONNECT`