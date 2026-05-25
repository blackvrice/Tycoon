# Tycoon 개발 현황 및 다음 단계

갱신일: 2026-05-25
목적: 지금까지 만든 기능을 한눈에 확인하고, 다음 개발 우선순위를 잃지 않기 위한 작업 메모입니다.

## 현재 한 줄 요약

농장 기본 플레이 루프는 코드상 연결되었고, 런타임 씬 자동 조립 도구와 PlayMode/EditMode 자동 QA까지 통과했습니다. 현재 기준으로는 에디터 메뉴 또는 명령줄로 Lobby/Farm/Marketplace/Main 씬을 재설치하고, 필수 런타임 오브젝트와 참조가 빠졌는지 자동 검증할 수 있으며, 실제 Lobby/Farm/Main 씬을 PlayMode에서 로드하는 스모크 QA까지 통과합니다. 잘못된 농장 액션, 상점 구매/판매, Save / Load / New Game, 설정 UI 버튼 경유 저장 흐름, 기본 재투자 경제 흐름, FarmGridController 좌표/Tilemap 갱신, 시작 인벤토리 지급 규칙, 씬 설치 도구의 최소 계약도 자동 검증합니다. HUD/상점/설정/인벤토리의 표시 문구는 한국어 기준으로 1차 통일했습니다.

## 전체 소스 상황

현재 소스는 싱글 플레이 농장 루프를 중심으로 꽤 명확하게 층이 나뉘어 있습니다.

### 핵심 런타임 축

- `Assets/Scripts/Core`
  - 게임 전체 상태, 세션 유지, Tick, 저장/불러오기, 골드, 설정, 씬 이동, 런타임 데이터베이스를 담당합니다.
  - 현재 프로젝트의 중심축은 `GameSession`, `GameManager`, `TickManager`, `SaveManager`, `GameDatabase`입니다.
- `Assets/Scripts/Data`
  - 아이템, 작물, 상점 상품 ScriptableObject 정의가 있습니다.
  - 앞으로 콘텐츠를 늘릴 때 가장 먼저 만지게 될 데이터 계층입니다.
- `Assets/Scripts/Runtime`
  - 저장 파일에 들어가는 DTO 성격의 데이터 클래스가 있습니다.
  - 저장 포맷이 바뀌면 여기와 `SaveManager`, `FarmGrid`를 같이 봐야 합니다.

### 플레이 루프 축

- `Assets/Scripts/Farm`
  - 밭 타일 상태, 작물 성장, 농장 그리드, Tilemap 반영, 플레이어 농장 액션을 담당합니다.
  - 현재 핵심 루프인 밭 갈기, 씨앗 심기, 물 주기, 성장, 수확이 여기서 완성되어 있습니다.
- `Assets/Scripts/Inventory`
  - 인벤토리 슬롯, 아이템 스택, 시작 아이템 지급 규칙을 담당합니다.
  - 핫바와 상점, 저장/불러오기 흐름의 공통 기반입니다.
- `Assets/Scripts/Shop`
  - 구매/판매, 가격, 최대 구매/판매 수량 계산을 담당합니다.
  - 경제 밸런스와 상품 잠금/해금 규칙을 붙일 다음 후보입니다.
- `Assets/Scripts/Player`, `Assets/Scripts/Input`, `Assets/Scripts/Hotbar`
  - 플레이어 이동/선택 슬롯, 입력 액션, 핫바 표시를 담당합니다.
  - 농장 조작감 QA와 키 바인딩 정리 때 같이 봐야 합니다.

### 화면과 씬 조립 축

- `Assets/Scripts/UI`, `Assets/UI`
  - HUD, 인벤토리, 상점, 설정 UI와 UXML/USS가 있습니다.
  - 현재 핵심 문구는 한국어 기준으로 1차 정리되었지만, 실제 해상도별 겹침 QA는 남아 있습니다.
- `Assets/Scripts/Tilemap`, `Assets/Scripts/Camera`
  - Tilemap 보조 기능과 카메라 이동 관련 코드가 있습니다.
  - FarmGrid 시각 품질과 카메라 프레이밍을 다듬을 때 보게 됩니다.
- `Assets/Editor`
  - 런타임 오브젝트 자동 설치/검증, 농장 씬 참조 연결, 테스트 데이터 생성 도구가 있습니다.
  - 현재는 수동 씬 세팅 실수를 줄이는 안전장치 역할을 합니다.

### 아직 뒤쪽으로 미룰 축

- `Assets/Scripts/Network`
  - 네트워크 메시지, 역할, 패킷, 동기화 초안이 있습니다.
  - 아직 실제 플레이 루프와 연결하지 않았고, 지금은 싱글 플레이 안정화가 우선입니다.

## 현재 개발 판단

