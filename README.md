# 월드 트레이더스

친구와 가상의 세계 경제를 만들어가는 실시간 멀티플레이 보드게임 프로토타입입니다. React, TypeScript, Vite, Express, Socket.IO, Zod, Prisma를 사용합니다. 보드와 카드 문구, 규칙 표현, 브랜드 그래픽은 독자적으로 구성했습니다.

## 구조와 핵심 데이터 흐름

```text
React 화면 (src/App.tsx)
  ├─ REST: 방 생성·조회·참가 (/api/rooms)
  └─ Socket.IO: 준비·시작·굴림·매입·개발·경매·턴 종료
              ↓ Zod 입력 확인 + 방 멤버/방장/차례/단계 권한 확인
Express 게임 서버 (server/index.ts)
  ├─ 공개 방 상태를 해당 방에만 전송
  ├─ 주사위, 이동, 보상, 자금, 소유권, 로그를 서버에서 결정
  └─ PostgreSQL/Prisma 데이터 모델 (prisma/schema.prisma)
```

게임 클라이언트는 행동 의도만 전송하며 잔액, 위치, 주사위, 소유권을 제출하지 않습니다. 방 코드는 서버가 생성합니다. 개발 중 `DATABASE_URL`을 설정하지 않으면 메모리 저장 모드로 빠르게 시작할 수 있습니다. Prisma 스키마는 PostgreSQL 운영 데이터 모델의 기반입니다.

## 로컬 실행

Node.js 20 이상과 npm이 필요합니다.

```bash
npm install
cp .env.example .env
npm run dev
```

PowerShell에서는 `Copy-Item .env.example .env`를 사용하세요. 개발 서버는 Vite 화면 `http://localhost:5173`, API 및 Socket.IO 서버 `http://localhost:3001`에서 실행됩니다. 홈 화면에서 새 방을 만들고, 다른 브라우저/시크릿 창에서 초대 코드로 참가할 수 있습니다. **데모 교역실 둘러보기**로 현재 브라우저에서 바로 체험할 수도 있습니다.

## 환경 변수

| 변수 | 설명 |
| --- | --- |
| `PORT` | Express 포트 (기본 `3001`) |
| `CLIENT_ORIGIN` | 허용할 프런트엔드 origin. 여러 개는 쉼표로 구분 |
| `DATABASE_URL` | PostgreSQL 연결 문자열. 로컬 데모에서 생략 가능 |

비밀키나 데이터베이스 암호를 소스에 넣지 마세요. `.env`는 Git에 포함되지 않습니다.

## Prisma 및 PostgreSQL

로컬 PostgreSQL을 준비한 다음 `.env`에 `DATABASE_URL="postgresql://USER:PASSWORD@HOST:5432/worldtraders?schema=public"`를 설정합니다. 데이터베이스 계정과 암호는 개인 환경의 값으로 채우세요.

```bash
npm run db:generate
npm run db:dev -- --name init
npm run dev
```

`prisma/schema.prisma`에는 사용자, 방/멤버, 게임/플레이어 상태, 보드 공간, 소유권/개발, 카드, 턴, 로그, 거래, 경매, 재접속 세션 모델을 정의했습니다. 금액은 정수입니다.

## Render 배포

저장소를 Render에 연결하고 `render.yaml` Blueprint를 사용합니다. Blueprint는 Web Service와 PostgreSQL 데이터베이스를 정의합니다. 배포 빌드에서 Prisma Client를 생성하고, 시작 단계에서 `prisma migrate deploy` 후 서버를 실행합니다. Render 대시보드에서 `CLIENT_ORIGIN`을 실제 서비스 도메인과 일치시키세요. PostgreSQL 연결 문자열은 Render가 `DATABASE_URL`로 주입합니다.

수동 배포 시 빌드 명령은 `npm install && npm run db:generate && npm run build`, 시작 명령은 `npm run db:migrate && npm start`입니다.

## 구현 현황과 남은 작업

현재 동작하는 수직 흐름: 방 생성/참가, 게스트 이름, 초대 코드 복사, 로비와 준비, 방장 시작, 36칸 독창 보드, 서버 주사위/이동, 출발 보상, 구매 선택, 상대 임대료, 그룹 완성 보너스, 최대 3단계 개발, 세금/이벤트 효과, 라운드 종료, 실시간 로그, 연결 재접속(프로세스 생존 중), 반응형 보드/자산/기록 탐색, 규칙 안내입니다. 입력은 Zod로 검증하고, 소켓 연결은 게임 플레이어 식별 및 방 멤버 확인 후 허용합니다.

미완성 항목:

- Prisma 모델과 마이그레이션 구성은 포함했지만, 현재 서버 런타임은 빠른 개발을 위해 메모리 방 저장소를 사용합니다. PostgreSQL 영속화 어댑터와 장애 후 복구 연결이 필요합니다.
- 거래 제안/승인·거절 API, 경매 시간 만료 자동 판정/클라이언트 입찰 패널은 남았습니다. 서버 경매 이벤트 처리의 기반만 포함합니다.
- 대출, 자산 매각을 통한 채무 해결, 파산 시 소유권 이전은 남았습니다. 현재 자금 부족은 알림 로그를 남기고 잔액을 0으로 제한합니다.
- 턴 제한 시간 자동 처리, 초대 코드 만료/재생성, 계정 인증, 요청 속도 제한은 운영 전 추가해야 합니다.
- `resolve_event` 카드 선택/개별 비공개 상태는 아직 사용하지 않습니다. 현재 이벤트는 서버가 무작위로 결정하고 공개 로그로 보냅니다.

이 기능을 프로덕션 운영 준비 완료 상태로 간주하지 마세요. 핵심 플레이 프로토타입은 실행 가능하지만, 위 서버 영속성 및 경제 규칙은 추가 구현이 필요합니다.
