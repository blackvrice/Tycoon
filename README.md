# Tycoon

Unity 6와 C#으로 Farm·Inventory·Shop·Economy·Save 상태를 연결한 싱글 플레이 농장 경영 Vertical Slice입니다.

**Gameplay GIF:** To be added

**Gameplay Video:** To be added

| 개발 | Engine | Language | 핵심 | 검증 |
|---|---|---|---|---|
| 개인 프로젝트 / 1인 개발 | Unity 6 `6000.3.10f1` | C# | State Flow · ScriptableObject · Save/Load · UI Toolkit | PlayMode / EditMode · Scene Smoke |

## Overview

밭을 갈고 작물을 키워 판매한 뒤 씨앗과 콘텐츠에 재투자하는 농장 경영 게임입니다. 핵심 목적은 기능 수를 늘리는 것이 아니라 Farm Domain, Inventory, Shop, Economy, Scene과 Save/Load의 책임을 분리하고 하나의 플레이 흐름에서 상태가 일관되게 이어지도록 만드는 것입니다.

## Core Gameplay

`Till → Plant → Water → Grow → Harvest → Sell → Reinvest → Save / Load`

새 게임에서 도구와 씨앗을 받고, Farm에서 작물을 재배·수확해 지역 상점에서 판매합니다. Main, Farm, Pasture, Livestock를 이동하며 공통 UI를 재사용하되 지역별 상품은 분리됩니다. 설정 UI를 통해 Save, Load, New Game 흐름을 실행할 수 있습니다.

## Technical Highlights

1. **Farm Domain / Tilemap Separation** — `FieldTile`과 `CropInstance`가 경작·수분·성장·수확 상태를 소유하고, `FarmTileControl`이 Tilemap과 crop sprite 표시를 갱신합니다.
2. **Farm–Inventory–Shop–Economy State Flow** — 씨앗 소모, 수확물 추가, stack/space 검사, 구매·판매와 Gold 변경을 독립 시스템으로 나누고 실제 재투자 루프에서 연결했습니다.
3. **Composite Save / Load Restoration** — `SaveManager`가 Tick, Gold, Inventory, Farm과 player/확장 콘텐츠 상태를 DTO로 묶고, ID lookup을 통해 ScriptableObject 참조를 복원합니다.
4. **Scene Contract Verification** — `RuntimePrefabSceneInstaller`가 씬별 필수 runtime object와 reference를 설치·검증하고, PlayMode Scene Smoke가 실제 Title/Main/Farm/Pasture/Livestock 로드 경로를 확인합니다.

## Architecture

```text
Farm Domain ───── Inventory / Hotbar
     │                    │
     └──── Player Interaction
                          │
              Shop / Economy / Scene Catalog
                          │
                     GameManager
                          │
                     SaveManager
                          │
               UI Toolkit / Tilemap / Scene Flow
```

`GameSession`은 씬 전환 뒤에도 Inventory, Economy, 선택 slot과 필요한 cache를 유지합니다. UI는 상태를 직접 수정하지 않고 `GameManager`, `ShopSystem`, Player/Farm interaction 경로에 명령을 위임합니다. Item, Crop과 Shop 구성은 `ScriptableObject`와 `GameDatabase`를 중심으로 관리합니다.

## Problem Solving

### Scene 전환 직후 재진입 또는 범위 밖 Spawn

**Problem** 고정 좌표로 귀환시키면 목적지의 실제 walkable bounds 밖에 놓이거나, 출구 trigger를 즉시 다시 밟아 씬이 재전환될 수 있었습니다.

**Cause** 목적지 Tilemap 크기와 출발 방향을 고려하지 않은 하나의 spawn 좌표를 사용했습니다.

**Solution** `GameWorldGrid`의 현재 경계를 조회하고 이동 방향의 반대쪽으로 한 칸 들어온 walkable cell을 계산해 `SceneTravelPosition`으로 전달했습니다.

**Verification** Main↔Farm/Pasture/Livestock 왕복 방향과 유효 위치를 `SceneTravelPositionPlayModeTests`와 실제 scene smoke에서 확인합니다.

### 공통 Shop UI의 지역 상품 누출

**Problem** 공통 상점 UI와 전역 bootstrap을 함께 사용하면서 동물·시설 상품이 관련 없는 씬의 상점에도 노출될 수 있었습니다.