현재 프로젝트는 "기능이 없어서 막힌 상태"라기보다 "기본 루프가 생겼으니 실제 플레이 감각과 콘텐츠 밀도를 올려야 하는 상태"입니다. 즉, 당장 큰 구조를 갈아엎기보다 지금 연결된 농장 루프를 Unity Editor에서 눈으로 확인하고, 화면 겹침, 조작감, 밸런스, 데이터 콘텐츠를 순서대로 다듬는 쪽이 좋습니다.

지금 기준으로 안정성이 비교적 높은 부분:

- 핵심 농장 도메인 로직
- 인벤토리/핫바 기본 동작
- 상점 구매/판매와 수량 제한
- Save / Load / New Game 기본 흐름
- 씬 자동 조립과 런타임 필수 오브젝트 검증
- PlayMode/EditMode 자동 테스트 기반

아직 수동 확인이나 후속 설계가 필요한 부분:

- 실제 Editor Play Mode에서의 화면 배치, 카메라, UI 겹침
- 한국어 문구 적용 후 버튼/라벨 길이 문제
- 실제 아이템/작물 데이터 이름과 아이콘 정리
- 상점 잠금/해금, 가격, 성장 시간, 시작 골드 밸런스
- 마우스/키보드 조작감과 상호작용 거리
- 네트워크 구조의 실제 적용 여부와 권위 모델 결정

## 개발 방향

### 1. 먼저 눈으로 플레이 가능한 싱글 루프를 잠근다

가장 가까운 목표는 Farm 씬을 열고 Play Mode에서 아래 흐름이 끊기지 않게 만드는 것입니다.

1. 시작 인벤토리 지급
2. 핫바 도구 선택
3. 밭 갈기
4. 씨앗 심기
5. 물 주기
6. Tick 성장
7. 수확
8. 상점 판매
9. 씨앗 재구매
10. 저장/불러오기
11. 새 게임

이 흐름이 눈으로 봐도 자연스러워지면, 그때부터 콘텐츠를 늘리는 비용이 크게 줄어듭니다.

### 2. 콘텐츠는 데이터 중심으로 늘린다

다음 콘텐츠 추가는 코드를 많이 고치기보다 `ItemData`, `CropData`, `ShopItemData`, `GameDatabase` 중심으로 가는 것이 좋습니다.

- 작물 2종에서 4~6종으로 확장
- 씨앗 구매가, 수확물 판매가, 성장 Tick 조정
- 아이콘 Sprite 누락분 정리
- 시작 골드와 시작 아이템 수량 조정
- 상점 상품 잠금/해금 조건 추가

### 3. UI는 실제 플레이 화면 기준으로 다듬는다

UI는 기능 단위 구현은 되어 있으므로, 이제는 "보이는 품질" 기준으로 작업해야 합니다.

- 한국어 긴 문구가 버튼과 패널 안에서 잘리는지 확인
- HUD, 핫바, 인벤토리, 상점, 설정 패널이 겹치지 않는지 확인
- 상점 상세 패널의 정보 밀도 조정
- 아이콘 없는 아이템의 임시 이니셜 표시를 실제 Sprite로 교체
- 해상도별 USS 여백, 폰트 크기, 패널 폭 조정

### 4. 자동 테스트는 지금처럼 기능을 고정하는 데 쓴다

현재 테스트는 좋은 안전망입니다. 앞으로도 기능을 늘릴 때마다 "한 번 고친 기능이 다시 깨지지 않게" 작게 추가하는 방향이 좋습니다.

- 작물 추가 시 성장/수확/판매 루프 테스트 확장
- 상점 잠금/해금 추가 시 구매 가능 여부 테스트 추가
- UI 문구/요소 이름 변경 시 UXML 계약 테스트 갱신
- 씬 자동 설치 규칙이 바뀌면 EditMode 설치 테스트 갱신

### 5. 네트워크는 싱글 플레이가 안정된 뒤 다시 본다

네트워크 초안은 남겨두되, 지금 당장 연결하면 디버깅 축이 너무 늘어납니다. 싱글 플레이 농장 루프, 저장, 상점, UI가 안정된 뒤에 권위 구조를 결정하는 것이 좋습니다.

권장 순서:

1. 싱글 플레이 루프 Editor QA
2. UI 겹침/문구/아이콘 정리
3. 작물/상점 콘텐츠 확장
4. 경제 밸런스 조정
5. 저장 포맷 안정화
6. 네트워크 권위 구조 결정

## 이번 작업 완료

