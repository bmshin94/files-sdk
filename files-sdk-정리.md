# files-sdk 정리 노트

> 이 저장소(`files-sdk`)가 무엇인지, 어떻게 쓰는지, 어디에 활용할 수 있는지를
> 저장소 내용을 직접 확인해서 정리한 문서입니다.

## 🔗 링크 모음

| 항목 | 주소 |
|---|---|
| **이 저장소 (포크)** | https://github.com/bmshin94/files-sdk |
| **원본 저장소 (upstream)** | https://github.com/haydenbleasel/files-sdk |
| **npm 패키지** | https://www.npmjs.com/package/files-sdk |
| **공식 문서 사이트** | https://files-sdk.dev |
| **이슈 트래커** | https://github.com/haydenbleasel/files-sdk/issues |

- 원작자: **Hayden Bleasel** / 라이선스: **MIT** / 패키지 버전: **2.5.0**
- 이 저장소는 원본을 **포크**한 것이며, 직접 추가한 커밋은 `CLAUDE.md` 하나입니다.

---

## 1. 이게 뭐야? (한 줄 요약)

**여러 파일 저장소(S3, R2, GCS, Azure, Vercel Blob, Dropbox, 로컬 폴더 …)를
똑같은 코드 한 벌로 다루게 해주는 TypeScript 라이브러리.**

해외여행용 멀티 어댑터와 같은 역할입니다. 저장소 회사마다 사용법이 전부 다른데,
그 사이에 끼어서 통역을 해줍니다.

```
내 코드  →  files-sdk (통역사)  →  아마존 / 구글 / MS / 클라우드플레어 ...
```

```ts
import { Files } from "files-sdk";
import { s3 } from "files-sdk/s3";          // ← 이 줄만 바꾸면

const files = new Files({ adapter: s3({ bucket: "uploads" }) });

await files.upload("avatars/abc.png", file);  // ← 아래 코드는 그대로
await files.download("avatars/abc.png");
```

`files-sdk/s3` → `files-sdk/r2` 로 import 한 줄만 바꾸면 클라우드 이사가 끝납니다.

---

## 2. 폴더 구조

| 경로 | 설명 |
|---|---|
| `packages/files-sdk/` | **본체.** npm에 배포되는 SDK (`src/` 아래 85개 폴더) |
| `packages/videos/` | Remotion으로 만든 홍보 영상 프로젝트 (비공개) |
| `apps/web/` | 공식 문서 사이트 (files-sdk.dev), Astro + Cloudflare Workers |
| `skills/files-sdk/` | AI(Claude)용 사용설명서 `SKILL.md` + 레퍼런스 8종 |
| `.changeset/`, `turbo.json`, `bun.lock` | 모노레포 관리 (Bun + Turborepo + Changesets) |

### 본체 구성

- **어댑터 50개 이상** — s3, r2, gcs, azure, vercel-blob, supabase, firebase,
  dropbox, google-drive, onedrive, box, sharepoint, cloudinary, minio,
  backblaze-b2, wasabi, ftp, sftp, webdav, `fs`(로컬), `memory`(테스트용) …
- **프레임워크 연동** — next, hono, express, fastify, koa, nestjs, astro,
  sveltekit, tanstack-start, react, vue, svelte
- **플러그인 17종** — encryption, compression, cache, versioning, soft-delete,
  audit, failover, dedup, validation, tiering, zip, tracing, usage,
  content-type, signed-url-policy, api
- **CLI + MCP 서버** — 터미널 명령어 및 AI 에이전트 연결
- **AI 툴** — Vercel AI SDK / OpenAI / Claude Agent SDK 바인딩
- **UI 컴포넌트 13종** — shadcn 스타일 레지스트리
  (dropzone, file-list, file-browser, file-preview, upload-progress,
  multipart-uploader, trash-bin, version-history, share-dialog, file-search …)

---

## 3. 설치 및 사용법

### 설치

```sh
npm install files-sdk
```

저장소별 SDK는 **optional peer dependency** 라 필요한 것만 따로 설치합니다.

