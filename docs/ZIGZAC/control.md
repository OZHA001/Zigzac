# Claude Agent 제어 도구 목록 (cli.js 추출)

> **출처:** `cli.js` 내 `pc` 배열(`BROWSER_TOOLS`) + macOS 컴퓨터 제어 함수  
> **총계:** 42개 도구 (브라우저 18 + OS 24)  
> **분해 깊이:** 1단계 (name / description / parameters key / required)

---

## 브라우저 제어 도구 (`pc` 배열, 18개)

> Claude-in-Chrome MCP 서버를 통해 크롬 브라우저를 직접 조작.  
> 도구명은 `mcp__claude-in-chrome__[name]` 형식으로 노출됨.

```json
{
  "source": "pc (BROWSER_TOOLS)",
  "count": 18,
  "tools": [
    {
      "name": "javascript_tool",
      "description": "현재 페이지 컨텍스트에서 JS 코드 실행. DOM, window, 페이지 변수 접근 가능. 마지막 표현식 결과 반환.",
      "parameters": {
        "action": "string — 반드시 'javascript_exec' 고정",
        "text": "string — 실행할 JavaScript 코드",
        "tabId": "number — 실행 대상 탭 ID"
      },
      "required": ["action", "text", "tabId"]
    },
    {
      "name": "read_page",
      "description": "페이지의 접근성 트리(Accessibility Tree) 반환. 기본 50000자 제한. 비가시 요소 포함.",
      "parameters": {
        "filter": "string — 'interactive' | 'all'",
        "tabId": "number — 대상 탭 ID",
        "depth": "number — 트리 최대 깊이 (기본 15)",
        "ref_id": "string — 특정 부모 요소 ref ID",
        "max_chars": "number — 최대 출력 문자 수 (기본 50000)"
      },
      "required": ["tabId"]
    },
    {
      "name": "find",
      "description": "자연어로 페이지 요소 탐색. '검색창', '로그인 버튼' 같은 목적어로 검색. 최대 20개 결과 반환.",
      "parameters": {
        "query": "string — 자연어 탐색 쿼리",
        "tabId": "number — 대상 탭 ID"
      },
      "required": ["query", "tabId"]
    },
    {
      "name": "form_input",
      "description": "read_page에서 얻은 ref_id로 폼 요소 값 설정. 체크박스는 boolean, select는 option 값/텍스트.",
      "parameters": {
        "ref": "string — read_page에서 얻은 요소 ref ID",
        "value": "string | boolean | number — 설정할 값",
        "tabId": "number — 대상 탭 ID"
      },
      "required": ["ref", "value", "tabId"]
    },
    {
      "name": "computer",
      "description": "마우스/키보드로 브라우저 직접 조작 (클릭, 입력, 스크롤, 드래그, 스크린샷 등 복합 액션 툴).",
      "parameters": {
        "action": "string — 수행할 액션 (left_click / right_click / double_click / triple_click / type / key / scroll / drag / screenshot 등)",
        "coordinate": "array — [x, y] 좌표",
        "text": "string — 입력할 텍스트 또는 키",
        "duration": "number — wait 액션 시간 (최대 30초)",
        "scroll_direction": "string — 스크롤 방향",
        "scroll_amount": "number — 스크롤 틱 수 (기본 3)",
        "start_coordinate": "array — 드래그 시작 좌표",
        "region": "array — 스크린샷 영역",
        "repeat": "number — 키 반복 횟수 (1~100)",
        "ref": "string — 요소 ref ID",
        "modifiers": "string — 수정자 키",
        "tabId": "number — 대상 탭 ID"
      },
      "required": ["action", "tabId"]
    },
    {
      "name": "navigate",
      "description": "URL로 이동하거나 브라우저 히스토리 앞/뒤로 탐색.",
      "parameters": {
        "url": "string — 이동할 URL (또는 'back' | 'forward')",
        "tabId": "number — 대상 탭 ID"
      },
      "required": ["url", "tabId"]
    },
    {
      "name": "resize_window",
      "description": "브라우저 창 크기를 지정된 픽셀로 변경. 반응형 디자인 테스트에 유용.",
      "parameters": {
        "width": "number — 목표 너비 (px)",
        "height": "number — 목표 높이 (px)",
        "tabId": "number — 대상 탭 ID"
      },
      "required": ["width", "height", "tabId"]
    },
    {
      "name": "gif_creator",
      "description": "브라우저 자동화 세션의 GIF 녹화 및 내보내기 관리. 클릭 표시, 액션 레이블, 진행 바, 워터마크 오버레이 포함.",
      "parameters": {
        "action": "string — 'start_recording' | 'stop_recording' | 'export' | 'clear'",
        "tabId": "number — 대상 탭 그룹 ID",
        "download": "boolean — export 시 브라우저로 다운로드 여부",
        "filename": "string — 내보내기 파일명 (기본: recording-[timestamp].gif)",
        "options": "object — GIF 옵션 객체",
        "showDragPaths": "boolean — 드래그 경로 표시 (기본 true)",
        "showActionLabels": "boolean — 액션 레이블 표시 (기본 true)",
        "showProgressBar": "boolean — 진행 바 표시 (기본 true)",
        "showWatermark": "boolean — Claude 로고 워터마크 표시 (기본 true)",
        "quality": "number — GIF 압축 품질 1~30 (낮을수록 고품질, 기본 10)"
      },
      "required": ["action", "tabId"]
    },
    {
      "name": "upload_image",
      "description": "캡처된 스크린샷 또는 사용자 업로드 이미지를 파일 input/드래그&드롭 대상에 업로드.",
      "parameters": {
        "imageId": "string — 이전에 캡처한 스크린샷 ID 또는 사용자 업로드 이미지 ID",
        "ref": "string — 파일 input 요소 ref ID",
        "coordinate": "array — 드래그&드롭 대상 좌표",
        "tabId": "number — 대상 탭 ID",
        "filename": "string — 업로드 파일명"
      },
      "required": ["imageId", "tabId"]
    },
    {
      "name": "get_page_text",
      "description": "페이지의 원문 텍스트 추출 (기사 콘텐츠 우선). HTML 포맷 없이 플레인 텍스트 반환.",
      "parameters": {
        "tabId": "number — 대상 탭 ID"
      },
      "required": ["tabId"]
    },
    {
      "name": "tabs_context_mcp",
      "description": "현재 MCP 탭 그룹 정보 조회. 그룹 내 모든 탭 ID 반환. 다른 브라우저 도구 사용 전 반드시 먼저 호출.",
      "parameters": {
        "createIfEmpty": "boolean — 탭 그룹이 없으면 새 창/그룹/탭 생성"
      },
      "required": []
    },
    {
      "name": "tabs_create_mcp",
      "description": "MCP 탭 그룹 내 새 빈 탭 생성. 사용 전 tabs_context_mcp로 컨텍스트 획득 필수.",
      "parameters": {},
      "required": []
    },
    {
      "name": "update_plan",
      "description": "액션 실행 전 사용자에게 계획 승인 요청. 방문할 도메인과 접근 방식 표시. 승인 후 해당 도메인에서 추가 권한 없이 진행.",
      "parameters": {
        "domains": "array — 방문 예정 도메인 목록",
        "approach": "array — 실행 단계 목록"
      },
      "required": ["domains", "approach"]
    },
    {
      "name": "read_console_messages",
      "description": "특정 탭의 브라우저 콘솔 메시지 읽기 (console.log/error/warn 등). 현재 도메인 메시지만 반환.",
      "parameters": {
        "tabId": "number — 대상 탭 ID",
        "onlyErrors": "boolean — true면 에러/예외만 반환 (기본 false)",
        "clear": "boolean — 읽은 후 메시지 지우기 (기본 false)",
        "pattern": "string — 메시지 필터 정규식",
        "limit": "number — 최대 반환 메시지 수 (기본 100)"
      },
      "required": ["tabId"]
    },
    {
      "name": "read_network_requests",
      "description": "특정 탭의 HTTP 네트워크 요청 기록 조회 (XHR, Fetch, 문서, 이미지 등). 크로스 오리진 요청 포함.",
      "parameters": {
        "tabId": "number — 대상 탭 ID",
        "urlPattern": "string — URL 필터 패턴 (예: '/api/')",
        "clear": "boolean — 읽은 후 요청 기록 지우기 (기본 false)",
        "limit": "number — 최대 반환 요청 수 (기본 100)"
      },
      "required": ["tabId"]
    },
    {
      "name": "shortcuts_list",
      "description": "사용 가능한 단축키/워크플로우 목록 반환. 명령어, 설명, 워크플로우 여부 포함.",
      "parameters": {
        "tabId": "number — 대상 탭 ID"
      },
      "required": ["tabId"]
    },
    {
      "name": "shortcuts_execute",
      "description": "단축키 또는 워크플로우를 새 사이드패널 창에서 실행. 즉시 반환 (완료 대기 없음).",
      "parameters": {
        "tabId": "number — 대상 탭 ID",
        "shortcutId": "string — 실행할 단축키 ID",
        "command": "string — 단축키 명령어 이름 (슬래시 제외)"
      },
      "required": ["tabId"]
    },
    {
      "name": "switch_browser",
      "description": "브라우저 자동화에 사용할 Chrome 브라우저 전환. 확장이 설치된 모든 Chrome에 연결 요청 브로드캐스트.",
      "parameters": {},
      "required": []
    }
  ]
}
```

