# Weekly Synthesis Scheduler — 통합 설치·운영 가이드 (인수인계용)

연구실 합성 스케줄 토큰제 시스템. 완전 무료(카드 등록 불필요), 서버 상시 운영 불필요.
구성: **웹앱 1개 + Firebase(데이터·보안) + Google Apps Script 봇(Slack 공지·캘린더 동기화)**

**파일명 규약**: `Synthesis_scheduler.html` / `server/Synthesis_gas.txt` / `server/INNO_firestore.rules`(합성+광학 통합 규칙 — 유일한 규칙 파일). GitHub에도 **파일명 그대로 업로드**하며, 접속 주소는 파일명까지 포함한 `https://inno-kaist.github.io/INNO-Scheduler/Synthesis_scheduler.html` 형식. (루트 주소는 사용하지 않음 — 대문 페이지 index.html을 별도로 두면 루트 접속도 가능)

---

# ⚡ A. 직접 입력해야 하는 값 (설치·이관 시 필수)

| # | 값 | 입력 위치 | 얻는 곳 |
|---|---|---|---|
| 1 | `FIREBASE_CONFIG` (6개 값) | `Synthesis_scheduler.html` 상단 | Firebase 콘솔 → ⚙프로젝트 설정 → 일반 → 내 앱 |
| 2 | `apiKey` / `projectId` / `appUrl` | `server/Synthesis_gas.txt` 상단 `GAS_CONFIG` | 1번과 동일 값 + 배포된 웹앱 주소 |
| 3 | `slackWebhookUrl` | `GAS_CONFIG` | 파트2-1단계에서 발급 (⚠비밀 — 코드 밖 공유 금지) |
| 4 | `calendarId` | `GAS_CONFIG` | 구글캘린더 → 설정 및 공유 → 캘린더 통합 |
| 5 | `SLACK_MENTION` 표 (이름↔Slack ID) | `server/Synthesis_gas.txt` | 아래 B 참고 |

**현재 운영 정보 (2026-07 기준)** — 이관 시 후임자에게 함께 전달할 것:

| 항목 | 값 |
|---|---|
| 웹앱 주소 | https://inno-kaist.github.io/INNO-Scheduler/Synthesis_scheduler.html |
| 코드 저장소 | GitHub `INNO-KAIST/INNO-Scheduler` |
| Firebase 프로젝트 (데이터·보안규칙) | `inno-synthesis-scheduler` |
| Slack 발표 채널 | `#synthesis-schedule` |
| GAS 봇 프로젝트 | script.google.com (설치한 구글 계정 소유) |
| 접근 권한 계정 | GitHub·Firebase·GAS를 만든 구글/GitHub 계정 — **이 계정 접근 권한이 곧 관리 권한** |

---

# 🔄 B. 멤버 변동 시 반드시 할 일

**신규 멤버 (4곳):**
1. 웹앱 관리자 탭 → "명단에 추가" (이름 입력)
2. 본인에게 첫 로그인 안내 → 이름 선택 + 원하는 PIN 입력(그 PIN이 등록됨). **명단 추가 후 가급적 빨리** (PIN 등록 전 계정은 이론상 선점 가능)
3. Slack `#synthesis-schedule` 채널 초대
4. GAS 코드 `SLACK_MENTION` 표에 `'이름': 'Slack멤버ID'` 추가 → 저장 (Slack ID: 프로필 → ⋮ → 멤버 ID 복사)

**퇴출/휴면 멤버:**
1. 관리자 탭 → "비활성화" (삭제보다 권장 — 과거 기록의 이름 표시 유지)
2. GAS `SLACK_MENTION` 표에서 해당 줄 삭제 → 저장

**주의사항:**
- `SLACK_MENTION`의 이름은 웹앱 '명단'의 표기와 **한 글자도 다르면 안 됨** (다르면 그 사람만 멘션 누락)
- 표에 없는 사람은 리마인드 멘션 대상에서 빠질 뿐, 웹앱 사용에는 지장 없음
- GAS 코드는 저장만 하면 즉시 적용 (setup 재실행 불필요)
- 본인 계정 비활성화·삭제, 마지막 관리자의 권한 해제는 앱이 차단함 (잠금 사고 방지)

---

# 시스템 구조 한눈에

