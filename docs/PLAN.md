# 혼자쓰는앱 (app-for-one) 기획서

> 나 한 명만 쓰는 인생 CRM. 건강, 목표, 하루, 사람, 돈을 한 곳에서 기록하고 돌아본다.
>
> - 작성일: 2026-10-04
> - 상태: 초안 (v0.1)
> - 사용자: 나 1명

---

## 1. 왜 만드나

- 회사에서 직접 설계하고 만든 **건강기록 앱(식단·수분·체중·수면·운동)** 의 UX가 내 생활에도 잘 맞았다. 회사 서비스에 묶인 기능을 **개인용으로 떼어** 계속 쓰고 싶다.
- 건강만이 아니라 **목표, 회고, 사람, 돈** 등 인생 전반을 한 앱에서 관리하고 싶다. 노션이나 시트로는 기록하기가 귀찮아서 오래 못 간다.
- 폰에서 3초 만에 기록하고, 큰 화면(웹)에서 돌아보고 계획한다.

## 2. 원칙

| 원칙 | 의미 |
| --- | --- |
| **사용자는 나 1명** | 온보딩, 다국어, 권한, 결제, 소셜 기능 없음. 대신 내 습관에 딱 맞춘다. |
| **기록은 3초** | 앱을 열면 오늘 화면이 바로 보이고, 탭 한두 번으로 기록이 끝난다. 상세 입력은 선택. |
| **앱과 웹 동시** | iOS, Android, Web을 같은 코드로 만든다. 폰은 기록 위주, 웹은 회고와 계획 위주. |
| **데이터는 내 것** | 표준 Postgres와 순수 SQL을 쓰고, 언제든 덤프해서 다른 백엔드(Java)로 옮길 수 있게 한다. |
| **작게, 단계적으로** | Phase 1은 건강기록과 목표·계획만. 나머지는 필요해질 때 붙인다. |
| **public 레포** | 시크릿과 개인 데이터는 절대 커밋하지 않는다. 코드와 문서만 공개한다. |

## 3. 로드맵

| Phase | 영역 | 내용 |
| --- | --- | --- |
| **M0** | 기반 | 모노레포 세팅, RN 앱과 웹 셸, Supabase 연결, 로그인, CI |
| **Phase 1** | 🩺 건강기록 | 식사, 수분, 체중, 수면, 운동 기록 + 오늘 홈 + 캘린더 + 추이 리포트 |
| **Phase 1** | 🎯 목표·계획 | 연·분기·월 목표, 할 일, 건강 지표와 연동된 자동 진척도 |
| Phase 1.x | 🩺 확장 | 식단 사진 AI 추론, 식사 리마인더 알림, HealthKit/Health Connect 연동 |
| Phase 2 | 📓 일기·회고 | 하루 한 줄, 주간·월간 회고 (목표 리뷰와 연결) |
| Phase 3 | 👥 사람 관리 | 지인 목록, 연락 주기, 만남 기록, 생일·기념일 |
| Phase 4 | 💰 돈·자산 | 지출, 구독, 자산 스냅샷 |
| 언젠가 | 🏗 백엔드 이전 | Supabase를 Spring Boot(Java)와 자체 Postgres로 이전 (§8) |

---

## 4. Phase 1 기능 상세

### 4.1 오늘 (홈)

- **주간 날짜 스트립**: 좌우로 스와이프해서 날짜를 옮긴다. 각 날짜에 **기록 링**이 붙는다.
  - 기록 링은 하루를 **8칸**(아침, 점심, 저녁, 간식, 수분, 체중, 수면, 운동)으로 나누고, 채운 칸만큼 호를 그린다.
  - 판정 규칙은 `packages/core`의 **한 함수**로만 계산한다. 화면마다 계산이 달라서 링이 어긋나는 일을 막기 위해서다.

    | 칸 | 채워지는 조건 |
    | --- | --- |
    | 아침, 점심, 저녁, 간식 | 해당 끼니 기록 1건 이상, 또는 "건너뜀" 체크 |
    | 수분 | 섭취량 > 0 |
    | 체중 | 기상 후 또는 취침 전 체중 중 하나 |
    | 수면 | 취침 시각과 기상 시각이 둘 다 있음 |
    | 운동 | 운동 1건 이상, 또는 "쉬었어요" |