- `RuntimePrefabSceneInstaller`에 씬 검증 메뉴와 명령줄 검증 메서드를 추가했습니다.
- Lobby/Farm/Marketplace/Main 씬을 대상으로 `ReinstallAndValidateConfiguredScenesFromCommandLine`을 실행했습니다.
- Unity batch-mode에서 `Configured runtime scene validation: validation passed.`를 확인했습니다.
- `HUDUI`가 없는 씬에서는 `FarmInteractionController.hudUI`를 선택 참조로 취급해 불필요한 경고가 나오지 않게 했습니다.
- `FarmInteractionController`에 밭 갈기, 심기, 물 주기, 수확 실패 이유 메시지를 추가했습니다.
- HUD 상호작용 힌트가 현재 선택한 도구/씨앗 기준으로 더 구체적인 안내를 표시합니다.
- `FarmInteractionPlayModeTests`를 추가해 실제 컴포넌트 조합으로 농장 루프를 자동 검증합니다.
- 농장 상호작용 추가 후 Unity Test Runner batch-mode PlayMode 테스트 통과를 확인했습니다.
- `InventoryShopUIController`에 구매/판매 수량 선택을 추가했습니다.
- 상점/인벤토리 화면에 선택 아이템 상세 패널을 추가했습니다.
- 아이콘이 없는 아이템은 이름 첫 글자를 기본 아이콘처럼 표시하도록 했습니다.
- `UIContractPlayModeTests`에 상점 수량/상세 패널 UXML 계약을 추가했습니다.
- `GameManager`의 New Game 흐름에 `StartingInventoryLoader`를 연결해 새 게임 후 시작 아이템을 다시 지급하도록 했습니다.
- `GameManagerSaveLoadPlayModeTests`를 추가해 `GameManager` 기준 Save / Load / New Game 통합 흐름을 검증합니다.
- Save / Load가 골드, 인벤토리, Tick, 농장 작물 상태를 복원하는지 확인했습니다.
- New Game이 저장 파일 삭제, 골드/틱/선택 슬롯/농장 초기화, 시작 인벤토리 재지급을 수행하는지 확인했습니다.
- 저장/새 게임 통합 테스트 추가 후 Unity Test Runner batch-mode PlayMode 테스트 통과를 확인했습니다.
- `ShopSystem`에 현재 골드와 인벤토리 공간 기준 최대 구매 수량 계산을 추가했습니다.
- `ShopSystem`에 현재 보유량 기준 최대 판매 수량 계산을 추가했습니다.
- 상점 UI 수량 조절이 선택 아이템의 구매/판매 가능 수량 한계를 따라가도록 개선했습니다.
- 상점 상세 패널에 최대 구매/판매 가능 수량을 표시하도록 했습니다.
- 씨앗 구매 후 수확물을 판매하면 다음 씨앗을 다시 살 수 있는 기본 재투자 루프 테스트를 추가했습니다.
- Unity Test Runner batch-mode PlayMode 테스트 결과 `17 passed / 0 failed`를 확인했습니다.
- `SettingUIControllerPlayModeTests`를 추가해 실제 설정 UI 버튼 경유 Save / Load / New Game 흐름을 검증합니다.
- 저장 파일이 없을 때 Load 버튼이 상태 메시지를 표시하는지 확인했습니다.
- Save / Load 버튼 클릭 후 골드, 인벤토리, Tick, 농장 작물 상태가 복원되는지 확인했습니다.
- New Game 버튼과 확인 팝업을 거쳐 저장 파일 삭제, 런타임 초기화, 시작 인벤토리 재지급이 되는지 확인했습니다.
- Unity Test Runner batch-mode PlayMode 테스트 결과 `19 passed / 0 failed`를 확인했습니다.
- `FarmGridControllerPlayModeTests`를 추가해 `originCell`, Tilemap 레이아웃, 셀 중심/크기 계산을 검증합니다.
- `FarmGridController`의 밭 갈기, 심기, 물 주기, 성장, 수확 시 Tilemap 타일과 작물 Sprite가 갱신되는지 확인했습니다.
- Unity Test Runner batch-mode PlayMode 테스트 결과 `21 passed / 0 failed`를 확인했습니다.
- `StartingInventoryLoaderPlayModeTests`를 추가해 시작 도구/씨앗 지급 규칙을 검증합니다.
- 선호 슬롯 배치, 선호 슬롯 충돌 시 빈 슬롯 fallback, 중복 지급 방지, 기존 인벤토리 교체 지급을 확인했습니다.
- Unity Test Runner batch-mode PlayMode 테스트 결과 `24 passed / 0 failed`를 확인했습니다.
- `RuntimePrefabSceneInstaller`가 `Player`도 생성/정리 대상에 포함하도록 보강했습니다.
- Farm 씬 자동 조립 시 `GameManager.startingInventoryLoader` 참조를 명시적으로 연결하고, 검증 도구가 누락을 잡도록 했습니다.
- `RuntimePrefabSceneInstallerEditModeTests`를 추가해 임시 Farm 씬 설치, 필수 런타임 계약, 반복 설치 시 중복 제거를 검증합니다.
- Lobby/Farm/Marketplace/Main 씬을 재설치하고 새 검증 조건으로 통과를 확인했습니다.
- Unity Test Runner batch-mode EditMode 테스트 결과 `2 passed / 0 failed`를 확인했습니다.
- Unity Test Runner batch-mode PlayMode 테스트 결과 `24 passed / 0 failed`를 다시 확인했습니다.
- `RuntimePrefabSceneInstaller`가 비농장 씬에 남은 `FarmGridController`, `FarmInteractionController`, `StartingInventoryLoader` 같은 농장 런타임 컴포넌트를 제거하도록 보강했습니다.
- 예전 자동 조립 잔재였던 `FarmGroundTileMap`, `FarmCropTileMap`, `FarmWateredOverlayTileMap`, `FieldTile Debug Tester` 정리도 설치 도구에 포함했습니다.
- 비농장 씬 검증에서 농장 런타임 컴포넌트가 남아 있으면 실패하도록 했습니다.
- `SceneSmokePlayModeTests`를 추가해 실제 `FarmScene`, `MainScene`, `LobbyScene` 로드 후 런타임/UI/카메라/농장 타겟/비농장 씬 잔재를 검증합니다.
- Unity Test Runner batch-mode PlayMode 테스트 결과 `27 passed / 0 failed`를 확인했습니다.
- `DEVELOPMENT_STATUS.md`를 현재 검증 결과와 다음 개발 순서에 맞게 갱신했습니다.
- UI 표시 언어 기준을 한국어로 잡고 `InventoryUI`, `InventoryShop`, `SettingUI` UXML의 고정 라벨을 한국어로 정리했습니다.
- `HUDUI` 안에 공용 UI 문자열/포맷 헬퍼를 추가해 골드, 날짜, Tick, 선택 아이템, 상점 상세 패널, 설정 상태 메시지 문구를 일관되게 표시하도록 했습니다.
- `InventoryShopUIController`의 구매/판매 버튼, 가격, 선택 아이템, 상세 패널, 잠금/판매 불가/구매 불가 문구를 한국어 기준으로 정리했습니다.
- `SettingUIController`의 저장/불러오기/새 게임 상태 메시지를 한국어로 정리하고 기존 PlayMode 테스트 기대값을 갱신했습니다.
- `FarmInteractionController`의 HUD 메시지와 타겟 힌트, 타일/작물 상태 표시를 한국어로 정리했습니다.
- `UIContractPlayModeTests`에 주요 UXML 표시 문구가 한국어인지 확인하는 계약 테스트를 추가했습니다.
- Unity Test Runner batch-mode PlayMode 테스트 결과 `28 passed / 0 failed`를 확인했습니다.
- 전체 소스 상황, 현재 개발 판단, 앞으로의 개발 방향을 상단 요약으로 다시 정리했습니다.