| 구성요소 | 역할 | 관리 위치 |
|---|---|---|
| GitHub Pages | 웹앱 파일 호스팅 | github.com 저장소 |
| Firebase Firestore | 데이터 저장 + 보안규칙(마감·토큰·본인확인·선착순을 서버가 강제) | console.firebase.google.com |
| GAS 봇 | 월화수 12시 리마인드 · 목 12시 발표 · 캘린더 1분 동기화 | script.google.com |
| Slack Webhook | 봇 → 채널 발송 통로 | api.slack.com/apps |
| 공용 구글캘린더 | 확정 스케줄 자동 반영 (읽기 전용 거울 — **직접 수정 금지**, 앱에서 조정) | calendar.google.com |

**주간 사이클(KST)**: 월 00시~수 24시 투표(주 4토큰, 다음 주 대상) → 목 12시 자동 추첨 발표 + Day 탈락자 우선예약(기본 60분) → 전체 실시간 예약(대상 주 일요일까지). 월요일 토큰 초기화는 자동(주차별 저장 구조).

---

# 파트 1. 스케줄러 웹앱 설치 (총 ~15분)

1. **Firebase 프로젝트**: console.firebase.google.com → 프로젝트 추가(애널리틱스 사용 안 함) → 요금제 Spark(무료) 확인. Blaze 업그레이드 절대 불필요
2. **익명 인증**: 빌드 → Authentication → Sign-in method → **익명** 사용 설정
3. **Firestore 생성**: 빌드 → Firestore Database → 위치 **asia-northeast3(서울)** → **프로덕션 모드**
4. **보안 규칙**: Firestore → 규칙 탭 → 기존 내용 전부 삭제 → `server/INNO_firestore.rules` 전체 붙여넣기 → 게시
5. **웹앱 등록**: ⚙프로젝트 설정 → 내 앱 → 웹(</>) → 등록 → 표시되는 `firebaseConfig` 6개 값을 `Synthesis_scheduler.html` 상단 `FIREBASE_CONFIG`에 입력 (공개 식별자라 노출되어도 안전 — 보안은 4번 규칙이 담당)
6. **배포 (GitHub Pages)**: github.com → New repository(Public) → `Synthesis_scheduler.html` 업로드(파일명 그대로) → Settings → Pages → Deploy from a branch / main / root → 1~2분 후 `https://아이디.github.io/저장소명/Synthesis_scheduler.html` 접속 가능
7. **승인 도메인**: Firebase → Authentication → Settings → 승인된 도메인 → `아이디.github.io` 추가
8. **첫 관리자 등록**: 배포 주소 접속(데모 배너 없어야 정상) → 멤버 로그인 → 초기 설정 화면에서 본인 이름+PIN 등록 → 관리자 탭에서 멤버 명단 추가 (이후 B 절차)

---

# 파트 2. 자동화 봇 설치 (총 ~15분)

1. **Slack Webhook 발급**: api.slack.com/apps → Create New App(From scratch) → Incoming Webhooks **On** → Add New Webhook → 채널 `#synthesis-schedule` 선택 → URL 복사
2. **GAS 프로젝트**: script.google.com → 새 프로젝트 → 기본 코드 삭제 → `server/Synthesis_gas.txt` 전체 붙여넣기 → 상단 `GAS_CONFIG`에 A표의 값 입력 + `SLACK_MENTION` 표 확인 → 저장
3. **시간대 확인**: ⚙프로젝트 설정 → **(GMT+09:00) Seoul** (GAS와 구글캘린더 모두)
4. **캘린더 연동**: 구글캘린더 → 공용 캘린더 설정 및 공유 → 캘린더 통합 → **캘린더 ID** 복사 → `GAS_CONFIG.calendarId` 입력. GAS 실행 계정에 해당 캘린더 "일정 변경" 권한 필요
5. **테스트 3종** (함수 드롭다운에서 선택→실행. 최초 실행 시 권한 승인 — "확인되지 않은 앱" 경고는 본인 스크립트라 정상, 고급→이동→허용):
   - `testSlack` → 채널에 테스트 메시지 도착 확인
   - `previewRemind` → 리마인드 미리보기 발송, **멘션이 정확한 사람에게 걸리는지 확인**
   - `testCalendar` → 캘린더에 테스트 일정 생성 확인 후 삭제
6. **활성화**: `setup` 실행 → ⏰트리거 메뉴에 `armDaily`(매일 오전 9시대)와 `calendarSyncJob`(1분마다) 2개 확인

