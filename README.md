# Airborne Zombie Cleanup — UE5 Blueprint 투척 전투 실습

> 공을 집어 던지고, 충돌한 좀비의 머티리얼과 상태 변화를 처리하는 Unreal Engine 5 개인 실습입니다.

## 프로젝트 개요

| 항목 | 내용 |
|---|---|
| 개발 형태 | 개인 실습 |
| 엔진 | Unreal Engine 5.5 |
| 구현 방식 | Blueprint-only, Blueprint Interface |
| 대표 맵 | `Content/_TEST3/CharacterAnimationMontageMap.umap` |
| 렌더링 | Windows, DirectX 12, Shader Model 6, Lumen |
| 현재 상태 | Blueprint 구조 정적 확인, UE 5.5 실행 확인 필요 |

대표 맵에는 공 7개와 좀비 5개가 배치되어 있습니다. 공의 획득·투척 요청과 피격·사망 처리를 Blueprint Interface로 분리해 캐릭터와 상호작용 대상 사이의 직접 의존을 줄였습니다.

## 구현 범위

- 캐릭터 이동·시점·점프 입력 구성
- 공 획득, 부착, 분리, 투척 흐름
- Interface 기반 투척·피격·사망 메시지 구성
- Projectile Movement 기반 공 이동
- 공 충돌 시 머티리얼·파티클·제거 처리
- 좀비의 단순 전진, 피격 표현과 사망 처리
- 캐릭터 Blend Space와 Animation Blueprint 연결
- HUD 조준점과 대표 맵 구성

캐릭터·애니메이션·텍스처와 Epic Starter Content는 직접 제작 범위에 포함하지 않습니다.

## 핵심 구현

### Interface 기반 공 상호작용

캐릭터와 공의 상호작용을 `BPI_Ballinteration`으로 연결합니다.

- `Throw(location, velocity)` 메시지로 투척 위치와 속도 전달
- 획득 시 공을 캐릭터에 부착
- 투척 시 분리 후 초기 속도 적용
- 여러 공에서 동일한 상호작용 규약 재사용

### 충돌 결과와 머티리얼 변화

`BP_Ball`은 Projectile Movement로 이동하고 충돌 이벤트에 따라 효과와 수명 주기를 처리합니다.

- `BPI_hit`을 통한 피격 메시지 전달
- 충돌 시 얼음 계열 머티리얼 적용
- 파티클 생성 후 지연 제거

### 좀비 상태 처리

`BP_Zombie`는 단순한 방향 기반 전진과 피격·사망 반응을 담당합니다.

- 피격 시 머티리얼 변경
- 사망 Animation 재생
- 충돌 비활성화 후 Actor 제거

Behavior Tree, Blackboard, NavMesh 경로 탐색은 사용하지 않았으므로 경로 탐색 AI가 아닌 상태 반응형 대상으로 구분합니다.

## 주요 Blueprint

| 자산 | 역할 |
|---|---|
| `Content/_TEST3/Granny.uasset` | 캐릭터 이동과 공 획득·투척 |
| `Content/_TEST3/GrannyController.uasset` | 입력 매핑과 시점 제어 |
| `Content/_TEST3/BP_Ball.uasset` | 공 부착·투척·충돌·효과 처리 |
| `Content/_TEST3/BP_Zombie.uasset` | 단순 이동과 피격·사망 처리 |
| `Content/_TEST3/BPI/BPI_Ballinteration.uasset` | 투척 메시지 규약 |
| `Content/_TEST3/BPI/BPI_hit.uasset` | 피격 메시지 규약 |
| `Content/_TEST3/BPI/BPI_death.uasset` | 사망 메시지 규약 |
| `Content/_TEST3/GrannyAnimation.uasset` | 캐릭터 Animation 상태 연결 |

## 조작법

| 입력 | 동작 |
|---|---|
| `WASD` | 이동 |
| 마우스 이동 | 시점 회전 |
| `SpaceBar` | 점프 |
| 마우스 버튼 | 공 획득·투척 이벤트 |

마우스 좌·우 버튼별 최종 동작과 투척 감도는 UE 5.5 실행 후 확인이 필요합니다.

## 실행 방법

1. Unreal Engine **5.5**를 설치합니다.
2. 저장소를 clone합니다.
3. 루트의 `.uproject` 파일을 엽니다.
4. `Content/_TEST3/CharacterAnimationMontageMap.umap`을 엽니다.
5. Blueprint를 Compile한 뒤 Play In Editor로 실행합니다.

기본 에디터·게임 시작 맵은 `CharacterAnimationMontageMap`으로 설정되어 있습니다.

## 개발 상태

공·좀비·Interface 흐름은 구성되어 있습니다. 저장된 자산에는 일부 유효하지 않은 Target과 미연결 Animation 전이 진단 흔적이 있어, 정확한 UE 5.5 환경에서 Compile All과 플레이 흐름을 다시 확인해야 합니다.

## 외부 자산

외부 FBX로 임포트된 캐릭터·Animation, Epic Starter Content, `LandscapeLab` 플러그인과 별도 텍스처가 포함되어 있습니다. 해당 콘텐츠는 직접 제작물로 주장하지 않으며, 공개 배포 전 원 배포처와 재배포 조건을 별도로 확인해야 합니다.

## 프로젝트 구조

```text
.
├─ *.uproject
├─ Config/
├─ Content/
│  ├─ _TEST3/               # 커스텀 캐릭터·공·좀비·맵
│  ├─ Characters/           # 외부 캐릭터 자산
│  └─ StarterContent/       # Epic 제공 콘텐츠
├─ Plugins/LandscapeLab/    # 외부 플러그인
└─ README.md
```

