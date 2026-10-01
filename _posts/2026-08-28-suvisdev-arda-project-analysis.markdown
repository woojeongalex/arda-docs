---
layout: default
title: "Arda 프로젝트 구조 분석"
date: 2026-08-28
categories: [analysis]
owner: 진수택
owner_color: "#7aa2f7"
summary: "저장소 구조(backend 라우터 11종·에이전트 모듈) · 도메인 오너제와 W1 협업 규칙 진화 · ERD/API 문서=계약 원칙 · ADR 15건 · 게이트 일정(09/04·09/30) 분석"
---

# Arda 프로젝트 구조 분석

> 작성: suvisdev (에이전트 도메인) · 기준: 2026-08-28 저장소 실측

W1을 마치며 저장소 전체를 훑고 구조·협업 체계를 분석한 기록이다. 새로 합류하거나 다른 도메인을 들여다볼 때 시작점으로 쓰면 된다.

---

## 한 줄 요약

Arda는 **채용 지원자 관리 시스템(ATS)** — 공고 등록부터 지원서 제출(S3), 칸반 보드 단계 심사, 단계 변경 시 자동 메일(SES·SQS)까지의 흐름을 5인·5주에 만드는 프로젝트다. 이름은 퀘냐로 "영역(Realm)" — 판단하는 주체가 아니라 판단이 일어나는 장소라는 뜻을 담았다(ADR-0014).

## 저장소 구조

```
backend/    FastAPI 서버
  app/
    api/        라우터 11종 — auth · postings · applications · assignments
                · evaluations · notes · files · search · public · agent
    agent/      에이전트 도메인 — runtime(도구 호출 루프) · entity_resolver
                · extract(이력서 구조화) · summarizer · stt · prompts · tools
    models.py · schemas/ · stages.py(단계 정책) · mail.py · s3.py · worker.py
frontend/   React (Vite · TS) — Vercel 배포
mobile/     Flutter (Android APK 데모 예정, ADR-0010)
infra/      Docker · AWS(EC2·S3·SES·SQS) · GitHub Actions
docs/       00_overview(계약 문서) · 01_role(도메인 로드맵)
            · 02_tasks(지시서) · 03_decision(ADR 15건) · 04_planning
```

`docker-compose.yml`은 postgres:16-alpine(healthcheck 포함) + api 두 서비스 — 로컬은 `docker compose up` 한 번으로 뜬다.

## 협업 체계 — 도메인 오너제

이 프로젝트에서 가장 특징적인 부분. 사람마다 폴더 단위 도메인을 소유하고, 자기 로드맵(`docs/01_role/`)이 곧 작업 큐다.

| 도메인 | 오너 | 폴더 |
|---|---|---|
| 인프라·총괄 | bestcow | `infra/` · `.github/` · AWS |
| 프론트엔드 | cloverky | `frontend/` |
| 에이전트 | suvisdev | `backend/app/agent/` · `api/agent.py` |
| 앱 | minahdev | `mobile/` |
| 백엔드 | woojeongalex | `backend/` (agent 제외) |

W1 동안 협업 규칙이 두 번 진화했다: "작업 풀 + 팀장 지시서" → **도메인 오너제**(08/24, ADR-0007) → **리뷰 게이트 폐지·전원 main 직접 push**(08/28). 승인 대기를 없애는 대신 ① push 전 `git pull --rebase`+검증, ② 스키마·API 변경은 같은 커밋에서 계약 문서(ERD `01-erd.md`·API `02-api.md`) 갱신 + 사후 공지, ③ 깨진 main은 묻지 말고 fix-forward, ④ 금요일 팀장 주간 다이제스트로 흐름 파악 — 이 네 가지가 안전망이다.

## 문서가 계약이다

- **ERD·API 문서 = 전원의 계약.** 코드와 같은 커밋에서 갱신하지 않는 스키마 변경은 금지.
- **ADR 15건** — "왜 안 썼는가"까지 기록한다: K8s 제외(0001), 표정분석 제외(0002), **AI는 추천만·합불 확정은 항상 사람**(0003), React 확정(0006), 에이전트·앱 정식 트랙 승격(0008), Flutter 확정(0010), 에이전트 모델·비용(0011), 엔티티 오탐 방지(0015) 등.
- 기능은 번호로 부른다(A=인증, B=공고, D=칸반, G=메일, H=검색, J=배포…) — 커밋 제목에 번호를 붙여 이력 자체가 기록이 되게 한다.

## 일정 구조

게이트 2개: **초기 버전 09/04(금)** → **1차 완성 09/30(수)**. W1(08/24~28)에서 이미 실배포 1차(W2 마일스톤)를 앞당겨 완료했고, 에이전트 트랙은 W2~W4 작업을 대폭 선행했다(E2E 데모·비용 실측 — [1주차 팀 활동 보고]({% post_url 2026-08-26-team-progress %}) 참고).

## 분석 소감 — 잘 굴러가는 이유

1. **폴더 경계 = 권한 경계 = 책임 경계**가 일치해서 "누구에게 물어야 하나"가 없다.
2. 리뷰를 없앴는데 계약 문서 동기화 규칙이 그 빈자리를 채운다 — 문서가 낡지 않는 구조.
3. 제외 결정(ADR)을 기록해 둔 덕에 "이거 왜 안 해요?"가 반복되지 않는다.

리스크는 하나: 5명이 같은 문서·같은 지킬을 만질 때의 **git 충돌**이다. 이건 팀원별 소유 파일로 분리하는 [팀 칸반]({{ site.baseurl }}/kanban/)으로 풀었다 — 별도 글 없이 칸반 페이지의 사용 규칙 절 참고.
