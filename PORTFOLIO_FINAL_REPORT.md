# Tycoon Portfolio Final Report

작성일: 2026-09-01 · 대상: Unity 6000.3.10f1 / C# / `main`

## 1. 프로젝트 분석 요약

프로젝트에는 Farm Domain, Inventory/Hotbar, Shop/Economy, Save/Load, Scene Flow, UI Toolkit, 동물/시설 확장과 자동 테스트가 이미 존재했습니다. 문제는 기능 부재보다 최신 시스템의 씬 wiring과 데이터 등록, 저장 포맷의 일부 누락, HUD/상점의 시각 충돌, 오래된 문서가 서로 어긋난 점이었습니다. 따라서 같은 기능을 새로 만들지 않고 기존 `GameSession`, `GameManager`, `FarmGrid`, `ShopSystem`, `RuntimePrefabSceneInstaller`를 연결·보강했습니다.

## 2. 가장 큰 문제 5개와 해결

1. **저장 후 작물 성장 진행도 되감김** — `GrowthTick` 누락을 Save DTO와 Farm load 경로에 추가했습니다.
2. **선택 Hotbar 슬롯 미복원** — Save v2에 슬롯을 기록하고 Inventory/Hotbar 유효 범위로 보정해 Player와 UI에 재적용했습니다.
3. **Sunflower가 코드에는 있으나 실제 씬 데이터에 불완전 등록** — Item/Crop/Shop/seed binding을 installer, Farm setup, authored Farm/Main scene에 연결했습니다.
4. **Unity 6 자동 Farm 구성의 레거시 Tile 경로** — 현재 RuleTile과 기본 walkable 계약으로 교체하고 재설치 검증을 통과시켰습니다.
5. **전체 화면 Shop/Settings와 HUD 겹침** — 기존 패널 공개 상태를 `UIController`가 관찰해 모달 동안 HUD/Hotbar VisualElement를 숨기고 닫을 때 상태 손실 없이 복원합니다.

## 3. Core Gameplay Loop 최종 상태

`Title → New Game → Main → Farm → Till → Plant → Water → Grow → Harvest → Sell → Buy Seed → Save/Load`가 연결됐습니다.

- 100G와 4개 시작 stack으로 즉시 플레이 가능
- Farm 타일 규칙과 Inventory 소비/수확물이 동일 상태 흐름 사용
- 상인의 실제 지역 catalog에서 수확물 판매와 씨앗 구매
- Wheat/Tomato/Sunflower의 시간 대비 이익 progression
- 첫 플레이 가이드가 도메인 상태를 변경하지 않고 7단계 진행 감지
- Settings와 Title에서 Save/Load/New Game 진입 가능

## 4. 변경한 파일과 이유

### Core / Save

- `GameSaveData.cs`, `FieldTileSaveData.cs` — Save v2의 선택 슬롯과 성장 진행 Tick
- `SaveManager.cs`, `GameManager.cs` — 저장/복원, 범위 보정, Player/Hotbar 재적용
- `FarmGrid.cs`, `FieldTile.cs` — Crop stage 내부 진행도의 round-trip

### Data / Scene Wiring

- `SunflowerCropData.asset`, seed/harvest ItemData — 세 번째 progression 콘텐츠
- `RuntimePrefabSceneInstaller.cs`, `FarmSceneSetupTool.cs` — 3종 작물과 Unity 6 Tile 계약
- `FarmScene.unity`, `MainScene.unity`, `TitleScene.unity` — installer로 검증된 authored wiring

### UI / UX

- `GameplayHUD.uxml`, `GameplayHUD.uss`, `HUDUI.cs` — 공통 상태/가이드/피드백
- `FirstPlayGuideController.cs` — 상태 관찰형 첫 플레이 안내
- `GameplayHudBootstrap.cs`, `GameplayHudInputBridge.cs` — 씬 공통 HUD와 기존 입력 연결
- `UIController.cs`, `HotbarControl.cs` — 모달 겹침 제거와 상태 보존 표시 전환
- Inventory/Shop/Settings/Title UXML·Controller — 한국어 정보 구조와 실제 System 위임

### Tests / Tools / Docs

- Save/Load, Farm Interaction, Scene Smoke, Scene Catalog, UI Contract, First Guide 테스트 보강
- `UIResponsiveLayoutPlayModeTests.cs` — 세 해상도 layout, modal visibility, 선택적 PNG 캡처
- `RuntimePrefabSceneInstallerEditModeTests.cs` — 8 Item/3 Crop/3 seed binding 계약
- `PortfolioBuildTool.cs` — Windows build 메뉴와 CLI
- README, Development Status, Capture Guide, Final Report — 현재 구현과 검증을 기준으로 재작성

