# 카카오 이모티콘 매니저 — Claude Code 컨텍스트

## 프로젝트 배경
엄마(비개발자)의 카카오 이모티콘 제작/출시 작업을 돕기 위한 웹앱.
아이패드 + 애플펜슬 + Procreate로 이모티콘을 제작하고,
이 웹앱으로 제출/심사/판매 전 과정을 관리한다.

**핵심 원칙: 엄마가 직접 쓰는 앱 → UI는 최대한 단순하고 직관적으로**

---

## 기술 스택
- **Frontend**: HTML/CSS/JS (PWA, 아이패드 Safari 최적화, 단일 파일 구조)
- **Database**: Supabase (프로젝트 ID: `nyxcgkhwiwfgvofioqyh`, 서울 리전 ap-northeast-2)
- **AI**: Claude API (`claude-sonnet-4-20250514`) — 이모티콘 이미지 분석/피드백
- **배포**: Netlify
- **스타일**: 카카오 옐로우(`#FEE500`) 기반 디자인 시스템, 한국어 UI 전용

---

## Supabase 테이블 구조

### `series` — 이모티콘 시리즈
```sql
id            uuid primary key
title         text not null          -- 시리즈명
status        text default 'drafting' -- drafting | submitted | reviewing | approved | rejected
submitted_at  timestamptz
memo          text                   -- 카카오 연락 메모
rejection_count int default 0
created_at    timestamptz default now()
```

### `emoticons` — 개별 이모티콘 시안
```sql
id          uuid primary key
series_id   uuid references series(id)
image_url   text                     -- Supabase Storage URL
ai_feedback text                     -- Claude AI 분석 결과 (JSON)
created_at  timestamptz default now()
```

### `revenue` — 월별 수익 정산
```sql
id           uuid primary key
month        text not null           -- 'YYYY-MM' 형식
gross_amount int default 0           -- 총 수익 (원)
kakao_fee    int default 0           -- 카카오 수수료
tax          int default 0           -- 세금
net_amount   int default 0           -- 실 수령액
note         text
created_at   timestamptz default now()
```

### Storage bucket: `emoticons`
- PNG/GIF 파일, 5MB 제한
- Public URL 사용

---

## 핵심 기능 (5개 탭)

1. **🏠 홈** — 전체 현황 통계, 최근 시리즈, 오늘의 심사 팁
2. **📁 시리즈** — 시리즈 CRUD, 상태 추적 (drafting→submitted→reviewing→approved/rejected), 카카오 연락 메모
3. **✅ 체크리스트** — 제출 전 12개 항목 체크 (파일 규격, 금지 요소 등)
4. **🤖 AI 분석** — 이모티콘 이미지 업로드 → Claude API로 카카오 심사 기준 피드백
5. **💰 수익** — 월별 정산 기록, 수수료/세금 자동 계산

---

## 아이패드 최적화 필수 사항
- 최소 터치 타겟: 44px 이상
- 폰트 크기: 최소 16px (Safari 자동 줌 방지)
- `-webkit-tap-highlight-color: transparent`
- Safe area inset 처리 (`env(safe-area-inset-*)`)
- PWA manifest 포함 (`standalone` 모드)
- 가로/세로 모두 대응

---

## 파일 구조
```
kakao-emoticon-manager/
├── CLAUDE.md          ← 이 파일 (Claude Code 컨텍스트)
├── index.html         ← 메인 앱 (단일 HTML 파일)
├── manifest.json      ← PWA 매니페스트
├── sw.js              ← Service Worker (오프라인 대응)
├── icons/             ← PWA 아이콘
│   ├── icon-192.png
│   └── icon-512.png
└── netlify.toml       ← Netlify 배포 설정
```

---

## 개발 시 주의사항
- Supabase anon key는 `index.html` 내 JS 상단에 상수로 선언
- Claude API key는 사용자가 앱 설정에서 직접 입력 후 localStorage 저장 (보안)
- 모든 에러는 한국어 토스트 메시지로 표시
- 로딩 상태 반드시 표시 (엄마가 "멈춘 건지" 헷갈리지 않도록)
- 삭제 기능은 반드시 확인 다이얼로그 포함

---

## 카카오 이모티콘 규격 (체크리스트 기준)
- 이모티콘 수: 24개 (멈춰있는 이모티콘 기준)
- 파일 형식: PNG (투명 배경)
- 사이즈: 360×360px
- 용량: 개당 최대 500KB
- 금지: 타 브랜드 로고, 실존 인물, 폭력/선정적 표현
- 필수: 메인 이미지 1장(360×360), 탭 이미지 1장(130×130)