## 구현 완료

### 1. 코어 런타임

- `GameSession`
  - 씬 전환 후에도 유지되는 인벤토리, 골드, 선택 핫바 슬롯 상태를 관리합니다.
- `GameManager`
  - 게임 상태 전환, 시작/일시정지/재개/종료, 저장/불러오기/새 게임 흐름을 관리합니다.
  - `NewGame()`과 `ResetRuntimeState()`로 골드, 인벤토리, 핫바, 농장, Tick 상태를 초기화합니다.
  - 새 게임 초기화 후 `StartingInventoryLoader`를 통해 시작 도구/씨앗을 다시 지급합니다.
- `TickManager`
  - Tick, Day, TickInDay 계산과 `ITickable` 등록/해제, 강제 Tick, Pause/Resume을 지원합니다.
- `EconomyManager`
  - 골드 추가/차감, 구매 가능 여부를 관리합니다.
- `SaveManager`
  - JSON 기반 저장/불러오기/삭제를 지원합니다.
  - 저장 대상은 Tick, Day, Gold, Inventory, Farm field 상태입니다.
- `AudioManager`
  - 음악/SFX 소스와 볼륨 제어 구조가 준비되었습니다.
- `SettingsManager`
  - 마스터/BGM/SFX 볼륨, 전체화면, VSync 설정을 저장하고 `AudioManager`와 동기화합니다.
- `SceneFlowManager`
  - Lobby/Farm/Marketplace/Main 씬 이름과 씬 이동 흐름을 관리합니다.
- `GameDatabase`
  - 런타임에서 사용할 아이템, 작물, 상점 상품 데이터를 한곳에서 조회할 수 있게 준비되었습니다.

### 2. 인벤토리와 핫바

- `ItemData`, `ItemStack`, `InventorySlot`, `Inventory`
  - 아이템 ID, 이름, 타입, 스택, 가격, 아이콘 데이터와 인벤토리 조작을 처리합니다.
- `HotbarControl`
  - `GameSession.SelectedHotbarSlotIndex`를 기준으로 선택 슬롯을 동기화합니다.
  - 인벤토리 변경 후 핫바 새로고침을 지원합니다.
- `StartingInventoryLoader`
  - 플레이 시작 시 기본 도구와 씨앗을 지급합니다.
  - 기본 슬롯 예시: Hoe 0번, WateringCan 1번, Wheat Seed 2번, Carrot Seed 3번.
  - 선호 슬롯, fallback 슬롯, 중복 지급 방지, 기존 인벤토리 교체 지급이 PlayMode 테스트로 검증되었습니다.