봇 동작: 매일 오전 armDaily가 당일 12시 정각 실행을 장전 → 월화수엔 **채널 멤버 중 미투표자만 멘션**하는 리마인드(전원 투표 시 미발송), 목요일엔 추첨 확정·발표. 캘린더는 1분마다 동기화되되 변화 없으면 즉시 종료(무료 한도 안전). 일정 형식: `[합성 #1·#2] 이름`, Day 06–18시 / Night 18시–익일 06시 / 주말 종일.

---

# 운영 요령

- **관리자 권한 (웹앱)**: 명단 관리(추가/비활성화/PIN·기기 초기화/관리자 지정), 우선예약 시간 조정, 예약 단계 Day 2배치 제한 토글, 결과 재생성. **실시간 예약 탭에서 모든 칸 직접 관리 가능** — 빈 칸 클릭 시 임의 멤버 명의 대리 예약(우선창 무시), 타인 예약 취소, 타인 확정 취소. 변경은 1분 내 캘린더 자동 반영
- **멤버 안내사항**: 확보한 시간을 못 쓰게 되면 **즉시 앱에서 취소** (규칙 탭 최상단 명시). 본인 추첨 확정 칸도 실시간 예약 탭에서 취소 가능(권리 영구 상실·토큰 미반환). 이미 확정된 이번 주 스케줄의 빈 칸은 일정 탭(이번 주 보기)에서 실시간 신청/본인 취소 가능
- **공정성 검증**: 투표 결과 탭의 "✓ 검증 완료" = 브라우저가 투표 원본으로 결과를 재계산해 대조했다는 뜻. "⚠ 불일치"가 뜨는 것이 비정상 → 관리자 "결과 삭제 후 재생성"
- **PIN 분실**: 관리자 탭 → PIN 초기화 → 본인이 다시 첫 로그인으로 새 PIN 등록. **기기 변경**: 새 기기에서 이름+PIN 로그인만 하면 됨
- **파일 업데이트 시 교체 위치**: `Synthesis_scheduler.html` → GitHub에 같은 이름으로 덮어쓰기(1~2분 후 반영, 강력 새로고침) / `server/INNO_firestore.rules` → Firebase 콘솔 규칙 탭 게시(합성·광학 공용 — 반드시 통합본 사용) / `server/Synthesis_gas.txt` → GAS 편집기 전체 교체 후 저장(단, GAS_CONFIG·SLACK_MENTION 값 유지 확인)
- **봇 중단/재개**: GAS 트리거 메뉴에서 삭제 / `setup` 재실행
- **실행 이력·오류 확인**: GAS 왼쪽 "실행" 메뉴

# 문제 해결

| 증상 | 원인 / 해결 |
|---|---|
| "데모 모드" 배너 | FIREBASE_CONFIG 미입력본이 배포됨, 또는 file://로 열었음 |
| "Firebase 인증 실패" | 익명 인증 미활성화 (파트1-2) |
| 저장 시 permission denied | 규칙 미게시, 해당 작업의 허용 시간이 아님(정상), 또는 미로그인 |
| 로그인 목록에 이름이 없음 | 계정이 '비활성화' 상태 → 다른 관리자가 활성화. 관리자 전원이 잠겼다면 Firebase 콘솔 → Firestore → members → 해당 문서 `active`를 `true`로 직접 수정 |
| 리마인드에 멘션 누락 | SLACK_MENTION 표의 이름이 명단 표기와 다름 (B 참고) |
| 12시 발표가 안 옴 | GAS 실행 메뉴에서 오류 확인. slackWebhookUrl 공백 여부, 트리거 2종 존재 여부 확인 후 setup 재실행 |
| 캘린더가 이상함 | 캘린더에서 직접 수정했는지 확인(자동 원상복구됨 — 앱에서 조정할 것). 즉시 복구: GAS `forceCalendarResync` 실행 |
| 캘린더 날짜가 하루 밀림 | 캘린더 또는 GAS 시간대가 Seoul이 아님 |
| GAS 한도 경고 | 코드 상단 `CAL_SYNC_EVERY_MINUTES`를 1→5로 변경 후 setup 재실행 |
| 무료 한도 걱정 | 연구실 규모에서는 Firebase·GAS 모두 한도의 수 % 수준. 초과해도 과금 없음(일시 차단 후 자정 리셋) |
