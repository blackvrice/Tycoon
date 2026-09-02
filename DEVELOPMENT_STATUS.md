# Tycoon Development Status

갱신일: 2026-09-01 · 기준 버전: Unity 6000.3.10f1 / `main`

## Current Status

**Portfolio Ready — 코드, 자동 테스트, Windows 빌드, README GIF, 64초 무음 시스템 프리뷰와 최종 Gameplay Video까지 준비 완료.**

핵심 3~5분 루프는 `Title → New Game → Main → Farm → Till → Plant → Water → Grow → Harvest → Sell → Buy Seed → Save/Load`로 연결되어 있습니다. 첫 플레이 가이드는 Farm/Inventory/Economy 상태를 관찰하며 7단계를 안내합니다.

## Completed

### Core Gameplay

- Farm Domain: Empty/Tiled/Planted/Grown, 물주기, 성장 Tick, 수확 가능 조건
- Inventory/Hotbar: 36 slot, stack/move, 9-slot selection, 시작 아이템 지급
- Shop/Economy: 지역별 stock, 구매/판매 수량 제한, Gold, 양의 재투자 구조
- Data: Wheat/Tomato/Sunflower의 Fast/Balanced/Premium 성장·이익 진행
- Scene: Title, Main, Farm, Pasture, Livestock와 경계 기반 왕복 위치

### Save / Load / New Game

- Save v2: Scene, 위치, 선택 슬롯, Tick/Day, Gold, Inventory, Farm, Animal, Structure
- 작물 단계 내부 `GrowthTick`까지 저장·복원
- New Game: 저장 삭제, 세션 초기화, 시작 인벤토리 재지급
- 씬 전환 전 자동 저장과 Title Load의 저장 Scene 복귀

### UI / UX

- 공통 UI Toolkit HUD, Hotbar, Inventory, Shop, Settings
- 날짜/시간, Gold, 선택 도구, 대상 타일/작물, 행동 결과 메시지
- 7단계 상태 기반 첫 플레이 가이드와 Skip
- 모달 UI 동안 HUD/Hotbar를 숨겨 제목·닫기 버튼 겹침 제거
- 1920×1080, 1600×900, 1280×720 렌더 캡처 확인

### Tooling / Verification

- Runtime scene installer와 반복 설치 검증
- Farm scene setup 도구의 Unity 6 Tile 경로와 3종 작물 wiring
- Windows Portfolio Build 메뉴/CLI
- C# compile 3종 warning 0 / error 0
- Unity EditMode 4/4, PlayMode 74/74
- 실제 FarmScene 코어 루프 10단계 캡처와 11.15초 README GIF
- 같은 캡처를 설명 흐름으로 편집한 64초 H.264 무음 시스템 프리뷰
- Windows Standalone build 성공: `Builds/Windows/Tycoon.exe`, 약 103.4MB
- Standalone headless boot에서 Unity 6000.3.10f1과 Title 입력 시스템 초기화 확인

## Known Issues

- Headless Standalone 로그에서 Unity 외부 서비스 인증서 경고가 발생할 수 있습니다. 게임 코드 예외나 오프라인 루프 실패는 확인되지 않았습니다.

## Deferred / Out of Scope

- `Assets/Scripts/Network`의 Multiplayer 초안 연결과 권위 모델
- 계절, 퀘스트, 대규모 콘텐츠 확장
- 동물/시설을 핵심 경제 루프와 같은 깊이로 확장하는 작업
- 세이브 마이그레이션 UI와 복수 저장 슬롯

## Submission Checklist

- [x] Unity 6000.3.10f1에서 C# compile
- [x] EditMode 전체 통과
- [x] PlayMode 전체 통과
- [x] 5개 Build Settings Scene smoke
- [x] 3개 대표 해상도 UI 렌더 QA
- [x] Windows Standalone build
- [x] README / Status
- [x] Gameplay GIF 추가
- [x] 64초 무음 시스템 프리뷰 추가
- [x] Gameplay Video 촬영·업로드 및 README 링크 반영