```sh
# S3 계열
npm install files-sdk @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
# Google Cloud Storage
npm install files-sdk @google-cloud/storage google-auth-library
# Vercel Blob
npm install files-sdk @vercel/blob
```

### 가장 쉬운 시작 (가입/토큰 불필요)

```ts
import { Files } from "files-sdk";
import { fs } from "files-sdk/fs";        // 내 컴퓨터 폴더

const files = new Files({ adapter: fs({ root: "./uploads" }) });

await files.upload("hello.txt", "안녕!");
const got = await files.download("hello.txt");
console.log(await got.text());
```

나중에 `fs` → `s3` 로 바꾸기만 하면 실서비스로 전환됩니다.

### 자주 쓰는 메서드

```ts
await files.upload("a.png", file);              // 올리기
await files.download("a.png");                  // 받기
await files.exists("a.png");                    // 존재 확인
await files.head("a.png");                      // 메타데이터만
await files.delete("a.png");                    // 삭제
await files.copy("a.png", "b.png");             // 복사
await files.move("a.png", "b.png");             // 이동/이름변경
await files.list({ prefix: "photos/" });        // 목록
await files.url("a.png", { expiresIn: 300 });   // 임시 공유 링크
```

### CLI

```sh
npx -p files-sdk files --provider s3 --bucket uploads list --prefix reports/
```

---

## 4. 플러그인? 스킬? MCP? → 전부 다

**라이브러리가 본체**이고, 나머지는 같은 기능을 다른 창구로 열어준 것입니다.

| 형태 | 정체 | 사용 주체 |
|---|---|---|
| npm 라이브러리 | `import { Files }` | 내 앱 코드 |
| CLI 도구 | `files upload ...` | 터미널 |
| MCP 서버 | `files ... mcp` | Claude 등 AI 앱 |
| Skill | `skills/files-sdk/SKILL.md` | AI에게 주는 사용설명서 |

### MCP 설정 예시

```jsonc
{
  "mcpServers": {
    "files-sdk": {
      "command": "files",
      "args": ["--provider", "s3", "--bucket", "uploads", "mcp"],
      "env": {
        "AWS_ACCESS_KEY_ID": "...",
        "AWS_SECRET_ACCESS_KEY": "..."
      }
    }
  }
}
```

**보안 설계**

- 기본이 **읽기 전용** (download, head, exists, list, url, capabilities)
- 쓰기는 `--allow-writes` 를 명시해야 열림
- 자격증명은 서버 시작 시에만 바인딩 → **AI에게 비밀키가 전달되지 않음**
- 다운로드 상한 10MB (에이전트 컨텍스트 폭발 방지)

---

## 5. API 토큰이 필요한가?

| 어댑터 | 토큰 | 비고 |
|---|---|---|
| `fs` | 불필요 | 로컬 폴더 |
| `memory` | 불필요 | 메모리(테스트용) |
| `s3`, `r2`, `minio` … | 필요 | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` |
| `vercel-blob` | 필요 | `BLOB_READ_WRITE_TOKEN` |
| `gcs`, `firebase` | 필요 | 서비스 계정 JSON 키 |
| `dropbox`, `google-drive`, `box` | 필요 | OAuth |
| `uploadthing` | 필요 | `UPLOADTHING_TOKEN` |

각 어댑터가 환경변수를 알아서 읽으므로 코드에 키를 적을 필요가 없습니다.

### 보안 주의사항 2가지

```ts
// 1) 브라우저 업로드용 서명 URL에는 maxSize 필수
//    없으면 그 URL로 무제한 크기 업로드가 가능 → 요금 폭탄
await files.signedUploadUrl("a.png", { expiresIn: 300, maxSize: 5_000_000 });