## 5. 핵심 클래스 책임

| 클래스 | 책임 |
|---|---|
| `GameSession` | 씬 사이의 Inventory, Economy, 선택 슬롯, Farm/확장 cache 유지 |
| `GameManager` | 게임 상태, Save/Load/New Game, 런타임 관리자 조율 |
| `SaveManager` | 상태 스냅샷 JSON 직렬화와 ID 기반 복원 |
| `FarmGrid` | FieldTile 좌표 컬렉션, Tick 전달, 저장 데이터 변환 |
| `FieldTile` | 타일 상태, 수분, 심기/수확 전이 |
| `CropInstance` | 성장 단계와 단계 내부 Tick |
| `FarmInteractionController` | Player 대상 판정과 도메인 명령, 결과 피드백 |
| `Inventory` | slot/stack/add/remove/move와 저장 데이터 |
| `ShopSystem` | 구매/판매 가능 수량, 재고 이동, Gold 원자적 변경 |
| `GameDatabase` | Item/Crop ID lookup과 데이터 등록 |
| `SceneFlowManager` | fade 전환, 중복 방지, 이동 전 저장 |
| `RuntimePrefabSceneInstaller` | authored scene 런타임 계약 설치와 검증 |

## 6. 자료구조와 상태 모델

- `List<InventorySlot>`: 고정 순서 36-slot Inventory와 9-slot Hotbar mapping
- `Dictionary<string, ItemData/CropData>`: 저장 ID를 ScriptableObject로 복원하는 O(1) lookup
- 2차원 `FieldTile[,]`: Farm 좌표 기반 조회와 전체 Tick 순회
- DTO `List<FieldTileSaveData>`: JSON이 다차원 배열을 직접 다루지 않아 좌표와 상태를 평탄화
- `GameState`, `TileState`, `FirstPlayGuideStep`: 허용된 상태 전이를 enum으로 명시
- Scene별 `SceneShopCatalog`: 전역 데이터 등록과 현재 지역 판매 stock의 책임 분리

## 7. Data-driven Design 적용

Wheat, Tomato, Sunflower의 ID, 표시 이름, 성장 단계, Tick, 구매가, 판매가, harvest item과 Sprite는 ScriptableObject가 소유합니다. `FarmInteractionController`는 `SeedCropBinding`을 통해 seed ItemData를 CropData로 해석합니다. Shop은 `ShopItemData`로 가격과 잠금 상태를 받으므로 작물 추가에 새로운 switch/if 분기를 만들지 않습니다.

## 8. 실제 수행한 자동 테스트

- Farm: 경작, 파종, 물주기, 성장, 수확, 잘못된 행동의 무변경
- Inventory/Hotbar: stack, 이동, 공간 부족, 선택 슬롯과 UI 갱신
- Shop/Economy: 구매/판매, 최대 수량, 지역 stock, Gold, 재투자
- Save/Load/New Game: Scene, 위치, 선택 슬롯, Tick, Gold, Inventory, 성장 진행, Farm reset
- UI: UXML 계약, 한국어 문구, 설정 버튼, 첫 플레이 guide state machine
- Responsive UI: 1920×1080, 1600×900, 1280×720 경계와 modal/HUD visibility
- Scene: Title/Main/Farm/Pasture/Livestock 실제 load와 필수 object 유일성, 카메라/출구/상인/Tilemap
- Editor: installer 설치, 재설치 멱등성, 필수 참조와 3종 작물 데이터

## 9. 테스트 결과

| 검증 | 결과 |
|---|---|
| `dotnet build Game.Runtime.csproj` | warning 0 / error 0 |
| `dotnet build Assembly-CSharp-Editor.csproj` | warning 0 / error 0 |
| `dotnet build PlayMode.csproj` | warning 0 / error 0 |
| Unity EditMode | 4 passed / 0 failed |
| Unity PlayMode | 73 passed / 0 failed |
| Runtime scene reinstall/validation | passed |
| UI visual capture | 3 resolutions inspected |
| Windows Standalone build | succeeded, 약 103.4MB |
| Standalone headless boot | engine/Title input initialized, game code exception 없음 |

## 10. 해결한 버그와 설계 문제

- 부분 성장 저장 누락
- 선택 Hotbar 슬롯 저장 누락과 범위 초과 방어
- Sunflower 실제 Scene/GameDatabase/Shop/Seed binding 누락
- Scene installer의 Unity 6-invalid Tile 경로
- 공통 상점 catalog의 신규 작물 기대값 불일치
- HUD/가이드/Hotbar와 Shop/Settings의 화면 겹침
- 모달 숨김 과정에서 Hotbar GameObject 재활성화 시 아이콘 상태가 흔들리는 문제
- 실제 해상도를 바꿀 수 없는 batch-mode에서 RenderTexture viewport를 사용하는 UI 검증 경로

