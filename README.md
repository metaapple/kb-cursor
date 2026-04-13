# Vue 3 가계부 앱 (kb-cursor)

원본 React/TypeScript 가계부 UI를 **Vue 3** 기반으로 옮긴 웹 애플리케이션입니다. 거래를 기록·조회하고, 요약 지표·차트로 흐름을 보여 주며, **금융 리터러시 퀴즈(게임)**로 학습 요소를 더했습니다. 백엔드 대신 **json-server**와 `db.json`으로 REST API를 흉내 내어 로컬에서 바로 동작하게 구성되어 있습니다.

---

## 목차

- [기능 개요](#기능-개요)
- [기술 스택](#기술-스택)
- [아키텍처 요약](#아키텍처-요약)
- [시작하기](#시작하기)
- [환경 변수](#환경-변수)
- [프로젝트 구조](#프로젝트-구조)
- [데이터 모델 (db.json)](#데이터-모델-dbjson)
- [라우팅·인증](#라우팅인증)
- [빌드·배포](#빌드배포)
- [알려진 제한](#알려진-제한)

---

## 기능 개요

### 인증·프로필

- **회원가입 / 로그인**: 이메일·비밀번호 기준. 가입 시 `users`에 저장되고, 로그인 성공 시 **공개 사용자 정보**(id, name, email, avatarUrl)만 `localStorage`에 유지합니다.
- **세션 복원**: 새로고침 후에도 로그인 상태를 유지합니다.
- **프로필**: 서버(`GET /users/:id`)에서 최신 이름·이메일을 다시 불러와 세션과 동기화할 수 있습니다.

### 대시보드

- **지표 카드**: 총 수입, 총 지출, 잔액(수입 − 지출), 흑자/적자 표시.
- **거래 목록**: 날짜 기준 정렬, **페이지네이션**, 항목 **추가·수정·삭제**.
- **계좌 카드**: 사용자별 계좌 목록 연동(스토어·API).
- **차트**
  - **월별 수입/지출**: Chart.js(`vue-chartjs`) 기반.
  - **Google Charts 패널**: 별도 컴포넌트로 확장 가능한 시각화 영역.
- **테마**: 라이트/다크 전환, `data-bs-theme` 및 `localStorage`에 저장.

### 금융 게임 (`/game`)

- **난이도 티어**(입문·중급·심화)별 객관식 문제로 가계부·소비 습관 개념을 학습합니다.
- **XP·레벨**: 정답 시 경험치 누적, 일정 XP마다 레벨 상승(로직은 Pinia `game` 스토어).
- **티어별 통계**: 시도·정답·라운드 등을 `localStorage`에 사용자별로 저장합니다.
- **맞춤 인사이트**: `financialGameInsights` 유틸이 거래 내역·게임 기록·티어 통계·XP를 묶어 개선 제안 문구를 생성합니다.
- 게임 **결과/이력**은 API(`gameResults` 등)와 연동되는 스토어가 있습니다.

### 기타

- `scripts/generate-db.mjs`: 시드/DB 생성 보조 스크립트.
- Axios **공통 클라이언트**: 개발·프로덕션별 `baseURL` 규칙이 주석과 코드에 정리되어 있습니다.

---

## 기술 스택

| 구분 | 사용 |
|------|------|
| UI 프레임워크 | Vue 3 (Composition API, `<script setup>`) |
| 빌드 | Vite 7 |
| 상태 관리 | Pinia |
| 라우팅 | Vue Router 4 |
| 스타일 | Tailwind CSS 4, PostCSS, **Bootstrap 5**, Bootstrap Icons |
| HTTP | Axios |
| 차트 | Chart.js, vue-chartjs |
| 로컬 API | json-server (`db.json`) |
| 병렬 실행 | concurrently (`dev:full`) |

---

## 아키텍처 요약

```text
브라우저 (Vue SPA)
    │  Axios (REST)
    ▼
json-server ← db.json  (users, transactions, accounts, gameResults 등)
```

- **영구 저장(서버)**: `db.json`에 반영되는 리소스는 json-server를 통해 CRUD.
- **클라이언트 전용**: 로그인 세션(`currentUser`), 다크 모드, 게임 XP/통계 등은 **`localStorage`**에 저장.

개발 시 API 기본 주소는 `src/api/client.js`에서 `VITE_API_BASE_URL` 또는 개발용 기본 호스트(`window.location.hostname` + 포트)로 결정됩니다. `package.json`의 `npm run api`는 **3001** 포트를 사용하므로, 기본 개발 URL과 맞지 않으면 [환경 변수](#환경-변수)로 맞춰 주세요.

프로덕션에서는 보통 **`/api`** 로 요청하고, Nginx 등에서 실제 백엔드로 프록시하는 방식을 가정합니다(Vite `preview`에도 `/api` 프록시 설정이 있습니다).

---

## 시작하기

### 요구 사항

- Node.js(프로젝트에 맞는 LTS 권장)
- npm 또는 yarn

### 설치

```bash
npm install
```

### 프론트만 실행 (API 없이 UI만 보려는 경우)

```bash
npm run dev
```

API가 없으면 로그인·목록 요청이 실패할 수 있습니다.

### API + 프론트 동시 실행 (권장)

```bash
npm run dev:full
```

- Vite 개발 서버와 json-server가 함께 뜹니다.
- json-server는 `db.json`을 감시하며 **포트 3001**에서 동작합니다(`package.json`의 `api` 스크립트).

API 주소가 어긋나면 프로젝트 루트에 `.env.development`를 두고 아래를 설정하세요.

### 기타 스크립트

```bash
npm run api          # json-server만 (포트 3001)
npm run build        # 프로덕션 빌드 → dist/
npm run preview      # 빌드 결과 미리보기
```

---

## 환경 변수

| 변수 | 설명 |
|------|------|
| `VITE_API_BASE_URL` | API 베이스 URL. 미설정 시 개발에서는 코드에 정의된 기본값, 프로덕션에서는 `/api`를 사용합니다. json-server를 **3001**에 둔 경우 예: `http://127.0.0.1:3001` |

`.env`, `.env.development` 등은 저장소에 올리지 마세요(`.gitignore`에 포함).

---

## 프로젝트 구조

```text
vueapp/
├── public/                 # 정적 자산
├── scripts/                # DB 생성 등 보조 스크립트
├── src/
│   ├── api/client.js       # Axios 인스턴스, baseURL 규칙
│   ├── components/         # 카드, 모달, 차트, 목록 등 UI 조각
│   ├── composables/        # 예: Google Charts 로더
│   ├── router/index.js     # 라우트·네비게션 가드
│   ├── stores/             # Pinia: auth, transactions, accounts, theme, game, gameResults …
│   ├── utils/              # 예: financialGameInsights
│   ├── views/              # Login, Signup, Dashboard, Game
│   ├── App.vue
│   ├── main.js
│   └── style.css
├── db.json                 # json-server 데이터 시드
├── vite.config.js          # 별칭 @ → src, /api 프록시(프리뷰·개발)
├── package.json
└── README.md
```

---

## 데이터 모델 (db.json)

- **`users`**: id, name, email, password, avatarUrl 등. 데모용 평문 비밀번호이므로 **실서비스에 그대로 쓰면 안 됩니다.**
- **`transactions`**: userId, date, category, description, amount, type(`income` | `expense`).
- **계좌·게임 결과** 등 다른 컬렉션은 스토어·API 사용처에 맞게 확장되어 있습니다.

기본 데모 계정 예시는 `db.json`의 `users`를 참고하세요(예: `demo@example.com`).

---

## 라우팅·인증

| 경로 | 설명 |
|------|------|
| `/` | `/dashboard`로 리다이렉트 |
| `/login`, `/signup` | 비로그인 전용(`guestOnly`). 이미 로그인 시 대시보드로 보냄 |
| `/dashboard`, `/game` | 로그인 필요(`requiresAuth`) |

`router.beforeEach`에서 `authStore.restoreSession()` 후 위 규칙을 적용합니다.

---

## 빌드·배포

```bash
npm run build
```

산출물은 `dist/`입니다. 정적 호스팅 시 **API는 별도 서버** 또는 **경로 프록시**로 제공해야 하며, 클라이언트의 `VITE_API_BASE_URL` 또는 `/api` 프록시 설정과 일치시켜야 합니다.

---

## 알려진 제한

- **json-server + 평문 비밀번호**는 개발·데모용입니다. 실제 서비스에는 해시·HTTPS·토큰 기반 인증 등이 필요합니다.
- 로그인 시 이메일로 사용자를 조회한 뒤 클라이언트에서 비밀번호를 비교하는 방식은 **교육/로컬 목적**에 가깝습니다.
- 브라우저·포트·프록시 조합에 따라 API URL을 환경 변수로 조정해야 할 수 있습니다.

---

## 라이선스·기여

개인/팀 프로젝트에 맞게 `package.json`의 `license` 필드와 저장소 정책을 설정하세요.