// 2) 사용자 업로드 파일 링크에는 attachment 지정
//    없으면 악성 HTML/SVG가 버킷 오리진에서 실행됨 (저장형 XSS)
await files.url("user.html", { responseContentDisposition: "attachment" });
```

---

## 6. 왜 GitHub에서 인기가 있을까

> 실제 스타 수는 확인하지 않았습니다. 아래는 매력 요인 분석입니다.

1. **원작자 인지도** — `next-forge` 등을 만든 유명 개발자
2. **모두가 겪는 불편 해결** — AWS SDK 문서 지옥을 3줄로 축약
3. **압도적 완성도** — 어댑터 50개+, 프레임워크 17개+, 플러그인 17종,
   문서 사이트와 홍보 영상까지 자체 제작
4. **타이밍** — MCP + Claude/OpenAI/Vercel AI SDK 툴 기본 제공 (AI 에이전트 붐)
5. **정직한 문서** — "이 어댑터는 이 기능이 안 됨"까지 명시.
   지원하지 않는 기능은 조용히 무시하지 않고 `FilesError` 를 던지는 설계 철학

---

## 7. 로컬 에이전트 구축에 도움이 되는가 → 매우 도움됨

### 방법 A: MCP로 붙이기 (코딩 0줄)

```jsonc
{
  "mcpServers": {
    "files": {
      "command": "npx",
      "args": ["-p", "files-sdk", "files", "--provider", "fs",
               "--root", "./workspace", "mcp"]
    }
  }
}
```

### 방법 B: 코드로 툴 주입

```ts
import { Files } from "files-sdk";
import { createClaudeFileTools } from "files-sdk/claude";
import { fs } from "files-sdk/fs";

const files = new Files({ adapter: fs({ root: "./workspace" }) });

const tools = createClaudeFileTools({
  files,
  readOnly: false,
  requireApproval: { deleteFile: true, uploadFile: false },
});
```

`files-sdk/ai-sdk`(Vercel), `files-sdk/openai`, `files-sdk/claude` 세 종류 제공.

### 에이전트에 유용한 안전장치

| 기능 | 효과 |
|---|---|
| `readOnly: true` | AI가 변경 불가 |
| `requireApproval` | 위험 작업만 사람 승인 |
| `maxBytes` | 대용량 파일로 컨텍스트 폭발 방지 |
| `prefix` / `authorize` | AI를 특정 폴더에 격리 |
| `audit()` | AI 행동 전체 로그 |
| `softDelete()` | AI가 지워도 휴지통행 (복구 가능) |
| `memory` 어댑터 | 가짜 저장소로 테스트 |

---

## 8. React / PHP 에서 쓸 수 있는가

### React → 전용 훅과 UI 컴포넌트가 이미 존재

```tsx
import { useFiles } from "files-sdk/react";

function Uploader() {
  const { upload, uploads, progress, isUploading, error } = useFiles({
    endpoint: "/api/files",
  });

  return <input type="file" onChange={(e) => upload(e.target.files[0])} />;
}
```

Vue, Svelte, React Native 클라이언트도 제공됩니다.

**구조 (비밀키는 절대 브라우저에 두지 않음)**

```
[브라우저 React]  →  [내 서버 /api/files]  →  [S3]
   useFiles            createFilesRouter