- **위젯형 홈**: 식사, 수분, 체중, 수면, 운동 위젯과 **오늘의 목표·할 일** 위젯. 위젯마다 그 자리에서 바로 기록할 수 있다.
- 오늘 할 일 상단 노출: 오늘 마감이거나 이번 주 목표에 묶인 할 일.

### 4.2 건강기록

| 항목 | MVP | 상세 (선택 입력) |
| --- | --- | --- |
| **식사** | 끼니(아침/점심/저녁/간식), 시각, 메뉴명, 사진 | 메뉴별 종류(음식/음료/술), 양(포션 슬라이더), 포만감, 장소, 함께한 사람, 식사 시간, 메모, 건너뜀 |
| **수분** | +/- 버튼 (1잔 = 250ml, 설정 가능) | 하루 목표량 |
| **체중** | 기상 후 / 취침 전 kg | 메모 |
| **수면** | 취침 시각, 기상 시각 | 수면 질 (1~5) |
| **운동** | 종류, 시간(분), 강도 | 사진, 메모, 사용자 정의 운동 종류, "쉬었어요" 토글 |

- 식사 사진: 모바일은 카메라와 앨범(크롭 포함), 웹은 파일 업로드와 드래그앤드롭을 쓴다. 사진은 Supabase Storage에 저장한다.
- 날짜와 시각 입력은 휠 피커(모바일)와 기본 입력(웹)으로 나눠서 구현한다.

### 4.3 캘린더

- 월 달력에 날짜별 기록 링을 표시한다. 날짜를 누르면 그날 홈으로 이동한다.
- 월 단위 데이터는 **쿼리 1번**으로 가져온다. 날짜마다 API를 부르지 않는다.

### 4.4 리포트 (추이)

- 지표: 체중, 수면 시간, 수분, 운동 시간
- 기간: 주, 월, 연. 라인과 바 차트는 `react-native-svg`로 직접 그린다 (웹 호환).
- 평균, 최고, 최저, 지난 기간 대비 증감.

### 4.5 목표·계획

**구조**

```
목표 (Goal)
 ├─ 기간 단위: 연 / 분기 / 월 / 주
 ├─ 상위 목표 (parent) → 연 목표 아래 분기 목표 … 트리
 ├─ 진척 방식: 수동 체크인 | 자동 지표(건강기록 연동) | 하위 할 일 완료율
 └─ 할 일 (Task) 여러 개
```

- **자동 지표 연동**은 이 앱만의 핵심 차별점이다.
  - "올해 말 체중 70kg" → 체중 기록으로 자동 진척
  - "주 3회 운동" → 운동 기록 횟수로 자동 계산
  - "하루 물 2L" → 수분 기록 달성일 비율
  - MVP 지표: `WEIGHT`, `EXERCISE_COUNT`, `EXERCISE_MINUTES`, `WATER_DAYS`, `SLEEP_HOURS`
- **할 일**: 제목, 마감일, 연결된 목표(선택), 완료 처리. 반복 할 일은 Phase 1.x.
- **목표 화면**: 기간 탭(올해, 이번 분기, 이번 달, 이번 주)마다 목표 카드와 진척 바를 보여 준다.
- **주간 리뷰** (웹 위주): 이번 주 목표 달성률, 건강 요약, 다음 주 할 일 정리. Phase 2 회고와 연결할 자리다.

### 4.6 Phase 1에서 하지 않는 것

- CGM(연속혈당), 체성분 등 외부 기기 연동
- 설문 기반 점수, 등급 리포트
- 다국어, 다중 사용자, 공유
- 오프라인 우선 동기화 (온라인 전제, 실패 시 재시도만)

---

## 5. 기술 스택

