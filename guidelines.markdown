---
layout: default
title: 주요 개발 수행 지침
nav_order: 5
permalink: /guidelines/
---

# 주요 개발 수행 지침

> 최종 갱신 2026-09-28 — 인스턴스·STT 경로·헬스체크 간격을 실제 운영값으로 정정(코드는 09-18 동결 그대로).

> **기준 시점: 2026-09-22 · 저장소 `Seuk-Team/Arda` 커밋 `8071fd9`.** 이 문서는 실제 저장소 파일(`pyproject.toml` · `package.json` · `pubspec.yaml` · `.github/workflows/ci.yml` · `docs/00_overview/03-conventions.md` · `07-deploy.md` · `docs/03_decision/` ADR 36편)과 대조해 작성했다. 수치·버전은 그 시점의 값이다.

## 일반 사항

### 개발 방법론

**도메인 오너제**([ADR-0007](https://github.com/Seuk-Team/Arda/blob/main/docs/03_decision/0007-도메인-오너제-전환.md)) — 한 사람이 도메인 하나(폴더)를 소유하고, 자기 도메인 로드맵(`docs/01_role/<도메인>.md`)이 작업의 기준이다. 도메인 안의 판단은 묻지 않고 진행하며, 팀 채널에 올리는 것은 도메인 밖뿐이다 — 범위·마일스톤 변경, 다른 도메인이 기다리는 의존, 30분 넘게 막힌 문제. 팀장(인프라·총괄)은 승인 게이트가 아니라 **서버 자원·비용·발표 문구 통일**만 조율한다.

| 도메인 | 오너 | 폴더 |
|---|---|---|
| 인프라·총괄 | 진수택 (suvisdev) | `infra/` · `.github/` · `docker-compose.yml` · AWS |
| 백엔드 | 이우정 (woojeongalex) | `backend/` (agent 제외) |
| 에이전트 | 박소연 (cloverky) | `backend/app/agent/` |
| 앱·프론트엔드 | 김민아 (minahdev) | `mobile/` · `frontend/` |

**계약 우선** — ERD(`01-erd.md`)·API 문서(`02-api.md`)가 전원의 계약이다. 스키마·API 변경은 **코드와 같은 커밋**에서 문서와 alembic 리비전을 함께 갱신해야 하며, 문서 갱신 없는 스키마 변경은 금지한다. 요청·응답의 진실은 Swagger(`/docs`)이고 `02-api.md`는 "무엇이 있는가"만 유지한다. 프론트·앱은 계약 문서로 병렬 개발한다.

**ADR 기반 의사결정** — 기술·범위·윤리 결정 **36건**을 ADR(`docs/03_decision/`)로 남겼다. "안 한 것"에도 ADR이 있다 — Kubernetes 제외(0001), 실시간 공동편집 제외(0005), SQS 워커 폐기(0036). **개정은 원문을 지우지 않고 절을 덧붙인다.** 오너가 개정 ADR을 쓰면 그것으로 확정이며 "팀 확정 대기" 상태를 두지 않는다. 대표 개정 사례 — 표정분석 제외(0002) → 표정·음성 보조 신호 도입(ADR-0029, 단 **점수 재료로 쓰지 않음**) · AI는 추천까지(0003) → 서류 단계 자동 판정, 최종 합불만 사람(0034) · AWS 8종 → 3종(0031·0036) · 헥사고날 부분 적용 Bounded Context 4개(0035) · 실시간 전사 OpenAI API(0038).

주차별 실행 계획은 [개발 일정]({{ site.baseurl }}/schedule/) 참조.

### 협업 운영 규칙

- **브랜치 → PR → 자체 머지** (2026-09-04 개정) — `main` 직접 push·force push는 GitHub 브랜치 보호가 차단한다. 자기 PR은 **CI 초록이면 자기가 머지**한다. 사람 승인 게이트는 없다 — 리뷰가 필요하면 담당 도메인 오너에게 요청(선택).
- **`main` 머지 = 2분 후 프로덕션 배포** — 서버가 2분마다 `main`을 확인해 새 커밋이면 pull→build→up 한다. CI 빨간불인 PR은 머지하지 않는다.
- **push 전 검증 필수** — 자기 파트 실행·테스트 결과를 커밋 메시지 한 줄에 남긴다.
- **테스트를 통과시키려고 우회·주석 처리 금지** — 실패하면 실패한 대로 기록한다.
- **깨진 `main`은 묻지 않고 즉시 고친다** — fix-forward 또는 revert. CI 빨간불은 다른 작업보다 우선.
- **되돌릴 수 있는 일은 그냥 한다** — 실험·프로토타입은 묻지 않고 결과만 공유. 물어야 할 것은 되돌리기 어려운 것뿐(스키마, 외부 서비스 계약, 마일스톤).
- **사후 공지** (팀 채널 한 줄) — 스키마 · API · 공용 문서(`docs/00_overview/` · `CLAUDE.md` · `.github/`) · 남의 도메인 폴더를 고쳤을 때. 큰 변경이면 직접 고치지 말고 이슈로 오너에게.
- **진행 상태는 팀 칸반**([/kanban/]({{ site.baseurl }}/kanban/)) · **받은 피드백은 피드백 트래커**([/feedback/]({{ site.baseurl }}/feedback/))에 기록한다. 로드맵은 범위·큐 순서, 칸반은 현재 상태 — 이중 관리하지 않는다.
- **시크릿은 `.env`(git 제외)에만** — 키 이름은 `.env.example`에 반영해 팀원이 알 수 있게 한다.

---

## 개발 표준 및 산출물

### 기술 스택

| 영역 | 스택 | 근거 파일 |
|---|---|---|
| **BACKEND** | Python 3.12 · FastAPI · SQLAlchemy 2.0 · psycopg 3 · PostgreSQL 16 + pgvector(`pgvector/pgvector:pg16`) · alembic(리비전 26개) · uv(의존 잠금) · pytest + pytest-timeout(60초) · ruff `--select F` | `backend/pyproject.toml` |
| **아키텍처** | 헥사고날 부분 적용 — Bounded Context 4개(`hiring` · `talent` · `application` · `interview`) + `ports/output` 포트 · `adapter/outbound/{pg,llm,mail}` 어댑터 (ADR-0035) | `backend/app/` |
| **FRONTEND** | React 19 · Vite 8 · TypeScript 6 · react-router 7 · three.js(아르 3D 캐릭터) · CSS Modules + 디자인 토큰 · oxlint | `frontend/app/package.json` |
| **APP** | Flutter (CI 3.44.8 · Dart SDK ^3.12) · `http` · `flutter_secure_storage`(토큰은 Android Keystore) · Android APK | `mobile/pubspec.yaml` |
| **AI · 판단** | **Anthropic Claude**(`claude-haiku-4-5`) — 서류 요약·평가·추천 3단 체인(ADR-0022) · 아르 도구 호출 에이전트 · 지원자 FAQ. 규칙 의도 라우터가 빈출 요청·기본 질문을 LLM 없이 처리($0) | `backend/app/agent/` |
| **AI · 검색** | sentence-transformers(ko-sroberta) 임베딩 + pgvector 시맨틱 검색 (ADR-0021) | `backend/app/agent/embedder.py` |
| **AI · 음성·표정** | Whisper STT — 업로드 답변·재전사·아르 음성 = whisper-1 API(백엔드가 직접 호출) · 실시간 면접 전사 = CPU 로컬 faster-whisper (09-17 운영 실측, `STT_BACKEND` 빈 값 — ADR-0038 「같은 변수, 두 해석」. 09-18 이후 값은 미확인) · lie-detection 서비스(Python 3.13 · MediaPipe 얼굴 랜드마크 · ViT 표정 · librosa 음성) — **담당자 화면 참고 지표 전용, 점수 미반영**(ADR-0029·0032) | `ai/lie-detection/` |
| **AI · R&D 자산** | Qwen3-8B QLoRA 어댑터 3갈래(chat v9 · summary · interview) — 동일 채점기로 학습 전 26.1% → v9 73.9%. 심사 서빙엔 쓰지 않고 `AGENT_*_BACKEND=ollama` 스위치로 교체 가능 | `ai/qwen-training/` |
| **무결성** | web3 · Ethereum Sepolia 테스트넷 앵커 · OpenTimestamps — 제출물 SHA-256 해시 사슬을 추가 전용 원장에 쌓고 DB 트리거가 수정·삭제를 거부(ADR-0028). 서명 키는 서버 밖(GitHub Actions) | `backend/app/chain.py` |
| **메일** | n8n 웹훅 + SMTP(지메일 앱 비밀번호) — n8n이 죽어도 API가 5분마다 밀린 메일을 재발송(최대 3회). SES·SQS는 폐기(ADR-0030·0031·0036) | `backend/app/shared/mail_smtp.py` |
| **INFRA** | Docker Compose(db · api · caddy · lie-detection · n8n) · AWS EC2 t3.large(서울, Elastic IP, EBS 50GB) · S3(presigned URL 직접 업로드, CORS) · Caddy(443, 인증서 자동) · Vercel(프론트) · systemd 타이머 자동 CD · CloudWatch + SNS 경보 · 매일 S3 DB 백업(30일 수명주기) | `infra/` · `07-deploy.md` |
| **CI** | GitHub Actions 5잡 — 백엔드 린트(ruff F) · 백엔드 테스트(alembic upgrade → 모델↔이행 비교 → pytest) · 프론트 빌드·린트(oxlint + tsc -b + vite build) · AI 면접 서버 시험 · 앱 시험(flutter test) | `.github/workflows/ci.yml` |

### 코드 관리 규칙

| 항목 | 기준 |
|------|------|
| 버전 관리 | Git · GitHub 조직 `Seuk-Team` (저장소 `Seuk-Team/Arda`, 2026-09-04 이관) |
| 브랜치 전략 | `git switch -c <type>/<주제>` → 실행·테스트 → push → **PR 오픈 → CI 초록 확인 → 자체 머지**. `main` 직접 push·force push는 브랜치 보호로 차단. 충돌 시 브랜치에서 `main`을 받아 풀고 다시 push |
| 코드 리뷰 | 사람 승인 게이트 없음(2026-08-28 폐지 → 09-04 PR 경로만 추가). **CI가 안전망**, 이의는 사후 revert가 기본값. 리뷰 요청은 선택 |
| 커밋 메시지 | `<type>(<기능번호|도메인>): <한국어 요약>` — `feat` `fix` `docs` `test` `chore` `refactor`. 예: `feat(J7): 더미 지원서 10만 건 생성기` · `fix(agent): 무관 질문 거절을 '데이터 근거' 기준으로` |
| 스키마·API 변경 | 코드 + `01-erd.md`/`02-api.md` + alembic 리비전을 **같은 커밋**에 · 사후 공지 |
| 시크릿 | `.env`(git 제외) + `.env.example`(키 이름만). 코드·커밋·로그에 키 금지 |

### 주요 산출물

| 구분 | 산출물 | 위치 · 비고 |
|------|--------|------|
| 설계 | 시스템 아키텍처 다이어그램 | [/about/]({{ site.baseurl }}/about/) architecture.svg |
| 설계 | ERD — 테이블 28개 (v2.8) · alembic 리비전 26개 | `docs/00_overview/01-erd.md` · erd.png |
| 설계 | API 명세 — 라우트 107개 | Swagger `/docs` 자동 생성 · `02-api.md` 목록 |
| 결정 | ADR 36편 | `docs/03_decision/` |
| 개발 | 소스 코드 — Backend · Frontend · App · AI(lie-detection · qwen-training) | GitHub `Seuk-Team/Arda` |
| 개발 | AI 프롬프트 18개(버전 관리) · 에이전트 도구 명세 `TOOLS.md` | `backend/app/agent/prompts/` |
| 품질 | 테스트 스위트 — 백엔드 pytest 1,165건 · 앱 위젯 테스트 30파일 · AI 면접 서버 pytest · 에이전트 회귀 하네스 | 저장소 · CI |
| 평가 | Qwen vs Claude 동일 채점기(`judge.py`) 비교 보고서 · 서류 판정 정확성 분석(fit-check 24명) | `ai/qwen-training/` · `docs/07_eval/` |
| 인프라 | Docker 구성 · Caddy · 배포 스크립트(자동 CD) · 권한 스크립트 | `infra/` · `backend/scripts/` |
| 인프라 | CI/CD 파이프라인 | `.github/workflows/ci.yml` |
| 문서 | 개발 진행 보고서 · 칸반 · 피드백 트래커 · 개발 로그 | 본 Jekyll 사이트 |

---

## 배포 · 운영

```
브라우저 ── https ──> Vercel (frontend/app · main 머지 시 자동 배포 · seuk.suvisdev.cloud)
브라우저/앱 ── https ──> Caddy(443, 인증서 자동) ──> FastAPI api:8000        ┐
                                             PostgreSQL 16 + pgvector (db)  ├ EC2 t3.large · docker compose
                                             lie-detection (/ai/*)          │   (api.seuk.suvisdev.cloud)
                                             n8n (/n8n/*)                   ┘
파일: 브라우저 ── presigned URL ──> S3 (서버 미경유 · SSE)
메일: api ── 웹훅 ──> n8n ──> SMTP   (실패 시 api가 5분마다 직접 재발송)
```

| 항목 | 방식 |
|---|---|
| 배포 트리거 | `main` 머지 → 서버 systemd 타이머(`arda-deploy.timer`)가 2분마다 확인 → pull → build → `up -d` (`deploy-arda.sh`, 로그 `~/deploy.log`). 백엔드 변경만 api 이미지 재빌드 |
| 스키마 이행 | 컨테이너 이미지가 기동 시 `alembic upgrade head` 실행 — 호스트 마운트 불필요 |
| 환경 파일 | `~/arda/.env`(compose: DB 비번 · n8n) + `~/arda/backend/.env`(앱 전체). **값 변경 후 `docker compose up -d --force-recreate`** — `restart`는 env_file을 다시 읽지 않는다 |
| 관측·경보 | CloudWatch 사용자 지정 지표(디스크·백업 나이·API 헬스) → SNS 메일 · GitHub Actions 외부 헬스체크(cron 15분 — 실제 실행 간격은 GitHub 스케줄 지연으로 더 벌어질 수 있음 · 실패 시 이슈 자동 개폐) · 컨테이너 로그 상한 20MB×3 |
| 백업 | 매일 04:00 KST DB `pg_dump` → gzip 검증 → S3(최소 권한 PutObject) · 30일 자동 만료 |
| 복구 | EC2 시스템 상태 검사 실패 시 자동 복구 알람 · 컨테이너 `restart: unless-stopped` |
| 프론트 | Vercel · 환경변수 `VITE_API_BASE` · 백엔드 `CORS_ORIGINS`에 프론트 도메인 필수 |
| 앱 | `flutter build apk --dart-define=API_BASE=<API 주소>` |

---

## 보안 · 개인정보

| 항목 | 기준 |
|---|---|
| 인증 | JWT Bearer 12시간 · **비활성 계정은 이미 발급된 토큰도 401** · 지원자는 이메일 + 생년월일 8자리 로그인 또는 링크 안 일회성 토큰(`/public/*`) · 판정 워커·n8n은 서비스 토큰(`/internal/*`) |
| 권한 | `admin` · `member` 둘(ADR-0017). admin 전용 — 면접관 배정·해제 / 계정 생성 / 메일 템플릿 / 타인 가용 시간. 조회는 로그인만 하면 전부 허용 |
| 운영 잠금 | `APP_ENV=production`에서 공개 회원가입 차단 · 기본 시크릿 사용 불가. 오타·미설정도 production으로 잠김 |
| 데이터 규칙 | 사용자는 삭제하지 않고 비활성화만(이력 보존) · 불합격은 `reason` 없으면 422 · 일괄 변경은 전부 성공 아니면 전체 롤백(409) · 단계 역행은 항상 허용, 전진은 한 칸씩 |
| AI 정책 | 프롬프트에 주민번호·연락처 등 개인정보 미포함 · **표정·음성 신호는 담당자 참고 지표로만 표시, 점수·합불 판정에 미반영**(ADR-0029) · 최종 합불은 사람이 확정(ADR-0003·0034) · AI 요약은 측정된 수치의 재서술만 허용, 수치에 없는 숫자나 사람을 단정하는 문구가 나오면 문장을 버리고 고정 틀로 대체 |
| 음성 데이터 경로 | `STT_BACKEND` 값으로 결정 — `openai`면 OpenAI API로 전송, `faster_whisper`면 서버 내 로컬 전사. **빈 값은 백엔드(openai)와 실시간 서버(로컬)가 다르게 읽으므로 반드시 명시**(ADR-0038 「같은 변수, 두 해석」) — 09-17 운영 실측은 빈 값이라 업로드는 API, 실시간은 로컬로 갈려 돌았다 |
| 무결성 | 이력서·자소서 SHA-256을 추가 전용 원장에 해시 사슬로 기록, 트리거가 UPDATE·DELETE·TRUNCATE 거부, 사슬 머리를 Sepolia에 매일 앵커 · 서명 개인키는 서버에 두지 않음(ADR-0028) |
| 시크릿 | 서버 `.env`에만 · IAM은 서버 전용 유저(최소 권한) · 팀장 개인 키는 전부 폐기 후 팀 발급분으로 교체(ADR-0025) |

---

## 품질 관리 및 테스트

### 품질 요구 사항

| 요구 ID | 항목 | 검수 기준 · 현황(2026-09-22) |
|---------|------|----------|
| PER-001 | **성능** — 지원자 검색·필터 | 더미 10만 건 기준 인덱스 튜닝 실측 **111ms → 7.8ms** |
| PER-002 | **성능** — 실시간 면접 STT | 오디오 청크 1.5초 예산 안에 전사 (09-17 GPU 벤치 Tesla T4 275ms/3초 음성 — 운영 실시간 전사는 CPU 로컬) |
| QUA-001 | **테스트** — 자동화 | CI에서 push·PR마다 백엔드 pytest(실제 PostgreSQL+pgvector, alembic 이행 후) · 프론트 oxlint + tsc + build · 앱 flutter test · AI 서버 pytest 전부 초록이어야 머지 |
| QUA-002 | **문서화** — API | FastAPI Swagger `/docs`에서 전체 API 명세(107 라우트) 조회 가능 |
| QUA-003 | **문서화** — 스키마 | 모델↔alembic 이행 결과를 CI가 비교해 이행 누락을 잡는다(2026-09-17) |

### 테스트 전략

| 단계 | 대상 | 도구 | 시점 · 실측 |
|------|------|------|------|
| 백엔드 린트 | 미정의 이름·미사용 import (배포 후 500의 주원인) | `ruff check --select F` | push·PR마다 (CI) |
| 백엔드 단위·API | 실제 PostgreSQL+pgvector에 alembic 이행 후 실행, 이행 결과↔모델 비교 선행 | pytest + pytest-timeout(60초) — **1,165건 통과** (커밋 `a650123`) | push·PR마다 (CI) |
| 프론트 | 타입·린트·빌드 | oxlint + `tsc -b` + `vite build` (Node 22) | push·PR마다 (CI) |
| 앱 | 위젯·API 클라이언트 | `flutter test` — 테스트 파일 **30개** | push·PR마다 (CI) |
| AI 면접 서버 | 실시간 전사·판정 중계·화자 매칭 | pytest 3파일 (Python 3.13) | push·PR마다 (CI) |
| 에이전트 회귀 하네스 | 실서버 `/agent/chat` 시나리오 40개(기본 10 + 변형 30) — 정답·속도·창작 여부 | 자체 하네스 | 모델·프롬프트 변경 시 |
| 모델 비교 | 동일 케이스 23건 · 동일 채점기 `judge.py`(도구 이름·no-tool 위반·확인 문구·인자·응답) | Qwen 학습 전 26.1% → Claude Haiku 69.6% → Qwen v9 73.9% | 어댑터 학습마다 |
| 서류 판정 정확성 | fit-check 지원자 24명 3축 점수(요건·우대·인재상) 근거 분석 | 분석 리포트 → PR#182 가중치 개정 | 판정 규칙 변경 시 |
| 계약 대조 | 배포본 OpenAPI ↔ `main` 경로·스키마·인증 | `check_public_contract.py` (08/31 실측 35/35 · 62/62 일치) | 배포 후 |
| E2E·QA | 필수 27기능 시나리오(전제→절차→기대) | qa-scenarios 문서 | 게이트 판정 |
| 성능 | 더미 10만 건 검색 | 111ms → 7.8ms | 인덱스 변경 시 |

### 사고가 남긴 장치 (재발 방지 규칙으로 승격된 것)

| 사고 | 장치 |
|---|---|
| 깨진 `main`을 하루 두 번 우연히 발견 | CI 도입 — "깨졌다는 사실만 즉시 보이게" |
| 디스크 고갈로 배포 정지 | 컨테이너 로그 상한(20MB×3) · 배포 때 이미지·캐시 prune · EBS 50GB |
| Docker가 죽어 pytest 24분 침묵 | `pytest-timeout 60초` — 멈춤을 실패로 |
| 미정의 이름 2건이 프로덕션 500 | CI `ruff --select F` |
| n8n이 웹훅 받고 죽어 메일 `queued` 잔류 | API가 5분마다 SMTP 재발송(최대 3회) |
| 면접 링크 즉시 시작 시 422 | 기본 질문 3개를 세션과 같은 커밋에 삽입 |
| `.env` 수정 후 `restart`해도 미반영 | `up -d --force-recreate` 규칙 문서화 |
| 로컬 sLLM 도구 오호출·예시 숫자 베끼기·JSON 깨짐이 심사 직전 누적 | 심사 서빙을 클라우드 + Claude로 확정, 온프레미스는 R&D 자산으로 보존(2026-09-18) |