```

서버 게이트웨이(`files-sdk/api`)는 **기본이 전부 차단(deny-by-default)** 이며
`authorize` 로 허용한 작업만 통과합니다. 멀티테넌트도 간단합니다.

```ts
const router = createFilesRouter({
  files: createFiles({ adapter: minio({ bucket: process.env.S3_FILES_BUCKET }) }),
  authorize: async ({ req }) => {
    const userId = await requireUser(req);      // throw 하면 거부
    return { keyPrefix: `users/${userId}/` };   // 사용자별 격리
  },
});
```

### PHP → 직접 사용 불가 (JS/TS 전용), 우회로 3가지

1. **Node 서버 분리 (권장)** — PHP → HTTP → 작은 Node 게이트웨이 → S3
2. **CLI 호출** — `shell_exec('files ... list ...')`, 출력이 기본 JSON.
   단, 사용자 입력을 그대로 넣으면 명령어 주입 위험
3. **PHP 네이티브 대안** — `league/flysystem` (같은 컨셉의 PHP 라이브러리)

---

## 9. 수익화 아이디어

### 전제

files-sdk 자체는 MIT 무료라 이것만으로는 돈이 되지 않습니다.
**"개발 기간을 3개월 → 3주로 줄여서 남들보다 먼저 판다"** 가 핵심 전략입니다.

### 활용 가능한 무기 (저장소에서 확인함)

| 무기 | 기능 | 수익 연결점 |
|---|---|---|
| `usage()` | 작업 횟수 + 업/다운 바이트 집계 | **종량제 과금 미터기** |
| `dedup()` | SHA-256 콘텐츠 주소화, 동일 파일 1회만 저장 | 스토리지 원가 절감 = 마진 |
| `tiering()` | hot/cold 자동 분산 (크기·prefix·나이 기준) | 원가 절감 |
| `audit()` | who/what/when 기록을 await 보장으로 기록 | 컴플라이언스 = 기업 결제 |
| `softDelete()` | 휴지통 (`trashed`/`restore`/`purge`) | 상위 요금제 기능 |
| `versioning()` | 덮어쓰기 전 스냅샷 + 롤백 | 상위 요금제 기능 |
| `encryption()` | AES-256-GCM 봉투 암호화 (Web Crypto) | 기업 계약 요건 |
| `failover()` | 장애 시 보조 어댑터로 자동 전환 | SLA 보장 |
| `sync()` | `prune`, `compare:"etag"`, **`dryRun`** | 마이그레이션 툴 |
| 멀티테넌트 게이트웨이 | `authorize` → `keyPrefix` | SaaS 뼈대 |

### 티어 A — 빨리 현금이 되는 것 (외주/수주형)

**A-1. 사내 파일 AI 챗봇 구축** (최고 추천)
- 대상: 직원 20~200명 중소기업
- 구성: 사내문서 → S3/NAS → files-sdk MCP(읽기전용) → Claude
- 강점: MCP는 설정 파일 1개, 읽기 전용이 기본값, `audit()` 로 조회 이력 전체 기록
- 가격: 구축 400~1,500만원 + 유지보수 월 30~100만원 / 기간 2~4주
- 리스크: 검색 품질(RAG)은 별도 작업. 초기에는 문서 정리가 된 고객만 수주

**A-2. 클라우드 이사 대행 (비용 절감 컨설팅)**
- 대상: AWS S3 전송료(egress) 부담이 큰 회사 (R2는 전송료 0원)
- 강점: `sync(from, to, { dryRun: true })` 로 **견적서 자동 생성**
- 가격: 절감액의 20~30%를 6개월 셰어 (성과보수형) / 기간 견적 1일, 실행 2~5일
- 리스크: 데이터 유실 공포. `prune: false` + 원본 유지 + 검증기간 2주 필수

**A-3. 파일 업로드 모듈 납품**
- 대상: 업로드 기능이 필요한 스타트업/에이전시
- 강점: UI 컴포넌트 13종이 이미 존재
- 가격: 건당 200~500만원 또는 소스 패키지 30~50만원 반복 판매
- 기간: 첫 개발 2주, 이후 납품 2~3일

### 티어 B — 제품형 (SaaS)

**B-1. AI 에이전트 전용 스토리지** (성장성 최고)
- 대상: AI 에이전트를 만드는 개발자/스타트업
- 해결: "AI에게 파일 권한을 주기는 무섭고, 안 주면 쓸모없다"
- 구성: `softDelete` + `versioning` + `audit` + `prefix` + `requireApproval` + `usage`
- 가격: 무료(1GB) / Pro 월 2만원 / Team 월 10만원 + 종량제 / MVP 6~10주
- 리스크: 개발자 대상은 지불 의사가 낮음 → 직접 만들기 귀찮은 영역(감사 로그, 롤백)에 집중

**B-2. 안심 백업 서비스 (개인/소상공인)**
- 구성: `encryption` + `versioning` + `dedup` + `failover`
- 수익 구조: 중복률 20~40% → 100GB 판매, 실제 65GB 저장 = 차액이 마진
- 가격: 100GB 월 4,900원 / 1TB 월 14,900원 / MVP 8~12주
- 리스크: 구글·네이버 무료 용량과 경쟁. "암호화 + 국내 저장" 같은 차별점 필요

**B-3. 규제 대응 파일 보관함 (B2B, 마진 최고)**
- 대상: 병원, 법률·회계 사무소, 금융 등 기록 보존 의무 업종
- 구성: `audit` + `encryption` + `versioning` + `softDelete`
- 강점: `audit()` 가 await 보장이라 "기록 누락 없음"을 기술적으로 주장 가능
- 가격: 기업당 월 30~200만원 / MVP 8주 + 인증·영업 6개월~
- 리스크: ISMS-P·의료법·개인정보보호법 등 진입장벽 높음. 업계 파트너 없이는 비권장

### 티어 C — 비용 0원, 리턴이 큰 것

**C-1. 국내 클라우드 어댑터 기여** (가성비 1위)
- 현재 어댑터 50여 개 중 **국내 클라우드가 하나도 없음**
  (네이버클라우드, 카카오클라우드, NHN Cloud, KT Cloud)
- 대부분 S3 호환이라 `s3()` 를 감싸고 기본값만 지정하면 됨
  (기존 `wasabi`, `tigris` 어댑터가 그 패턴)
- 얻는 것: 해외 인기 OSS 컨트리뷰터 이력, 국내 포지션 선점, A-1·A-2 영업 시 신뢰도

**C-2. 콘텐츠로 영업 깔때기 만들기**
- "Next.js 파일 업로드 완벽 가이드" 등 → 구축 문의 → A-1/A-3 외주 연결

### 추천 실행 순서

```
1개월차   C-1 어댑터 기여 + A-3 모듈 1개 완성   (비용 0원, 이력·실력)
2~3개월차 A-1 사내 AI 챗봇 고객 1~2곳 수주     (현금 확보 + 시장 검증)
4~6개월차 반복되는 요구사항을 제품화           (B-1 또는 B-3)
```

이유: ① SaaS는 6개월간 수익이 없으므로 외주 현금이 필요 ②
상상으로 만든 제품은 실패하기 쉬우므로 고객 목소리를 먼저 확보 ③
A-1을 수행하는 과정에서 B-1/B-3에 필요한 기술이 자연히 축적됨

### 리스크

| 리스크 | 현실 | 대응 |
|---|---|---|
| 기술적 해자 없음 | 누구나 무료로 사용 가능 | 고객관계·도메인지식으로 승부 |
| 경쟁자 존재 | Uploadthing, Cloudinary, Filestack | 정면승부 대신 국내·특정 업종 니치 |
| 클라우드 재판매 마진 | 전송료가 변수 | 종량제 필수, 정액제는 위험 |
| AI 챗봇 품질 | MCP만으로는 검색 정확도 부족 | RAG 별도 구축, 기대치 관리 |
| 개인정보 | 파일 취급 시 필연적 | 처리방침·위탁계약 사전 준비 |
| MIT 라이선스 | 상업적 사용 가능 | 단, 저작권 고지 포함 의무 |
| 원본 저장소 변경 | 방향 전환 가능성 | 버전 고정 사용 (`files-sdk@2.5.0`) |

---

## 10. 핵심 정리

| 질문 | 답 |
|---|---|
| 설치 | `npm i files-sdk` + 사용할 저장소 SDK. `fs` 로 시작하면 가입 불필요 |
| 정체 | 라이브러리가 본체. CLI·MCP·Skill은 같은 기능의 다른 창구 |
| 토큰 | `fs`/`memory` 는 불필요, 클라우드는 환경변수로 주입 |
| 인기 요인 | 원작자 인지도 + 보편적 불편 해결 + 완성도 + AI 타이밍 |
| 로컬 에이전트 | 매우 유용. MCP 설정 한 개로 연결, 안전장치 완비 |
| React | 전용 훅 + UI 컴포넌트 13종 제공 |
| PHP | 직접 사용 불가. Node 게이트웨이 분리 / CLI 호출 / Flysystem |
| 수익화 | 사내 AI 챗봇 구축 → 클라우드 이사 대행 → 제품화 순 |

> **결론: files-sdk가 대단해서 돈이 되는 것이 아니라,
> 이것을 쓰면 훨씬 빨리 만들 수 있어서 기회가 생긴다.**
