# MariaDB MCP 서버 (웹/모바일 세션용)

맥에 로컬로 설치한 MCP 서버는 그 기기 안에서만 동작하고, Claude Code 웹/모바일 세션이
실행되는 **클라우드 컨테이너**로는 따라오지 않습니다. 이 저장소의 `.mcp.json`은 웹/모바일
세션에서도 동일하게 MariaDB에 접속할 수 있도록 **공식 `mariadb-mcp-server`(PyPI 0.1.1)** 를
등록합니다.

## 동작 방식 (`.mcp.json`)

- 서버 기동 시 네이티브 라이브러리(MariaDB Connector/C, `mariadb_config`)가 없으면
  `libmariadb-dev`를 자동 설치한 뒤 `uvx`로 서버를 실행합니다. (컨테이너는 매 세션 새로
  생성되므로 자동 설치가 필요)
- 실행 명령: `uvx --from 'mariadb-mcp-server==0.1.1' python -m mariadb_mcp_server.server`
  (이 버전은 콘솔 스크립트 진입점이 비어 있어 모듈을 직접 실행합니다.)
- 노출 툴: **`query_database(query)`** 하나. 서버에 read-only 가드가 내장되어 있어
  `SELECT / SHOW / DESCRIBE / DESC / EXPLAIN` 으로 시작하는 쿼리만 허용됩니다.
- 단일 커넥션이지만 `db.table` 형태의 정규화 이름으로 **여러 DB를 교차 조회**할 수 있습니다
  (계정에 해당 DB SELECT 권한이 있어야 함).

## 사용자가 설정해야 할 것 (커밋 금지 — 시크릿)

웹 환경(Environment) 설정의 **환경변수/시크릿**에 다음을 등록하세요. `.mcp.json`이 이 값을
참조합니다 (평문으로 저장소에 들어가지 않음):

| 환경변수 | 필수 | 설명 |
|----------|------|------|
| `MARIADB_HOST` | ✅ | 공인 호스트(IP/도메인) |
| `MARIADB_USER` | ✅ | **읽기전용 권한** 계정 권장 |
| `MARIADB_PASSWORD` | ✅ | 비밀번호 |
| `MARIADB_PORT` | ⬜ | 기본 `3306` |
| `MARIADB_DATABASE` | ⬜ | 기본 접속 DB, 기본값 `da` |

> 보안: 서버 자체 read-only 가드가 있지만, 안전장치로 **SELECT 전용 DB 계정**을 사용하세요.
> 접근 대상: `da`, `da_001`, `da_test`, `sangdofurniture`, `sangdofurniture001`.

## 적용 순서

1. 위 환경변수를 웹 환경 설정에 등록
2. 이 브랜치를 머지(또는 체크아웃)하여 `.mcp.json`이 세션에 포함되도록 함
3. 새 세션 시작 → `.mcp.json` 신뢰 승인
4. `mcp__mariadb__query_database` 툴이 보이면 준비 완료
