---
layout: post
title: "Next.js 15 + Supabase + PWA로 구축하는 오프라인 우선 하이브리드 웹앱"
date: 2026-02-13
categories: [DevLog, Nextjs]
tags: [PWA, Supabase, Vercel, TailwindCSS, Sonner, Offline-First]
---

# 🚀 기회비용 계산기(OP-CO-CA) 제작기

앱처럼 설치 가능한 하이브리드 웹앱을 구축한 과정을 정리합니다.

---

## 1. 초기 환경 구축 (Next.js & shadcn)

최신 Next.js 15와 UI 라이브러리인 shadcn을 활용하여 기초를 잡았습니다.

```bash
# 1. Next.js 프로젝트 생성
npx create-next-app@latest op-co-ca --typescript --tailwind --eslint

# 2. shadcn 초기화 및 필수 컴포넌트 추가 (2025년 부터 shadcn-ui가 아닌 shadcn으로 입력해야 됩니다.)
npx shadcn@latest init
npx shadcn@latest add button input card textarea

# 3. 가볍고 강력한 토스트 라이브러리 'sonner' 설치(toast는 shadcn에서 더 이상 지원하지 않습니다. 대신 sonner를 사용합시다.)
npm install sonner
```

---

## 2. 백엔드 연동 (Supabase)

공유 기능을 위해 Supabase를 활용했습니다. `.env.local`에 환경 변수를 세팅하는 것이 첫 단계입니다.

```env
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
```

---

## 3. 핵심 아키텍처: Offline-First 하이브리드 전략

이 프로젝트의 핵심은 **서버 데이터와 로컬 데이터의 공존**입니다.



### 💡 주요 인사이트
1. **실시간 상태 감지**: 클라이언트 컴포넌트(`OfflineBanner`)를 통해 `navigator.onLine` 상태를 체크하고, 연결이 끊기면 즉시 사용자에게 알립니다.
2. **저장 로직 분기**: 
   - **온라인**: Supabase DB에 저장하여 전 세계와 공유.
   - **오프라인/프라이빗**: `localStorage`에만 저장하여 개인 데이터 보안 유지.
3. **ID 체계 분리**: 로컬 데이터에는 `local_` 접두사를 붙여 서버 DB와 충돌하거나 UUID 형식 에러가 나는 것을 원천 차단했습니다.

---

## 4. PWA 설정 (설치 가능한 웹)

`@ducanh2912/next-pwa` 라이브러리를 사용하여 브라우저 주소창에서 바로 설치 가능한 환경을 만들었습니다.

```javascript
// next.config.mjs 설정 예시
import withPWAInit from "@ducanh2912/next-pwa";

const withPWA = withPWAInit({
  dest: "public",
  disable: process.env.NODE_ENV === "development", // 개발 환경에선 끄기
});

export default withPWA({
  // Next.js config...
});
```

---

## 5. 배포 (Vercel & GitHub)

GitHub 저장소와 Vercel을 연동하여 CI/CD 환경을 구축했습니다.

1. **Vercel**: 환경 변수 등록 및 HTTPS 도메인 배포 (PWA는 보안을 위해 HTTPS가 필수입니다!)
2. **Sonner 연동**: 저장 성공/실패 메시지를 `sonner`를 통해 구현했습니다.

