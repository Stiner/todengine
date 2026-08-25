# analysis

TodEngine 코드베이스 분석 문서 모음.

| 문서 | 내용 |
|---|---|
| [`todengine-analysis.html`](todengine-analysis.html) | 리플렉션 코어를 중심으로 한 아키텍처·결함·되살리기 경로 분석 |

브라우저에서 파일을 직접 열면 됩니다. 외부 스크립트 없이 동작하며, 웹폰트(Google Fonts)만 네트워크를 사용합니다 — 오프라인에서도 시스템 폰트로 폴백해 정상적으로 읽힙니다. 라이트/다크 테마는 OS 설정을 따릅니다.

공유용 웹 페이지: <https://claude.ai/code/artifact/877bb660-892e-4930-bc96-5164ef9bbcbc>

## 분석 시점

- 대상 리비전: `857673c` (2022-11-08)
- 분석 일자: 2026-08-25
- 대상 범위: `code/tod/**` (외부 라이브러리 `code/external/**` 제외)

## 요약

TodEngine은 2009–2011년에 개발된 Windows / Direct3D 9 실시간 3D 엔진으로, 원저장소는
`code.google.com/archive/p/todengine`에서 export되었습니다. 코드 커밋은 2011년 11월 이후
멈춰 있습니다.

핵심은 렌더링이 아니라 **리플렉션 코어**입니다. `DECLARE_CLASS` / `IMPLEMENT_CLASS` 매크로 쌍이
클래스마다 정적 `Type` 인스턴스를 심고, 그 안의 메서드·프로퍼티 테이블을 세 소비자가 공유합니다.

- `XmlSerializer` — 씬 `.xml` 저장/복원 (별도 스키마 정의 없음, 테이블이 곧 포맷)
- `todpython` / `todlua` — 클래스당 0줄의 바인딩 코드로 속성·메서드 자동 노출
- `PropertyGrid` (wxPython) — 편집 UI 자동 생성

설계는 Radon Labs의 Nebula Device 2 계보를 따릅니다 (Kernel, NOH 경로 체계, cwn 스택,
`v_setName_s` 형태의 프로토타입 시그니처).

## 현재 상태

현대 툴체인에서 빌드되지 않습니다. 차단 요인:

| 위치 | 문제 |
|---|---|
| `code/tod/core/typeid.h:22` | `reinterpret_cast<int>(typeid(T).name())` — 64비트에서 포인터 절단, MSVC x64 컴파일 에러 |
| `code/tod/core/define.h:99` | `#define override virtual` — C++11 `override` 키워드와 충돌 |
| `code/tod/core/define.h:49` | `#innclude <sys/stat.h>` 오타 — 비-Windows 분기가 컴파일된 적 없음 |
| `code/tod/core/define.h:42,69` | `stdext::hash_map` / `__gnu_cxx::hash_map` — 제거된 컴파일러 확장 |

전체 결함 목록과 되살리기 단계별 순서는 분석 문서를 참고하세요.