### 3. 농장 도메인 로직

- `CropData`
  - 작물 ID, 이름, 성장 단계, 단계별 Tick, 단계별 Sprite, 수확 아이템을 가집니다.
- `CropInstance`
  - 개별 작물의 성장 Tick, 현재 단계, 수확 가능 상태를 관리합니다.
- `FieldTile`
  - `Empty`, `Tiled`, `Planted`, `Grown` 상태를 관리합니다.
  - 밭 갈기, 심기, 물 주기, Tick 성장, 수확을 처리합니다.
- `FarmGrid`
  - 2차원 농장 타일 배열과 저장 데이터 생성/복원을 담당합니다.
- `FarmGridController`
  - 실제 씬의 Tilemap과 `FarmGrid`를 연결합니다.
  - `TickManager`에 등록되어 작물 성장을 진행합니다.
  - Tilemap 갱신, 저장/불러오기/새 게임, 좌표 변환을 제공합니다.
  - `GridToCellWorldBounds()`와 `GetCellWorldSize()`로 타겟 셀 표시를 지원합니다.
  - 좌표 변환과 Tilemap 갱신 흐름이 PlayMode 테스트로 검증되었습니다.

### 4. 플레이어 농장 상호작용

- `FarmInteractionController`
  - 선택 아이템에 따라 농장 액션을 실행합니다.
  - Hoe: 빈 땅을 갈아진 밭으로 변경합니다.
  - Watering Can: 심어진 작물 타일에 물을 줍니다.
  - Seed: 갈아진 밭에 작물을 심고 씨앗 1개를 소비합니다.
  - Interact: 다 자란 작물을 수확하고 인벤토리에 추가합니다.
  - 실패 시 이미 심어진 작물, 갈 수 없는 타일, 물 줄 수 없는 상태, 수확 불가 상태, 인벤토리 가득 참 등을 HUD 메시지로 안내합니다.
- 타겟 지정
  - 마우스가 가까우면 마우스 위치를 우선 사용합니다.
  - 아니면 플레이어가 마지막으로 바라본 방향의 셀을 사용합니다.
- HUD 피드백
  - 현재 타겟 좌표, 타일 상태, 물 준 상태, 작물 성장 단계, 선택 아이템 기준 행동 힌트를 표시합니다.
- 타겟 마커
  - `LineRenderer`로 현재 조작 대상 셀의 테두리를 화면에 표시합니다.

### 5. UI

- `InventoryUIController`
  - UI Toolkit 기반 인벤토리 UI를 관리합니다.
  - 슬롯 표시, 클릭 이동, 핫바 갱신과 연결되어 있습니다.
- `InventoryShopUIController`
  - 상점/인벤토리 패널을 표시합니다.
  - UXML 루트 이름을 `InventoryShopRoot` 기준으로 맞췄습니다.
  - `ShopSystem` 기반 구매/판매 흐름을 사용합니다.
  - 선택 아이템 판매 UI가 추가되었습니다.
  - 구매/판매 수량 조절, 선택 아이템 상세 패널, 아이콘 없는 아이템의 기본 이니셜 표시를 지원합니다.
  - 구매/판매/상세/상태 문구를 한국어 기준으로 표시합니다.
- `SettingUIController`
  - 설정 패널, 볼륨, 전체화면, VSync 제어를 담당합니다.
  - Save / Load / New Game 버튼을 제공합니다.
  - New Game 확인 팝업을 제공합니다.
  - 버튼 클릭 경유 저장/불러오기/새 게임 흐름과 한국어 상태 메시지가 PlayMode 테스트로 검증되었습니다.
- `HUDUI`
  - 날짜, Tick, 시간, 골드, 선택 아이템, 농장 타일/작물 정보, 메시지, 상호작용 힌트를 표시합니다.
  - 화면 표시용 공용 한국어 문자열/포맷 헬퍼를 함께 제공합니다.

### 6. 상점과 경제

- `ShopItemData`, `ShopItemEntry`, `ShopSystem`
  - 구매/판매 가능 여부, 가격 계산, 골드 차감/추가, 인벤토리 추가/제거를 처리합니다.
  - 골드, 인벤토리 공간, 보유량 기준으로 최대 구매/판매 가능 수량을 계산합니다.
- 상점 UI
  - 씨앗 구매, 수확물 판매, 선택 아이템 판매 흐름이 준비되었습니다.
  - 수량 선택으로 여러 개를 한 번에 구매/판매할 수 있습니다.
  - 수량 선택 버튼은 현재 구매/판매 가능한 최대치에 맞춰 비활성화됩니다.

### 7. 씬 자동 조립 도구