| 영역 | 선택 | 이유 |
| --- | --- | --- |
| 앱 | **React Native (bare CLI)** + TypeScript | 이미 익숙한 스택이다. 네이티브 모듈을 제약 없이 쓸 수 있다. |
| 웹 | **react-native-web** + Vite | 같은 RN 컴포넌트로 웹을 렌더링한다. 웹은 정적 빌드로 배포한다. |
| 네비게이션 | React Navigation v7 | 앱과 웹 모두 지원한다. 웹은 `linking` 설정으로 URL 라우팅. |
| 서버 상태 | **TanStack Query** | 날짜별 캐시, 중복 요청 합류, 쓰기 후 무효화를 직접 짜지 않아도 된다. |
| 클라이언트 상태 | Zustand | 선택 날짜, UI 상태 등 |
| 폼·검증 | react-hook-form + **zod** | zod 스키마를 core에 두고 앱과 백엔드가 공유한다. |
| 스타일 | `StyleSheet` + 디자인 토큰 (`packages/ui`) | 웹에서도 동작한다. 반응형은 `useWindowDimensions`로 처리. |
| 차트 | react-native-svg 직접 구현 | 웹 호환, 의존성 최소 |
| 날짜 | dayjs | |
| 백엔드 | **Supabase** (Postgres, Auth, Storage, Edge Functions) | §8 이전 전략을 전제로 선택했다. |
| 테스트 | Vitest (core), Jest (RN) | |
| CI | GitHub Actions: typecheck, lint, test | |
| 패키지 매니저 | **npm workspaces** | pnpm 심볼릭 링크는 Metro와 RN autolinking에서 자주 깨진다. 호이스팅되는 npm이 가장 무난하다. |

### 플랫폼별 분기

`*.native.ts(x)` / `*.web.ts(x)` 확장자로 나눈다.

| 기능 | 모바일 | 웹 |
| --- | --- | --- |
| 사진 | react-native-image-picker, crop-picker | `<input type="file">`, 드래그앤드롭 |
| 로컬 저장 (세션 등) | MMKV | localStorage |
| 햅틱 | react-native-haptic-feedback | no-op |
| 알림 (1.x) | notifee 로컬 알림 | 없음 |
| 휠 피커 | 커스텀 휠 | 기본 `<input type="time/date">` |

---

## 6. 레포 구조 — 모노레포로 간다

**왜 모노레포인가:** 모바일 셸과 웹 셸이 화면 코드를 공유해야 하고, DB 스키마(`supabase/`)와 나중에 붙을 Java 백엔드(`apps/api`)까지 한 레포에서 같이 버전 관리하는 게 혼자 하기에 가장 편하다. 대신 도구는 최소로만 쓴다 (npm workspaces만, Turborepo는 필요해지면 추가).

```
app-for-one/
├─ apps/
│  ├─ mobile/            # RN CLI 셸 — ios/, android/, index.js, metro.config.js
│  ├─ web/               # Vite 셸 — index.html, vite.config.ts (react-native → react-native-web alias)
│  └─ api/               # (나중) Spring Boot
├─ packages/
│  ├─ app/               # 화면, 기능, 네비게이션 — 앱과 웹이 공유하는 본체
│  │  └─ src/features/{today,health,calendar,report,goals}/
│  ├─ ui/                # 디자인 토큰 + 기본 컴포넌트 (Text, Button, Card, Sheet …)
│  └─ core/              # 순수 TS: 도메인 타입, zod 스키마, 판정 로직(기록 링), repository 인터페이스와 구현
├─ supabase/
│  ├─ migrations/        # 순수 SQL 마이그레이션 (나중에 Flyway로 그대로 이전)
│  ├─ functions/         # Edge Functions (1.x: 식단 AI 추론)
│  └─ seed.sql
├─ docs/
│  └─ PLAN.md            # 이 문서
├─ .github/workflows/
├─ .env.example
└─ package.json          # workspaces
```

**의존 방향:** `apps/*` → `packages/app` → `packages/ui`, `packages/core`. `core`는 RN을 import하지 않는다 (나중에 Node나 다른 클라이언트에서도 쓸 수 있게).

**모노레포 + RN CLI 주의점**

