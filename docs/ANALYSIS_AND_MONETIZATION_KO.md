# AutoResearchClaw 전수조사 분석 & 수익화 전략 정리 (한국어)

> 작성일: 2026-09-22
> 대상 저장소: **https://github.com/bmshin94/AutoResearchClaw**
> 원본(업스트림): **https://github.com/aiming-lab/AutoResearchClaw**
> 관련 자료: [arXiv 논문](https://arxiv.org/abs/2605.20025) · [ARC-Bench 데이터셋](https://huggingface.co/datasets/AIMING-Lab-UNC/ARC-Bench) · [Discord](https://discord.gg/u4ksqW5P)
> 라이선스: MIT (Copyright (c) 2026 Aiming Lab) · 버전: 0.5.0 · Python 3.11+

---

## 목차
1. [프로젝트 전수조사 결과](#1-프로젝트-전수조사-결과)
2. [쉽게 풀어쓴 개념 설명](#2-쉽게-풀어쓴-개념-설명)
3. [핵심 질문 7선 Q&A](#3-핵심-질문-7선-qa)
4. [수익화 아이디어 상세](#4-수익화-아이디어-상세)
5. [최종 로드맵](#5-최종-로드맵)

---

## 1. 프로젝트 전수조사 결과

### 1.1 한 줄 정의
> **연구 주제 한 줄을 입력하면 → 학회 제출 수준의 논문 초안 한 편이 통째로 산출되는 자율 AI 연구 파이프라인**

### 1.2 규모 (실측)
| 항목 | 수치 |
|---|---|
| 전체 파일 수 | 757개 |
| Python 파일 / 라인 | 275개 / 약 81,500줄 |
| 테스트 파일 | 120개 (README 배지: 2,699 테스트 통과) |
| 서브패키지 | 33개 |
| 파이프라인 단계 | 23 스테이지 / 8 페이즈 |

### 1.3 폴더 구조 실측

| 경로 | 내용 |
|---|---|
| `researchclaw/` | 본체 패키지 (pipeline, llm, experiment, hitl, literature, knowledge, mcp, server, skills, evolution, memory, domains, docker, overleaf, trends, calendar, voice, wizard 등) |
| `researchclaw/pipeline/` | `stages.py`(23단계 IntEnum·전이·롤백), `contracts.py`(단계별 입출력 계약), `executor.py`(단계 실행기), `runner.py`(구동), `debate.py`, `tournament.py`, `code_agent.py`, `experiment_diagnosis.py`, `experiment_repair.py`, `paper_verifier.py`, `verified_registry.py`, `opencode_bridge.py` |
| `researchclaw/hitl/` | Human-in-the-Loop 코파일럿 (smart_pause, cost_guard, claim_verifier, branching, escalation, learning, workshops, tui, adapters 등 20+ 모듈) |
| `researchclaw/mcp/` | MCP 서버/클라이언트/툴/트랜스포트/레지스트리 |
| `researchclaw/server/` | 웹서버 + WebSocket + 라우트(chat, pipeline, projects, voice) |
| `researchclaw/skills/` | 스킬 loader / matcher / registry / schema / builtin |
| `researchclaw/experiment/` | 샌드박스, 러너, AST 검증기, 시각화, git 매니저 |
| `.claude/skills/` | Claude Code 스킬 9종 (researchclaw, a-evolve, literature-search, scientific-writing, scientific-visualization, statistical-reporting, hypothesis-formulation, chemistry-rdkit, biology-biopython) |
| `external/agents/` | 도메인 특화 에이전트 3종: Biology-Agent(COBRApy 대사모델), ColliderAgent(입자물리 FeynRules→MadGraph5→Delphes), stat_research_agent |
| `experiments/arc_bench/` | ARC-Bench 55개 연구주제 벤치마크(ML 25, HEP 10, 양자 10, 생물 7, 통계 3) + manifests + rubrics |
| `tests/` | 120개 테스트 파일 |
| `docs/` | 9개 언어 README(한국어 포함), HITL 가이드, 도메인 통합 가이드, 생성 논문 PDF 8편 쇼케이스 |
| `website/` | 정적 소개 사이트 (HTML/CSS) |
| `frontend-legacy/` | 구버전 대시보드 (순수 바닐라 JS, 빌드 스텝 없음) — 컴포넌트 7종 |
| `prompts.default.yaml` | 단계별 프롬프트 전량 (23KB) |
| `config.researchclaw.example.yaml` | 전체 설정 예시 (9.5KB) |
| `sentinel.sh` | 백그라운드 품질 감시 워치독 |

### 1.4 23단계 파이프라인

```
Phase A 스코핑       1. TOPIC_INIT          2. PROBLEM_DECOMPOSE
Phase B 문헌조사     3. SEARCH_STRATEGY     4. LITERATURE_COLLECT
                     5. LITERATURE_SCREEN [게이트]   6. KNOWLEDGE_EXTRACT
Phase C 종합         7. SYNTHESIS           8. HYPOTHESIS_GEN (멀티에이전트 토론)
Phase D 실험설계     9. EXPERIMENT_DESIGN [게이트]  10. CODE_GENERATION  11. RESOURCE_PLANNING
Phase E 실험실행    12. EXPERIMENT_RUN     13. ITERATIVE_REFINE (자가치유)
Phase F 분석        14. RESULT_ANALYSIS    15. RESEARCH_DECISION (PROCEED/REFINE/PIVOT)
Phase G 논문작성    16. PAPER_OUTLINE      17. PAPER_DRAFT   18. PEER_REVIEW   19. PAPER_REVISION
Phase H 마무리      20. QUALITY_GATE [게이트]  21. KNOWLEDGE_ARCHIVE  22. EXPORT_PUBLISH  23. CITATION_VERIFY
```

- **게이트 단계 3개(5, 9, 20)**: 사람 승인 대기. `--auto-approve`로 생략 가능.
- **결정 루프**: 15단계에서 REFINE(→13) 또는 PIVOT(→8)로 되돌아감. 아티팩트 자동 버저닝.

### 1.5 산출물
```
artifacts/rc-YYYYMMDD-HHMMSS-<hash>/deliverables/
├── paper_draft.md            # 5,000~6,500 단어 논문 초안
├── paper.tex                 # NeurIPS/ICLR/ICML LaTeX
├── references.bib            # 실제 BibTeX (인라인 인용 기준 자동 정리)
├── verification_report.json  # 4겹 인용 무결성 검증 결과
├── charts/                   # 에러바·신뢰구간 포함 비교 차트
├── reviews.md                # 멀티에이전트 동료심사
├── evolution/                # 실행별 추출 교훈
└── experiment runs/          # 생성 코드 + 샌드박스 결과 + JSON 지표
```

### 1.6 언제 쓰나
- 신규 연구 주제의 빠른 탐색/서베이
- 실험 코드 생성 → 실행 → 자동 수정 반복
- 학회 템플릿 LaTeX 초안 확보
- **AI 에이전트 아키텍처 학습 교보재** (실사용 가치 최상)

### 1.7 개발자에게 주는 가치
1. 에이전트 설계 교과서 — 상태머신, 단계 계약, 롤백, 게이트, 자가치유
2. HITL 설계 패턴 — 6개 개입 모드, 비용 가드레일, 신뢰도 기반 자동 일시정지
3. 안티-할루시네이션 실전 코드 — 4겹 인용 검증 + VerifiedRegistry
4. MCP / Skills / ACP 연동 레퍼런스가 한 레포에 전부 존재

---

## 2. 쉽게 풀어쓴 개념 설명

### 2.1 비유: "논문 자동 요리 로봇"
| 요리 로봇 | AutoResearchClaw |
|---|---|
| 레시피 검색 | 논문 DB 3곳(OpenAlex/Semantic Scholar/arXiv)에서 실제 논문 수집 |
| 재료 손질 | 지식 카드 추출 + 연구 공백(gap) 분석 |
| 조리 | 파이썬 실험 코드 자동 생성 → 샌드박스 실행 |
| 맛보기 | NaN/에러 감지 → 코드 자동 수리 → 최대 10회 재시도 |
| 간 조절 | REFINE(파라미터 수정) / PIVOT(방향 전환) 자동 판단 |
| 손님 평가 | 멀티에이전트 동료심사 (7차원 점수, NeurIPS 체크리스트) |
| 플레이팅 | LaTeX + BibTeX + 차트 |

### 2.2 핵심 개념 5가지

**① 23단계 상태머신** — 각 단계가 `required_keys` / `produced_keys` 계약을 가짐. 중단 시 `--resume`, `--from-stage`로 해당 지점부터 재개.

**② 게이트 3개** — 5/9/20단계에서 사람 승인 대기. `--auto-approve`로 무인 완주.

**③ 자가치유** — 실험 에러 로그를 LLM에 재투입 → 수정 코드 → 재실행, 최대 10라운드.

**④ 안티-할루시네이션 (가장 중요)**
- 인용: arXiv ID → CrossRef/DataCite DOI → Semantic Scholar 제목 매칭 → LLM 관련성 점수. 미통과 시 자동 삭제.
- 수치: `VerifiedRegistry`에 실제 실험 결과로 등록된 값만 논문 사용 허용, 나머지는 sanitize.

**⑤ 자기진화** — 실행마다 교훈 추출 → 30일 시간감쇠 적용 → 다음 실행 프롬프트에 주입. MetaClaw 연동 시 교훈이 재사용 가능한 스킬로 변환.

---

## 3. 핵심 질문 7선 Q&A

### Q1. 설치 및 사용법

```bash
git clone https://github.com/bmshin94/AutoResearchClaw.git
cd AutoResearchClaw
python3 -m venv .venv && source .venv/bin/activate   # Python 3.11+
pip install -e .            # 기본: pyyaml, rich, arxiv, numpy
pip install -e ".[all]"     # 전체: httpx, scholarly, crawl4ai, PyMuPDF, matplotlib, scipy

researchclaw setup     # OpenCode 설치, Docker/LaTeX 점검
researchclaw init      # 대화형 → config.arc.yaml 생성
researchclaw doctor    # 환경 건강검진

researchclaw run --config config.arc.yaml --topic "연구 주제" --auto-approve
researchclaw run --topic "연구 주제" --mode co-pilot
researchclaw run --resume
researchclaw run --from-stage PAPER_OUTLINE
```

최소 설정:
```yaml
project: { name: "my-research" }
research: { topic: "연구 주제" }
llm:
  base_url: "https://api.openai.com/v1"
  api_key_env: "OPENAI_API_KEY"
  primary_model: "gpt-4o"
  fallback_models: ["gpt-4o-mini"]
experiment:
  mode: "sandbox"
  sandbox: { python_path: ".venv/bin/python" }
```

CLI 전체 명령 (cli.py 실측):
`run` `validate` `doctor` `init` `setup` `info` `report` `serve` `dashboard` `wizard` `project` `mcp` `overleaf` `trends` `profile` `skills` `calendar` `attach` `status` `approve` `reject` `guide`

주의: Python 3.11+, Docker 모드엔 Docker, LaTeX 컴파일엔 TeX Live 필요. 1회 완주에 LLM 호출 수백 회 → 토큰 비용 주의.

### Q2. 플러그인인가 / 스킬인가 / MCP인가
**본질은 독립 파이썬 애플리케이션이며, 동시에 전부 지원한다.**

| 형태 | 지원 | 근거 |
|---|:---:|---|
| 독립 CLI 앱 (본체) | O | `[project.scripts] researchclaw = "researchclaw.cli:main"` |
| Python 라이브러리 | O | `from researchclaw.pipeline.runner import execute_pipeline` |
| MCP 서버 | O | `researchclaw/mcp/server.py`, 툴 6종: `run_pipeline`, `get_pipeline_status`, `get_experiment_results`, `search_literature`, `review_paper`, `get_paper` |
| Claude Code 스킬 | O | `.claude/skills/researchclaw/SKILL.md` |
| 스킬 호스트 | O | `researchclaw/skills/` 로더가 SKILL.md 20종을 프롬프트에 자동 주입 |
| OpenClaw 서비스 | O | `RESEARCHCLAW_AGENTS.md` + `openclaw_bridge` 어댑터 6종(cron/message/memory/sessions_spawn/web_fetch/browser) |
| ACP 백엔드 사용 | O | Claude Code / Codex / Copilot / Gemini / Kimi / OpenCode를 LLM 엔진으로 |
| 웹 서버 | O | `researchclaw serve` |

### Q3. API 토큰이 필요한가
- **LLM**: API 키 방식 **또는** ACP 방식(키 불필요) 중 선택.
```yaml
# A. API 키
llm: { base_url: "https://api.openai.com/v1", api_key_env: "OPENAI_API_KEY", primary_model: "gpt-4o" }
# B. ACP — API 키 없이 에이전트 CLI 자체 인증 사용
llm: { provider: "acp", acp: { agent: "claude", cwd: "." } }
```
- **논문 DB**: OpenAlex(이메일만) / Semantic Scholar(선택) / arXiv(무료) / CrossRef·DataCite(무료) → **키 없이 동작**
- **비용 방어**: `cost_budget_usd` 설정 시 50%/80%/100% 경고 + 초과 시 자동 일시정지 (`hitl/cost_guard.py`)

### Q4. 왜 GitHub에서 유명한가
1. **주제 선점** — "AI Scientist" 카테고리에서 오픈소스 + MIT + 실제 동작하는 드문 케이스
2. **데모 임팩트** — 생성 논문 PDF 8편을 `docs/showcase/`에 직접 공개 (말이 아닌 결과물)
3. **신뢰 장치** — arXiv 논문, HuggingFace 데이터셋(ARC-Bench 55토픽), 테스트 2,699개, 대학 연구실(UNC AIMING Lab) 배경
4. **진입장벽 제거** — README 9개 국어, Discord, 테스터 모집, OpenClaw에 URL만 던지면 자동 설치
5. **생태계 호환성** — Claude Code/Codex/Copilot/Gemini/Kimi/OpenCode + Discord/Telegram/Lark/WeChat
6. **릴리즈 속도** — v0.1.0(3/15) → v0.5.0(5/19), 2개월간 5회 메이저 릴리즈

> 오픈소스 그로스 전략 자체가 벤치마킹 가치 있음.

### Q5. 로컬 에이전트 구축에 도움이 되는가 — **매우 그렇다**

재사용 가능한 설계 패턴:

| 패턴 | 파일 | 가치 |
|---|---|---|
| 단계 계약(Contract) | `pipeline/contracts.py` | LLM 오작동 시 검증 가능 — 최우선 학습 대상 |
| 상태머신 + 롤백 | `pipeline/stages.py` | 장시간 에이전트 작업의 정석 |
| 체크포인트/재개 | `--resume`, `--from-stage` | 중단 복구 |
| 게이트 + HITL | `hitl/` | 6개 개입 모드, SmartPause |
| 자가치유 루프 | `experiment_diagnosis.py`, `experiment_repair.py` | 에러→진단→수리→재실행 |
| AST 코드 검증 | `experiment/validator.py` | LLM 생성 코드 실행 전 문법/보안 스캔 (필수) |
| 샌드박스 격리 | `experiment/sandbox.py`, `docker/` | 네트워크 정책 none/setup_only/pip_only/full |
| 멀티에이전트 토론 | `pipeline/debate.py`, `tournament.py` | 다관점 합의 |
| 스킬 시스템 | `skills/loader.py` 등 | SKILL.md 상황별 자동 주입 |
| 메모리/지식베이스 | `memory/`, `knowledge/` | 6개 카테고리 구조화 저장 |
| 자기진화 | `evolution.py` | 실패→교훈→개선, 30일 감쇠 |
| 비용 가드 | `hitl/cost_guard.py` | 예산 초과 자동 정지 |
| 재현성 | `hitl/checksums.py` | SHA256 아티팩트 체크섬 |

로컬 무료 구성 예:
```yaml
llm:
  provider: "openai-compatible"
  base_url: "http://localhost:11434/v1"   # Ollama
  primary_model: "qwen2.5-coder:32b"
  api_key: "ollama"
experiment:
  mode: "docker"
  docker: { network_policy: "none" }
```
한계: 8만 줄 규모로 무거움, 연구 워크플로에 강결합, 소형 로컬 모델 단독으로 23단계 완주 시 품질 저하 → 하이브리드(문헌조사=API 모델, 코드생성=로컬 모델) 권장.
추천 학습 순서: `pipeline/` → `hitl/` → `experiment/validator.py`

### Q6. 수익화 가능성 — 가능 (4장 참조)
- MIT 라이선스로 상업적 이용/수정/비공개화 전부 허용 (저작권 고지 유지 필요)
- 핵심 인사이트: **엔진 자체가 아니라 "엔진이 못 하는 부분"을 판다**

### Q7. React / PHP로 만들 수 있는가

| 접근 | 평가 | 내용 |
|---|:---:|---|
| **React로 프론트엔드/래퍼** | 강력 추천 | `researchclaw serve`(REST+WebSocket)가 이미 존재. `frontend-legacy`는 바닐라 JS라 Next.js+TS로 교체 가능. 추천 스택: Next.js 15 + TypeScript + TanStack Query + shadcn/ui + Recharts + Zustand |
| **PHP로 포털/결제 레이어** | 가능 | Laravel 포털(회원/결제/주문) → Redis Queue → Python 워커(researchclaw) → S3 결과 → PHP 다운로드. 빠른 MVP에 적합 |
| **엔진 자체 재작성** | 비추천 | 8만 줄 재작성 부담 + 과학 생태계(numpy/scipy/torch/RDKit/COBRApy/Biopython) Python 독점 + 실험 코드 생성/실행이 Python 전제 |

결론: **엔진은 Python 유지 + React 프론트엔드 신규 구축**이 가성비 최적.

---

## 4. 수익화 아이디어 상세

### 4.0 법적/윤리 체크
- 라이선스 MIT → 상업적 이용·수정·재배포·비공개화 허용. 저작권 고지 + 라이선스 전문 포함 필요.
- 저작권자는 Aiming Lab → "Powered by AutoResearchClaw" 표기 권장.
- **AI 생성 논문의 학술지 직접 투고는 대부분 학회 규정 위반.** "논문 대필"이 아니라 **"연구 보조/초안 생성 도구"**로 포지셔닝 필수.

### 아이디어 1. 한국형 연구 SaaS (난이도 高 / 수익성 최상 / 3~6개월)
타겟: 대학원생, 신진연구자, 기업 R&D
차별점: 영어 + CLI + API키 장벽 제거 → 한글 + 웹 + 클릭 3번

| 플랜 | 가격 | 내용 |
|---|---|---|
| Free | 0원 | 월 1회, 문헌조사(1~6단계)까지 |
| Starter | 29,000원/월 | 월 5회 완주, 기본 모델 |
| Pro | 99,000원/월 | 월 20회, 고성능 모델, 코파일럿, Overleaf 연동 |
| Lab | 390,000원/월 | 팀 5인, 공유 지식베이스, 우선 GPU |

스택: Next.js + FastAPI 게이트웨이 + Celery 워커 + researchclaw + PostgreSQL + S3
추가 차별화: RISS/KCI/DBpia 커넥터, 한글 초안, 국내 학회 템플릿

### 아이디어 2. 셋업·운영 대행 서비스 (난이도 低 / 수익성 中上 / 1~2주 — 가장 빠른 현금화)

| 상품 | 가격 |
|---|---|
| 원격 설치 + 세팅 대행 | 15~30만원 |
| 연구실 온보딩 교육 (2h) | 50~100만원 |
| 월간 운영 관리 | 30~50만원/월 |
| 도메인 커스터마이징 | 200~500만원 |

근거: Python 3.11/Docker/LaTeX/API키/YAML 장벽에서 대부분의 연구자가 이탈. 초기 투자 0원, 국내 선점 효과.
채널: 산학협력단, 연구실 커뮤니티, 크몽/숨고, LinkedIn

### 아이디어 3. 버티컬 특화 SaaS (난이도 최상 / 수익성 최상 / 6~12개월)
`external/agents/`의 Biology-Agent, ColliderAgent, stat_research_agent 패턴을 고부가 산업으로 확장.

| 버티컬 | 제공 가치 |
|---|---|
| 제약/바이오 | 후보물질 문헌조사 + RDKit 분석 + 실험설계안 |
| 소재/화학 | 소재 특성 예측 실험 파이프라인 |
| 금융 퀀트 | 전략 가설 → 백테스트 코드 → 리포트 |
| 의료 임상 | 관찰연구 설계 + 통계 (`docs/examples/medical_observational_demo.md` 존재) |

가격대: 엔터프라이즈 연 3,000만~2억원. 동일 엔진, 도메인 특화로 100배 단가.

### 아이디어 4. 마이크로 SaaS — "연구 부품" 판매 (난이도 低~中 / 2~4주)

| 상품 | 활용 단계 | 가격 |
|---|---|---|
| **인용 검증 API** | 23단계 4겹 검증 | 건당 100원 / 월 19,000원 |
| 자동 문헌조사 리포트 | 3~7단계 | 건당 5,000원 |
| AI 동료심사 봇 | 18단계 | PDF 1편당 3,000원 |
| AI-slop 탐지기 | 품질감사 4라운드 | 기관 구독 |

특히 **인용 검증 API**는 가짜 인용 이슈로 학술지·대학·출판사 수요가 명확하고 MVP 2주면 가능.

### 아이디어 5. 콘텐츠 & 교육 (난이도 低 / 즉시 시작 가능)

| 상품 | 가격 |
|---|---|
| 유튜브 "AI가 논문 쓴다" 시리즈 | 광고+협찬 |
| 온라인 강의 "AI 에이전트 아키텍처 실전" | 99,000원 |
| 유료 뉴스레터 | 9,900원/월 |
| 전자책 "AutoResearchClaw 완전정복" | 29,000원 |
| 기업 출강 | 100~300만원/일 |

이 레포는 계약/상태머신/롤백/HITL/자가치유/안티-할루시네이션 패턴의 집합체라 커리큘럼화가 쉬움. 아이디어 1~3의 마케팅 깔때기 역할.

### 아이디어 6. React 프론트엔드 오픈소스화 (비용 0 / 리턴 高)
1. `frontend-legacy`(바닐라 JS)를 Next.js + TypeScript로 재구축, 별도 오픈소스 공개
2. 업스트림 PR → 공식 웹 UI 지위 가능
3. 인지도 확보 → 아이디어 1(호스팅 SaaS)로 전환

---

## 5. 최종 로드맵

```
1개월차    콘텐츠(아이디어 5) + 셋업 대행(아이디어 2)
           → 리스크 0으로 현금흐름 + 시장 반응 검증

2~3개월차  React 프론트엔드 오픈소스(아이디어 6)
           → 기술력 증명 + 인지도 + SaaS 기반 코드 확보

4~6개월차  한국형 SaaS 런칭(아이디어 1)
           → 앞선 단계의 고객/인지도로 초기 유저 확보

7~12개월차 버티컬 특화(아이디어 3)
           → 고단가 도메인 엔터프라이즈 전환
```

### 전략 3원칙
1. **엔진을 팔지 말고, 엔진을 쉽게 쓰게 만든 것을 판다** (원본이 무료 오픈소스이므로)
2. **한국화가 최고의 해자다** (영어 + CLI 장벽이 곧 기회)
3. **"논문 대필"이 아니라 "연구 가속 도구"로 포지셔닝한다** (윤리·법적 안전선)

---

## 참고 링크
- 본 저장소: https://github.com/bmshin94/AutoResearchClaw
- 업스트림 원본: https://github.com/aiming-lab/AutoResearchClaw
- arXiv 논문: https://arxiv.org/abs/2605.20025
- ARC-Bench 데이터셋: https://huggingface.co/datasets/AIMING-Lab-UNC/ARC-Bench
- 한국어 README: [docs/README_KO.md](README_KO.md)
- HITL 가이드: [docs/HITL_GUIDE.md](HITL_GUIDE.md)
- 통합 가이드: [docs/integration-guide.md](integration-guide.md)
- 논문 쇼케이스: [docs/showcase/SHOWCASE.md](showcase/SHOWCASE.md)