## 11. 남은 기술 부채

- Save v2 이후의 명시적 migration pipeline과 복수 slot UI는 없습니다.
- Network 폴더는 실험 초안이며 실제 gameplay에 연결하지 않았습니다.
- First Play Guide 진행은 PlayerPrefs 한 slot 기준이며 프로필 시스템은 없습니다.
- 동물/시설은 보조 콘텐츠라 Farm 경제만큼 회귀 범위가 깊지 않습니다.
- Windows의 장시간 수동 조작, 최종 오디오 mix와 Gameplay 영상은 촬영 단계에서 마지막 확인이 필요합니다.

## 12. 채용 담당자에게 강조할 포인트 5개

1. 이미 존재하는 시스템을 분석하고 책임 경계를 유지한 채 실제 vertical slice로 연결했습니다.
2. Farm 규칙, UI 표현, 데이터와 저장 DTO를 분리해 콘텐츠 추가 비용을 낮췄습니다.
3. Scene/Prefab/ScriptableObject wiring을 Editor tool과 실제 scene smoke로 검증합니다.
4. Save/Load를 단순 JSON 예제가 아니라 성장 중 상태와 선택 UI까지 round-trip 했습니다.
5. 73 PlayMode + 4 EditMode, 세 해상도 렌더 QA와 Windows build까지 같은 제출 기준으로 운영합니다.

## 13. 면접 예상 질문 10개와 코드 위치

1. **왜 Crop을 MonoBehaviour가 아니라 상태 객체로 뒀나요?** — `Assets/Scripts/Farm/CropInstance.cs`
2. **FieldTile의 허용 상태 전이는 어떻게 보장하나요?** — `Assets/Scripts/Farm/FieldTile.cs`
3. **씬 전환 후 Inventory와 Gold가 유지되는 이유는?** — `Assets/Scripts/Core/GameSession.cs`
4. **ScriptableObject 참조를 JSON에서 어떻게 복원하나요?** — `Assets/Scripts/Core/GameDatabase.cs`, `SaveManager.cs`
5. **부분 성장 중 Save/Load 버그는 어떻게 찾고 고쳤나요?** — `FieldTileSaveData.cs`, `GameManagerSaveLoadPlayModeTests.cs`
6. **상점 구매가 인벤토리 공간과 Gold를 함께 지키나요?** — `Assets/Scripts/Shop/ShopSystem.cs`
7. **지역마다 상점 상품이 다른데 전역 DB와 어떻게 분리했나요?** — `Assets/Scripts/Shop/SceneShopCatalog.cs`
8. **씬 자동 설치가 수동 Scene 작업을 덮어쓰지 않게 어떻게 검증하나요?** — `Assets/Editor/RuntimePrefabSceneInstaller.cs`와 EditMode tests
9. **반응형 UI를 batch-mode에서 어떻게 테스트했나요?** — `Assets/Tests/PlayMode/UIResponsiveLayoutPlayModeTests.cs`
10. **AI 사용 결과를 어떻게 검증했나요?** — C# compile, Unity Test Runner, scene smoke, RenderTexture visual QA, Windows build의 다층 기준

## 14. 포트폴리오 품질 점수

**91 / 100**

- 코드 구조와 핵심 루프: 24/25
- 데이터/저장 설계: 18/20
- 자동 테스트와 Editor tooling: 20/20
- UI/UX와 시각 피드백: 16/20
- 문서/빌드/제출 준비: 13/15

감점은 실제 Gameplay GIF/Video 미첨부, 장시간 수동 조작·오디오 최종 QA, 동물/시설 확장 깊이에서 발생합니다. 핵심 Farm vertical slice의 코드·테스트·빌드는 제출 가능한 상태입니다.

## 15. 최종 판정과 다음 최소 개선

**Portfolio Ready:** Yes — 핵심 Farm → Sell → Reinvest → Save/Load vertical slice 기준.

**미디어 제출 Ready:** 촬영 문서 기준 준비 완료, 실제 GIF/Video 파일 추가 필요.

최소 후속 작업은 3~5분 연속 수동 플레이 1회, 60~90초 영상 촬영, README 링크 교체입니다. 신규 대형 기능보다 현재 장면의 애니메이션 타이밍, SFX 밸런스와 영상 편집 완성도가 점수 상승에 더 직접적입니다.