- `metro.config.js`: `watchFolders`에 레포 루트를 넣고, `resolver.nodeModulesPaths`에 루트 `node_modules`를 넣는다.
- `react`, `react-native`는 루트에 한 벌만 있게 고정한다 (중복 설치 시 hooks 에러).
- Vite: node_modules 안의 JSX `.js` 파일 처리(`optimizeDeps.esbuildOptions.loader`), `__DEV__` define, 일부 RN 라이브러리용 alias가 필요하다. **M0에서 가장 먼저 검증할 리스크다.**

---

## 7. 데이터 모델 (초안)

모든 테이블에 공통 규칙을 적용한다. PK는 `uuid`, `user_id uuid not null`, 시각은 `created_at/updated_at timestamptz`, 이름은 snake_case.

```sql
-- 프로필 (1행)
profiles (id uuid pk = auth uid, display_name, timezone, height_cm, birth_date,
          water_cup_ml default 250, water_goal_ml default 2000)

-- 🩺 건강기록
meals        (id, user_id, date, meal_type  -- BREAKFAST|LUNCH|DINNER|SNACK
              , eaten_at time, skipped bool, fullness, place, companion, duration, memo)
meal_items   (id, meal_id fk, name, kind -- FOOD|DRINK|ALCOHOL
              , portion numeric, sort_order)
meal_photos  (id, meal_id fk, storage_path, taken_at, sort_order)

daily_logs   (user_id, date, pk(user_id, date),   -- 하루 1행
              water_ml int default 0,
              weight_morning_kg numeric, weight_night_kg numeric,
              slept_at timestamptz, woke_at timestamptz, sleep_quality smallint,
              rested bool default false)          -- "쉬었어요"

exercise_types (id, user_id, name, is_custom)
exercises      (id, user_id, date, exercise_type_id fk, started_at time,
                duration_min int, intensity -- LOW|MEDIUM|HIGH|VERY_HIGH
                , memo)

-- 🎯 목표·계획
goals        (id, user_id, parent_id fk null, title, why text,
              horizon     -- YEAR|QUARTER|MONTH|WEEK
              , period_start date, period_end date,
              status      -- ACTIVE|DONE|DROPPED
              , progress_mode -- MANUAL|METRIC|TASKS
              , metric    -- WEIGHT|EXERCISE_COUNT|EXERCISE_MINUTES|WATER_DAYS|SLEEP_HOURS (null)
              , target_value numeric, unit, sort_order)
goal_checkins(id, goal_id fk, date, value numeric, note)   -- MANUAL 모드용
tasks        (id, user_id, goal_id fk null, title, due_date, done_at timestamptz, sort_order)
```

- enum은 Postgres `enum` 타입 대신 **`text` + `check` 제약**으로 둔다. 이전과 변경이 쉽다.
- 자동 지표 진척도는 `packages/core`에서 계산한다 (DB 함수 금지, §8).

---

## 8. Supabase를 쓰되 나중에 Java로 쉽게 옮기기

Supabase는 결국 **그냥 Postgres**다. 아래 규칙만 지키면 나중에 Spring Boot가 같은 DB에 그대로 붙을 수 있다.

### 지금 지킬 규칙

1. **마이그레이션은 순수 SQL 파일로 관리한다** (`supabase/migrations/*.sql`). 이름만 `V1__init.sql` 식으로 바꾸면 Flyway에서 그대로 쓸 수 있다.
2. **비즈니스 로직을 DB에 넣지 않는다.** plpgsql 함수, 트리거, RLS 안의 로직을 금지한다. RLS는 `user_id = auth.uid()` 소유권 체크만 둔다.
3. **앱은 repository 인터페이스를 통해서만 데이터에 접근한다.** 화면에서 `supabase.from(...)`을 직접 호출하지 않는다.
   ```ts
   // packages/core
   interface MealRepository {
     listByDate(date: string): Promise<MealsByType>;
     listByMonth(yearMonth: string): Promise<DayRecordFlags[]>;
     create(input: MealCreate): Promise<Meal>;
     // …
   }
   // 지금: SupabaseMealRepository / 나중: RestMealRepository (Spring API)
   ```
