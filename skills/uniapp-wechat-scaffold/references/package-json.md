# package.json 标准模版

```json
{
  "name": "your-project-name",
  "version": "1.0.0",
  "scripts": {
    "dev": "uni --mode test",
    "dev:prod": "uni --mode prod",
    "dev:h5": "uni --mode test",
    "dev:mp-weixin:test": "uni -p mp-weixin --mode test",
    "dev:mp-weixin:prod": "uni -p mp-weixin --mode prod",
    "build": "node build.mjs",
    "build:mp-weixin:test": "uni build -p mp-weixin --mode test",
    "build:mp-weixin:prod": "uni build -p mp-weixin --mode prod",
    "format": "prettier --write src",
    "type-check": "vue-tsc --noEmit",
    "prepare": "husky install"
  },
  "dependencies": {
    "@dcloudio/uni-app": "3.0.0-4050720250324001",
    "@dcloudio/uni-mp-weixin": "3.0.0-4050720250324001",
    "@dcloudio/uni-h5": "3.0.0-4050720250324001",
    "pinia": "^2.0.36",
    "vue": "^3.5.13",
    "vue-i18n": "^9.14.4"
  },
  "devDependencies": {
    "@dcloudio/types": "*",
    "@dcloudio/vite-plugin-uni": "3.0.0-4050720250324001",
    "@vue/tsconfig": "*",
    "vite": "5.2.8",
    "typescript": "^4.9.4",
    "vue-tsc": "^1.0.24",
    "husky": "^8.0.3",
    "lint-staged": "^13.2.3",
    "prettier": "^3.0.3",
    "chalk": "^5.4.1",
    "inquirer": "^12.5.2",
    "shelljs": "^0.9.2",
    "dotenv": "^16.4.7",
    "jsonminify": "^0.4.2",
    "io-spin": "^0.4.1",
    "log-symbols": "^7.0.0"
  },
  "lint-staged": {
    "src/**/*.{js,jsx,ts,tsx,vue,json,css,scss,html,md}": [
      "prettier --write"
    ]
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

## Node 版本要求

- 最低：18.0.0
- 推荐：20.19.0
- 在项目根目录创建 `.nvmrc` 写入 `20.19.0`

## 包管理器

使用 **pnpm**（build.mjs 内置 pnpm → npm 的回退机制）。
