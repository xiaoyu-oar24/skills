# 代码生成模板

## 页面模板（分包页面）

```vue
<!-- src/pagesXxx/moduleName/index.vue -->
<script lang="ts" setup>
import { ref, onMounted } from 'vue';
import { onLoad, onShow } from '@dcloudio/uni-app';
import { useUserStore } from '@/stores/user';
import { getXxxList } from '@/api/xxx';

const userStore = useUserStore();
const loading = ref(false);
const list = ref<XxxItem[]>([]);

interface XxxItem {
  id: number;
  name: string;
}

onLoad(options => {
  console.log('页面参数', options);
});

onShow(async () => {
  await fetchData();
});

async function fetchData() {
  if (loading.value) return;
  loading.value = true;
  try {
    const res = await getXxxList({});
    list.value = res.data ?? [];
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <view class="page">
    <uv-loading-page v-if="loading" />
    <view v-else class="content">
      <view v-for="item in list" :key="item.id" class="item">
        {{ item.name }}
      </view>
    </view>
  </view>
</template>

<style lang="scss" scoped>
.page {
  min-height: 100vh;
  background: #f2f4f6;
}
.content {
  padding: 24rpx;
}
.item {
  background: #fff;
  border-radius: 16rpx;
  padding: 24rpx;
  margin-bottom: 16rpx;
}
</style>
```

## 公共组件模板

```vue
<!-- src/components/xxxComponent/xxxComponent.vue -->
<script lang="ts" setup>
interface Props {
  title: string;
  visible?: boolean;
}

interface Emits {
  (e: 'close'): void;
  (e: 'confirm', value: string): void;
}

const props = withDefaults(defineProps<Props>(), {
  visible: false
});

const emit = defineEmits<Emits>();

function handleClose() {
  emit('close');
}
</script>

<template>
  <view v-if="visible" class="xxx-component">
    <view class="title">{{ title }}</view>
    <uv-button @click="handleClose">关闭</uv-button>
  </view>
</template>

<style lang="scss" scoped>
.xxx-component {
  padding: 24rpx;
}
</style>
```

## API 接口模板

```typescript
// src/api/xxx/index.ts
import { request } from '@/utils/request';

export interface XxxItem {
  id: number;
  name: string;
  createdAt: string;
}

export interface GetXxxListParams {
  pageNo?: number;
  pageSize?: number;
  keyword?: string;
}

export const getXxxList = (params: GetXxxListParams) =>
  request<{ list: XxxItem[]; total: number }>({
    url: '/api/module/list',
    method: 'POST',
    data: params
  });

export const getXxxDetail = (id: number) =>
  request<XxxItem>({
    url: '/api/module/detail',
    method: 'GET',
    data: { id }
  });

export const createXxx = (data: Omit<XxxItem, 'id' | 'createdAt'>) =>
  request<{ id: number }>({
    url: '/api/module/create',
    method: 'POST',
    data
  });

export const updateXxx = (data: Partial<XxxItem> & { id: number }) =>
  request<void>({
    url: '/api/module/update',
    method: 'POST',
    data
  });
```

## Pinia Store 模板（Setup 风格）

```typescript
// src/stores/xxx.ts
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import type { XxxItem } from '@/api/xxx';

export const useXxxStore = defineStore('xxx', () => {
  const list = ref<XxxItem[]>([]);
  const currentItem = ref<XxxItem | null>(null);
  const loading = ref(false);

  const count = computed(() => list.value.length);

  function setList(data: XxxItem[]) {
    list.value = data;
  }

  function setCurrentItem(item: XxxItem | null) {
    currentItem.value = item;
  }

  function resetAll() {
    list.value = [];
    currentItem.value = null;
    loading.value = false;
  }

  return { list, currentItem, loading, count, setList, setCurrentItem, resetAll };
});
```

## utils/request.ts 核心封装

