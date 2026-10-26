# CodexLab 작업 대화 내보내기

> 숨겨진 시스템·개발자 지침과 내부 도구 세부정보는 제외했습니다. 개인 경로, 계정명, 이메일 등 개인 식별정보는 제거했습니다.

## 저장소 요약

사용자 요청: 저장소의 기능, 주요 파일 위치, `GET /tasks` 요청 흐름, 필터링 로직을 파악하고 신규 엔지니어를 위한 요약 작성.

요약:

- FastAPI 기반의 로컬 JSON 태스크 관리 API
- `app/main.py`: HTTP 라우트
- `app/schemas.py`: Pydantic 요청·응답 모델
- `app/service.py`: 비즈니스 로직과 필터링
- `app/store.py`: JSON 파일 읽기·쓰기
- `data/tasks.json`: 태스크 데이터
- 지원 엔드포인트: `/health`, `/tasks`, `/tasks/{task_id}`, `/tasks/{task_id}/complete`

`GET /tasks` 흐름:

```text
HTTP 요청
  → app/main.py 라우트
  → status/q 파라미터 검증
  → service.list_tasks()
  → store.load_tasks()
  → data/tasks.json 읽기
  → 필터링
  → Task 응답 모델 검증
  → JSON 응답
```

## 의도된 버그 수정

사용자 요청: 두 버그를 재현·진단·수정하고 diff와 검증 결과를 확인.

수정 내용:

1. 상태 필터가 쿼리 파라미터가 아닌 문자열 `"status"`와 비교하던 문제를 수정했습니다.
2. 태스크 완료 시 수정된 복사본만 반환하고 저장하지 않던 문제를 수정했습니다.

검증:

- 상태 필터 결과 정상
- 완료 결과가 `done`으로 변경됨
- `completed_at`이 설정됨
- 변경 목록 저장 함수가 호출됨
- `git diff --check` 통과
- Python 컴파일 검사 통과

## `q` 검색 구현

사용자 요구사항:

- 선택적 `q` 쿼리 매개변수
- 대소문자 구분 없는 부분 일치
- 제목과 설명 모두 검색
- 상태 필터와 함께 동작
- `q` 생략 시 기존 동작 유지
- `curl`로 검증

`app/service.py`에서 `casefold()`를 사용해 제목 또는 설명을 검색하도록 구현했습니다. 상태 필터와 검색 필터는 AND 조건으로 적용됩니다.

검증한 요청:

```bash
curl 'http://127.0.0.1:8000/tasks?status=open'
curl 'http://127.0.0.1:8000/tasks?q=OAUTH'
curl 'http://127.0.0.1:8000/tasks?status=open&q=REPORT'
curl 'http://127.0.0.1:8000/tasks'
```

모든 검증이 통과했습니다.

## PR 준비

확인 결과:

- `data/tasks.json`은 변경되지 않았습니다.
- 런타임 데이터 복원은 필요하지 않았습니다.
- 최종 코드 변경은 `app/service.py` 하나로 제한했습니다.
- 원본 저장소는 `upstream`으로 보존했습니다.
- 개인 포크는 `origin`으로 설정했습니다.
- 작업 브랜치는 `codex/lab-fixes`입니다.
- 커밋: `94482a7 Fix task filters and persistence`
- 작업 트리는 clean 상태였습니다.

## 최종 수락 검사

- 상태 필터링 정상: 통과
- 완료 상태 유지 및 `completed_at` 설정: 통과
- 대소문자 무시 검색: 통과
- 검색과 상태 필터 조합: 통과
- `q` 생략 시 기존 전체 목록 동작: 통과
- 최종 diff 범위 검토: 통과
- `data/tasks.json` 변경 없음: 통과

완료 상태 영속성은 원본 데이터를 변경하지 않도록 임시 JSON 저장소에서 검증했습니다.