---

## macOS 컴퓨터 제어 도구 (OS Control Function, 24개)

> 네이티브 macOS 접근성/입력 API를 통해 OS를 직접 제어.  
> 앱 허용 목록(allowlist) 기반 권한 체계로 보호됨.

```json
{
  "source": "getComputerTools() return value",
  "count": 24,
  "tools": [
    {
      "name": "request_access",
      "description": "세션에서 제어할 앱 목록 권한 요청. 모든 다른 도구보다 먼저 호출 필수. 사용자에게 일괄 승인/거부 다이얼로그 표시.",
      "parameters": {
        "apps": "array — 제어 허용 요청할 앱 이름 목록",
        "reason": "string — 사용자에게 보여줄 한 문장 이유",
        "clipboardRead": "boolean — 클립보드 읽기 권한 추가 요청",
        "clipboardWrite": "boolean — 클립보드 쓰기 권한 추가 요청",
        "systemKeyCombos": "boolean — 시스템 레벨 키 조합 권한 요청 (앱 종료, 앱 전환, 화면 잠금 등)"
      },
      "required": ["apps", "reason"]
    },
    {
      "name": "screenshot",
      "description": "기본 디스플레이 전체 스크린샷 캡처. 모든 열린 창 포함 (필터 없음). 허용 목록 외 앱 조작 거부.",
      "parameters": {
        "save_to_disk": "boolean — 디스크에 저장하여 사용자에게 첨부 (경로 반환)"
      },
      "required": []
    },
    {
      "name": "zoom",
      "description": "마지막 전체 스크린샷의 특정 영역을 고해상도로 확대 캡처. 소형 텍스트, 버튼 레이블 확인에 활용. 이후 클릭 좌표는 전체 화면 기준.",
      "parameters": {
        "region": "array — [x, y, width, height] 영역 좌표",
        "save_to_disk": "boolean — 디스크에 저장 여부"
      },
      "required": ["region"]
    },
    {
      "name": "left_click",
      "description": "지정 좌표에서 좌클릭. 최전면 앱이 허용 목록에 있어야 함.",
      "parameters": {
        "coordinate": "array — [x, y] 클릭 좌표"
      },
      "required": ["coordinate"]
    },
    {
      "name": "double_click",
      "description": "지정 좌표에서 더블클릭. 대부분의 텍스트 편집기에서 단어 선택.",
      "parameters": {
        "coordinate": "array — [x, y] 클릭 좌표"
      },
      "required": ["coordinate"]
    },
    {
      "name": "triple_click",
      "description": "지정 좌표에서 트리플클릭. 대부분의 텍스트 편집기에서 줄 선택.",
      "parameters": {
        "coordinate": "array — [x, y] 클릭 좌표"
      },
      "required": ["coordinate"]
    },
    {
      "name": "right_click",
      "description": "지정 좌표에서 우클릭. 컨텍스트 메뉴 열기.",
      "parameters": {
        "coordinate": "array — [x, y] 클릭 좌표"
      },
      "required": ["coordinate"]
    },
    {
      "name": "middle_click",
      "description": "지정 좌표에서 가운데 클릭(스크롤 휠 클릭).",
      "parameters": {
        "coordinate": "array — [x, y] 클릭 좌표"
      },
      "required": ["coordinate"]
    },
    {
      "name": "type",
      "description": "현재 키보드 포커스 위치에 텍스트 입력. 줄바꿈 지원. 단축키는 key 도구 사용.",
      "parameters": {
        "text": "string — 입력할 텍스트"
      },
      "required": ["text"]
    },
    {
      "name": "key",
      "description": "키 또는 키 조합 입력 (예: 'cmd+c', 'Return', 'escape'). 반복 횟수 지정 가능.",
      "parameters": {
        "text": "string — 키 이름 또는 조합",
        "repeat": "integer — 반복 횟수 (기본 1)"
      },
      "required": ["text"]
    },
    {
      "name": "scroll",
      "description": "지정 좌표에서 스크롤.",
      "parameters": {
        "coordinate": "array — [x, y] 스크롤 위치",
        "scroll_direction": "string — 'up' | 'down' | 'left' | 'right'",
        "scroll_amount": "integer — 스크롤 틱 수"
      },
      "required": ["coordinate", "scroll_direction", "scroll_amount"]
    },
    {
      "name": "left_click_drag",
      "description": "마우스 누름 → 목표 좌표로 이동 → 해제 (드래그 앤 드롭).",
      "parameters": {
        "coordinate": "array — [x, y] 드래그 목표 좌표",
        "start_coordinate": "array — [x, y] 드래그 시작 좌표"
      },
      "required": ["coordinate"]
    },
    {
      "name": "mouse_move",
      "description": "클릭 없이 마우스 커서만 이동. 호버 상태 트리거에 활용.",
      "parameters": {
        "coordinate": "array — [x, y] 이동 목표 좌표"
      },
      "required": ["coordinate"]
    },
    {
      "name": "open_application",
      "description": "앱을 최전면으로 가져옴 (필요 시 실행). 허용 목록에 있는 앱만 가능.",
      "parameters": {
        "app": "string — 앱 이름"
      },
      "required": ["app"]
    },
    {
      "name": "switch_display",
      "description": "스크린샷을 캡처할 모니터 전환. 다른 모니터에 앱이 있을 때 사용.",
      "parameters": {
        "display": "string — 대상 디스플레이 식별자"
      },
      "required": ["display"]
    },
    {
      "name": "list_granted_applications",
      "description": "현재 세션 허용 목록의 앱과 활성 권한 플래그, 좌표 모드 반환. 부작용 없음.",
      "parameters": {},
      "required": []
    },
    {
      "name": "read_clipboard",
      "description": "현재 클립보드 내용을 텍스트로 읽기. clipboardRead 권한 필요.",
      "parameters": {},
      "required": []
    },
    {
      "name": "write_clipboard",
      "description": "텍스트를 클립보드에 쓰기. clipboardWrite 권한 필요.",
      "parameters": {
        "text": "string — 클립보드에 쓸 텍스트"
      },
      "required": ["text"]
    },
    {
      "name": "wait",
      "description": "지정된 시간만큼 대기.",
      "parameters": {
        "duration": "number — 대기 시간 (초, 0~100)"
      },
      "required": ["duration"]
    },
    {
      "name": "cursor_position",
      "description": "현재 마우스 커서 위치 반환. 최근 스크린샷 기준 픽셀 좌표, 없으면 논리적 포인트 좌표.",
      "parameters": {},
      "required": []
    },
    {
      "name": "hold_key",
      "description": "키 또는 키 조합을 지정 시간만큼 누른 채 유지 후 해제. 시스템 콤보는 systemKeyCombos 권한 필요.",
      "parameters": {
        "text": "string — 키 이름 또는 조합",
        "duration": "number — 유지 시간 (초, 0~100)"
      },
      "required": ["text", "duration"]
    },
    {
      "name": "left_mouse_down",
      "description": "현재 커서 위치에서 마우스 왼쪽 버튼 누름 유지. mouse_move로 위치 잡은 후 사용. left_mouse_up으로 해제.",
      "parameters": {},
      "required": []
    },
    {
      "name": "left_mouse_up",
      "description": "현재 커서 위치에서 마우스 왼쪽 버튼 해제. left_mouse_down과 쌍으로 사용. 버튼이 눌려있지 않아도 안전하게 호출 가능.",
      "parameters": {},
      "required": []
    },
    {
      "name": "computer_batch",
      "description": "여러 액션을 단일 도구 호출로 순차 실행. 개별 호출 대비 왕복 시간(수 초) 절약. 반복적 조작 시 효율적.",
      "parameters": {
        "actions": "array — 순차 실행할 액션 객체 목록"
      },
      "required": ["actions"]
    }
  ]
}
```

