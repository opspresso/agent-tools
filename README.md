# agent-tools

Agent Studio가 동기화하는 MCP 도구 목록 저장소예요. Agent Studio의 `/tools` →
**Sync from GitHub** 버튼(또는 `POST /api/mcps/sync`)이 여기 내용을 가져가요.

## 구조

```
tools/
  <tool-name>/         # 소문자-하이픈 슬러그 (레지스트리 엔트리 이름)
    TOOL.md            # frontmatter + 마크다운 본문
```

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

`tools/url-fetch/TOOL.md`를 그대로 옮긴 예시예요. `name`은 디렉터리 이름과 같아야
하고, Service 이름(`mcp-url-fetch`)과 다를 수 있어요 — 엔트리 이름은 슬러그, 주소는
배포가 정하는 것이라서요.

- `url`은 필수예요. 없으면 등록할 대상이 없으므로 건너뛰고 사유를 보고해요.
- `description`은 **한 줄**로 써요. 모델의 시스템 프롬프트에서 "Connected MCP Servers"
  표의 한 칸이 되기 때문에 여러 줄이면 표가 깨져요. 생략하면 본문의 첫 제목을 써요.
- 본문은 운영자용 메모예요. 스킬과 달리 모델에게 가지 않아요.

## 인증 헤더는 여기에 두지 않아요

시크릿이 git에 들어갈 경로를 만들지 않기 위해, sync는 헤더를 비운 채로 엔트리를
만들어요. 토큰이 필요한 서버는 등록 후 Agent Studio 콘솔에서 헤더 값을 입력하거나
OAuth로 연결해요.

## 반영 방법과 우선순위

main에 머지하고 Sync 버튼을 누르면 **아직 등록되지 않은 이름만** 새로 등록돼요.

**이미 등록된 이름은 건드리지 않아요.** 스킬 저장소와 정반대인데, 이유가 있어요.
스킬은 문서가 곧 스킬 전체지만, MCP 엔트리는 저장소가 담을 수 없는 것들을 함께
들고 있어요 — 암호화된 헤더, 서버에서 발견한 OAuth 설정, 콘솔에서 손본 주소나 설명.
덮어쓰면 그게 전부 사라지니까, 저장된 쪽이 이겨요. 이 저장소는 *무엇이 있어야 하는지*를
선언하고, *이미 있는 것*은 소유하지 않아요.

그래서 등록된 도구의 URL을 바꾸려면 여기 파일이 아니라 콘솔에서 고쳐야 해요.

## 클러스터 내부 주소

`*.svc.cluster.local` 같은 사설 주소는 Agent Studio 배포의
`MCP_INTERNAL_HOST_SUFFIXES`에 해당 네임스페이스 suffix가 들어 있어야 등록돼요.
없으면 SSRF 가드가 거절하고, sync 결과에 사유와 함께 표시돼요.

**포트는 Service 포트예요.** 컨테이너 포트가 아니에요. `mcp-*` 차트(서버마다
하나)의 네 서버는 모두 Service `port: 80`으로 받아 각자의 `targetPort`(url-fetch·
memory 3000, grafana 8000, kubernetes 8080)로 넘기니까, 주소에는 포트를 적지
않아요. 컨테이너 포트를 그대로 적으면 Service에 없는 포트라 연결되지 않아요.
