# Airborne Zombie Cleanup — UE5 Blueprint 투척 전투 실습

공중에 배치된 공을 집어 던지고, 충돌한 좀비의 머티리얼과 상태 변화를 처리하는 Unreal Engine 5 개인 실습 프로젝트입니다.

> 이 문서는 프로젝트 설정, 대표 맵 참조, Blueprint 직렬화 정보로 교차 확인했습니다. 정확한 개발 버전인 Unreal Engine 5.5가 로컬에 없어 에디터 실행·패키징 검증은 아직 진행하지 않았습니다.

## 프로젝트 정보

| 항목 | 내용 |
| --- | --- |
| 개발 형태 | 개인 실습 |
| 엔진 | Unreal Engine 5.5 |
| 구현 방식 | Blueprint only |
| 대표 맵 | `Content/_TEST3/CharacterAnimationMontageMap.umap` |
| 렌더링 설정 | Windows / DirectX 12 / Shader Model 6 / Lumen / Ray Tracing |
| 현재 상태 | 정적 검증 완료, UE 5.5 실행·진단 검증 대기 |

## 프로젝트 개요

캐릭터가 맵에 놓인 공을 집어 원하는 방향으로 던지고, 공과 좀비의 충돌 결과를 시각적으로 확인하는 투척 전투 실습입니다.

대표 맵에는 공 7개와 좀비 5개가 배치되어 있습니다. 공의 획득·투척 요청과 피격·사망 처리를 Blueprint Interface로 분리해, 캐릭터와 상호작용 대상 사이의 직접 의존을 줄이는 구조를 연습했습니다.

## 담당 범위

저장소의 커밋 기여자는 `douyun0623` 한 명입니다. 아래 범위는 커스텀 Blueprint와 맵에서 확인한 구현이며, 캐릭터·애니메이션·텍스처와 Epic Starter Content는 직접 제작 범위에 포함하지 않습니다.

- 캐릭터 이동·시점·점프 입력 구성
- 공 획득, 부착, 분리, 투척 흐름
- 인터페이스 기반 투척·피격·사망 메시지 설계
- Projectile Movement 기반 공 이동
- 공 충돌 시 머티리얼·파티클·제거 처리
- 좀비의 단순 전진, 피격 표현, 사망 처리
- 캐릭터 Blend Space와 Animation Blueprint 연결
- HUD 조준점과 대표 맵 구성

## 핵심 구현

### 1. 인터페이스 기반 공 상호작용

캐릭터와 공의 상호작용을 `BPI_Ballinteration`으로 연결합니다.

- `Throw(location, velocity)` 메시지로 투척 위치와 속도 전달
- 획득 시 공을 캐릭터에 부착
- 투척 시 분리 후 초기 속도 적용
- 공마다 동일한 상호작용 규약 재사용

### 2. 충돌 결과와 머티리얼 변화

`BP_Ball`은 Projectile Movement로 이동하고 충돌 이벤트에 따라 시각 효과와 수명 주기를 처리합니다.

- `BPI_hit`을 통한 피격 대상 메시지 전달
- 충돌 시 얼음 계열 머티리얼 적용
- 파티클 생성 후 지연 제거

### 3. 좀비 상태 처리

`BP_Zombie`는 전방 이동과 피격·사망 반응을 담당합니다.

- 단순한 방향 기반 전진
- 피격 시 머티리얼 변경
- 사망 애니메이션 재생
- 충돌 비활성화 후 Actor 제거
- `BPI_death` 참조는 캐릭터·좀비 측에 있으며, 유효하지 않은 Target 진단이 남아 있어 실행 전 재검증 필요

이 저장소에서는 Behavior Tree, Blackboard, NavMesh 경로 탐색을 확인하지 못했습니다. 따라서 좀비를 **AI 구현**으로 설명하지 않고, 이동·피격·사망 상태를 가진 Blueprint 대상이라고 구분합니다.

## 주요 Blueprint

| 자산 | 역할 |
| --- | --- |
| `Content/_TEST3/Granny.uasset` | 캐릭터 이동과 공 획득·투척 |
| `Content/_TEST3/GrannyController.uasset` | 입력 매핑과 시점 제어 |
| `Content/_TEST3/BP_Ball.uasset` | 공 부착·투척·충돌·효과 처리 |
| `Content/_TEST3/BP_Zombie.uasset` | 단순 이동과 피격·사망 처리 |
| `Content/_TEST3/BPI/BPI_Ballinteration.uasset` | 투척 메시지 규약 |
| `Content/_TEST3/BPI/BPI_hit.uasset` | 피격 메시지 규약 |
| `Content/_TEST3/BPI/BPI_death.uasset` | 사망 메시지 규약 |
| `Content/_TEST3/GrannyAnimation.uasset` | 캐릭터 애니메이션 상태 연결 |

## 조작법

| 입력 | 동작 | 검증 상태 |
| --- | --- | --- |
| `WASD` | 이동 | 입력 매핑에서 확인 |
| 마우스 이동 | 시점 회전 | `Mouse2D` 매핑에서 확인 |
| `SpaceBar` | 점프 | 입력 매핑에서 확인 |
| 마우스 버튼 | 공 획득·투척 이벤트 | Blueprint 이벤트 확인, 최종 동작 검증 필요 |

왼쪽·오른쪽 마우스 이벤트가 공 상호작용에 사용되지만, 버튼별 최종 동작과 투척 감도는 UE 5.5 실행 후 확인해야 합니다.

## 실행 방법

