# 초등 교육과정 기반 독서전략 수업설계 시스템

2022 개정 초등 교육과정 13개 TTL과 `readstrategy.ttl`, `text.ttl`, `가네.ttl`을 사용하는 교육과정 기반 수업설계 **시제품(MVP)**입니다. 일반 AI 프롬프트만 이용하는 방식과 달리 파서가 읽은 온톨로지의 `hasArea`, `hasCoreIdea`, `hasContentElement`, `hasAchievementStandard`, `hasVerb`, `hasStrategy`, `sequenceNumber` 관계를 사용합니다.

## 실행

Node.js 20 이상, Python 3.10+ 및 `rdflib`(데이터 재컴파일할 때만 필요).

```bash
npm install
npm start
```

브라우저에서 `http://localhost:3000` 열기. 기본 포트는 3000이고 Render는 `PORT` 환경변수를 자동 제공합니다.

## GitHub / Render 배포

1. ZIP을 풀고 **압축 해제한 폴더 안의 파일을 GitHub 저장소 루트에 업로드**하세요.
2. Render > New > Web Service > GitHub 저장소를 연결하세요.
3. Build Command: `npm install`, Start Command: `npm start`, Runtime: Node.
4. 환경변수 `APP_PASSWORD`에 긴 임의의 사이트 접속 암호를 등록하세요. (`render.yaml` Blueprint 이용 시 자동 생성됩니다.)
5. 선택 사항: `GEMINI_API_KEY` 설정 시 AI 읽기자료/지도안 초안 생성 지원. `GEMINI_MODEL` 기본 `gemini-2.5-flash`는 서비스 제공 시점에 변경될 수 있습니다.
6. **데이터 보존이 중요하면 PostgreSQL을 연결하고** `DATABASE_URL`에 접속 문자열을 등록하세요. Render 무료 웹 인스턴스 로컬 파일은 배포·재시작 시 유실될 수 있습니다.
7. 배포 주소의 `/api/config`가 JSON을 반환하는지 확인하세요.

주의: `APP_PASSWORD`는 공유 접속 암호입니다. **교사별 개별 계정 및 독립된 보관함, Firebase Auth/Rules는 구현되지 않았습니다.** 실사용에서 민감한 학생정보를 입력하지 마세요. 암호 설정만으로 다중 사용자 데이터 격리가 되지는 않습니다. DB가 연결되면 현재 모든 접속자가 같은 저장 목록을 공유합니다. 공개 서비스 전에는 로그인·권한 구조 보완이 필요합니다.

## 주요 기능

- 13개 교과, 611개 성취기준 및 영역/핵심 아이디어/내용 요소 탐색
- 성취기준·학습목표 동사 기반 독서 기능 후보 제안 (휴리스틱, 교사가 선택/수정)
- TTL `hasStrategy`의 실제 그래프 연결을 이용한 독서전략 추천
- 읽기자료 직접 편집 및 Gemini 옵션 생성 (API 키는 서버에만 보관)
- `가네.ttl`의 9개 수업사태 템플릿, 교사/학생 활동·시간·평가 편집
- 저장·다시 열기·삭제, 인쇄/PDF, JSON, DOCX 출력
- 데이터 현황 및 영역 미확정 성취기준 별도 표시

## 온톨로지 및 데이터 품질

- 사용된 TTL 파일은 `data/ttl/`에 포함되어 있습니다.
- 재컴파일: `pip install rdflib` 후 `python scripts/compile.py`
- 데이터: 총 13개 초등 교과, 611개 성취기준, 독서기능 15개, 가네 사태 9개.
- 원본 자동 추출 데이터는 **검수 완료본이 아닙니다.** 사회 49, 과학 102, 영어 40개 성취기준은 영역 매핑이 미확정입니다. 음악·미술·통합교과의 내용 요소/핵심 아이디어에는 누락이 있습니다. 원문 속 수식·표와 대조해 검수하세요.
- 성취기준→읽기 기능은 RDF에서 정의된 관계가 아니라 앱의 동사 어휘 매칭 추천입니다. 독서기능→전략은 `readstrategy.ttl`에서 정의된 관계입니다. 추천 전략→가네 수업사태 배치는 편집 가능한 설계 예시이지 TTL에서 검증된 연계가 아닙니다.

## MD 마스터 가이드 대비 상태

**구현된 핵심 MVP**: 교과/성취기준 탐색, 읽기자료 작성, 독서전략 추천, 가네 9사태 편집, PDF(인쇄), DOCX, JSON, 저장 기능, Gemini 선택 연동, Render 배포 구성.

**아직 구현되지 않은 기능**: Firebase 계정별 로그인 및 Security Rules, 관리자 전용 TTL 업로드/변경 차이/버전 롤백, 정밀 형태소 기반 동사 매칭, 검수된 독서전략 적합 텍스트 유형/사태 연결, 본문 사실 검증, 자동 저장, 개별 활동지 생성, 완전한 Playwright E2E 테스트, 사용자별 접근권한, 이미지 분석. 원본 MD 문서에 있는 전체 Phase 0~11 완성품이 아닙니다.

## 제작 파일

- `public/app.js`, `public/index.html`, `public/styles.css` : 교사용 UI
- `server.js`: Express API / PostgreSQL 또는 로컬 저장 / Gemini / DOCX 생성
- `scripts/compile.py`: RDF Turtle → 정규화된 JSON
- `data/compiled.json`: 앱에서 사용하는 컴파일 데이터
- `data/ttl/`: 원본 TTL 16개 (13개 교과 + 3개 읽기/수업 온톨로지; 기존 중학교 예시 3개가 추가로 포함됨)
- `render.yaml`: Render Blueprint 배포 설정

## 로컬 검증 사항

파이썬 `rdflib`으로 13개 초등 교과 및 독서전략·텍스트·가네 TTL을 구문 분석하여 JSON을 생성했습니다. `node --check`로 `server.js`와 `public/app.js` 문법을 확인했습니다. 현재 실행 환경에서는 `npm install`이 시간 초과하여 **실제 서버 구동과 브라우저 E2E 테스트까지 완료했다고 주장할 수 없습니다.** Render에서 설치·실행 후 브라우저로 검수하세요.