- `RuntimePrefabSceneInstaller`
  - 런타임 공통 오브젝트, 입력, UI, 코어 매니저, 플레이어, 농장 오브젝트를 씬에 설치합니다.
  - 메뉴:
    - `Tycoon/Prefabs/Install Runtime Objects In Open Scene`
    - `Tycoon/Prefabs/Install Runtime Objects In Configured Scenes`
    - `Tycoon/Prefabs/Validate Runtime Objects In Open Scene`
    - `Tycoon/Prefabs/Validate Runtime Objects In Configured Scenes`
    - `Tycoon/Prefabs/Reinstall And Validate Configured Scenes`
  - 명령줄:
    - `Tycoon.Editor.RuntimePrefabSceneInstaller.InstallInConfiguredScenesFromCommandLine`
    - `Tycoon.Editor.RuntimePrefabSceneInstaller.ValidateConfiguredScenesFromCommandLine`
    - `Tycoon.Editor.RuntimePrefabSceneInstaller.ReinstallAndValidateConfiguredScenesFromCommandLine`
  - 반복 실행 시 기존 런타임 오브젝트를 정리하고 다시 설치한 뒤, 필수 컴포넌트와 참조를 검증합니다.
  - `Player`도 설치 도구의 생성/정리 대상에 포함되어, 예전 씬에 남은 농장 상호작용/시작 인벤토리 컴포넌트와 새 FarmRuntime 오브젝트가 중복되지 않게 했습니다.
  - 비농장 씬에서는 남아 있는 농장 런타임 컴포넌트를 제거하고, 검증 단계에서도 농장 컴포넌트 잔재를 오류로 처리합니다.
  - 예전 자동 조립 이름인 `FarmGroundTileMap`, `FarmCropTileMap`, `FarmWateredOverlayTileMap`, `FieldTile Debug Tester`도 재설치 시 정리합니다.
  - Farm 씬에서는 `GameManager.startingInventoryLoader`까지 자동 연결하고 검증합니다.
  - 검증 대상에는 `RuntimeCore`, `GameSession`, `GameManager`, `TickManager`, `AudioManager`, `SettingsManager`, `SceneFlowManager`, `GameDatabase`, `RuntimeInput`, `RuntimeUIRoot`, `Player`, `FarmGridController`, `FarmInteractionController`, `StartingInventoryLoader` 등이 포함됩니다.
- `FarmSceneSetupTool`
  - 농장 테스트에 필요한 참조를 현재 씬 또는 Main/Farm 씬에 자동 연결합니다.
  - 메뉴:
    - `Tycoon/Farm/Setup Active Farm Scene`
    - `Tycoon/Farm/Setup Main And Farm Scenes`
- 자동 연결 대상
  - `GameSession`, `GameManager`, `TickManager`, `RuntimeInput`, `RuntimeUIRoot`, `Player`
  - `FarmGridController`, 농장 Tilemap, `FarmInteractionController`, `StartingInventoryLoader`
  - Hoe, Watering Can, Wheat/Carrot Seed, Wheat/Carrot Crop, 수확 아이템

### 8. 테스트 데이터

- 작물 데이터
  - `Assets/TestData/Crops/Test_Wheat_Crop.asset`
  - `Assets/TestData/Crops/Test_Carrot_Crop.asset`
- 아이템 데이터
  - `Test_Wheat_Item`
  - `Test_Carrot_Item`
  - `Test_Wheat_Seed_Item`
  - `Test_Carrot_Seed_Item`
  - `Tool_Hoe_Item`
  - `Tool_WateringCan_Item`
- 작물 단계 Sprite
  - Wheat/Carrot 0~3 단계 Sprite가 준비되어 있습니다.

### 9. 테스트

- `CoreSystemsPlayModeTests`
  - Inventory 추가/이동/삭제
  - FieldTile 성장/수확
  - ShopSystem 구매/판매/잠긴 상품
  - ShopSystem 최대 구매/판매 가능 수량
  - 씨앗 구매, 수확물 판매, 다음 씨앗 재구매로 이어지는 경제 루프
  - SaveManager 저장/불러오기
- `UIContractPlayModeTests`
  - UXML 요소 이름과 컨트롤러 쿼리 계약 확인
  - 인벤토리/상점/설정 UXML의 주요 표시 문구가 한국어인지 확인
- `TickManagerPlayModeTests`
  - Tick 진행과 Pause 동작 확인
- `FarmInteractionPlayModeTests`
  - `FarmInteractionController` 기준 밭 갈기, 씨앗 심기, 물 주기, Tick 성장, 수확 루프 확인
  - 잘못된 농장 액션이 타일/인벤토리 상태를 변경하지 않는지 확인
- `FarmGridControllerPlayModeTests`
  - `originCell`, `Grid`/`Tilemap` 셀 크기, 월드-그리드 좌표 변환 확인
  - 밭 상태 변경, 작물 Sprite, 물 표시 Overlay가 Tilemap에 반영되는지 확인
- `StartingInventoryLoaderPlayModeTests`
  - 시작 아이템 선호 슬롯 배치와 fallback 슬롯 배치 확인
  - 빈 인벤토리에서만 지급하는 규칙으로 중복 지급 방지 확인
  - 기존 인벤토리를 비우고 시작 아이템으로 교체하는 지급 흐름 확인
