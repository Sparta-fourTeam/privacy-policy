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
| Unity Diagnostics | [UnityConnectSettings.asset](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/ProjectSettings/UnityConnectSettings.asset#L14), [Android_Release.asset](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/Assets/Settings/Build%20Profiles/Android_Release.asset#L918) | `InsightsSettings.m_EngineDiagnosticsEnabled=1`; Android profile의 `m_BuildProfileEngineDiagnosticsState=2`. 아래 enum에서 2는 ProjectSettings 상속이므로 설정상 활성. 아래 Dashboard 증거로 1.0.1/build1뿐 아니라 현재 1.0.2/code2의 실제 수신도 확인. |
| Unity 버전·enum | [ProjectVersion.txt](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/ProjectSettings/ProjectVersion.txt), [Unity 6000.6.3f1 대응 reference enum](https://github.com/Unity-Technologies/UnityCsReference/blob/ed99dc1ddc11ec2f31893417d1ad7c142977d402/Modules/InsightsEditor/EngineDiagnosticsSettings.bindings.cs#L14-L29) | `6000.6.3f1`; Disabled=0, Enabled=1, ProjectSettings=2. legacy UnityAnalytics 및 CloudDiagnostics 비활성은 Engine Diagnostics 비활성을 의미하지 않는다. |
| Crashlytics·GameAnalytics | [manifest.json](https://github.com/Sparta-fourTeam/client/blob/10090077763379220123eabbabaedf7ae2ea3ed6/Packages/manifest.json), packages-lock.json, Assets 파일 및 런타임 소스 검색 | 현재 해당 SDK/런타임 참조가 없는 것을 확인. 도입 예정이라는 정보는 사용자 제공이며 현 빌드의 처리로 기재하지 않는다. |
| 빌드 1.0.2/code2 | 사용자가 제공한 검증된 빌드 정보 | 위 소스 SHA의 직렬화된 기본 PlayerSettings/Profile에는 1.0/code1이 남아 있다. 소스 기본값을 빌드 산출물 버전으로 오인하지 않는다. 이번 작업에서 AAB/APK 자체를 다시 검사한 것은 아니다. |
| 광고·대상 국가·연령·문의 | 사용자 확인 및 승인 | 현재 광고 없음, 한국 대상, 대상은 만 13세 이상 청소년·성인으로 확정. 공개 문의·피드백 주소 `teamdoodle.dev@gmail.com`. 대상 연령의 확정은 연령 확인·아동 관련 운영 절차의 구현이나 이메일 운영 세부 승인까지 의미하지 않는다. |
| 운영주체·요청 창구 | 사용자 확정 | 운영주체는 TeamDoodle, 개인정보 문의·삭제 요청 접수 이메일은 `teamdoodle.dev@gmail.com`. 요청 창구 확정은 시행일, 자동 삭제나 실제 삭제 처리 절차, 게임 내 Unity 설치 ID 확인 수단까지 확정한 것이 아니다. |

## Dashboard 및 빌드별 수신 추가 증거

2026-10-13 보완 작업에 전달받은 Unity Dashboard 읽기 전용 확인 결과를 반영했다. 이번 HTML 수정 작업에서 Dashboard 설정을 바꾸거나 원본 세션·설치 ID 값을 저장소로 전송하지 않았다.

- ProjectNova 프로젝트 ID `23795b90-83b5-4aa3-8841-29e4ac7fcf22`가 출시 소스와 일치함을 확인한 결과가 전달됐다.
- 최초 Collection 이력: 필수 수집 활성, Additional Recommended ON. 아래 1.0.1/build1과 1.0.2/code2 수신 관측은 이 ON 기간의 근거이며, 이후 OFF 기간이나 ON 복원 후 단말 동기화·수집 범위를 입증하지 않는다.
- 일시 OFF 이력: 사용자가 Additional Recommended Data OFF를 승인했고, 2026-10-13 16:52 UTC (2026-10-14 01:52 KST)에 ProjectNova에서 해당 설정만 OFF로 저장한 결과가 전달됐다. Unity 성공 알림, 새로고침·재진입 후 OFF 유지 및 `Collection settings: Essential`을 확인했다. 이 상태는 아래 ON 복원으로 종료됐다.
- 현재 Collection: 사용자가 다시 ON을 승인했고, 2026-10-13 17:17:53 UTC 전후 (2026-10-14 02:17:53 KST)에 ProjectNova Additional Recommended Data를 ON으로 복원한 결과가 전달됐다. 적용 성공 알림과 새로고침·재진입 후 `Essential + Recommended` 유지를 확인했다. Essential·Usage 네 항목·Ads 추가 액세스는 변경하지 않았다. 이번 문서 수정에서 진단 설정을 변경한 것은 아니다.
- Usage: 고객 중심 기능, 모델링/집계 인사이트, 기술 지원, 비즈니스 권장 사항 네 항목 모두 ON. 활성 설정을 해당 용도의 실제 처리 완료 증거로 확대하지 않는다.
- 2026-10-14 추가 전달 증거: 실제 Usage 화면에 `2026-10-07` controller-controller 조건 수락 표시가 확인됐다. 이것만으로 기본 4개 유형 전부의 공유 Yes 또는 공유 없음은 확정하지 않는다. 아래 목적별 원문과 필드별 적용 관계를 구분한다.
- Google Analytics는 `Not linked` 상태다. 추가 권장 수집 및 Usage 활성과 별개의 확인이며, Google Analytics 미연결이 모든 추가 이용 없음의 증거는 아니다.
- Additional Access: Unity Ads는 액세스 허용 버튼이 표시되는 미허용 상태. Usage 옵션 활성이나 지역 메타데이터 수신을 광고 사용의 증거로 해석하지 않는다.
- 변경 전 Additional Recommended ON 기간의 실제 Play 설치 패키지 `com.teamdoodle.doodledefense`, 앱 1.0.1/build1의 세션에서 설치 ID 및 기기·OS·메모리·GPU·화면·언어·로케일·국가/지역 메타데이터 수신을 확인한 결과가 전달됐다. 모든 빌드의 진단 전송이 미확인이라고 설명하면 안 된다.
- 변경 전 Additional Recommended ON 기간인 2026-10-13 KST 추가 확인 결과, 현재 문서 대상과 정확히 일치하는 `com.teamdoodle.doodledefense` / 앱 1.0.2 / build2 / AndroidPlayer / Play 설치 세션이 Dashboard에서 확인됐다. 현재 배포 빌드의 설치 ID·세션·기기 메타데이터·진단 자료 전송 및 수신이 실증되어 이전의 실제 수신 미확인 항목은 해소했다. 전달받은 확인 결과를 반영하며 개별 식별자·팀원 기기 상세나 원본 세션 자료는 저장소에 기재하지 않는다.
- 다른 과거 패키지 `com.teamdoodle.doodledoodle`의 build2 세션을 현재 출시 패키지의 1.0.2/code2 수신 증거로 혼동하지 않는다.
- 지역 메타데이터의 출처와 Google 신고 분류는 미확정이다. 나라/지역 메타데이터 존재만으로 위치정보 수집·정밀도·필수성·공유 분류를 임의 확정하지 않는다.
- 일시 OFF 기간 이후 새 세션에서도 지역 필드가 관측된 결과가 전달됐다. 해당 단말에 설정이 동기화됐는지는 불명이며, 이 관측만으로 OFF의 효과나 지역 필드 생성 경로·신고 분류를 확정하지 않는다. 전체 수집 범위 변화·정확한 적용 지연은 미검증이다. 기존 수신 데이터도 삭제하지 않았다. OFF 저장이나 ON 복원을 “수집 없음” 또는 기존 자료 삭제 완료로 해석하지 않는다. Usage 네 항목 ON과 Unity Ads 추가 액세스 미허용은 유지된다.
- 삭제 경로 확인: 실제 ProjectNova `Diagnostics > Data Privacy > Delete Diagnostics Data` 화면의 `Specific user with installation ID`와 `Submit Deletion Request`를 읽기 전용으로 확인한 결과가 전달됐다. 개발자 삭제 요청 경로 자체의 미확인은 해소했으나 삭제는 실행하지 않았고 접수부터 완료까지 검증한 것은 아니다.
- [client PR420](https://github.com/Sparta-fourTeam/client/pull/420)은 2026-10-14 조회 시 OPEN, head `66c019953c324714bdfa43019d99818c2cf8e6e6`이었다. 문의 자동 양식은 출시 빌드 반영·실기기 검증 전 기능이다. 현 1.0.2/code2에서 이미 제공 중이라고 서술하지 않는다. 다른 PR의 변경은 이 문서 작업에서 수정하지 않았다.

## Unity 공식 자료와 앱 사실의 구분

2026-10-14에 아래 원문을 다시 확인했다. 제품 기준, Dashboard 전달 증거와 앱의 실증 결과를 구분한다.

| 원문 | 확인한 내용과 적용 한계 |
| --- | --- |
| [Diagnostics privacy overview](https://docs.unity.com/en-us/cloud/developer-data/privacy-overview) | 기본 설치 ID와 안정성 진단, 제품 기본 보관 90일 및 기본 Diagnostics의 처리자 관계 설명. 이 기간·관계를 활성 추가 이용 전체에 그대로 확장하지 않는다. |
| [Google Play data safety](https://docs.unity.com/en-us/cloud/developer-data/google-data-safety) | 명시적으로 Diagnostics only 범위. 기본 4개 유형은 크래시 로그, 앱 진단, 기타 앱 성능, 기기/기타 ID이며 표의 목적은 Analytics다. 해당 표를 앱 전체 수집·공유·위치·삭제 신고의 최종 답으로 복사하지 않는다. |
| [Collection settings](https://docs.unity.com/en-us/cloud/developer-data/collection-settings) | Essential은 서비스에 필요한 크래시·텔레메트리·구성 자료; Additional Recommended는 사용 서비스에 필수인 범위를 넘어설 수 있다. Analytics 직접 미사용·미연결이 추가 수집 없음을 뜻하지 않는다. 실제 추가 필드 범위는 아래 3개 보류 항목에서 확인한다. |
| [Usage settings](https://docs.unity.com/en-us/cloud/developer-data/data-usage-settings) | 활성 선택은 집계·추상화된 개발자 간 기능, 익명 벤치마크·집계 모델 개선, 지원 접근, 추천을 허용한다. 원시 식별 자료를 다른 개발자에게 공개한다는 설명은 아니다. 실제 처리 필드·익명화 단계와 신고의 공유 여부는 별도 확인한다. 아동 대상·비Ads 앱의 앞 두 Usage 옵션에 관한 제한 안내도 있으므로 적용 여부를 검토한다. |
| [Developer Data Framework Additional Terms](https://unity.com/legal/developer-data-framework-additional-terms) | 현재 원문 최종 갱신은 2026-06-30; 1.1·2.1·2.2에서 설정에 따른 목적·처리 통제와 특정 목적의 독립 개인정보처리자 관계를 구분한다. 실제 화면의 controller-controller 수락 표시를 기본 Diagnostics의 위탁 설명과 혼동하지 않는다. 새 약정 수락은 하지 않았다. |
| [Manage Diagnostics data](https://docs.unity.com/en-us/cloud/developer-data/manage-diagnostics-data) | 이용자의 설치 ID로 개발자가 삭제 요청하는 경로 및 역할 권한·앱 설치 ID 확인 수단 필요성을 설명한다. 요청 처리 중 확인과 완료는 구분된다. 실제 화면 경로 확인은 이용자 접수→삭제 완료의 실증이 아니다. |

[프로젝트 데이터 설정 공식 문서](https://docs.unity.com/en-us/cloud/developer-data/configure-data-project)는 Dashboard 설정 변경이 기존 출시 앱에 자동 동기화되며 새 빌드가 필요 없다고 설명한다. 제품 설명과 별도로 이 앱의 설정 전환 후 단말 동기화 여부·전체 수집 범위와 정확한 적용 지연은 아직 확인되지 않았다.

개발자 삭제 요청 경로는 원문과 실제 화면에서 확인됐다. 이용자 접수 창구는 `teamdoodle.dev@gmail.com`이다. 출시 빌드에 Unity 설치 ID 확인·문의 전달 수단을 반영하고 실기기 검증한 뒤, 운영자가 문의로 받은 Unity 설치 ID로 진단 자료 삭제를 요청하는 절차를 적용할 예정이다. 로컬 게임 저장 및 서버 계정 데이터와 별개이며 자동·즉시 삭제나 확정 처리일을 약속하지 않는다. PR420 자동 양식과 접수→완료 처리는 아직 출시·실기기 검증 전이다. Diagnostics 기본 90일은 추가 이용의 필드별 보관이나 실제 저장 국가가 확인됐다는 뜻이 아니다.

## 데이터 보안 분류의 추가 확인 보류 — 3개

- [ ] 독립 처리에 들어가는 개인 연결 가능 필드와 익명화 시점. Usage 네 항목 ON·controller-controller 조건 수락만으로 기본 4개 유형 모두를 공유 Yes 또는 No로 확정하지 않는다.
- [ ] `country/subdivision`이 물리 위치 추정인지 단순 locale인지. IP나 locale에서 생성된다는 원문 근거 없이 출처·정밀도·위치 신고 분류를 단정하지 않는다.
- [ ] Additional Recommended가 기본 4개 진단 유형 외 앱 상호작용 등을 포함하는지. 공개 원문에는 이 앱의 필드별 적용 매핑이 없어 확정하지 않는다.

이 세 항목은 확인된 수집·설정·목적과 분리한다. 공식 지원 확인이 필요할 수 있으나 이번 작업에서 문의·지원 웹폼 제출·실제 사용자 데이터 삭제는 하지 않았다.

## 공개 반영 전 미확정 항목

- [x] 운영주체 TeamDoodle과 개인정보 문의·삭제 요청 접수 창구 `teamdoodle.dev@gmail.com`을 사용자 확정값으로 반영했다.
- [ ] 개인정보 담당자, 필요한 추가 연락 정보 및 시행일을 확정한다. 운영주체·접수 이메일 확정만으로 실제 삭제 처리나 운영 세부사항이 확정된 것은 아니다.
- [x] 대상 연령은 만 13세 이상 청소년·성인으로 사용자 확정. 이전 “12세 이용가?” 질문으로 추정한 것이 아니다.
- [ ] 타깃에 만 13세가 포함되므로 만 14세 미만 개인정보 처리 절차를 검토한다. Unity 원문의 아동 대상 비Ads 앱과 앞 두 Usage 옵션 제한 안내가 이 타깃에 어떻게 적용되는지도 검토한다. 연령 확인·법정대리인 관련 절차가 구현·운영 중이라고 주장하지 않는다.
- [x] 정확한 출시 패키지 `com.teamdoodle.doodledefense`의 1.0.2/build2 AndroidPlayer·Play 설치 세션으로 현재 code2의 설치 ID·세션·기기 메타데이터·진단 자료 실제 전송과 수신을 확인했다. 다른 패키지나 1.0.1/build1 증거로 대체한 것이 아니다.
- [x] 승인된 Additional Recommended Data OFF 저장·재확인과 이후 ON 복원·재확인 결과를 이력으로 보존하고, 현재 설정은 Essential + Recommended / ON으로 반영했다. Essential 활성, Usage 네 항목 ON, Unity Ads 추가 액세스 미허용은 유지한다.
- [ ] 설정 전환 후 출시 앱 세션으로 단말 동기화 여부·실제 수집 범위·지역 필드 변화와 적용 지연을 검증한다. 최초 ON 기간의 수신이나 일시 OFF 이후 지역 필드 관측만으로 대체하지 않는다.
- [ ] 데이터 보안 분류는 위 추가 확인 3개로 좁혀 보류한다. 확인된 수집·목적은 초안에 반영하고 Usage 설정의 실제 필드 매핑을 꾸며 채우지 않는다.
- [ ] 실제 저장 국가, 추가 이용의 필드별 보관 범위 및 적용되는 제공·위탁·국외 이전 고지와 처리 근거를 확정한다. 기본 Diagnostics의 90일과 관계를 추가 이용 전체에 적용하지 않는다.
- [ ] Android에서 앱 데이터 삭제·제거·재설치 시 PlayerPrefs와 save.json이 각각 어떻게 제거되는지 실기기로 확인한다. 기기·OS 백업 및 복원, 남은 사본과 복구 가능성을 검증하고 그 결과로만 사용 절차를 작성한다.
- [ ] 공통 표시값 LOCAL-0001 대신 사용할 안전한 문의·진단 데이터 식별 및 본인 확인 절차를 설계·검증한다. 사용자에게 원본 저장 파일·비밀 토큰 등 민감 정보를 보내도록 요구하지 않는다.
- [x] 개발자용 설치 ID별 Diagnostics 삭제 요청 화면과 공식 경로를 확인했다. 삭제 실행은 하지 않았다.
- [ ] PR420 문의 양식·Unity 설치 ID 확인/전달 수단을 실제 출시 빌드에 반영하고 실기기로 검증한다. 수신 값 식별과 접수부터 삭제 완료까지는 별도 검증이며, 반영 전에 이미 제공 중이라고 안내하지 않는다.
- [ ] Unity 진단 자료 열람·삭제 요청을 실제로 처리할 수 있는지, 담당자·권한·절차·기한과 실패/확인 방법을 검증한다. 로컬 삭제와 외부 자료 삭제를 연동됐다고 단정하지 않는다.
- [ ] 문의 이메일의 수신·접근 가능 담당자, 계정 보안·접근 제한, 목적별 보관·삭제 기간, 요청 처리와 기록 보관, 이메일 제공업체의 처리·국외 이전 조건을 결정한다. 이 초안은 그러한 통제나 삭제 운영이 이미 시행 중이라고 주장하지 않는다.
- [ ] 한국 대상 최종 방침의 필수 내용과 처리 근거를 검토한다. 실제 서비스 사실이 확정된 뒤 한·영 정책 및 삭제 안내를 함께 갱신한다. 이 PR로 법적 적합성·출시 준비·Play 신고가 완료되지 않는다.
- [ ] 향후 계정 서버·인증·진행 동기화·광고·Crashlytics/GameAnalytics 도입 시 새 빌드의 사실과 처리 항목·보관·삭제를 다시 검토한다.

## 검증 결과

검증 기록은 PR에도 요약한다. 앱 코드를 변경하지 않으므로 이 문서 수정에서 Unity 테스트나 실제 기기 삭제·진단 전송 테스트를 실행했다고 주장하지 않는다. 정책 저장소에는 기존 자동 테스트가 없다.

- `git diff --check`: 통과.
- 네 HTML의 태그 짝·문서 구조, 언어 속성, 초안 표시, script/form/이벤트 핸들러 부재를 검사: 통과.
- 네 파일의 `<style>` 블록을 기준 main과 문자열 비교: 모두 동일. 기존 레이아웃 CSS 변경 없음.
- 48개 링크 출현을 검사: 상대 링크는 기존 네 HTML 경로로 해결되며, mailto는 승인된 문의 주소만 사용하고 고유 ID·삭제 신청 제목 등의 사전 입력을 넣지 않았다. 외부 링크는 위 Unity 공식 원문 여섯 개이며 열리는 것을 확인했다. 이메일은 전송하지 않았다.
- ko/en 정책은 11개 절과 2개 표, 삭제 안내는 ko/en 각 5개 절. 각 언어에서 빌드 버전, 로컬 저장, 인증 미연결, Diagnostics, SDK 부재, 표시 ID·삭제 한계 및 문의 주소를 교차 검토했다.
- 정책 HTML 디렉터리만 `127.0.0.1` 로컬 서버로 열어 Chrome에서 루트→ko→en→삭제 안내 링크를 실제 탐색하고, 정책·삭제 안내의 표·초안 박스·탐색·문의 버튼 렌더를 확인했다. 외부 공개 서버·Pages로 배포하지 않았다. 모바일 화면 및 실제 앱 기능은 이 검사 범위에 포함하지 않았다.
- Dashboard·빌드별 수신·연령 및 현재 code2 실제 수신 확인 보완 후 같은 HTML/CSS/42개 링크 검사를 다시 통과했다. 운영주체·이메일 접수 창구 확정 보완에도 같은 검사와 Chrome localhost 한·영 정책·삭제 안내 확인을 수행했고, 시행일·설치 ID 확인 수단·삭제 처리 절차·지역 분류의 미확정을 유지했다. ko/en의 연령, Dashboard 옵션 및 패키지·빌드 구분을 교차 확인하고, Chrome localhost에서 수정된 정책 표와 한·영 삭제 안내를 다시 렌더·탐색했다. 전달받은 Dashboard 증거의 범위를 명시했으며, 원본 진단 세션이나 설치 ID 값은 게시하지 않았다.
- Additional Recommended Data OFF 및 이후 ON 복원 보완 후 HTML/CSS/42개 링크·diff 검사와 Chrome localhost 한·영 정책 확인을 수행했다. 현재 설정과 일시 OFF·지역 필드 관측 이력을 구분했고, 단말 동기화·지역 출처/신고 분류·적용 지연의 미검증을 유지했다.
- 2026-10-14 개발자 삭제 경로·기본/추가 이용 보완 후 HTML/CSS/48개 링크·diff 검사와 Chrome localhost 한·영 정책·삭제 안내 렌더/탐색을 확인했다. 정책 11개 절·2개 표와 삭제 안내 언어별 5개 절을 유지했다. 추가 분류 보류 3개와 PR420 미출시·실기기 검증 전 조건을 교차 확인했다. 약정 수락·이메일/지원 문의·데이터 삭제는 수행하지 않았다.
