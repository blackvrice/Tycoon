# Tycoon Portfolio Gameplay Capture

목표: 60~90초 안에 게임의 장르, 핵심 루프, 상태 피드백, 경제 재투자와 Save/Load를 채용 담당자가 이해하게 합니다. 실제 플레이는 3~5분으로 진행하고 성장 대기 구간만 타임랩스 또는 점프 컷으로 압축합니다.

## Capture Build

- Unity: `6000.3.10f1`
- 권장 실행: Windows build `Builds/Windows/Tycoon.exe`
- 권장 녹화 해상도: 1920×1080, 60fps
- 권장 최종 길이: 75~85초
- 오디오는 게임 BGM/SFX가 음성을 덮지 않도록 Master 70%, Music 45%, SFX 70%부터 조정

## Preflight

1. Unity 메뉴 `Tycoon/Build/Build Windows Portfolio`로 최신 build를 생성합니다.
2. 기존 저장으로 시작하지 않도록 Title에서 `새 게임`을 선택합니다.
3. 시작 상태가 100G, 괭이, 물뿌리개, 밀 씨앗 12, 토마토 씨앗 8인지 확인합니다.
4. HUD, 가이드, Player, Farm 타일, 상인, 카메라가 화면 안에 있는지 확인합니다.
5. I, B, Esc, E, 1~4, 이동/도구 사용 입력을 한 번씩 확인합니다.
6. 녹화 전 알림, 메신저, 마우스 커서 강조와 불필요한 Editor 창을 정리합니다.

## 75~85 Second Shot List

| 구간 | 화면 | 보여줄 내용 | 편집 포인트 |
|---:|---|---|---|
| 0–5초 | Title | 로고, `새 게임` 클릭 | 첫 1초 안에 장르가 보이게 시작 |
| 5–11초 | Main | Player 이동, 우측 Farm 길, 첫 플레이 가이드 | 씬 전환 fade 포함 |
| 11–25초 | Farm | 괭이 선택 → 경작 → 씨앗 선택 → 파종 → 물주기 | 대상 타일 marker와 HUD 메시지를 읽을 시간 확보 |
| 25–34초 | Farm | 작물 단계 변화 | 성장 대기만 4~8배 타임랩스 |
| 34–42초 | Farm | 성숙 작물 앞 E → 수확물 Inventory 반영 | 수확 전후 아이콘/수량이 한 화면에 보이게 |
| 42–55초 | Shop | 상인 E → 수확물 판매 → Gold 증가 | 상점 제목, 수확물, 판매 버튼, Gold를 순서대로 강조 |
| 55–64초 | Shop | Sunflower Seed 구매 → Gold 감소/씨앗 증가 | `판매 → 재투자`가 명확한 한 컷 |
| 64–74초 | Settings | Esc → 저장 → 상태 변경 → 불러오기 | 선택 슬롯/Gold/Inventory가 복원되는 장면 유지 |
| 74–82초 | Farm | 가이드 완료 또는 해바라기 파종, 전체 농장 | 엔드 카드에 기술 키워드 표시 |

## Suggested On-screen Captions

- `Data-driven Crop Progression`
- `Domain State: Till → Plant → Water → Grow`
- `Harvest → Sell → Reinvest`
- `Composite Save / Load`
- `73 PlayMode + 4 EditMode Tests`

각 캡션은 2~3초만 표시하고 코드 용어보다 플레이 결과를 먼저 보여줍니다.

## Save / Load Proof Shot

1. 수확물을 판매하고 Sunflower Seed를 구매합니다.
2. Hotbar에서 Sunflower Seed 슬롯을 선택합니다.
3. Settings에서 `저장`을 누르고 완료 메시지를 촬영합니다.
4. 씨앗을 하나 더 구매하거나 다른 슬롯을 선택해 화면 상태를 바꿉니다.
5. `불러오기`를 누릅니다.
6. Gold, Inventory 수량, 선택 Hotbar 슬롯이 저장 시점으로 돌아온 화면을 2초 유지합니다.

## Final Visual QA

- [ ] HUD 상태 카드와 가이드가 서로 겹치지 않음
- [ ] Shop/Settings가 열릴 때 HUD와 Hotbar가 패널 제목/닫기 버튼을 가리지 않음
- [ ] 1920×1080에서 상점 목록과 상세 패널이 잘리지 않음
- [ ] 1280×720에서도 Settings 하단 버튼이 화면 안에 있음
- [ ] Player와 현재 대상 타일이 카메라 안에 있음
- [ ] 도구 사용 애니메이션 방향과 타일 marker가 실제 대상과 일치함
- [ ] Farm → Main 귀환 시 출구 재진입 루프가 없음
- [ ] 판매/구매 직후 Gold와 Inventory가 즉시 갱신됨
- [ ] Save/Load 전후 Scene, 위치, 선택 슬롯, 작물 상태가 자연스럽게 복원됨
- [ ] 녹화에 Console, 테스트 창, 개인 경로, 알림이 노출되지 않음

## Upload Package

- `Tycoon_Gameplay_80s.mp4`: 1080p H.264
- `Tycoon_CoreLoop.gif`: README용 8~12초, 15~20fps, 10MB 이하 권장
- 썸네일: 성숙 작물, Player, HUD가 한 프레임에 보이는 Farm 화면
- README의 `Gameplay GIF`와 `Gameplay Video` placeholder를 실제 링크로 교체