**Cause** 저장용 데이터 등록과 현재 씬의 판매 stock 구성을 같은 전역 목록으로 처리했습니다.

**Solution** `SceneShopCatalog`가 Main/Farm/Pasture/Livestock별 stock을 구성하고, 공통 `InventoryShopUIController`에는 현재 지역 목록만 주입하도록 분리했습니다.

**Verification** `SceneShopCatalogPlayModeTests`가 각 실제 씬을 로드해 기대한 Item ID 집합과 무관한 상품의 부재를 검사합니다.

## Testing / Verification

자동 검증은 클래스 수를 나열하기보다 아래 gameplay 계약을 보호하는 데 사용합니다.

- **Farm** — Till/Plant/Water/Grow/Harvest와 잘못된 행동의 무변경 보장
- **Inventory / Shop / Economy** — Add/Remove/Stack/Move, 수용량, 구매·판매, Gold, 지역별 stock
- **Save / Load / New Game** — Tick, Gold, Inventory, Farm과 시작 상태 복원
- **Scene** — Title/Main/Farm/Pasture/Livestock 실제 로드, 필수 object의 유일성, 왕복 위치
- **Editor Contract** — 설치 반복 시 duplicate 제거, Player/Manager/UI/Database reference 검증

커밋된 `main`에는 위 범위를 검증하는 PlayMode/EditMode 테스트와 실제 Scene Smoke가 함께 있습니다. C# compile check는 아래 3개 target으로 빠르게 수행하고, 최종 회귀검증은 Unity Test Runner의 EditMode/PlayMode 전체 실행으로 확인합니다.

```powershell
dotnet build Game.Runtime.csproj --no-restore
dotnet build Assembly-CSharp-Editor.csproj --no-restore
dotnet build PlayMode.csproj --no-restore -maxcpucount:1
```

Unity Test Runner에서는 EditMode와 PlayMode 전체를 실행합니다. 실제 화면 배치, 카메라, 조작감과 해상도별 UI 겹침은 자동 테스트와 별도의 Editor Play Mode QA가 필요합니다.

## AI-assisted Development

생성형 AI는 코드 탐색, 반복 구현 초안, 리팩터링 후보와 테스트 작성 보조에 사용했습니다. 개발자가 요구사항, 시스템 책임, 게임 규칙과 합격 기준을 결정하고 변경 파일과 scene wiring을 직접 검토했습니다. 결과는 C# build, Unity Test Runner, 실제 scene load와 Play Mode로 다시 확인합니다.

## Technical Documentation

- [Core Systems](Assets/Scripts/Core) — Session, Game State, Economy, Save, Scene Flow
- [Farm Domain](Assets/Scripts/Farm) — FieldTile, Crop, Grid, Tilemap, Interaction
- [Inventory / Shop](Assets/Scripts/Inventory) · [Shop](Assets/Scripts/Shop) — 재투자 상태 흐름
- [PlayMode Tests](Assets/Tests/PlayMode) — Domain, Save, Shop, UI, Scene regression
- [Editor Installer Tests](Assets/Editor/Tests) — Runtime scene installation contract
- [Development Status](DEVELOPMENT_STATUS.md) — 장기 개발 기록과 후속 과제

## Build & Run

1. Unity Hub에서 Unity `6000.3.10f1`로 프로젝트를 엽니다.
2. `Assets/Scenes/TitleScene.unity`를 엽니다.
3. Play Mode에서 새 게임을 시작합니다.

| 입력 | 동작 |
|---|---|
| WASD / Arrow | 이동 |
| Left Click / Enter / Space | 선택 아이템 사용 |
| E | 상호작용 / 수확 |
| 1–9 / Wheel / Q·R | Hotbar 선택 |
| I | Inventory |
| 상인 근처 E | Shop |
| Esc | UI 닫기 / Settings |

## Current Scope

- `Assets/Scripts/Network`는 실제 플레이 루프에 연결되지 않은 초안이며 Multiplayer를 제출 기능으로 주장하지 않습니다.
- Gameplay GIF/Video와 해상도별 visual QA는 아직 필요합니다.
- First Play Guide, 공통 Gameplay HUD, Sunflower 확장은 제출된 `main`의 기능에 포함하지 않습니다.
- README와 코드가 다를 때는 default branch에 커밋된 source와 test를 최종 기준으로 봅니다.
