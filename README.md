# SignalDesk

`SignalDesk`는 한국주식과 미국주식을 나눠 보고, 수급·공포지표·뉴스·포트폴리오·AI 추천·모의투자와 Pizza·Bar 계열 실험 대체지표까지 한 제품 안에서 관리하는 주식 인텔리전스 서비스다.

핵심 방향:
- 실시간 단타 앱이 아니라 `개인 투자자용 의사결정 플랫폼`
- 한국/미국 시장을 큰 틀에서 분리
- 코스피/코스닥, 나스닥/S&P, 공포지표, 수급, 뉴스, 관심종목, 포트폴리오를 한 흐름으로 연결
- 무료/공식 데이터 소스 우선, 이후 유료 시세 API로 확장 가능한 구조

구성:
- `signal-desk-api`: Kotlin + Spring Boot API
- `signal-desk-web`: React + Vite 웹 대시보드
- `signal-desk-app`: Expo + React Native 모바일 컴패니언 앱
- `docker-compose.yml`: 로컬 web + api 통합 실행

역할 분리:
- `api`: 시장 데이터 수집, 장 상태 계산, 뉴스/차트/수급 응답, 워크스페이스 CRUD
- `web`: 전체 기능을 다루는 메인 투자 대시보드
- `app`: 시장/차트/AI 로그를 빠르게 확인하는 모바일 요약 앱

현재 구조:
- `signal-desk-api`: 워크스페이스 저장소를 인터페이스 뒤로 분리해 향후 PostgreSQL 전환 준비
- `signal-desk-web`: 탭 컴포넌트, 데이터 hook, 뉴스/포맷 유틸 기준으로 구조 정리
- `signal-desk-app`: 홈 탭에서 관심종목/포트폴리오 요약까지 확인 가능한 모바일 컴패니언 앱

현재 구현 범위:
- `시장` 탭
  - 한국/미국 분리
  - 코스피/코스닥, 나스닥/S&P
  - 공포/열기 지표
  - 실험 지표(Pentagon Pizza Index, Policy Buzz, Bar Counter-Signal 등 alt-data)
  - 개인/외국인/기관 수급
  - 주요 뉴스
- `종목` 탭
  - 대표 종목 리스트
  - 관심 종목
- `포트폴리오` 탭
  - 얼마에 샀는지
  - 현재 얼마인지
  - 수익률/평가금액
- `AI 추천` 탭
  - 오늘 추천 종목
  - 추천 근거
  - 신뢰도
  - 추천 후 실제 수익률 트래킹
- `모의투자` 탭
  - 가상 현금
  - 가상 보유 종목
  - 최근 거래 로그

1차 릴리즈 보강:
- 뉴스를 섹터 기준으로 군집화하고 시장 요약 문장 제공
- 뉴스 나열을 줄이기 위해 상위 묶음 우선 노출 + 펼치기 구조 적용
- AI 추천 종목에 연관 뉴스 묶음 연결
- 차트 라벨을 실제 날짜 기준으로 정규화(오늘 날짜 포함)
- 한국/미국 장 상태를 정규장/장전/장후/휴장으로 계산해 표시
- 차트 엔진을 교체해 가독성 개선

실데이터 현황:
- 한국 지수/수급: `KRX 정보데이터시스템`
- 한국 종목 현재가: `Naver Finance Realtime`
- 미국 공포지표: `CBOE VIX`
- 미국 지수: `FRED`
- 뉴스: `Google News RSS`

저장 구조:
- 시장 데이터는 외부 공개 데이터 기반으로 수집
- 관심종목, 포트폴리오, AI 추천 로그, 모의투자 데이터는 기본적으로 API의 로컬 파일 저장소로 관리
- `signal-desk-api`는 JDBC 저장소 모드를 이미 지원하며, PostgreSQL로 전환해 같은 워크스페이스 CRUD를 사용할 수 있다
- 루트 `docker-compose.yml`은 기본 파일 저장 모드 기준이고, JDBC/PostgreSQL 검증용 compose는 `signal-desk-api/docker-compose.jdbc.yml`에 별도로 둔다

속도 구조:
- 초기 진입은 `summary`, `sections`, `news`로 분리
- 탭 전환 시 `watchlist`, `portfolio`, `ai-recommendations`, `paper-trading`를 지연 로드
- 백엔드는 시장 코어 데이터를 60초 캐시, 뉴스는 5분 캐시
- 첫 실데이터 수집 이후에는 `summary/sections` 응답이 매우 빠르게 재사용됨
- 차트는 동적 로딩으로 분리해 초기 로딩 부담 완화
- 시장 탭 차트 모듈은 lazy import로 분리되어 첫 진입 번들 축소
- 웹은 탭별 에러/로딩 상태를 분리해 한 탭 실패가 전체 실패처럼 보이지 않게 처리

주요 API:
- `GET /api/v1/market/summary`
- `GET /api/v1/market/sections`
- `GET /api/v1/market/news`
- `GET /api/v1/market/watchlist`
- `GET /api/v1/market/portfolio`
- `GET /api/v1/market/ai-recommendations`
- `GET /api/v1/market/paper-trading`
- `GET /api/v1/market/overview`
- `POST/DELETE /api/v1/workspace/*`

다음 단계:
1. 미국 개별 종목 실데이터 범위 확대
2. 종목 검색/전체 종목 페이징
3. PostgreSQL 기반 관심종목/포트폴리오 저장
4. AI 추천 로직과 성과 트래킹 자동 계산
5. 모의투자 거래 엔진 고도화
6. 알림/장전 브리핑 자동화

로컬 실행:
```bash
cd /Users/g/workspace/public/finance/signal-desk
docker compose up --build
```

- compose에는 `restart: unless-stopped`와 healthcheck를 포함했다.
- `signal-desk-web`은 `signal-desk-api`가 healthy 상태가 된 뒤 기동한다.

모바일 앱 실행:
```bash
cd /Users/g/workspace/public/finance/signal-desk-app
npm install
npm run web
```

기본 포트:
- web: `http://localhost:4180`
- api: `http://localhost:8091/api/v1/market/overview`
- app(web preview): Expo가 사용 가능한 포트로 기동하며, 고정 포트로 가정하지 않는다

배포/검증 예시:
```bash
curl -i http://127.0.0.1:8091/api/v1/market/summary
curl -i http://127.0.0.1:8091/api/v1/market/watchlist
curl -i http://127.0.0.1:4180/
```

JDBC/PostgreSQL 모드 검증 예시:
```bash
cd /Users/g/workspace/public/finance/signal-desk-api
docker compose -f docker-compose.jdbc.yml up --build
```

- 루트 compose와 JDBC 검증 compose는 둘 다 `8091`을 사용하므로 동시에 실행하지 않는다.

릴리즈 체크:
1. `GET /api/v1/market/sections`에서 차트 라벨이 오늘 날짜(`MM/dd`)로 노출되는지 확인
2. 시장 탭 뉴스가 상위 묶음 중심으로 보이고 펼치기 동작이 정상인지 확인
3. AI 추천 카드에서 연관 뉴스 묶음이 연결되는지 확인
