# INNO Scheduler

연구실 장비 주간 예약. Synthesis(합성 장비: 토큰 투표·추첨·실시간 예약)와 Optics(광학방: 요일 투표·배정·조율 채팅) 두 앱 + 시작 화면.

## 폴더 지도
```
INNO Scheduler/
├─ index.html                ← 시작 화면(두 스케줄러 선택)
├─ Synthesis_scheduler.html  ← 합성 장비 스케줄러
├─ Optics_scheduler.html     ← 광학방 스케줄러
├─ favicon.ico · apple-touch-icon.png   ← 위 5개가 GitHub Pages 배포 대상
├─ README.md
├─ server/                   ← 서버 쪽 원본(콘솔에 붙여넣는 것)
│   ├─ INNO_firestore.rules  ← Firestore 보안 규칙(합성+광학 통합본, 유일한 규칙 파일)
│   ├─ Synthesis_gas.txt     ← Google Apps Script 봇(추첨·캘린더·Slack 알림)
│   └─ Optics_gas.txt        ← 광학 봇
├─ docs/
│   ├─ Synthesis 설치 가이드.md · Optics 설치 가이드.md
│   └─ HISTORY.md
└─ _drive_backup/
```

## 쓰는 법 · 고치는 법 · 배포
- HTML 자체가 원본. 배포: `python "../INNO Deploy/inno_deploy.py" push scheduler` → https://inno-kaist.github.io/INNO-Scheduler/ (파일명까지 포함한 주소로 접속).
- `server/` 파일은 배포 도구가 올리지 않는다 — Firebase 콘솔(규칙)·script.google.com(GAS)에 붙여넣는다(설치 가이드 참고). 붙여넣을 때 GAS_CONFIG·SLACK_MENTION 값 유지.

- 사용법: `docs/` 의 가이드 · 작업 이력과 교훈: `docs/HISTORY.md`
- 백업(Google Drive): `_drive_backup/백업 만들기.bat` 실행 → `_drive_backup` 폴더를 Drive 에 올림. 복구는 `_drive_backup/RESTORE.md`.
