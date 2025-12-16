---
title: 如何在 Astro 的 Fuwari 主題中對導覽列新增功能
published: 2024-10-22
description: 'Friends?'
image: ''
tags: [Guides, Astro, 自定義]
lang: 'zh-TW'
draft: false
---

## 新增功能?

Fuwari 主題的導覽列一開始只有 Home, Archive, About 這幾個功能，我想加入 Friends 功能，原本以為在 `config.ts` 中新增幾行程式碼就可以解決，但事情沒有我想的那麼簡單。

## 新增 Head 導航連結

在 `src/config.ts` 中，由 `navBarConfig` 參數控制導航，所以我們可以在此加入想新增的導航

首先編輯 `src/types/config.ts`，找到 `LinkPreset` 加入我們要的類別

```ts
export enum LinkPreset {
  Home = 0,
  Archive = 1,
  About = 2,
  Friends = 3,      // 等號前面的是導行列的名稱，後面則是順序
}
```

接著進入 `src/config.ts`，找到 `navBarConfig`，並加入我們剛才新增的類別

```ts
export const navBarConfig: NavBarConfig = {
  links: [
    LinkPreset.Home,
    LinkPreset.Archive,
    LinkPreset.About,
    LinkPreset.Friends,     // 引用 LinkPreset 中的 Friends 類別
  ],
}
```

修改 `src/constants/link-presets.ts`，並加入 `Friends`

```ts
export const LinkPresets: { [key in LinkPreset]: NavBarLink } = {
  [LinkPreset.Home]: {
    name: i18n(I18nKey.home),
    url: '/',
  },
  [LinkPreset.About]: {
    name: i18n(I18nKey.about),
    url: '/about/',
  },
  [LinkPreset.Archive]: {
    name: i18n(I18nKey.archive),
    url: '/archive/',
  },
  [LinkPreset.Friends]: {           // 增加 Friends 導航對應的路由
    name: i18n(I18nKey.friends),
    url: '/friends/',
  },
}
```

接下來要修改 i18n 相關參數以對應我們做的修改，修改 `src/i18n/i18nKey.ts`，新增 `friends`

```ts
enum I18nKey {
  home = 'home',
  about = 'about',
  archive = 'archive',
  search = 'search',
  friends = 'frineds',
  ...
}
```

修改 `src/i18n/languages/en.ts`

```ts
export const en: Translation = {
  [Key.home]: 'Home',
  [Key.about]: 'About',
  [Key.archive]: 'Archive',
  [Key.search]: 'Search',
  [Key.friends]: 'Friends',
  ...
}
```

修改 `src/i18n/languages/zh_TW.ts`

```ts
export const zh_TW: Translation = {
  [Key.home]: '首頁',
  [Key.about]: '關於',
  [Key.archive]: '彙整',
  [Key.search]: '搜尋',
  [Key.friends]: '朋友們',
}
```

複製 `src/pages/about.astro` 至 `src/pages/friends.astro`

```astro
---
import MainGridLayout from '../layouts/MainGridLayout.astro'

import { getEntry } from 'astro:content'
import Markdown from '@components/misc/Markdown.astro'
import I18nKey from '../i18n/i18nKey'
import { i18n } from '../i18n/translation'

// 重要的是下面三句
const friendsPost = await getEntry('spec', 'friends')       // 指定頁面進入點為 src/content/spec/，前面的參數名稱記得要改成 friendsPost

const { Content } = await friendsPost.render()              // 記得將 aboutPost 改成 friendsPost

// 下面的 i18nKey 也需要改成 friends
---
<MainGridLayout title={i18n(I18nKey.friends)} description={i18n(I18nKey.friends)}>
    <div class="flex w-full rounded-[var(--radius-large)] overflow-hidden relative min-h-32">
        <div class="card-base z-10 px-9 py-6 relative w-full ">
            <Markdown class="mt-2">
                <Content />
            </Markdown>
        </div>
    </div>
</MainGridLayout>
```

## 最後一步

在 `src/content/spec` 中新增 `friends.md`，內容就不打了。

這樣就大功告成了!