4. **Realtime 등 Supabase 전용 기능에 의존하지 않는다.**
5. Edge Functions는 꼭 필요한 것(AI 추론 등)만 두고, 입출력은 zod 스키마로 고정한다. 이전할 때 Spring 엔드포인트로 1:1로 옮긴다.
6. Storage 경로 규칙은 `{user_id}/meals/{meal_id}/{photo_id}.jpg`로 둔다. S3 호환이라 그대로 옮길 수 있다.

### 이전 시나리오 (언젠가)

1. `apps/api`에 Spring Boot를 추가하고, Supabase Postgres 연결 문자열로 **같은 DB**에 붙는다 (JPA 또는 jOOQ).
2. Spring은 Supabase Auth JWT를 JWKS로 검증한다. 인증은 당분간 Supabase를 그대로 쓴다.
3. OpenAPI 스펙에서 TS 타입을 생성하고, `packages/core`의 repository 구현을 REST로 교체한다. 화면 코드는 바뀌지 않는다.
4. 필요하면 `pg_dump`로 자체 Postgres에 옮기고, 인증과 스토리지를 차례로 교체한다.

---

## 9. 인증·보안 (public 레포 기준)

- Supabase Auth 이메일 매직링크를 쓰고, **내 계정 1개**만 허용한다. 가입은 대시보드에서 막는다.
- 커밋해도 되는 것은 `SUPABASE_URL`, `anon key`뿐이다 (RLS가 있어서 노출돼도 안전한 키). 그래도 `.env`로 분리하고, `.env.example`만 커밋한다.
- **절대 커밋 금지:** `service_role` 키, AI API 키, 개인 데이터 덤프. AI 키는 Supabase secrets에만 둔다.
- GitHub secret scanning과 push protection을 켠다.

---

## 10. 배포

| 대상 | 방법 |
| --- | --- |
| 웹 | Vite 정적 빌드를 Vercel 또는 Cloudflare Pages에 배포, 커스텀 도메인 (선택) |
| iOS | 개인 개발자 계정의 TestFlight (혼자 쓰니 스토어 출시 없음) |
| Android | 릴리스 APK 사이드로드 또는 내부 테스트 트랙 |
| DB | Supabase 무료 플랜. 주 1회 `pg_dump` 백업 (GitHub Actions에서 private 스토리지로) |

---

## 11. 마일스톤

| # | 목표 | 완료 기준 |
| --- | --- | --- |
| **M0** | 기반 | 모노레포에서 RN 앱(iOS/Android)과 웹이 같은 "Hello" 화면을 띄운다. Supabase 로그인, CI 통과. |
| **M1** | 건강기록 코어 | 수분, 체중, 수면, 운동 기록 + 오늘 홈 + 기록 링 |
| **M2** | 식사 | 끼니별 식사 기록, 사진 업로드(모바일과 웹), 상세 입력 |
| **M3** | 캘린더·리포트 | 월 달력, 주·월·연 추이 차트 |
| **M4** | 목표·계획 | 목표 트리, 할 일, 자동 지표 진척, 홈 위젯 |
| **M5** | 실사용 | 2주간 매일 써 보고 불편한 점 수정, Phase 1 마감 |

---

## 12. 열린 질문

- [ ] 목표 계층을 연→분기→월→주 4단으로 둘지, 연→월 2단으로 단순화할지
- [ ] 식단 AI 추론(사진 → 메뉴와 양)을 Phase 1에 넣을지 1.x로 미룰지. 넣는다면 Edge Function + Claude 비전 모델
- [ ] HealthKit/Health Connect에서 걸음 수와 수면을 자동으로 가져올지
- [ ] 웹을 데스크톱 전용 레이아웃(사이드바, 넓은 대시보드)으로 따로 디자인할지, 모바일 레이아웃을 늘려 쓸지
- [ ] 디자인 시스템: 새로 만들지, 기존 작업물의 토큰 구조만 참고할지
- [ ] 라이선스 (MIT 등) 지정 여부