```typescript
// src/utils/request.ts
import config from '@/config/index';
import { useUserStore } from '@/stores/user';

let loadingCount = 0;
let loadingTimer: ReturnType<typeof setTimeout> | null = null;
let isRedirectingToLogin = false;

function showLoading() {
  loadingCount++;
  if (loadingCount === 1) {
    loadingTimer = setTimeout(() => {
      uni.showLoading({ title: '加载中...', mask: true });
    }, 300);
  }
}

function hideLoading() {
  loadingCount = Math.max(0, loadingCount - 1);
  if (loadingCount === 0) {
    if (loadingTimer) clearTimeout(loadingTimer);
    uni.hideLoading();
  }
}

interface RequestOptions {
  url: string;
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
  data?: Record<string, unknown>;
  silent?: boolean; // 不显示 loading 和 toast
}

interface ApiResponse<T = unknown> {
  code: string;
  msg: string;
  data: T;
}

const TOKEN_EXPIRED_CODES = ['TK01', 'TK02', 'TK05', 'TK08'];

export function request<T = unknown>(options: RequestOptions): Promise<ApiResponse<T>> {
  const { url, method = 'POST', data = {}, silent = false } = options;
  const userStore = useUserStore();
  const token = uni.getStorageSync(`${config.env}_token`);

  if (!silent) showLoading();

  data.platform = config.platform;

  return new Promise((resolve, reject) => {
    uni.request({
      url: config.apiUrl + url.replace(/^\/api\//, ''),
      method,
      data,
      timeout: 10 * 60 * 1000,
      header: { Authorization: token ? `Bearer ${token}` : '' },
      success(res) {
        const result = res.data as ApiResponse<T>;

        if (TOKEN_EXPIRED_CODES.includes(result.code)) {
          if (!isRedirectingToLogin) {
            isRedirectingToLogin = true;
            userStore.resetAll();
            uni.reLaunch({ url: '/pagesUser/login/login' });
            setTimeout(() => { isRedirectingToLogin = false; }, 3000);
          }
          return reject(result);
        }

        if (result.code !== '0' && result.code !== '200') {
          if (!silent) uni.showToast({ title: result.msg || '请求失败', icon: 'none' });
          return reject(result);
        }

        resolve(result);
      },
      fail(err) {
        if (!silent) uni.showToast({ title: '网络异常，请重试', icon: 'none' });
        reject(err);
      },
      complete() {
        if (!silent) hideLoading();
      }
    });
  });
}
```

## stores/user.ts 标准结构

```typescript
// src/stores/user.ts
import { defineStore } from 'pinia';
import { ref } from 'vue';
import config from '@/config/index';

interface UserInfo {
  userId: string;
  nickname: string;
  avatar: string;
  segNo?: string; // 有值=买家，空=卖家
  [key: string]: unknown;
}

export const useUserStore = defineStore('user', () => {
  const token = ref<string>(uni.getStorageSync(`${config.env}_token`) || '');
  const userInfo = ref<UserInfo | null>(
    JSON.parse(uni.getStorageSync(`${config.env}_userInfo`) || 'null')
  );
  const role = ref<boolean>(false); // true=买家, false=卖家
  const statusBarHeight = ref<number>(0);
  const bottomSafeHeight = ref<number>(20);

  function getToken() {
    return token.value;
  }

  function setToken(val: string) {
    token.value = val;
    uni.setStorageSync(`${config.env}_token`, val);
  }

  function getUserInfo() {
    return userInfo.value;
  }

  function setUserInfo(info: UserInfo) {
    userInfo.value = info;
    role.value = !!info.segNo;
    uni.setStorageSync(`${config.env}_userInfo`, JSON.stringify(info));
  }

  function getRole() {
    return role.value; // true=买家, false=卖家
  }

  function setStatusBarHeight(h: number) {
    statusBarHeight.value = h;
  }

  function setBottomSafeHeight(h: number) {
    bottomSafeHeight.value = h;
  }

  function resetAll() {
    token.value = '';
    userInfo.value = null;
    role.value = false;
    uni.removeStorageSync(`${config.env}_token`);
    uni.removeStorageSync(`${config.env}_userInfo`);
  }

  return {
    token,
    userInfo,
    role,
    statusBarHeight,
    bottomSafeHeight,
    getToken,
    setToken,
    getUserInfo,
    setUserInfo,
    getRole,
    setStatusBarHeight,
    setBottomSafeHeight,
    resetAll
  };
});
```