- `GameManagerSaveLoadPlayModeTests`
  - `GameManager` 기준 저장/불러오기 후 골드, 인벤토리, Tick, 농장 작물 상태 복원 확인
  - `NewGame()` 실행 후 저장 파일 삭제, 런타임 초기화, 시작 인벤토리 재지급 확인
- `SettingUIControllerPlayModeTests`
  - 설정 UI Save / Load 버튼 클릭 후 런타임 상태 저장/복원 확인
  - New Game 버튼, 확인 팝업, Confirm 버튼 클릭 후 저장 파일 삭제와 런타임 초기화 확인
- `RuntimePrefabSceneInstallerEditModeTests`
  - 임시 Farm 씬을 만들고 `Install Runtime Objects In Open Scene` 경로로 런타임 오브젝트 설치 확인
  - 필수 루트/컴포넌트/직렬화 참조, FarmGrid Tilemap 기본 페인트, Player 스폰 위치 확인
  - 같은 씬에 반복 설치해도 `RuntimeCore`, `RuntimeUIRoot`, `Player`, `FarmRuntime`, `FarmInteraction`, `StartingInventory` 등이 중복 생성되지 않는지 확인
- `SceneSmokePlayModeTests`
  - 실제 `FarmScene`을 PlayMode로 로드해 GameManager, TickManager, Player, FarmGrid, FarmInteraction, 시작 인벤토리, Hotbar/UI 바인딩 확인
  - 플레이어와 현재 농장 타겟 셀이 카메라 안에 들어오는지 확인
  - 타겟 마커가 생성되고 4개 포인트로 표시 준비되는지 확인
  - 실제 `MainScene`, `LobbyScene`을 로드해 공통 런타임/UI가 뜨고 농장 런타임 컴포넌트 잔재가 없는지 확인

## 최근 검증 결과

아래 명령은 최근 작업 후 통과했습니다.

```powershell
dotnet build Game.Runtime.csproj --no-restore -maxcpucount:1
dotnet build PlayMode.csproj --no-restore -maxcpucount:1
dotnet test PlayMode.csproj --no-build -maxcpucount:1
```

참고: `dotnet test PlayMode.csproj --no-build -maxcpucount:1`는 성공 종료했지만 Unity Test Runner의 상세 리포트를 별도로 출력하지는 않았습니다.

Unity batch-mode 검증도 통과했습니다.

```powershell
& 'C:\Program Files\Unity\Hub\Editor\6000.3.10f1\Editor\Unity.exe' -batchmode -quit -projectPath 'C:\Users\black\wkspaces\Tycoon' -executeMethod Tycoon.Editor.RuntimePrefabSceneInstaller.ReinstallAndValidateConfiguredScenesFromCommandLine -logFile 'C:\Users\black\wkspaces\Tycoon\Temp\runtime-scene-validation.log'
```

결과:

```text
Configured runtime scene validation: validation passed.
Exiting batchmode successfully now!
```

Unity Test Runner EditMode batch-mode도 통과했습니다.

```powershell
& 'C:\Program Files\Unity\Hub\Editor\6000.3.10f1\Editor\Unity.exe' -batchmode -projectPath 'C:\Users\black\wkspaces\Tycoon' -runTests -testPlatform EditMode -testResults 'C:\Users\black\wkspaces\Tycoon\Temp\editmode-test-results.xml' -logFile 'C:\Users\black\wkspaces\Tycoon\Temp\editmode-tests.log'
```

결과:

```text
EditMode test-run: Passed, total 2, passed 2, failed 0, skipped 0.
```

참고: `dotnet build Assembly-CSharp-Editor.csproj --no-restore -maxcpucount:1`는 Unity가 생성한 패키지 프로젝트의 `Temp\Bin` 메타데이터 경로 문제로 실패했습니다. 같은 코드에 대해 Unity batch-mode의 Bee 컴파일은 `Assembly-CSharp-Editor.dll`까지 성공했고, 이후 씬 재설치/검증도 통과했습니다.

Unity Test Runner PlayMode batch-mode도 통과했습니다.

```powershell
& 'C:\Program Files\Unity\Hub\Editor\6000.3.10f1\Editor\Unity.exe' -batchmode -projectPath 'C:\Users\black\wkspaces\Tycoon' -runTests -testPlatform PlayMode -testResults 'C:\Users\black\wkspaces\Tycoon\Temp\playmode-test-results.xml' -logFile 'C:\Users\black\wkspaces\Tycoon\Temp\playmode-tests.log'
```

결과:

```text
PlayMode test-run: Passed, total 28, passed 28, failed 0, skipped 0.
```

