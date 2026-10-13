# 로컬 비공개 테스트 정책 검토 체크리스트

**검토 초안 — 게시 준비 완료 문서가 아니다.** 대상은 끄적끄적 디펜스 1.0.2 / version code 2, 한국 대상 로컬 비공개 테스트 빌드다. 아래 미확정 항목을 해결하기 전 정책 확정·공개 반영·스토어 신고 근거로 사용하지 않는다.

## 변경 범위와 공개본 확인

- 확인일: 2026-10-13 UTC.
- 정책 저장소 main: `62cc12b8f3dc9db4fa1c80b264b564c5ddaee0b4` (작업 시작 시 원격 최신).
- `/`, `/ko/`, `/en/`, `/delete-account/` 공개 HTML을 내려받아 main의 해당 파일과 `cmp`로 비교: 네 파일 모두 동일했다.
- 기존 `/ko/`와 `/en/`은 서버 발급 게스트 ID, 선택적 Google 연동, 계정·진행 정보 삭제를 설명했다. 현 빌드의 기기 생성 ID·로컬 저장·외부 로그인 unavailable과 불일치한다.
- 기존 정책의 Crashlytics·GameAnalytics는 향후 계획으로 적혀 있었지만, 처리 항목 표에는 일반 오류·이용 분석 행이 섞여 있고 활성 Unity Diagnostics는 빠져 있었다. SDK 부재와 엔진 진단 활성은 서로 다른 사실이다.
- 기존 `/delete-account/`는 이메일 계정 삭제 준비, 인게임 사용자 ID 확인, Google 연동 정보 삭제를 설명했다. 현 빌드의 원격 삭제 기능 부재와 공통 표시값 `LOCAL-0001`에 맞지 않는다.
- 본 PR은 기존 네 경로, 언어 선택·상호 링크, CSS, header/main/footer 및 한·영 삭제 안내 구성을 유지한다. 정책 11개 절과 삭제 안내 언어별 5개 절의 틀을 유지한다. 루트·삭제 페이지에도 기존 정책 페이지와 같은 `noindex` 초안 메타를 추가한다. `noindex`는 접근 차단이 아니며 브랜치/PR은 공개 저장소에서 볼 수 있다.
- 관련 지침: 정책 저장소에는 추적되는 `AGENTS.md`, `.agents/skills`, `.pre-commit-config.yaml`, 테스트·빌드 설정이 없다. 앱 소스의 `AGENTS.md`, `docs/conventions.md`, `docs/architecture.md`, `docs/messages.md`를 읽었다. 앱 사본에도 `.agents/skills`는 없다. 조직 [CONTRIBUTING.md](https://github.com/Sparta-fourTeam/.github/blob/main/CONTRIBUTING.md)의 이슈·커밋 형식을 참고한다. 정책 저장소에는 `main`만 있으므로 dev 생성 없이 현재 main에서 작업 브랜치를 만든다.

## 배포 차단 확인

- 추적되는 `.github/workflows` 파일은 없다. GitHub Actions 목록에는 동적 `pages-build-deployment` workflow 하나가 있다.
- `gh api repos/Sparta-fourTeam/privacy-policy/pages` 확인: `build_type=legacy`, `source.branch=main`, `source.path=/`, `public=true`.
- 최근 Pages 실행 5개 모두 `event=dynamic`, `head_branch=main`; 최신 배포 SHA는 위 main과 같다.
- 이 설정에서 작업 브랜치 push는 Pages 공개 배포 대상이 아니다. 설정이나 workflow가 변경되면 push 전에 재확인하고, 작업 브랜치 배포 가능성이 있으면 중단한다.
- 본 작업은 draft PR까지만 한다. main merge, workflow dispatch, Pages 설정 변경·배포, 정책 확정, Play 신고는 하지 않는다. 향후 main merge는 현재 설정상 공개 배포를 유발할 수 있으므로 별도 검토가 필요하다.

## 사실 근거와 확인 수준

앱 코드 근거는 [Sparta-fourTeam/client의 고정 SHA](https://github.com/Sparta-fourTeam/client/tree/10090077763379220123eabbabaedf7ae2ea3ed6)다. 로컬 사본을 해당 SHA로 checkout하여 읽었으며 앱 파일은 수정하지 않았다. 문서·TODO보다 실제 코드와 설정을 우선했다.

| 항목 | 근거 | 확인 수준·해석 |
| --- | --- | --- |
| 게스트 ID | [LoginFlow.cs](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/Assets/_Project/Scripts/Runtime/Core/Auth/LoginFlow.cs#L71), [PlayerPrefsAuthCredentialStore.cs](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/Assets/_Project/Scripts/Runtime/Network/PlayerPrefsAuthCredentialStore.cs#L28) | 기기에서 `Guid.NewGuid()` 생성, 로그인 방식·ID를 PlayerPrefs에 저장. 서버 발급 ID가 아니다. |
| 게임 진행 | [LocalSaveStore.cs](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/Assets/_Project/Scripts/Runtime/Network/Local/LocalSaveStore.cs#L15), [LocalSave.cs](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/Assets/_Project/Scripts/Runtime/Network/Local/LocalSave.cs) | `Application.persistentDataPath/save.json` 저장. 지갑·경험치·진행·강화·전투·보상 원장 등 로컬 상태. 보관 기간이나 삭제·백업 복원 동작까지 검증한 것은 아니다. |
| 계정·진행 서버 | [RootLifetimeScope.cs](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/Assets/_Project/Scripts/Runtime/Boot/RootLifetimeScope.cs#L20) | LocalAuthApi 및 로컬 게임 API 등록. HTTP 구현 미등록. 서버 개발 중이라는 정보는 사용자 확인이며 이번 빌드에는 연결 없음. |
| Google·Apple 인증 | [PlaceholderSocialSignIn.cs](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/Assets/_Project/Scripts/Runtime/Network/PlaceholderSocialSignIn.cs#L22) | `IsAvailable=false`. 선택적 계정 연동으로 안내하면 안 된다. |
| 표시 ID | [ProfilePopupView.cs](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/Assets/_Project/Scripts/Runtime/View/Lobby/ProfilePopupView.cs#L15) | `LOCAL-0001` 상수. 고유 계정·Unity 설치 ID가 아니며 삭제 요청 식별에 사용 불가. |
| Unity Diagnostics | [UnityConnectSettings.asset](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/ProjectSettings/UnityConnectSettings.asset#L14), [Android_Release.asset](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/Assets/Settings/Build%20Profiles/Android_Release.asset#L918) | `InsightsSettings.m_EngineDiagnosticsEnabled=1`; Android profile의 `m_BuildProfileEngineDiagnosticsState=2`. 아래 enum에서 2는 ProjectSettings 상속이므로 설정상 활성. 실제 기기 전송·Dashboard 옵션은 미검증. |
| Unity 버전·enum | [ProjectVersion.txt](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/ProjectSettings/ProjectVersion.txt), [Unity 6000.6.3f1 대응 reference enum](https://github.com/Unity-Technologies/UnityCsReference/blob/ed99dc1ddc11ec2f31893417d1ad7c142977d402/Modules/InsightsEditor/EngineDiagnosticsSettings.bindings.cs#L14-L29) | `6000.6.3f1`; Disabled=0, Enabled=1, ProjectSettings=2. legacy UnityAnalytics 및 CloudDiagnostics 비활성은 Engine Diagnostics 비활성을 의미하지 않는다. |
| Crashlytics·GameAnalytics | [manifest.json](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/Packages/manifest.json), packages-lock.json, Assets 파일 및 런타임 소스 검색 | 현재 해당 SDK/런타임 참조가 없는 것을 확인. 도입 예정이라는 정보는 사용자 제공이며 현 빌드의 처리로 기재하지 않는다. |
| 빌드 1.0.2/code2 | 사용자가 제공한 검증된 빌드 정보 | 위 소스 SHA의 직렬화된 기본 PlayerSettings/Profile에는 1.0/code1이 남아 있다. 소스 기본값을 빌드 산출물 버전으로 오인하지 않는다. 이번 작업에서 AAB/APK 자체를 다시 검사한 것은 아니다. |
| 광고·대상 국가·문의 | 사용자 확인 및 승인 | 현재 광고 없음, 한국 대상, 공개 문의·피드백 주소 `teamdoodle.dev@gmail.com`. 목표 연령 선택이나 이메일 운영 세부 승인으로 확대 해석하지 않는다. |

## Unity 공식 자료와 앱 사실의 구분

2026-10-13에 [Diagnostics privacy overview](https://docs.unity.com/en-us/cloud/developer-data/privacy-overview)와 [Google Play data safety](https://docs.unity.com/en-us/cloud/developer-data/google-data-safety)를 확인했다. 제품 문서는 설치별 ID와 크래시·진단·성능 자료를 설명한다. 기본 보관 90일은 Unity 제품 기준이며 이 앱의 확인된 보관 기간으로 단정하지 않는다.

제품 문서의 Dashboard 삭제 경로는 이 앱에서 삭제 요청 처리 운영이 검증됐다는 뜻이 아니다. data-safety의 사용자 삭제 수단 항목과 개발자용 삭제 경로도 구분한다. 앱의 실제 전송·수집 조건, 설치 ID 취득·사용자 식별·삭제 요청 연결, Dashboard 설정과 권한을 확인해야 한다. Unity 문서의 목적·공유 분류를 앱 전체 Play 신고에 그대로 복사하지 않는다.

## 공개 반영 전 미확정 항목

- [ ] 운영주체의 정식 명칭·필수 연락 정보·개인정보 담당자, 시행일을 확정한다. TeamDoodle은 프로젝트명으로만 표시했다.
- [ ] 대상 연령과 아동 이용·진단 처리의 적정 절차를 검토한다. 사용자의 “12세 이용가?”는 질문이며 연령 선택이 아니다. 특정 연령 제한·보호자 동의를 이미 운영 중이라고 주장하지 않는다.
- [ ] 정확한 code2 산출물 및 설치 환경을 식별하고, Unity Diagnostics의 실제 네트워크 전송·크래시 경로·추가 메타데이터·Dashboard 옵션을 검증한다. 진단 유지/비활성 결정과 필요한 안내·처리 근거를 검토한다.
- [ ] Diagnostics 처리 주체·위탁/제공 관계·이전 국가·이전 항목·시점/방법·근거·거부 방법 및 실제 보관 조건을 확인한다. Unity 제품 기본 90일을 앱 운영 사실로 대체하지 않는다.
- [ ] Android에서 앱 데이터 삭제·제거·재설치 시 PlayerPrefs와 save.json이 각각 어떻게 제거되는지 실기기로 확인한다. 기기·OS 백업 및 복원, 남은 사본과 복구 가능성을 검증하고 그 결과로만 사용 절차를 작성한다.
- [ ] 공통 표시값 LOCAL-0001 대신 사용할 안전한 문의·진단 데이터 식별 및 본인 확인 절차를 설계·검증한다. 사용자에게 원본 저장 파일·비밀 토큰 등 민감 정보를 보내도록 요구하지 않는다.
- [ ] Unity 진단 자료 열람·삭제 요청을 실제로 처리할 수 있는지, 담당자·권한·절차·기한과 실패/확인 방법을 검증한다. 로컬 삭제와 외부 자료 삭제를 연동됐다고 단정하지 않는다.
- [ ] 문의 이메일의 수신·접근 가능 담당자, 계정 보안·접근 제한, 목적별 보관·삭제 기간, 요청 처리와 기록 보관, 이메일 제공업체의 처리·국외 이전 조건을 결정한다. 이 초안은 그러한 통제나 삭제 운영이 이미 시행 중이라고 주장하지 않는다.
- [ ] 한국 대상 최종 방침의 필수 내용과 처리 근거를 검토한다. 실제 서비스 사실이 확정된 뒤 한·영 정책 및 삭제 안내를 함께 갱신한다. 이 PR로 법적 적합성·출시 준비·Play 신고가 완료되지 않는다.
- [ ] 향후 계정 서버·인증·진행 동기화·광고·Crashlytics/GameAnalytics 도입 시 새 빌드의 사실과 처리 항목·보관·삭제를 다시 검토한다.

## 검증 결과

검증 기록은 PR에도 요약한다. 앱 코드를 변경하지 않으므로 이 문서 수정에서 Unity 테스트나 실제 기기 삭제·진단 전송 테스트를 실행했다고 주장하지 않는다. 정책 저장소에는 기존 자동 테스트가 없다.

- `git diff --check`: 통과.
- 네 HTML의 태그 짝·문서 구조, 언어 속성, 초안 표시, script/form/이벤트 핸들러 부재를 검사: 통과.
- 네 파일의 `<style>` 블록을 기준 main과 문자열 비교: 모두 동일. 기존 레이아웃 CSS 변경 없음.
- 42개 링크 출현을 검사: 상대 링크는 기존 네 HTML 경로로 해결되며, mailto는 승인된 문의 주소만 사용하고 고유 ID·삭제 신청 제목 등의 사전 입력을 넣지 않았다. 외부 링크는 위 Unity 공식 자료 두 개이며 열리는 것을 확인했다. 이메일은 전송하지 않았다.
- ko/en 정책은 11개 절과 2개 표, 삭제 안내는 ko/en 각 5개 절. 각 언어에서 빌드 버전, 로컬 저장, 인증 미연결, Diagnostics, SDK 부재, 표시 ID·삭제 한계 및 문의 주소를 교차 검토했다.
- 정책 HTML 디렉터리만 `127.0.0.1` 로컬 서버로 열어 Chrome에서 루트→ko→en→삭제 안내 링크를 실제 탐색하고, 정책·삭제 안내의 표·초안 박스·탐색·문의 버튼 렌더를 확인했다. 외부 공개 서버·Pages로 배포하지 않았다. 모바일 화면 및 실제 앱 기능은 이 검사 범위에 포함하지 않았다.
