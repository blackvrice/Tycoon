# Tycoon 개발 현황 정리

이 문서는 지금까지 구현된 내용과 다음 개발 우선순위를 빠르게 잡기 위한 작업 메모입니다.

## 프로젝트 상태

- Unity `6000.3.10f1` 기반 2D 프로젝트입니다.
- 주요 패키지는 Input System, UI Toolkit, URP, 2D Tilemap/RuleTile, Test Framework를 사용합니다.
- 현재 빌드 씬은 `Assets/Scenes/MainScene.unity`입니다.
- 추가 씬으로 `Assets/Scenes/MarketplaceScene.unity`가 있습니다.
- 런타임 프리팹으로 `GameSession`, `RuntimeInput`, `RuntimeUIRoot`가 준비되어 있습니다.

## 지금까지 구현된 것

### 1. 코어 게임 흐름

- `GameSession`
  - 씬 전환 후에도 유지되는 런타임 세션 싱글톤입니다.
  - 플레이어 인벤토리, 골드, 선택된 핫바 슬롯을 보관합니다.
- `GameManager`
  - 게임 상태를 `Title`, `Ready`, `Playing`, `Paused`, `Ended`, `GameOver`로 관리합니다.
  - 틱 시작/정지, 플레이어 조작 활성화, 저장/불러오기 진입점을 가지고 있습니다.
- `TickManager`
  - 일정 간격으로 Tick을 증가시키고 `ITickable` 대상에게 Tick을 전달합니다.
  - 일차 계산, 일 변경 이벤트, 일시정지/재개, 강제 Tick 기능이 있습니다.
- `EconomyManager`
  - 골드 추가, 사용, 구매 가능 여부 확인을 담당합니다.
- `SaveManager`
  - JSON 저장/불러오기/삭제 기능이 있습니다.
  - 저장 대상은 Tick, Day, Gold, Inventory, FieldTile입니다.

### 2. 인벤토리와 아이템

- `ItemData`
  - 아이템 ID, 이름, 타입, 최대 스택, 구매가, 판매가, 아이콘을 가진 ScriptableObject입니다.
- `Inventory`, `InventorySlot`, `ItemStack`
  - 아이템 추가, 제거, 슬롯 이동, 스왑, 병합, 저장 데이터 생성/복원을 지원합니다.
- `InventoryUIController`
  - UI Toolkit 기반 인벤토리 UI입니다.
  - 메인 인벤토리 27칸과 핫바 9칸을 표시하고 클릭 이동을 지원합니다.
- `HotbarControl`
  - 핫바 9칸 표시, 선택 슬롯 표시, 아이콘/수량 표시를 담당합니다.

### 3. 농장/작물 로직

- `CropData`
  - 작물 ID, 이름, 성장 단계, 단계별 Tick, 단계별 Sprite, 수확 아이템을 가진 ScriptableObject입니다.
- `CropInstance`
  - 개별 작물의 성장 Tick, 현재 단계, 수확 가능 상태를 관리합니다.
- `FieldTile`
  - 타일 상태를 `Empty`, `Tiled`, `Planted`, `Grown`으로 관리합니다.
  - 밭 갈기, 심기, 물주기, Tick 성장, 수확을 지원합니다.
- `FarmGrid`
  - 2차원 밭 타일 배열을 관리합니다.
  - 좌표 유효성 검사, 타일 조작, 전체 Tick 처리, 저장/복원 로직이 있습니다.
- `FarmTileView`
  - SpriteRenderer 기반 타일 표시용 뷰가 준비되어 있습니다.
- 테스트용 작물/아이템 데이터와 성장 단계 Sprite가 `Assets/TestData`에 있습니다.

### 4. 상점/경제 UI

- `ShopItemData`, `ShopItemEntry`, `ShopSystem`
  - 구매/판매 가능 여부, 가격 계산, 골드 차감/복구, 인벤토리 추가/제거 로직이 있습니다.
- `InventoryShopUIController`
  - 인벤토리와 상점 패널을 열고 닫는 UI가 있습니다.
  - 상점 아이템 목록, 구매 버튼, 골드 표시, 인벤토리 드래그 이동 기능이 구현되어 있습니다.

### 5. 플레이어와 입력

- `InputManager`
  - 이동, 상호작용, 아이템 사용, 핫바 선택/스크롤, 인벤토리/상점 토글, UI 닫기 이벤트를 제공합니다.
  - InputActionAsset이 없을 때 기본 액션맵을 생성합니다.
- `PlayerController`
  - Rigidbody2D 이동, Animator 파라미터 갱신, 핫바 슬롯 선택, 상호작용/아이템 사용 호출을 담당합니다.