---

## 요약 테이블

| # | 이름 | 분류 | 핵심 기능 |
|---|------|------|-----------|
| 1 | `javascript_tool` | 브라우저 | 페이지 JS 실행 |
| 2 | `read_page` | 브라우저 | DOM 접근성 트리 조회 |
| 3 | `find` | 브라우저 | 자연어 요소 탐색 |
| 4 | `form_input` | 브라우저 | 폼 요소 값 설정 |
| 5 | `computer` | 브라우저 | 마우스/키보드 복합 조작 |
| 6 | `navigate` | 브라우저 | URL 이동 / 히스토리 탐색 |
| 7 | `resize_window` | 브라우저 | 창 크기 변경 |
| 8 | `gif_creator` | 브라우저 | 세션 GIF 녹화/내보내기 |
| 9 | `upload_image` | 브라우저 | 이미지 파일 업로드 |
| 10 | `get_page_text` | 브라우저 | 페이지 텍스트 추출 |
| 11 | `tabs_context_mcp` | 브라우저 | 탭 그룹 컨텍스트 조회 |
| 12 | `tabs_create_mcp` | 브라우저 | 새 탭 생성 |
| 13 | `update_plan` | 브라우저 | 실행 계획 사용자 승인 |
| 14 | `read_console_messages` | 브라우저 | 콘솔 로그 읽기 |
| 15 | `read_network_requests` | 브라우저 | 네트워크 요청 기록 조회 |
| 16 | `shortcuts_list` | 브라우저 | 단축키/워크플로우 목록 |
| 17 | `shortcuts_execute` | 브라우저 | 단축키/워크플로우 실행 |
| 18 | `switch_browser` | 브라우저 | Chrome 브라우저 전환 |
| 19 | `request_access` | OS | 앱 제어 권한 요청 |
| 20 | `screenshot` | OS | 전체 화면 캡처 |
| 21 | `zoom` | OS | 영역 확대 캡처 |
| 22 | `left_click` | OS | 좌클릭 |
| 23 | `double_click` | OS | 더블클릭 |
| 24 | `triple_click` | OS | 트리플클릭 |
| 25 | `right_click` | OS | 우클릭 |
| 26 | `middle_click` | OS | 가운데 클릭 |
| 27 | `type` | OS | 텍스트 입력 |
| 28 | `key` | OS | 키/단축키 입력 |
| 29 | `scroll` | OS | 스크롤 |
| 30 | `left_click_drag` | OS | 드래그 앤 드롭 |
| 31 | `mouse_move` | OS | 마우스 이동 (클릭 없음) |
| 32 | `open_application` | OS | 앱 실행/전면 전환 |
| 33 | `switch_display` | OS | 모니터 전환 |
| 34 | `list_granted_applications` | OS | 허용 앱 목록 조회 |
| 35 | `read_clipboard` | OS | 클립보드 읽기 |
| 36 | `write_clipboard` | OS | 클립보드 쓰기 |
| 37 | `wait` | OS | 대기 |
| 38 | `cursor_position` | OS | 커서 위치 조회 |
| 39 | `hold_key` | OS | 키 누름 유지 |
| 40 | `left_mouse_down` | OS | 마우스 버튼 누름 유지 |
| 41 | `left_mouse_up` | OS | 마우스 버튼 해제 |
| 42 | `computer_batch` | OS | 액션 일괄 실행 |