1. Unreal Engine **5.5**를 설치합니다.
2. 저장소를 클론합니다.
3. `T3_2023180007.uproject`를 엽니다.
4. Content Browser에서 `Content/_TEST3/CharacterAnimationMontageMap.umap`을 직접 엽니다.
5. Blueprint 전체 컴파일 후 에디터의 Play 버튼으로 실행합니다.

`DefaultEngine.ini`의 `GameDefaultMap`은 현재 엔진의 OpenWorld 템플릿을 가리킵니다. 대표 맵을 직접 열지 않으면 이 프로젝트의 플레이 콘텐츠가 시작되지 않습니다.

## 검증 현황

| 검증 항목 | 결과 |
| --- | --- |
| UE 버전·프로젝트 설정 확인 | 완료 |
| 대표 맵과 배치 Actor 확인 | 완료 |
| 공·좀비·인터페이스 구조 확인 | 완료 |
| 입력 매핑 확인 | 완료 |
| UE 5.5 Blueprint Compile All | 미실시 |
| Play In Editor | 미실시 |
| Windows 패키징 | 미실시 |

로컬에는 정확한 UE 5.5가 없습니다. 원본 자산의 자동 변환을 막기 위해 상위 버전으로 열거나 저장하지 않았습니다.

## 재검증이 필요한 Blueprint 진단

직렬화된 자산 문자열에서 다음 컴파일 진단 흔적을 확인했습니다. 저장 당시의 오래된 메시지일 수 있으므로 현재 오류로 단정하지 않으며, UE 5.5에서 **Compile All Blueprints** 후 남는 항목만 수정해야 합니다.

- `death` 메시지의 대상 타입이 유효하지 않다는 진단
- 캐릭터 자신을 Spring Arm 대상으로 사용한 `Set bEnableCameraLag` 노드가 제거되었다는 진단
- `GrannyAnimation`의 Ground Moving → Winging, JumpUp → Winging 전이가 연결되지 않아 실행되지 않는다는 진단

## 대표 화면 준비 항목

현재 저장소에는 README에 사용할 플레이 스크린샷이 없습니다. UE 5.5 검증 시 다음 화면을 같은 해상도로 캡처해야 합니다.

1. 캐릭터, 공, 좀비, 조준점이 함께 보이는 대표 장면
2. 공을 손에 부착한 상태
3. 공이 좀비를 향해 이동하는 투척 장면
4. 충돌 전후 머티리얼 또는 파티클 변화
5. 좀비 사망 애니메이션과 제거 결과
6. 세 인터페이스 중 하나와 `BP_Ball` 호출부가 함께 보이는 Blueprint 그래프

## 외부 자산

게임 로직과 레벨 구성 외의 시각 자산은 직접 제작물이 아닙니다.

- 외부 FBX로 임포트된 Sporty Granny·Whiteclown 캐릭터와 애니메이션 — 원 배포처 확인 필요
- Epic Games Starter Content
- `LandscapeLab` 플러그인 및 콘텐츠
- 출처를 저장소에서 확인하지 못한 `starmap_2020_4k` 텍스처

저장소 메타데이터만으로 캐릭터·애니메이션의 Mixamo 출처를 확정할 수 없습니다. 원본 FBX 또는 배포 페이지로 출처와 이용 조건을 먼저 확인하고, Mixamo 자산으로 확인되는 경우 [Adobe Mixamo FAQ](https://helpx.adobe.com/kr/creative-cloud/faq/mixamo-faq.html)도 함께 확인해야 합니다. 특히 `LandscapeLab`의 플러그인 메타데이터에는 제작자·문서·라이선스가 비어 있고 대표 맵에서 사용 흔적을 확인하지 못했으므로, 필요하지 않다면 공개본에서 제외하고 필요하다면 출처와 재배포 허용 범위를 명시해야 합니다.

## 알려진 제한 사항

- UE 5.5에서 실행·컴파일·패키징하지 못했습니다.
- 대표 맵이 `GameDefaultMap`으로 지정되어 있지 않습니다.
- Blueprint 진단 흔적을 UE 5.5에서 재확인해야 합니다.
- 좀비에는 경로 탐색이나 의사결정 AI가 구현되어 있지 않습니다.
- 패키징된 실행 파일, 플레이 영상, 결과 스크린샷이 없습니다.
- 약 693MB의 저장소 중 약 608MB가 Starter Content입니다.
- Git LFS가 없어 대형 `.uasset`이 일반 Git 객체로 저장됩니다.
- 저장소에 `LICENSE`가 없어 코드 재사용 조건이 명시되지 않았습니다.
- 외부 자산의 원본 라이선스와 공개 저장소 재배포 범위를 추가 확인해야 합니다.
- `DefaultEngine.ini`의 Android File Server `SecurityToken`은 미사용 시 제거하고, 사용 중이면 기존 값을 폐기한 뒤 재발급해야 합니다.

## 저장소 구조

```text
.
├─ T3_2023180007.uproject
├─ Config/
├─ Content/
│  ├─ _TEST3/               # 커스텀 캐릭터·공·좀비·맵
│  ├─ Characters/           # 외부 캐릭터 자산
│  └─ StarterContent/       # Epic 제공 콘텐츠
├─ Plugins/LandscapeLab/    # 출처·사용 여부 확인 필요
└─ README.md
```

## 현재 완성도

공의 획득·투척과 인터페이스 기반 충돌 처리 구조는 자산에서 확인했습니다. 포트폴리오로 공개하기 전 UE 5.5 Blueprint 전체 컴파일, 대표 맵 실행, 입력·충돌·사망 흐름 검증, 외부 자산 정리가 필요합니다.