- `InteractionController`
  - 주변 Collider에서 `IInteractable` 대상을 찾아 상호작용하거나 선택 아이템을 사용합니다.

### 6. UI와 씬 구성

- `UIController`
  - 인벤토리, 상점, 설정 UI, 핫바 선택을 InputManager 이벤트와 연결합니다.
- `SettingUIController`
  - 설정 창 열기/닫기, 볼륨 라벨, 전체화면, VSync 토글을 처리합니다.
- `HUDUI`
  - 일자, Tick, 시간, 골드, 선택 아이템/도구, 타일/작물 정보, 메시지 표시용 API가 있습니다.
- `RuntimePrefabSceneInstaller`
  - 런타임 프리팹을 Main/Marketplace 씬에 설치하는 에디터 메뉴가 준비되어 있습니다.

### 7. 네트워크 초안

- `NetworkGameManager`
  - Host/Client 시작, 접속/해제, Join/Ready/TickSync/Input 패킷 흐름의 기본 구조가 있습니다.
- `INetworkTransport`, `NetworkPacket`, payload 클래스들이 있습니다.
- `ReplicationSystem`, `CommandDispatcher`는 아직 빈 껍데기입니다.

### 8. 테스트

- `TickManagerPlayModeTests`
  - Tick 실행과 Pause 동작에 대한 PlayMode 테스트가 있습니다.

## 현재 미완성/주의할 점

- `FarmGrid`를 실제 씬에서 생성하고 `TickManager`, `GameManager`, Tilemap과 연결하는 MonoBehaviour가 아직 보이지 않습니다.
- `FarmGrid.RefreshTileVisual()`은 구조만 있고 실제 `TileBase`/작물 Sprite를 Tilemap에 반영하지 않습니다.
- `FieldTile`, `FarmGrid`는 농장 규칙을 가지고 있지만 플레이어가 Hoe/WateringCan/Seed로 타일을 조작하는 연결부가 아직 없습니다.
- `InventoryShopUIController`는 UXML의 루트 이름이 `InventoryShopRoot`인데 코드에서는 `"Root"`를 찾고 있어 런타임 NullReference 가능성이 큽니다.
- `InventoryShopUIController`의 `createDebugRuntimeData`가 기본값 `true`라 실제 게임 시작 시 테스트 아이템이 자동 주입될 수 있습니다.
- 상점 구매 UI는 있으나 `ShopSystem`을 직접 사용하지 않고 컨트롤러 내부에서 구매 처리를 따로 합니다.
- 판매 UI, 수량 선택, 구매/판매 후 저장 연동은 아직 부족합니다.
- 저장 시스템은 준비되어 있지만 플레이어가 직접 저장/불러오기/새 게임을 실행하는 UI 흐름은 아직 없습니다.
- 네트워크 구조는 초안 단계입니다. 싱글플레이 핵심 루프가 안정화되기 전에는 우선순위를 낮게 두는 편이 좋습니다.
- `FieldTileDebugTester`는 수확 후 상태를 `Empty`로 기대하지만 실제 `FieldTile.Harvest()`는 `Tiled`로 유지합니다. 테스트/디버그 기대값을 현재 규칙에 맞춰 정리해야 합니다.

## 지금 바로 개발하면 좋은 것

가장 먼저 할 일은 **농장 로직을 실제 플레이 가능한 씬 기능으로 연결하는 것**입니다.

현재 코어 로직, 인벤토리, 작물 성장, 저장 구조는 꽤 많이 만들어져 있지만, 플레이어 입장에서는 아직 "밭을 갈고, 씨앗을 심고, 물을 주고, 자라면 수확한다"는 핵심 루프가 완전히 이어지지 않았습니다. 이 루프가 붙으면 이후 상점, 저장, UI, 밸런싱 작업이 모두 검증 가능해집니다.

## 추천 개발 순서

### 1. FarmGrid 씬 연결

- `FarmGridController` 같은 MonoBehaviour를 추가합니다.
- 씬의 Tilemap, 가로/세로 크기, 빈 땅/갈린 땅/물 준 땅 TileBase를 Inspector에서 연결합니다.
- `FarmGrid`를 생성하고 `GameManager.SetFarmGrid()`로 넘깁니다.
- `TickManager`에 농장 Tick 처리를 등록합니다.
- `RefreshTileVisual()`이 실제 Tilemap을 갱신하도록 TileBase/Sprite 표시 방식을 결정합니다.

완료 기준:

- Play 모드에서 특정 좌표의 타일 상태 변경이 화면에 보입니다.
- Tick이 지나면 작물 단계가 바뀌고 화면에 반영됩니다.
- 저장 후 불러오면 밭 상태와 작물 상태가 복원됩니다.

### 2. 플레이어 농장 상호작용

- 플레이어 앞 좌표 또는 마우스가 가리키는 좌표를 FarmGrid 좌표로 변환합니다.
- 선택 아이템 타입에 따라 동작을 연결합니다.
  - Hoe: Empty 타일을 Tiled로 변경
  - Seed: Tiled 타일에 CropData 심기
  - WateringCan: Planted 타일 물주기
  - 빈 손/상호작용: Grown 타일 수확
- 씨앗 사용 시 인벤토리에서 1개를 제거합니다.
- 수확 성공 시 수확 아이템을 인벤토리에 추가합니다.

완료 기준:

- 핫바에서 도구/씨앗을 선택해 농장 루프를 직접 플레이할 수 있습니다.
- 실패 조건이 명확합니다. 예를 들어 이미 심어진 곳에는 씨앗을 다시 심지 않습니다.

### 3. 인벤토리/핫바 동기화 안정화

- PlayerController의 선택 슬롯과 HotbarControl의 선택 표시가 항상 같은 값을 보도록 정리합니다.
- 인벤토리 이동, 구매, 수확 후 Hotbar/Inventory UI가 즉시 새로고침되게 만듭니다.
- ItemType이 `Tycoon.Data.ItemType`과 `Tycoon.Hotbar.ItemType`으로 나뉘어 있으므로 장기적으로 하나의 기준으로 정리합니다.

완료 기준:

- 아이템 획득/소비/이동 후 모든 UI가 같은 인벤토리 상태를 표시합니다.
- 핫바 선택, 마우스 휠, 숫자키 선택이 엇갈리지 않습니다.

### 4. 상점 UI 정리

- `InventoryShopUIController`의 루트 쿼리 이름을 UXML과 맞춥니다.
- 구매 처리를 `ShopSystem`으로 위임합니다.
- 판매 버튼 또는 선택 슬롯 판매 흐름을 추가합니다.
- `createDebugRuntimeData`는 개발 전용으로 끄거나 에디터/디버그 조건으로 제한합니다.

완료 기준:

- 상점에서 씨앗을 사고, 농장에 심고, 수확물을 팔아 골드를 얻을 수 있습니다.
- 골드 부족, 인벤토리 공간 부족, 잠긴 상품 메시지가 정상 표시됩니다.

### 5. 저장/불러오기 사용자 흐름

- 설정 UI 또는 별도 메뉴에 Save, Load, New Game 버튼을 추가합니다.
- 새 게임 시 `GameSession.ResetSession()`, FarmGrid 초기화, 저장 파일 삭제 흐름을 정리합니다.
- 자동 저장 시점을 정합니다. 예: 앱 종료, 하루 종료, 수동 저장.

완료 기준:

- 게임을 종료했다 다시 켜도 골드, 인벤토리, 작물 상태가 유지됩니다.
- 새 게임을 누르면 이전 저장 데이터가 남아 있지 않습니다.

### 6. 테스트 확장

- Inventory 추가/제거/이동 테스트를 추가합니다.
- FieldTile 성장/수확 테스트를 추가합니다.
- SaveManager 저장/복원 테스트를 추가합니다.
- ShopSystem 구매/판매 테스트를 추가합니다.
- UI는 최소한 UXML 이름과 컨트롤러 쿼리 이름이 맞는지 확인하는 스모크 테스트를 둡니다.

완료 기준:

- 핵심 데이터 로직은 Unity PlayMode 없이도 빠르게 검증할 수 있습니다.
- PlayMode 테스트는 씬 연결과 Tick 동작 위주로 유지합니다.

### 7. 네트워크는 나중으로 미루기

- 현재는 싱글플레이 농장 루프가 먼저입니다.
- 네트워크는 코어 루프가 안정된 뒤 다음 순서로 진행하는 것이 좋습니다.
  - 실제 Transport 구현
  - CommandDispatcher 구현
  - ReplicationSystem 구현
  - 호스트 권위 방식으로 농장/인벤토리 상태 동기화

## 한 줄 결론

다음 개발 목표는 **FarmGridController를 만들고, 플레이어 입력으로 밭 갈기/씨앗 심기/물주기/수확까지 이어지는 최소 플레이 루프를 완성하는 것**입니다.

이게 끝나면 상점과 저장은 이미 만들어둔 기반 위에 자연스럽게 붙일 수 있습니다.
