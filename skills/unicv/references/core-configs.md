# 核心配置文件

## vite.config.ts

```typescript
import { defineConfig } from 'vite';
import uni from '@dcloudio/vite-plugin-uni';

export default defineConfig({
  plugins: [uni()],
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "./src/uni.scss";`
      }
    }
  }
});
```

## tsconfig.json

```json
{
  "extends": "@vue/tsconfig/tsconfig.json",
  "compilerOptions": {
    "sourceMap": true,
    "baseUrl": ".",
    "paths": { "@/*": ["./src/*"] },
    "lib": ["esnext", "dom"],
    "types": ["@dcloudio/types"]
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"]
}
```

## .env.test

```
VITE_APP_TYPE=test
```

## .env.prod

```
VITE_APP_TYPE=prod
```

## .prettierrc

```json
{
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "trailingComma": "none",
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}
```

## .husky/pre-commit

```sh
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged
```

## src/uni.scss

```scss
// 全局 SCSS 变量，按项目需要定义
$uni-color-dominant: #your-brand-color;
$uni-color-primary: #your-primary-color;
```

## src/main.ts

```typescript
import { createSSRApp } from 'vue';
import App from './App.vue';
import * as Pinia from 'pinia';
import utils from './utils';
import uvUI from './uni_modules/uv-ui-tools'; // @ts-ignore

export function createApp() {
  const app = createSSRApp(App);
  app.use(Pinia.createPinia());
  app.use(uvUI);
  uni.$utils = utils;
  utils.init();
  return { app };
}
```

## src/App.vue

```vue
<script lang="ts" setup>
import { onLaunch, onShow, onHide } from '@dcloudio/uni-app';
import { useUserStore } from '@/stores/user';

const userStore = useUserStore();

onLaunch(async () => {
  await uni.$utils.wxLogin();
  userStore.getUserInfo();
  const res = uni.getSystemInfoSync();
  userStore.setStatusBarHeight(res.statusBarHeight);
  userStore.setBottomSafeHeight(res.screenHeight - res.safeArea.bottom || 20);
});
</script>

<style lang="scss">
@import '@/uni_modules/uv-ui-tools/index.scss';
page {
  background: #f2f4f6;
}
</style>
```

## src/config/index.ts

```typescript
const env = (import.meta.env.VITE_APP_TYPE as string) || 'test';

const baseUrlMap: Record<string, string> = {
  test: 'https://your-test-domain.com',
  prod: 'https://your-prod-domain.com'
};

const apiUrlMap: Record<string, string> = {
  test: 'https://your-test-domain.com/api/',
  prod: 'https://your-prod-domain.com/api/'
};

const baseUrl = baseUrlMap[env];
const apiUrl = apiUrlMap[env];
const imgUrl = 'https://your-cdn.com/static';
const platform = 'your_platform_id';  // 平台标识，与后端约定

export default { env, baseUrl, apiUrl, imgUrl, platform };
```
