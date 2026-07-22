# 项目目录结构规范

```
项目根/
├── src/
│   ├── api/                      # 接口定义（按业务模块划分）
│   │   ├── login/index.ts
│   │   ├── user/index.ts
│   │   ├── order/index.ts
│   │   └── [模块]/index.ts
│   ├── components/               # 全局公共组件
│   │   └── [组件名]/[组件名].vue
│   ├── config/
│   │   └── index.ts              # 环境配置（baseUrl、apiUrl、platform）
│   ├── pages/                    # 主包（TabBar 页面，≤2M）
│   │   ├── home/index.vue
│   │   ├── order/index.vue
│   │   └── main/index.vue
│   ├── pagesOrders/              # 订单分包
│   ├── pagesUser/                # 用户分包
│   ├── pagesCompany/             # 公司分包
│   ├── pagesSeller/              # 卖家分包
│   ├── pagesScan/                # 扫码分包
│   ├── pagesCommon/              # 公共分包（相机等）
│   ├── stores/
│   │   └── user.ts               # 用户 Pinia Store
│   ├── utils/
│   │   ├── index.ts              # 工具函数入口（挂载到 uni.$utils）
│   │   ├── request.ts            # HTTP 请求封装
│   │   └── uploadFile.ts         # 文件上传
│   ├── static/                   # 静态资源
│   ├── uni_modules/              # uv-ui 等本地插件
│   ├── App.vue
│   ├── main.ts
│   ├── pages.json                # 路由 + 分包配置
│   ├── manifest.json             # 应用配置（appid 等）
│   └── uni.scss                  # 全局 SCSS 变量
├── .env.test                     # 测试环境变量
├── .env.prod                     # 生产环境变量
├── .husky/
│   └── pre-commit                # lint-staged 钩子
├── .prettierrc
├── .nvmrc
├── build.mjs                     # 交互式打包脚本
├── vite.config.ts
├── tsconfig.json
└── package.json
```

## 分包策略

| 分包 | 路径 | 内容 |
|------|------|------|
| 主包 | `pages/` | TabBar 页面（首页、订单、我的等） |
| pagesOrders | `pagesOrders/` | 订单详情、编辑、合并、检验 |
| pagesUser | `pagesUser/` | 登录、实名认证、收款方式 |
| pagesCompany | `pagesCompany/` | 公司查询、添加车辆、待办列表 |
| pagesSeller | `pagesSeller/` | 卖家主页、估价、交车方式 |
| pagesScan | `pagesScan/` | 扫码结果、服务评价 |
| pagesCommon | `pagesCommon/` | 自定义相机等公共页面 |

**大小限制：** 单个分包 ≤ 2M，全部分包合计 ≤ 20M。

## pages.json 分包配置示例

```json
{
  "pages": [
    { "path": "pages/home/index", "style": { "navigationBarTitleText": "首页" } }
  ],
  "subPackages": [
    {
      "root": "pagesOrders",
      "pages": [
        { "path": "orderDetail/index", "style": { "navigationBarTitleText": "订单详情" } }
      ]
    },
    {
      "root": "pagesUser",
      "pages": [
        { "path": "login/login", "style": { "navigationBarTitleText": "登录" } }
      ]
    }
  ],
  "tabBar": {
    "list": [
      { "pagePath": "pages/home/index", "text": "首页" },
      { "pagePath": "pages/order/index", "text": "订单" },
      { "pagePath": "pages/main/index", "text": "我的" }
    ]
  },
  "globalStyle": {
    "navigationBarTextStyle": "black",
    "navigationBarBackgroundColor": "#FFFFFF",
    "backgroundColor": "#f2f4f6"
  }
}
```
