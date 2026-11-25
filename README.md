# 투표곰 (Team Poll Bot)

Teams 단체 대화방에서 후보를 올리고 함께 투표해서 결과를 모아 주는 봇입니다.

- GitHub 저장소: [namonak/teams-poll-bot](https://github.com/namonak/teams-poll-bot)
- 운영 주소: `https://poll-bot.namonak.dev`
- 구성: TypeScript, Microsoft Agents SDK, SQLite, Docker Compose (`teams-meal-bot`과 같은 서버·카드·배포 구조)
- 이름 규칙: GitHub 저장소만 `teams-poll-bot`이고, 로컬 폴더·컨테이너·DB·ZIP 파일은 `team-poll-bot`을 그대로 사용합니다.

## 사용 방법

Teams에서 `@투표곰`을 입력한 뒤, 목록에 나오는 봇 멘션을 선택해서 보내 주세요.

| 입력 | 동작 |
| --- | --- |
| `@투표곰` | 투표 준비 카드 표시 |
| `@투표곰 도움말` | 사용법 안내 |
| 그 밖의 입력 | 처리할 수 없다는 안내와 사용법 표시 |

### 투표 만들기

- 준비 카드에 제목, 후보 2~10개(한 줄에 하나), 마감, 복수 선택 여부를 적고 **투표 시작**을 누릅니다.
- 후보에는 링크를 함께 적을 수 있습니다. 예: `식당 이름 (https://naver.me/xxxx)`
- 준비 카드를 연 사람만 투표를 시작할 수 있으며, 준비 카드는 24시간 동안 유효합니다.
- 시작하면 준비 카드가 그 자리에서 투표 카드로 바뀝니다.
- 시작 후에는 제목·후보·설정을 바꿀 수 없습니다. 고치려면 새 투표를 만들어 주세요.

### 투표와 결과

- 투표 카드에 참여 인원, 총 표 수, 후보별 표 수와 투표한 사람 이름을 바로 갱신합니다.
- 마감 전에는 선택을 바꿀 수 있습니다. 같은 사람의 이전 응답은 새 선택으로 바뀝니다.
- 단일 선택과 복수 선택을 지원합니다. 복수 선택의 비율은 참여자 기준이라 합계가 100%를 넘을 수 있습니다.
- 공개 기명 투표입니다. 고른 후보와 이름이 같은 대화방 구성원에게 보입니다.
- 대화방마다 따로 저장합니다.
- 카드 갱신이 실패해도 선택은 보존하고 30초마다 다시 시도합니다. 카드의 **최신 결과 보기** 버튼으로도 갱신할 수 있습니다.

### 마감 시간

모든 마감은 한국 시간이며, 카드를 제출한 시각을 기준으로 해석합니다.

| 입력 | 의미 |
| --- | --- |
| 빈 값 | 자동 마감 없음. 작성자가 버튼으로 마감 |
| `30분`, `3시간`, `2일` | 제출 시각부터 해당 기간 뒤 마감 (양의 정수만) |
| `금요일 18시`, `금요일 18:30` | 가장 가까운 해당 요일·시각. 이미 지났으면 다음 주 |
| `8월 22일 18시`, `8월 22일 18:30` | 올해 해당 시각. 이미 지났으면 다음 해 |
| `2026-10-09 18:00` | 정확한 날짜와 시각. 지난 시각은 사용할 수 없음 |

- 마감은 365일 이내로만 정할 수 있습니다.
- 달력에 없는 날짜는 자동으로 고치지 않고 다시 입력을 요청합니다.
- 마감 시각부터 새 응답을 받지 않습니다. 카드의 마감 표시는 최대 약 30초 늦을 수 있습니다.

## 로컬 개발

준비물: Node 24 이상, npm, bash, zip, unzip

```bash
npm ci
cp .env.example .env    # 투표곰 전용 앱 정보를 입력해 주세요
npm test
npm run typecheck
npm run build
node --env-file=.env --import tsx src/index.ts    # 소스로 실행
node --env-file=.env dist/index.js                # 빌드 결과로 실행
```

- `npm start`는 `.env`를 읽지 않고 셸 환경 변수를 사용합니다.
- 인증 필수값이 없으면 서버가 시작되지 않습니다.
- `.env`와 SQLite 파일은 Git에 올리지 않습니다.

| 환경 변수 | 값 |
| --- | --- |
| `MicrosoftAppId` | Azure 봇 ID (Developer Portal의 앱 ID가 아닙니다) |
| `MicrosoftAppPassword` | 봇 클라이언트 비밀값 |
| `MicrosoftAppTenantId` | Microsoft 365 테넌트 ID |
| `PUBLIC_BASE_URL` | `https://poll-bot.namonak.dev` (기록용이며 서버 바인딩이나 프록시를 설정하지 않습니다) |
| `DATABASE_PATH` | 로컬 `./data/team-poll-bot.sqlite`, 컨테이너 `/data/team-poll-bot.sqlite` |
| `PORT` | 컨테이너 내부 `3978` |

## 보안 검사

`.github/workflows/security.yml`이 main 푸시, PR, 수동 실행, 매일 02:50 KST에 실행됩니다.

- 의존성 검사: 테스트와 타입 검사 뒤 `npm audit --omit=dev --audit-level=moderate` 실행
- 이미지 검사: Docker 이미지를 빌드해 Trivy(컨테이너 이미지 취약점 검사 도구) v0.76.2로 수정 가능한 HIGH·CRITICAL 취약점 검사
- 차단 기준: 두 검사 중 하나라도 취약점을 찾으면 워크플로가 실패합니다.
- 버전 고정: 간접 의존성 `body-parser` 2.4.0, `proxy-addr` 2.0.8을 `package.json`의 `overrides`로 고정합니다.
- 운영 이미지: 실행에 쓰지 않는 npm과 npx를 삭제합니다. 컨테이너 안에서는 `node`만 사용해 주세요.
- Dependabot: GitHub 취약점 알림과 자동 보안 업데이트를 켜 두었습니다.

## 서버 배포

1. 저장소를 내려받고 `.env`를 설정합니다. 다른 봇의 앱 ID와 비밀값은 재사용하지 않습니다.

   ```bash
   git clone https://github.com/namonak/teams-poll-bot.git ~/workspace/teams-poll-bot
   cd ~/workspace/teams-poll-bot
   cp .env.example .env
   chmod 600 .env
   ```

2. Caddyfile에 아래 설정을 추가하고, `caddy validate`로 검사한 뒤 `caddy reload`로 재시작 없이 반영합니다. HTTPS 인증서는 Caddy가 자동으로 발급하고 갱신합니다.

   ```caddyfile
   poll-bot.namonak.dev {
       reverse_proxy 127.0.0.1:3979
   }
   ```

3. `docker compose up -d --build`로 실행합니다. 구형 환경에서는 `docker-compose`를 사용합니다.
4. `curl https://poll-bot.namonak.dev/healthz`가 `{"ok":true}`를 반환하는지 확인합니다.

운영 기준

- 호스트 포트: 3979 (식사 봇 3978, 쿠웨이트박 3980과 겹치지 않음)
- 자동 실행: Compose의 `restart: unless-stopped`로 서버를 재부팅해도 다시 실행됩니다. systemd로 관리하려면 `deploy/team-poll-bot.service`의 WorkingDirectory를 서버 경로로 바꿔서 사용해 주세요.
- 데이터: `poll-data` 볼륨에 보관합니다. `docker compose down -v`는 볼륨을 삭제하므로 사용하지 않습니다.
- 실행 단위: 컨테이너 하나로만 실행합니다. 같은 DB를 여러 봇 인스턴스에 연결하지 않습니다.
- 도메인 변경 시 함께 바꿀 항목: `.env`, DNS·Caddy, Azure 봇 메시지 엔드포인트, `appPackage/manifest.json`의 `validDomains`

코드 업데이트 반영

```bash
cd ~/workspace/teams-poll-bot
git pull --ff-only
docker compose up -d --build
```

서버 코드만 바뀐 경우에는 Teams 앱을 다시 설치하지 않아도 됩니다.

## Teams 앱 등록과 설치

- 등록: Teams Developer Portal과 Azure Bot에서 투표곰 앱을 등록하고 Teams 채널을 연결합니다.
- 메시지 엔드포인트: `https://poll-bot.namonak.dev/api/messages`
- 서버 설정과 일치 필요: 봇 ID, 비밀값, 테넌트
- 조직에서 사용자 지정 앱 업로드를 허용해야 합니다.

ZIP 만들기(프로젝트 최상단에서 실행)

```bash
bash scripts/package-teams-app.sh
```

- 출력 파일: `dist/team-poll-bot-YYYYMMDD-HHMMSS.zip` (한국 시간 기준)
- 포함 파일: `manifest.json`, `color.png`(192×192), `outline.png`(32×32, 투명 배경). 비밀값은 들어가지 않습니다.
- 앱 ID: 환경 변수나 `.env`의 `MicrosoftAppId`(UUID 형식)로 manifest를 채웁니다.
- 현재 manifest 버전: 1.0.2

설치와 업데이트

- 처음 설치: Teams 앱 관리 화면에서 **사용자 지정 앱 업로드**로 ZIP을 올린 뒤 단체 대화방에 추가해 주세요.
- 업데이트: 이름·아이콘·manifest를 바꾸면 manifest의 `version`을 올리고, 새 ZIP을 같은 방법으로 업로드해 주세요. 앱을 삭제하고 다시 설치할 필요는 없습니다.
- 주의: ZIP으로 설치한 앱은 Developer Portal에서 버전을 올려도 업데이트되지 않습니다.

## 장애 확인과 복구

```bash
docker compose logs --tail=100 bot
```

| 로그 | 의미 |
| --- | --- |
| `투표 카드 갱신 재시도` | 응답은 저장했지만 카드 갱신에 실패한 상태. 30초마다 다시 시도하며, 재시작 후에도 이어서 처리합니다. |
| `투표 게시 확인 필요` | 게시 응답이 유실되었을 수 있는 상태. 카드 중복을 막기 위해 자동 재게시하지 않습니다. |

게시 여부가 불확실할 때

- HTTP 400·401·403·404·413·422처럼 게시가 명확히 거절된 경우: 원인을 해결한 뒤 원래 준비 카드를 다시 제출해 주세요.
- 그 밖의 경우: 봇을 멈추고 대화방에 실제로 게시되었는지 먼저 확인해 주세요. 확인 없이 게시 상태를 되돌리지 않습니다.
- 메시지 ID 확인: Teams 메시지의 **링크 복사** 주소에서 `/l/message/대화방/메시지ID` 부분을 확인합니다.

게시가 확인된 경우에는 아래 명령으로 메시지 ID를 연결합니다. 두 인수를 실제 값으로 바꿔서 실행해 주세요.

```bash
docker compose stop bot
docker compose run --rm --no-deps bot node --input-type=module -e '
import { DatabaseSync } from "node:sqlite";
const db = new DatabaseSync(process.env.DATABASE_PATH);
const [pollId, messageId] = process.argv.slice(1);
if (!pollId || !messageId) throw new Error("투표 ID와 메시지 ID가 필요해요");
const result = db.prepare("UPDATE poll SET message_id=?, status=\u0027open\u0027, published_revision=0 WHERE id=? AND status=\u0027publishing\u0027").run(messageId, pollId);
if (result.changes !== 1) throw new Error("복구할 준비 중 투표를 찾지 못했어요");
db.close();
' '투표 ID' 'Teams 메시지 ID'
docker compose start bot
```

게시되지 않은 것이 확실하면 같은 방식으로 `publish_state`를 `ready`로 되돌린 뒤 준비 카드를 다시 제출해 주세요. 오래된 준비 카드는 새 투표로 다시 만들어 주세요.

## 백업

- DB에는 투표 내용과 투표한 사람 이름이 들어 있습니다. 데이터와 백업 파일의 접근 권한을 제한해 주세요.

```bash
mkdir -p backups
docker compose stop bot
docker cp team-poll-bot:/data/team-poll-bot.sqlite "backups/team-poll-bot-$(date +%Y%m%d-%H%M%S).sqlite"
docker compose start bot
```

복원할 때는 봇을 멈추고 현재 DB를 따로 보관한 뒤 백업 파일을 복사해 주세요.

## Teams 실사용 확인 항목

두 계정으로 아래 항목을 확인해 주세요. 자동 테스트는 실제 Teams 설치, 화면 표시, DNS, 인증서를 확인하지 않습니다.

- `@투표곰`으로 준비 카드가, 다른 입력으로 안내와 사용법이 표시되는지
- 다른 사람이 준비 카드의 시작을 누르면 거부되는지
- 단일·복수 투표에서 선택, 응답 변경, 집계·이름·링크 표시
- 다른 사람의 수동 마감 거부, 작성자 마감과 자동 마감 후 응답 거부
- 데스크톱·모바일의 줄바꿈과 카드 갱신, 서버 재시작 후 결과 유지

## 참고 문서

- [설계 문서](docs/superpowers/specs/2026-10-05-team-poll-bot-design.md)
- [구현 계획](docs/superpowers/plans/2026-10-07-team-poll-bot.md)
