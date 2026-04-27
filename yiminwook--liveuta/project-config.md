---
trigger: always_on
description: ├── .cursor                      # cusror rules 폴더
---

## 프로젝트 구조: 주요 폴더 구조예시
project-root/
│
├── .cursor                      # cusror rules 폴더
├── .vscode                      # vsCode setting 폴더
├── analyze                      # next-bundle-analyzer 결과 폴더
├── e2e                          # e2e - playwright 테스트 코드 폴더
├── messages                     # i18next 번역 json 폴더
│   ├── en.json
│   ├── ja.json
│   ├── ko.json
│   └── README.md
├── public                       # public static assets 폴더
│   ├── assets
│   │   └── meta-image.png
│   ├── mockServiceWorker.js
│   ├── sw.js
│   └── theme.css
├── src           
│   ├── apis                       # client and server side fetch 함수 폴더
│   │   ├── cached.ts              # react.cache 캐시 모음
│   │   ├── fetcher.ts             # ky 라이브러리 fetch 함수 모음  
│   │   └── getQueryClient.ts      # react-query client
│   ├── app                        # app router 폴더
│   │   ├── [locale]               # 일반 유저들이 접근가능한 페이지 폴더
│   │   │   ├── [...catchAll]      # not-found catch all segments
│   │   │   │   └── page.tsx
│   │   │   ├── sentry-test
│   │   │   │   └── page.tsx
│   │   │   ├── forbidden.tsx   
│   │   │   ├── layout.tsx
│   │   │   ├── not-found.tsx
│   │   │   └── unauthorized.tsx
│   │   ├── admin                  # 관리자 페이지 폴더
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   │       
│   │   ├── api                   # api route handler 폴더
│   │   │   ├── auth
│   │   │   │   └── [...nextauth] # next-auth route handlers
│   │   │   │       └── route.ts
│   │   │   └── v1                # api version 
│   │   │       ├── blacklist
│   │   │       │   └── route.ts
│   │   │       ├── channel
│   │   │       │   └── route.ts
│   │   │       ├── featured
│   │   │       │   └── route.ts
│   │   │       ├── member
│   │   │       │   └── route.ts
│   │   │       ├── metadata
│   │   │       │   └── route.ts
│   │   │       ├── revalidate
│   │   │       │   └── route.ts
│   │   │       ├── schedule
│   │   │       │   └── route.ts
│   │   │       ├── setlist
│   │   │       │   └── route.ts
│   │   │       ├── whitelist
│   │   │       │   └── route.ts
│   │   │       └── youtube-channel
│   │   │           └── route.ts
│   │   ├── favicon.ico
│   │   ├── global-error.tsx
│   │   ├── layout.tsx
│   │   ├── manifest.ts
│   │   ├── page.tsx
│   │   ├── robots.ts
│   │   └── sitemap.ts
│   ├── assets                               # private static assets 폴더
│   │   └── image
│   ├── components                           # 컴포넌트 폴더 도메인 별로 구분
│   │   ├── common                           # 공통 컴포넌트 폴더
│   │   │   ├── authorization                # client side 세션관리 컴포넌트 폴더
│   │   │   │   ├── Administrator.tsx
│   │   │   │   ├── Authorized.tsx
│   │   │   │   ├── UnAuthorized.tsx
│   │   │   │   └── withSession.tsx
│   │   │   ├── button
│   │   │   │   ├── ClearButton.tsx
│   │   │   │   ├── CopyButton.tsx
│   │   │   │   ├── HamburgerBtn.module.scss
│   │   │   │   ├── HamburgerBtn.tsx
│   │   │   │   ├── MoreButton.module.scss
│   │   │   │   ├── MoreButton.tsx
│   │   │   │   ├── PasteButton.tsx
│   │   │   │   └── ToggleButton.tsx
│   │   │   ├── channelCard
│   │   │   │   ├── ChannelCard.module.scss
│   │   │   │   └── ChannelCard.tsx
│   │   │   ├── command
│   │   │   │   ├── CommandMenu.module.scss
│   │   │   │   ├── CommandMenu.tsx
│   │   │   │   ├── Context.tsx
│   │   │   │   └── GlobalCmd.tsx
│   │   │   ├── DataFetchingObserver.tsx
│   │   │   ├── Footer.module.scss
│   │   │   ├── Footer.tsx
│   │   │   ├── header
│   │   │   │   ├── DesktopNav.tsx
│   │   │   │   ├── Header.module.scss
│   │   │   │   ├── Header.tsx
│   │   │   │   ├── HeaderMenu.module.scss
│   │   │   │   └── HeaderMenu.tsx
│   │   │   ├── Iframe.module.scss
│   │   │   ├── Iframe.tsx
│   │   │   ├── input
│   │   │   │   ├── SearchInput.module.scss
│   │   │   │   └── SearchInput.tsx
│   │   │   ├── loading
│   │   │   │   ├── GlobalLoading.module.scss
│   │   │   │   ├── Loading.module.scss
│   │   │   │   ├── loading.tsx
│   │   │   │   ├── MainLoading.tsx
│   │   │   │   ├── RingLoader.tsx
│   │   │   │   ├── SquareToRound.tsx
│   │   │   │   └── Wave.tsx
│   │   │   ├── modal
│   │   │   │   ├── AlertModal.tsx
│   │   │   │   ├── ChannelCardModal.module.scss
│   │   │   │   ├── ChannelCardModal.tsx
│   │   │   │   ├── ConfirmModal.tsx
│   │   │   │   ├── ErrorModal.tsx
│   │   │   │   ├── Modal.module.scss
│   │   │   │   └── Modal.tsx
│   │   │   ├── Nodata.module.scss
│   │   │   ├── Nodata.tsx
│   │   │   ├── PageView.tsx
│   │   │   ├── player
│   │   │   │   ├── DndComponents.module.scss
│   │   │   │   ├── DndComponents.tsx
│   │   │   │   ├── GlobalPip.tsx
│   │   │   │   ├── LiveChat.tsx
│   │   │   │   ├── LiveChatPlayer.tsx
│   │   │   │   ├── Player.module.scss
│   │   │   │   ├── PlayerBase.tsx
│   │   │   │   └── PlayerPlaceholder.tsx
│   │   │   ├── scheduleCard
│   │   │   │   ├── Card.module.scss
│   │   │   │   ├── Card.tsx
│   │   │   │   ├── CardDesc.tsx
│   │   │   │   ├── CardImage.tsx
│   │   │   │   ├── CardMenu.module.scss
│   │   │   │   ├── CardMenu.tsx
│   │   │   │   ├── CardStatus.tsx
│   │   │   │   ├── CardViewer.tsx
│   │   │   │   ├── ScheduleCardSkeleton.module.scss
│   │   │   │   ├── ScheduleCardSkeleton.tsx
│   │   │   │   ├── SliderCard.module.scss
│   │   │   │   ├── SliderCard.tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yiminwook) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
