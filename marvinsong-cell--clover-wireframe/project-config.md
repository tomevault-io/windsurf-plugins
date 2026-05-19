---
trigger: always_on
description: 프리미엄 소개팅 서비스 클로버의 웹앱 프로젝트입니다.
---

# Clover - 소개팅 웹앱

프리미엄 소개팅 서비스 클로버의 웹앱 프로젝트입니다.

## Tech Stack

- **Frontend:** React 18 + TypeScript + Vite
- **Backend:** Supabase (Auth, Database, Storage, Realtime)
- **Styling:** Vanilla CSS with CSS Custom Properties
- **State:** Zustand (클라이언트) + React Query (서버)
- **Font:** Pretendard

## Project Structure

```
src/
├── app/                      # 앱 진입점
│   ├── App.tsx              # 루트 컴포넌트
│   ├── Router.tsx           # 라우팅 설정
│   └── providers/           # Context Providers
│       ├── AuthProvider.tsx
│       ├── QueryProvider.tsx
│       └── index.tsx
│
├── pages/                    # 페이지 컴포넌트 (라우트 단위)
│   ├── auth/                # 인증 관련
│   │   ├── LoginPage.tsx
│   │   ├── SignupPage.tsx
│   │   └── OnboardingPage.tsx
│   ├── main/                # 메인 앱
│   │   ├── FeedPage.tsx     # 매칭 피드
│   │   ├── ChatPage.tsx     # 채팅 목록
│   │   ├── ChatRoomPage.tsx # 채팅방
│   │   ├── MatchesPage.tsx  # 매칭 목록
│   │   └── ProfilePage.tsx  # 내 프로필
│   └── profile/             # 프로필 관련
│       ├── EditProfilePage.tsx
│       └── ViewProfilePage.tsx
│
├── components/               # 재사용 컴포넌트
│   ├── ui/                  # 기본 UI 컴포넌트 (디자인 시스템)
│   │   ├── Button/
│   │   ├── Input/
│   │   ├── Card/
│   │   └── index.ts
│   ├── layout/              # 레이아웃 컴포넌트
│   │   ├── Header.tsx
│   │   ├── BottomNav.tsx
│   │   ├── MobileFrame.tsx
│   │   └── Screen.tsx
│   └── features/            # 기능별 컴포넌트
│       ├── auth/
│       ├── matching/
│       ├── chat/
│       └── profile/
│
├── lib/                      # 라이브러리 & 유틸리티
│   ├── supabase/            # Supabase 클라이언트
│   │   ├── client.ts
│   │   ├── auth.ts
│   │   ├── database.ts
│   │   └── storage.ts
│   ├── hooks/               # 커스텀 훅
│   │   ├── useAuth.ts
│   │   ├── useProfile.ts
│   │   ├── useMatching.ts
│   │   └── useChat.ts
│   ├── api/                 # API 함수 (React Query용)
│   │   ├── auth.ts
│   │   ├── profiles.ts
│   │   ├── matching.ts
│   │   └── chat.ts
│   └── utils/               # 유틸리티 함수
│       ├── format.ts
│       ├── validation.ts
│       └── constants.ts
│
├── stores/                   # Zustand 스토어
│   ├── authStore.ts
│   ├── uiStore.ts
│   └── chatStore.ts
│
├── types/                    # TypeScript 타입 정의
│   ├── database.ts          # Supabase 테이블 타입
│   ├── auth.ts
│   ├── profile.ts
│   ├── matching.ts
│   └── chat.ts
│
├── styles/                   # 글로벌 스타일
│   ├── globals.css          # 전역 스타일 & CSS 변수
│   ├── reset.css            # CSS 리셋
│   └── animations.css       # 애니메이션
│
└── assets/                   # 정적 에셋
    ├── icons/
    └── images/

# 와이어프레임 (현재)
/wireframe
├── index.html
├── styles.css
├── App.jsx
└── components/
    ├── DesignSystem.jsx
    ├── Onboarding.jsx
    ├── PhoneAuth.jsx
    ├── BasicInfo.jsx
    ├── CloverInfo.jsx
    └── Complete.jsx
```

---

# Architecture & Design Patterns

## 1. Supabase Integration Pattern

### 클라이언트 초기화

```typescript
// lib/supabase/client.ts
import { createClient } from '@supabase/supabase-js';
import type { Database } from '@/types/database';

export const supabase = createClient<Database>(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
);
```

### Auth 패턴

```typescript
// lib/supabase/auth.ts
export const authApi = {
  // 휴대폰 인증
  signInWithPhone: async (phone: string) => {
    return supabase.auth.signInWithOtp({ phone });
  },

  // OTP 검증
  verifyOtp: async (phone: string, token: string) => {
    return supabase.auth.verifyOtp({ phone, token, type: 'sms' });
  },

  // 로그아웃
  signOut: async () => {
    return supabase.auth.signOut();
  },

  // 세션 가져오기
  getSession: async () => {
    return supabase.auth.getSession();
  },

  // 인증 상태 구독
  onAuthStateChange: (callback: (event, session) => void) => {
    return supabase.auth.onAuthStateChange(callback);
  }
};
```

### Database 패턴

```typescript
// lib/api/profiles.ts
export const profilesApi = {
  // 프로필 조회
  getProfile: async (userId: string) => {
    const { data, error } = await supabase
      .from('profiles')
      .select('*')
      .eq('user_id', userId)
      .single();

    if (error) throw error;
    return data;
  },

  // 프로필 업데이트
  updateProfile: async (userId: string, updates: ProfileUpdate) => {
    const { data, error } = await supabase
      .from('profiles')
      .update(updates)
      .eq('user_id', userId)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // 매칭 피드 조회
  getMatchingFeed: async (userId: string, filters: MatchFilters) => {
    const { data, error } = await supabase
      .rpc('get_matching_feed', {
        current_user_id: userId,
        ...filters
      });

    if (error) throw error;
    return data;
  }
};
```

### Realtime 패턴 (채팅)

```typescript
// lib/hooks/useChat.ts
export const useRealtimeChat = (roomId: string) => {
  const [messages, setMessages] = useState<Message[]>([]);

  useEffect(() => {
    // 기존 메시지 로드
    loadMessages();

    // 실시간 구독
    const channel = supabase
      .channel(`room:${roomId}`)
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'messages',
          filter: `room_id=eq.${roomId}`
        },
        (payload) => {
          setMessages(prev => [...prev, payload.new as Message]);
        }
      )
      .subscribe();


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marvinsong-cell/clover-wireframe](https://github.com/marvinsong-cell/clover-wireframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
