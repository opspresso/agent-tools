# agent-tools

Agent Studio가 동기화하는 MCP 도구 목록 저장소예요. Agent Studio의 `/tools` →
**Sync from GitHub** 버튼(또는 `POST /api/mcps/sync`)이 여기 내용을 가져가요.

## 구조

```
tools/
  <tool-name>/         # 소문자-하이픈 슬러그 (레지스트리 엔트리 이름)
    TOOL.md            # frontmatter + 마크다운 본문
```

엔트리 이름은 **디렉터리 이름**이에요. sync는 `<디렉터리>/TOOL.md`를 찾아 부모
디렉터리명을 쓰고, `tools/` 아래 두는 건 관례일 뿐이에요. 슬러그는 소문자·숫자·
하이픈만 허용돼요 (`^[a-z0-9-]+$`).

## TOOL.md 형식

```markdown
---
name: url-fetch
description: "Fetch a URL as usable content: images as bytes, documents (HTML, PDF, CSV, JSON) as text."
url: http://mcp-url-fetch.agent-mcps.svc.cluster.local/mcp
---

# url-fetch

운영 메모를 마크다운으로 적어요. 콘솔에만 보이고 모델에는 전달되지 않아요.
```

`tools/url-fetch/TOOL.md`를 그대로 옮긴 예시예요. 엔트리 이름은 Service
이름(`mcp-url-fetch`)과 다를 수 있어요 — 이름은 슬러그, 주소는 배포가 정하는
것이라서요.

- frontmatter에서 **실제로 읽는 건 `url`과 `description` 둘뿐이에요.** `name`은
  파싱하지 않아요. 사람이 읽기 좋으라고 남겨두는 건 괜찮지만 디렉터리가 이겨요.
- 완전한 YAML이 아니에요. 평평한 `key: value`와 `>`·`|` 폴딩 스칼라만 읽고 나머지는
  조용히 무시해요.
- `url`은 필수예요. 없으면 등록할 대상이 없으므로 `missing-url`로 건너뛰어요.
- `description`은 **한 줄로 짧게** 써요. 모델의 시스템 프롬프트에서 "Connected MCP
  Servers" 표의 한 칸이 되니까요. 줄바꿈과 `|`는 엔진이 접고 이스케이프하므로 표가
  깨지지는 않지만, 길면 그만큼 매 런의 프롬프트를 먹어요. 여러 줄로 써야 하면
  `>` 폴딩을 쓰면 한 줄로 접혀요 (`tools/memory`가 그렇게 해요).
- `description`을 생략하면 본문의 첫 제목(없으면 첫 줄)을 200자까지 잘라 써요.
- 본문은 운영자용 메모예요. 스킬과 달리 **모델에게 가지 않아요.** 설치 절차,
  자격증명 경로, 배포 쪽에서 켜고 끄는 것 같은 걸 적어요.

## 인증 헤더는 여기에 두지 않아요

시크릿이 git에 들어갈 경로를 만들지 않기 위해, sync는 헤더를 비운 채로 엔트리를
만들어요. 토큰이 필요한 서버는 등록 후 Agent Studio 콘솔에서 헤더 값을 입력하거나
OAuth로 연결해요.

## 반영 방법

main에 머지하고 Sync 버튼을 누르면 **아직 등록되지 않은 이름만** 새로 등록돼요.

**이미 등록된 이름은 건드리지 않아요.** 콘솔에서 누군가 고쳤을 수 있고, sync는
그것과 "문서가 그냥 앞서간 것"을 구분할 방법이 없어서요. 대신 문서가 바꿀 필드를
`existing`으로 보고하고, 콘솔에서 그 이름을 지목해야(`overwrite`) 실제로 덮어써요.
스킬 저장소도 지금은 똑같이 동작해요 — 예전엔 스킬만 무조건 덮어썼지만 아니에요.

덮어쓸 때 움직이는 건 문서가 소유한 **`url`·`description`·`content`** 셋뿐이에요.
암호화된 헤더, 서버에서 발견한 OAuth 블록, managed 엔트리의 주소는 손대지 않아요.
그래서 등록된 도구의 URL도 여기서 고치고 overwrite로 지목하면 바뀌어요. 다만:

- **주소를 옮기면 이전 서버에서 읽은 OAuth 블록이 버려져요.** Discover를 다시
  돌려야 해요.
- **managed 엔트리의 주소는 못 옮겨요.** 포트를 바인딩한 프로비저너가 기록한
  값이라서요. 나머지 필드는 적용되고 `managed-url` skip으로 보고돼요.

**디렉터리를 지워도 엔트리는 사라지지 않아요.** 이전 sync가 만든 엔트리가 저장소에서
없어지면 `orphaned`로 **보고만** 되고, 콘솔에서 지목해야(`remove`) 삭제돼요. MCP
엔트리는 자격증명을 들고 있어서, 브랜치에서 파일이 없어졌다는 이유만으로 지울 수는
없어요. 콘솔에서 직접 등록한 엔트리는 애초에 이 저장소 것이 아니라 목록에 뜨지도
않아요.

`POST /api/mcps/sync`는 admin 권한이 필요하고 `{ overwrite?: [...], remove?: [...] }`를
받아요. 건너뛴 문서는 `skipped`에 사유와 함께 담겨요 — `bad-name`, `missing-url`,
`invalid-url`, `managed-url`, `conflict`.

## 클러스터 내부 주소

`*.svc.cluster.local` 같은 사설 주소는 Agent Studio 배포의
`MCP_INTERNAL_HOST_SUFFIXES`에 해당 네임스페이스 suffix가 들어 있어야 등록돼요.
없으면 SSRF 가드가 거절하고, sync 결과에 `invalid-url`로 표시돼요.

매칭은 라벨 경계에 걸려요 — `agent-mcps.svc.cluster.local`은
`mcp-url-fetch.agent-mcps.svc.cluster.local`을 허용하지만
`evil-agent-mcps.svc.cluster.local`은 아니에요. 점이 없는 단일 라벨(`local`,
`internal`)은 거부되고, IP 리터럴은 어떤 suffix에도 매칭되지 않아요.

**포트는 Service 포트예요.** 컨테이너 포트가 아니에요. `mcp-*` 차트(서버마다 하나)의
일곱 서버는 모두 Service `port: 80`으로 받아 각자의 `targetPort`로 넘기니까, 주소에는
포트를 적지 않아요. 컨테이너 포트를 그대로 적으면 Service에 없는 포트라 연결되지
않아요.

| 서버 | targetPort |
|---|---|
| url-fetch, memory, youtube, argocd | 3000 |
| grafana | 8000 |
| kubernetes, brave-search | 8080 |