참고: Unity 로그에는 `Temp\editmode-test-results.xml`와 `Temp\playmode-test-results.xml` 저장이 찍혔지만, 상세 XML은 실행 후 `C:\Users\black\AppData\LocalLow\DefaultCompany\Tycoon\TestResults.xml`에서 확인했습니다.

## 현재 사용 가능한 플레이 루프

1. Unity에서 농장 테스트할 씬을 엽니다.
2. `Tycoon/Prefabs/Install Runtime Objects In Open Scene` 또는 `Tycoon/Prefabs/Reinstall And Validate Configured Scenes`를 실행합니다.
3. Play Mode 진입.
4. 핫바에서 Hoe를 선택해 밭을 갑니다.
5. Wheat/Carrot Seed를 선택해 씨앗을 심습니다.
6. Watering Can으로 물을 줍니다.
7. Tick이 지나 작물이 자라면 Interact로 수확합니다.
8. 인벤토리와 상점 UI에서 구매/판매 흐름을 확인합니다.
9. 설정 UI에서 Save / Load / New Game을 확인합니다.

## 남은 주의 사항

- Unity batch-mode 설치/검증은 통과했지만, 실제 Unity Editor Play Mode에서 전체 흐름을 눈으로 확인하는 QA는 아직 필요합니다.
- 씬 자동 조립 도구가 만든 오브젝트 배치와 기존 수동 배치가 겹치지 않는지는 커밋 전 씬 diff로 다시 확인해야 합니다.
- `FarmGrid` 내부의 예전 `RefreshTileVisual()` 주석 로직은 남아 있지만, 실제 시각 갱신은 현재 `FarmGridController`가 담당합니다.
- 에디터 메뉴 실행 후 씬 저장이 일어나므로, 커밋 전 씬 diff를 꼭 확인해야 합니다.
- 현재 Git 상태에 `_Recovery` 파일 삭제가 보입니다. 의도한 정리인지 커밋 전에 확인해야 합니다.
- 네트워크 관련 클래스는 초안 수준이며 아직 실제 플레이 루프와 연결하지 않았습니다.

## 다음 개발 예정

### 1. 실제 Unity Editor 화면 QA

- 자동화 가능한 씬 로드/런타임 잔재/기본 카메라 가시성 스모크 QA는 `SceneSmokePlayModeTests`로 추가했습니다.
- Unity Editor에서 Farm/Main 씬을 직접 열고 Play Mode로 진입합니다.
- 밭 갈기, 심기, 물 주기, 성장, 수확, 저장, 불러오기를 눈으로 확인합니다.
- 자동 생성 오브젝트 위치, 정렬 순서, Tilemap 레이어, UI 표시 겹침을 확인합니다.
- 저장/불러오기 후 농장과 인벤토리 상태가 유지되는지 확인합니다.

완료 기준:

- 메뉴 한 번 실행 후 Play Mode에서 농장 루프가 바로 동작합니다.
- 저장/불러오기 후 농장과 인벤토리 상태가 유지됩니다.
- 화면에서 타겟 마커, HUD, 인벤토리/상점/설정 UI가 겹침 없이 보입니다.

### 2. 상점/경제 콘텐츠 후속

- 상점 상품 잠금/해금 규칙 결정
- 실제 아이콘 Sprite 누락 아이템 정리
- 작물 종류가 늘어났을 때 씨앗 구매가, 수확물 판매가, 시작 골드 재조정
- 실제 플레이 시간 기준 수익률 조정

완료 기준:

- 여러 작물의 씨앗 구매, 작물 재배, 수확물 판매, 재투자 흐름이 자연스럽게 이어집니다.

### 3. UI와 현지화 정리

- HUD/상점/설정/인벤토리의 핵심 문구는 한국어 기준으로 1차 통일했습니다.
- 남은 영어 아이템 이름은 데이터 에셋의 표시 이름 정책을 정한 뒤 정리합니다.
- UI Toolkit 스타일을 실제 게임 화면 기준으로 다듬습니다.
- 모바일/다양한 해상도에서 텍스트가 겹치지 않는지 확인합니다.

완료 기준:

- 플레이 중 UI 문구가 일관되고, 화면에서 겹침 없이 읽힙니다.

### 4. 네트워크는 뒤로 미루기

네트워크 구조는 이미 일부 클래스가 있지만, 지금은 싱글 플레이 농장 루프 안정화가 우선입니다.

권장 순서:

1. 싱글 플레이 농장 루프 안정화
2. 저장/상점/인벤토리 안정화
3. 네트워크 권위 구조 결정
4. CommandDispatcher 구현
5. ReplicationSystem 구현
6. Host/Client 동기화 테스트

## 바로 다음 작업 추천

다음 작업 하나만 고르면, `실제 Unity Editor 화면 QA`가 좋습니다.
한국어 문구가 들어가면서 일부 버튼과 라벨 길이가 달라졌으니, Farm/Main 씬 Play Mode에서 HUD/상점/설정/인벤토리 겹침과 실제 조작감을 눈으로 확인하는 단계가 좋습니다.
