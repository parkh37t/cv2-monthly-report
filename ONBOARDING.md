# 컨버전스 2본부 실적관리 — Onboarding (Claude Code)

이 문서는 새 PC/노트북의 **Claude Code 에서 이 링크를 여는 즉시** Claude 가 자동으로 셋업을 진행하도록 작성되었습니다.

---

## 🤖 To Claude (자동 실행 지시)

사용자가 이 문서를 통해 Claude Code 를 열었다면, 아래 단계를 **순서대로 자동 진행**해주세요. 각 단계마다 사용자에게 진행상황을 한국어로 알려주세요.

### 0단계 · 환경 점검
다음 명령으로 git, node, npm 설치 여부와 현재 OS/PC 상황을 확인하세요:
```bash
git --version
node --version
npm --version
echo $USERNAME
```

미설치 시 안내:
- **Git 미설치** → https://git-scm.com/download/win
- **Node.js 미설치** → https://nodejs.org (LTS, v18+)

### 1단계 · 작업 폴더 결정
사용자에게 다음을 한 번 물어보세요(기본값으로 진행해도 좋다고 안내):
> "프로젝트를 어디에 클론할까요? 기본값: `C:\Users\<사용자명>\Documents\con-mgt` — 그대로 사용하려면 'ㅇ' 또는 Enter, 다른 위치는 경로 입력"

기본 경로로 진행 시 이미 폴더가 존재하면 → 4단계(.env 확인)로 점프.

### 2단계 · GitHub 저장소 clone
```bash
cd C:\Users\<사용자명>\Documents
git clone https://github.com/parkh37t/con-mgt.git
cd con-mgt
```

### 3단계 · 의존성 설치
```bash
npm install
```
약 1-2분 소요. 완료되면 다음 단계로.

### 4단계 · .env 파일 확인 / 안내
`C:\Users\<사용자명>\Documents\con-mgt\.env` 존재 여부 확인:

**존재하면** → 5단계로 진행
**없으면** → 사용자에게 다음 메시지 표시:

```
⚠️ .env 파일이 없습니다. 데이터베이스 연결을 위해 필요합니다.

방법 A (권장): 데스크탑 PC OneDrive 에서 복사
   기존 PC 에서 .env 를 OneDrive\_secrets\con-mgt.env 로 복사 후
   이 PC 에서: copy "%USERPROFILE%\OneDrive\_secrets\con-mgt.env" .env

방법 B: 직접 입력
   다음 4가지 값을 알려주시면 .env 파일을 만들겠습니다:
   1. DATABASE_URL (Supabase Postgres 연결 문자열)
   2. JWT_SECRET (랜덤 문자열, 32자 이상)
   3. ADMIN_USERNAME (보통: admin)
   4. ADMIN_PASSWORD (관리자 비밀번호)
```

방법 A 선택 시 자동 복사. 방법 B 선택 시 사용자 입력 받아 .env 생성.

### 5단계 · Cowork 세션 연결 안내
```
✅ 셋업 완료!

이제 좌측 사이드바 「Pinned」 영역에서
「컨버전스 2본부 실적관리」 세션을 클릭하면
데스크탑 PC 에서 작업하던 모든 대화가 그대로 이어집니다.

세션이 안 보이면:
- 데스크탑 PC 와 같은 Anthropic 계정인지 확인
- F5 로 새로고침
- 30초 대기 후 재시도
```

### 6단계 · 최신 코드 동기화 (이후 매번)
사용자가 이 PC 에서 작업을 시작할 때마다 자동으로:
```bash
cd C:\Users\<사용자명>\Documents\con-mgt
git pull origin master
```

---

## 📚 프로젝트 컨텍스트

### 무엇인가
- **컨버전스 2본부 실적관리** — Korean digital consultancy (와일리) 의 본부 손익/맨파워/프로젝트 관리 대시보드
- 단일 페이지 앱 (Vanilla JS + Express + Postgres)
- Vercel + Render dual deployment
- master 브랜치 직접 푸시 워크플로 (PR 미사용)

### 핵심 디렉토리
```
con-mgt/
├── public/
│   └── index.html       ← 메인 SPA (~660KB, Vanilla JS)
├── routes/
│   └── auth.js          ← 인증/회원/권한 API
├── api/                 ← Express API 엔드포인트
├── db.js                ← Postgres 연결 + 스키마 마이그레이션
├── server.js            ← Express 진입점
├── package.json
└── .env                 ← (gitignore) DB 연결 정보
```

### 기술 스택
- Frontend: Vanilla JS, SheetJS (xlsx), PptxGenJS, no build step
- Backend: Express.js, JWT, bcrypt
- DB: Supabase Postgres, JSON document store pattern (single `state` row)
- Brand: Wylie purple `#3A2A6B / #2A1D52 / #EDE8F5 / #8B7BB8`
- Migration: `_seedVersion` flag (current v16) for idempotent schema patches

### 배포
- master push → Render + Vercel 자동 배포 (1-2분)
- Render: https://con-mgt.onrender.com (Express 백엔드)
- Vercel: https://con-mgt.vercel.app (정적 + serverless)

### 작업 시 주의사항
- master 직접 푸시 OK (사용자 워크플로)
- `.env` 절대 커밋 금지 (.gitignore 됨)
- 큰 변경 후 `git push` 한 줄로 자동 배포
- index.html 은 단일 파일이므로 충돌 가능성 → 작업 시작 시 항상 `git pull` 먼저

### 데이터 모델 핵심
- `data.projects[]` — 프로젝트 마스터
- `data.execPnL[pid]` — 프로젝트별 손익 시뮬레이션 오버라이드 (`revenueOverride`, `extraPeople`, `peopleDraft` 등)
- `data.manpower.people[]` — 역할별 투입 일정 (단일 진실)
- `data.projects3` — 월간 결산 데이터 (xlsx 업로드로 갱신)
- `_buildExecMatrix(p)` — 단일 진실 매트릭스 빌더 (P&L + 인력)

### 자주 쓰는 패턴
- 매출 직접 수정: `execEditRevenueByMonth(monthKey, value)` → `execPnL[pid].revenueOverride['YYYY-MM']`
- 정직원 매핑 변경: `openEmployeePicker(rowKey, role)` → `pickEmployeeForExec(idx)`
- 현행화 (드래프트 → 본 데이터): `execRefreshFromSchedule()`

---

## 🔗 빠른 링크
- GitHub: https://github.com/parkh37t/con-mgt
- 운영 (Render): https://con-mgt.onrender.com
- 운영 (Vercel): https://con-mgt.vercel.app
- Supabase 콘솔: https://supabase.com/dashboard

---

## 💬 이 문서로 시작한 사용자에게

이 셋업이 끝나면, 평소처럼 자연어로 명령하시면 됩니다. 예:
- "메리츠화재 프로젝트 손익 시뮬레이션 열어줘"
- "5월 결산 엑셀 다시 업로드 했는데 정합성 검증해줘"
- "정직원 김선아 투입일 6월 1일로 바꿔줘"

이전 PC 에서 했던 모든 작업 컨텍스트는 Cowork 세션을 통해 동기화되어 있으니, "지난번에 ~했던 거" 같은 표현도 자유롭게 쓰셔도 됩니다.
