# nag_cat — Product Design v1.0

> Windows 화면 한구석에서 생활하며 사용자의 건강 루틴을 기억하고, 작업을 방해하지 않으면서 적절한 시점에 잔소리해주는 작은 데스크톱 펫 에이전트.

- 상태: **v1.0 / Phase 1 개발 착수 가능**
- 공식 Repository: `gmldbs25/health-cat-exe`
- 공식 앱/캐릭터 명칭: **`nag_cat` (내그캣)**
- 본 문서를 제품/UX/기술 설계의 기준선으로 사용한다.
- v1.0 이후 변경은 실제 사용 테스트 또는 구현 제약에 근거해 증분 반영한다.

---

## 1. 제품 목표

`nag_cat`을 실행하면 Windows 화면 한구석에 작은 고양이가 나타난다. 평소에는 주로 잠들어 있고, 건강 알림이 필요할 때 자연스럽게 깨어나 짧은 말풍선으로 알려준다.

단순 반복 알림 앱이 아니라, 향후 사내 LLM API를 연결해 사용자가 자연어로 건강 루틴을 등록/수정하고 고양이의 말투를 선택할 수 있는 **로컬 중심 건강 루틴 에이전트**로 발전시킨다.

핵심 가치는 다음 네 가지다.

1. 귀엽고 자연스러운 데스크톱 펫 경험
2. 현재 작업을 절대 방해하지 않는 건강 알림
3. 사용자별 건강 루틴의 자연어 관리
4. 같은 알림도 반복적으로 느껴지지 않는 캐릭터성

---

## 2. 절대 UX 원칙

### 2.1 Focus Non-Stealing

> **사용자가 먼저 상호작용하지 않는 한, nag_cat은 어떤 상황에서도 현재 활성 Window의 키보드 Focus를 가져오지 않는다.**

이 규칙은 옵션이 아니라 불변 조건이다.

자동 알림에서 금지한다.

- 현재 활성 Window 변경
- 키보드 커서 이동
- TextBox 자동 Focus
- 사용자가 입력 중인 키 이벤트 가로채기
- 알림 때문에 다른 프로그램이 비활성화되는 동작

Windows 구현에서는 No-Activate 동작을 보장하는 Window 설정 및 필요한 Win32 확장 스타일을 사용한다.

### 2.2 사용자 명시 상호작용만 Focus 허용

UI는 개념적으로 두 종류로 구분한다.

**Notification Bubble**
- 자동 표시
- Focusable = false
- No-Activate
- 일정 시간 후 자동 종료

**Conversation Bubble**
- 사용자가 고양이를 직접 클릭하거나 대화를 요청했을 때만 표시
- Text Input 포함 가능
- 이 경우에만 Focus 획득 허용

---

## 3. 고양이 동작

### 3.1 기본 상태

- 사용자가 마우스로 Drag하여 위치 이동 가능
- 화면을 넓게 돌아다니지 않음
- 사용자가 둔 위치 근처에서 생활
- 평소 대부분 Sleeping 상태
- 작은 Idle Animation만 사용
  - 귀 움직임
  - 꼬리 꿈틀
  - 호흡
  - 가끔 작은 자세 변화

### 3.2 알림 상태 전환

기본 흐름:

`Sleeping → Waking → Stretching → Talking → Response/Timeout → GoingToSleep`

알림 팝업이 갑자기 튀어나오는 느낌보다 **작은 생명체가 깨어나는 느낌**을 우선한다.

추가 상태 후보:

- Satisfied / Proud
- Annoyed / Nagging
- Dragged
- Grooming
- Yawn

Phase 1에서는 필요한 상태만 구현하고 점진적으로 확장한다.

---

## 4. 기본 표시와 위치

- Cat Window 기본 크기: **128 × 128 DIP**
- 사용자 설정 크기: **96 / 128 / 160 DIP**
- 첫 실행 위치: **Primary Monitor Work Area 우측 하단**
- Taskbar와 겹치지 않도록 우측/하단 **24 DIP Margin**
- Drag 종료 후 모니터 식별자와 좌표 저장
- 다음 실행 시 마지막 위치 복원
- 저장된 모니터가 사라졌거나 해상도/DPI 변경으로 Window의 50% 이상이 화면 밖이면 Primary Monitor 우측 하단으로 복구
- 말풍선은 화면 경계를 감지해 가능한 방향으로 자동 배치
- 말풍선 때문에 고양이의 저장 위치 자체를 변경하지 않음

---

## 5. 말풍선 / 알림 UX

### 5.1 Notification Bubble

- 고양이 이미지에 포함하지 않고 Avalonia UI로 렌더링
- 따뜻한 Off-White 배경
- 14~16 DIP 정도의 둥근 모서리
- 얇고 부드러운 Shadow
- 최대 폭 **300 DIP**
- 기본 본문 **2~3줄 이내**
- 짧은 Fade/Scale In은 허용
- Bounce 등 시선을 과하게 끄는 효과는 사용하지 않음
- 기본 표시 시간: **12초**
- Hover 중에는 자동 종료 Countdown 일시 정지
- Hover 종료 후 남은 시간부터 재개
- 자동 표시 중 항상 No-Activate / Non-Focus 유지

### 5.2 기본 반응 버튼

행동이 필요한 Reminder에는 다음 두 버튼을 제공한다.

- `했어`
- `10분 뒤`

**했어**
- Active-Use Interval Routine: 완료 기록 후 누적 Active-Use를 0으로 Reset
- Fixed-Time Routine: 해당 날짜의 Routine 완료 처리
- 짧은 Satisfied 반응 후 Sleep 복귀

**10분 뒤**
- 현재 Reminder에만 Wall Clock 기준 10분 One-Shot Snooze 생성
- 원래 Routine의 간격/고정 시각은 변경하지 않음
- 재알림 시 Interaction Gate가 닫혀 있으면 표시를 지연

### 5.3 무응답

1. 첫 Bubble이 12초 후 종료되면 `IgnoredOnce` 기록
2. 이후 **Active-Use 15분** 경과 후 같은 Reminder를 최대 1회 재알림
3. 두 번째도 무응답이면 추가 재촉 중단
4. Active-Use Interval은 두 번째 무응답 시점부터 원래 Interval을 다시 계산
5. Fixed-Time Routine은 두 번째 무응답 후 당일 추가 알림 없음

집요한 알림보다 조용한 기본 동작을 우선한다.

---

## 6. Windows Session / Display Awareness

Windows 상태를 이벤트 기반으로 추적한다.

- Session Locked / Unlocked
- Display On / Off / Dimmed
- Suspend / Resume

구현 방향:

- 세션 잠금/해제: `WTSRegisterSessionNotification` / `WM_WTSSESSION_CHANGE`
- 화면 상태: Power Setting Notification / `GUID_SESSION_DISPLAY_STATUS`
- Avalonia 공통 UI와 Windows Platform Service를 분리

### 6.1 Interaction Gate

자동 고양이 UI는 다음 조건을 모두 만족할 때만 허용한다.

`SessionUnlocked && DisplayOn && !HiddenMode && !QuietMode`

Gate가 닫혀 있으면:

- Wake Animation 실행하지 않음
- Notification Bubble 표시하지 않음
- Focus에 영향 주지 않음
- Scheduler / Routine 계산은 유지
- 필요한 알림은 Pending으로 관리

### 6.2 Unlock / Display Resume

- Unlock 후 Grace Period: **20초**
- 밀린 알림을 한꺼번에 표시하지 않음
- Pending이 여러 개면 Fixed-Time을 우선
- 그 외에는 가장 오래 지연된 Active-Use Reminder 우선
- 한 번에 최대 1개만 표시
- 나머지는 최소 **Active-Use 10분** 간격 후 재평가

### 6.3 Fixed-Time Pending TTL

- 기본 TTL: **60분**
- 예: 12:30 Reminder는 13:30까지 Pending 유효
- TTL이 지나면 조용히 폐기
- 향후 Routine별 TTL로 확장 가능

---

## 7. Hidden / Tray / Quiet Mode

### 7.1 Hidden Mode

`고양이 숨기기`를 선택하면:

- Cat Window: OFF
- Scheduler: ON
- Routine Manager: ON
- System Tray: ON

알림 시간이 와도 고양이를 강제로 다시 띄우지 않고 **Windows 시스템 알림**을 사용한다. 시스템 알림 자체도 Focus를 변경하지 않는다.

사용자가 Windows Notification을 직접 클릭한 경우에는 명시적 상호작용으로 보고 고양이를 마지막 위치에 다시 표시하고 관련 Action/Conversation Bubble을 열 수 있다.

### 7.2 오늘 조용히 있기

- Cat Notification과 Windows Notification 모두 중지
- Scheduler 상태와 Active-Use 계산은 유지
- 다음 로컬 날짜 **00:00**에 자동 해제
- 00:00에 즉시 알림을 띄우지 않고 다음 정상 Reminder부터 복귀

### 7.3 우클릭 메뉴 v1.0

1. `고양이 숨기기 / 다시 보이기`
2. `오늘 조용히 있기 / 조용히 해제`
3. `설정`
4. `Windows 시작 시 자동 실행` Toggle
5. 구분선
6. `종료`

Phase 1에서는 메뉴 항목을 더 늘리지 않는다.

---

## 8. Windows 시작 시 자동 실행

- 기본값: **OFF**
- 사용자가 우클릭 메뉴 또는 설정에서 직접 ON/OFF
- ON이면 Windows 부팅 시점이 아니라 **사용자 로그인 후** 자동 실행
- Standalone `.exe`에서도 지원
- 사용자별 자동 실행 영역(HKCU Run 등)을 우선 검토해 관리자 권한 없이 동작
- 자동 실행으로 시작되어도 Focus Non-Stealing 규칙 동일 적용
- 현재 실행 중인 exe의 절대 경로를 등록
- 사용자가 exe를 이동/삭제하면 경로가 깨질 수 있으므로 설정에서 유효성 재검증
- 초기 MVP에서 자기 자신을 별도 Install Path로 자동 복사하지 않음

---

## 9. Scheduler

Routine은 처음부터 두 종류로 분리한다.

### 9.1 Active-Use Interval Routine

PC를 실제 사용하는 시간만 누적한다.

누적 조건:

`SessionUnlocked && DisplayOn`

Lock / Display Off / Suspend 동안에는 누적 중단 후 복귀 지점에서 이어간다.

기본 Routine:

- 자리에서 일어나기: **Active-Use 60분**
- 물 마시기: **Active-Use 120분**

예: 40분 사용 → 30분 잠금 → 복귀 후 20분 사용 = 60분 Reminder 발생.

앱이 실행되지 않은 시간도 Active-Use로 계산하지 않는다. 앱 종료 전 누적 값은 LocalAppData에 저장하여 재실행 후 이어서 계산한다.

### 9.2 Fixed-Time Scheduled Routine

실제 Wall Clock 기준이다.

기본 Routine:

- 목/어깨 스트레칭: **매일 12:30**

예약 시각에 Interaction Gate가 닫혀 있으면 Pending 처리하고, TTL 내에서 사용자가 복귀하면 표시할 수 있다.

### 9.3 향후 자연어 수정

**Active-Use Interval과 Fixed-Time 모두 LLM 대화를 통해 수정 가능해야 한다.**

예:

- “물은 1시간 반마다 알려줘.”
- “일어나기는 50분마다 해줘.”
- “스트레칭은 1시에 알려줘.”
- “점심 비타민을 12시 40분에 챙겨줘.”

LLM은 시간을 직접 관리하지 않는다.

`사용자 자연어 → LLM 해석 → 구조화된 Routine 변경 → Local Scheduler 저장/실행`

---

## 10. Custom Routine / Skill

초기 Skill은 코드 실행형이 아니라 **선언형 Routine 데이터**로 구성한다.

초기 지원 범위:

- 특정 시간 매일
- N분 / N시간 간격 반복
- Enabled / Disabled

예시:

```yaml
id: morning_probiotics
name: 유산균 먹기
type: routine
schedule:
  type: daily
  time: "08:30"
enabled: true
message:
  style: nagging
```

평일만, 특정 요일, 복합 조건, 위치/실행 앱 기반 조건 등은 실제 사용 후 필요할 때 확장한다.

---

## 11. Personality

초기 4종:

1. **다정한 잔소리** — 기본값
2. **시크한 잔소리**
3. **츤데레**
4. **엄격한 잔소리**

공통 금지:

- 모욕
- 위협/공포 조장
- 수치심을 유발하는 건강 비난
- 의료 진단 또는 치료 판단

사용자는 설정에서 언제든 변경할 수 있고 Phase 3 이후 자연어 변경도 지원한다.

---

## 12. LLM Agent

### 12.1 역할

- 자연스러운 잔소리 문장 생성
- 사용자 자연어 Routine 요청 이해
- Routine 생성/수정/비활성화 요청 구조화
- Personality 적용
- 최근 알림을 참고해 표현 반복 최소화

### 12.2 담당하지 않는 것

- 시간 관리
- Scheduler 실행
- 의료 진단
- 약물 용량 변경 판단
- 증상 기반 치료 지시

LLM 장애가 Scheduler를 멈추게 해서는 안 된다.

### 12.3 Context

LLM 요청에는 필요한 최소 Context만 사용한다.

1. nag_cat Core System Prompt
2. 선택 Personality
3. 현재 Reminder/Routine 정보
4. 최근 관련 Notification Outcome 최대 **10건**
5. 현재 사용자 입력

현재 앱 세션에서 최근 **10 Turn** 정도만 Conversation Context로 유지한다.

전체 자연어 대화 원문은 장기 저장하지 않는다. 장기 기억이 필요한 내용은 구조화된 Routine/Preference로 변환해 저장한다.

### 12.4 Fallback

LLM API 실패/Timeout/인증 오류 시 Scheduler는 정상 동작한다.

기본 Routine별 Local Fallback 문구를 최소 8개 이상 준비하고 직전 문구와 중복되지 않도록 선택한다.

---

## 13. 사내 LLM API 인증

사내 LLM API는 승인된 사용자만 사용할 수 있고 각 사용자가 자신의 API Key/Token을 보유하는 구조를 전제로 한다.

### 13.1 배포 원칙

- GitHub Release의 `nag_cat.exe`에 API Key를 내장하지 않음
- 모든 사용자에게 동일한 바이너리 배포
- 각 사용자가 최초 1회 자신의 승인된 Key/Token 등록
- Source, Git, Release, 로그에 Secret을 포함하지 않음

### 13.2 최초 연결 UX

`설정 > LLM 연결`

- 사용 ON/OFF
- Endpoint
- Model
- API Key / Token
- 연결 테스트
- 저장

일반 사용자가 LocalAppData 파일을 직접 편집하게 하지 않는다.

공통으로 사용할 수 있는 비밀이 아닌 정보는 앱 기본값에 포함 가능하다.

- endpoint
- model
- request schema
- fixed headers
- timeout

사용자는 원칙적으로 개인별 Secret만 입력한다.

### 13.3 Secret 저장

API Key/Token은 평문 JSON에 저장하지 않는다.

우선 검토:

1. Windows Credential Manager
2. Windows DPAPI 기반 사용자 계정 종속 암호화

API Key 만료/폐기/권한 오류가 발생해도 Modal Popup으로 작업을 막지 않는다. 설정 또는 Conversation UI에서 연결 상태만 안내한다.

---

## 14. 로컬 데이터

실행파일과 사용자 데이터를 분리한다.

예상 루트:

```text
%LocalAppData%\HealthNaggingCat\
```

예시:

```text
HealthNaggingCat/
├─ config/
│  ├─ settings.json
│  └─ llm.json          # Secret 제외
├─ routines/
│  ├─ stand-up.json
│  ├─ water.json
│  ├─ stretch.json
│  └─ custom-*.json
├─ history/
│  └─ events.jsonl
└─ logs/
   └─ app.log
```

Installer 없이 최초 실행해도 앱이 필요한 디렉터리와 초기 설정을 생성한다.

`.exe`를 교체해도 사용자 설정/Routine은 유지한다.

- `.exe` = 고양이의 몸
- LocalAppData = 고양이의 기억

### 14.1 Logging

기본 로그 허용:

- 앱 시작/종료
- Routine ID와 Trigger/Complete/Snooze/Ignore 상태
- 오류 종류 / Stack Trace
- Windows Session/Display 기술 이벤트

기본 로그 금지:

- API Key/Token
- 전체 LLM Prompt/Response
- 전체 Conversation 원문
- 민감할 수 있는 Custom Routine 내용 전체

---

## 15. Visual Design

### 15.1 방향

레트로 저해상도 Pixel Art는 최종 방향에서 제외한다.

**Soft 2D / Chibi / Rounded Character**를 사용한다.

특징:

- 둥글고 부드러운 실루엣
- 조금 큰 머리와 짧고 둥근 몸
- 작은 크기에서도 읽히는 표정
- 너무 유아틱하지 않은 현대적인 Desktop Pet 느낌
- 기본 인터랙션은 3/4 시점
- Sleep은 옆으로 둥글게 웅크린 자세

### 15.2 Character Master Reference v1

공식 기준 파일:

`assets/cat/master/nag_cat_character_sheet_v1.png`

이 이미지를 **nag_cat Character Master Reference v1**로 고정한다.

후속 상태 이미지는 다음 요소를 유지해야 한다.

- Cream / Cheese 계열 밝은 털
- 둥글고 짧은 체형 / 상대적으로 큰 머리
- 얼굴/눈/귀 비율
- 꼬리 형태
- 따뜻하고 부드러운 외곽선/음영
- 갈색 목걸이와 원형 펜던트

Master v1은 개별 런타임 Sprite가 아니라 모든 후속 Asset의 일관성을 판단하는 기준이다. 수정이 필요하면 원본을 덮어쓰지 않고 v2로 만든다.

---

## 16. Runtime Asset / Animation

- 런타임 Asset: **투명 PNG Frame Sequence**
- Master Canvas: 상태별 **512 × 512 px**
- 모든 Frame 동일 Canvas Size / Anchor Point
- 앱에서는 기본 128 DIP로 축소 렌더링
- GIF/APNG/Animated WebP에 동작을 의존하지 않고 Animation State Machine이 Timing 제어
- 말풍선/버튼/입력창/Context Menu는 Raster 이미지로 만들지 않고 Avalonia UI로 구현

Phase 1 목표:

| State | 목표 |
|---|---|
| Sleeping | 4~6 frames, 3~5 fps |
| Waking | 6~8 frames, 8~10 fps |
| Stretching | 8~12 frames, 8~10 fps |
| Talking/Nagging | 4~6 frames, 4~6 fps |
| Satisfied | 4~6 frames |
| GoingToSleep | 6~8 frames |

고프레임 애니메이션보다 **느리고 귀여운 작은 움직임**을 우선한다.

---

## 17. 기술 스택

### Desktop

**C# / .NET / Avalonia UI**

목표 기능:

- Transparent / Borderless Window
- Always-on-top
- No-Activate
- Drag
- Tray
- Windows Notification
- DPI Scaling
- Multi Monitor
- Local Scheduler

Windows 고유 기능은 Platform Service로 분리하고 Avalonia 공통 UI와 직접 섞지 않는다.

### 개발 환경

주 개발:

- MacBook
- VS Code / Codex
- .NET SDK
- Git

Windows 실환경 검증:

- 개인 Windows PC 또는 회사 Windows PC

필수 검증:

- Focus 유지
- No-Activate
- Tray / Notification
- DPI Scaling
- Multi Monitor
- Session Lock/Unlock
- Display On/Off
- Windows Startup
- Fullscreen 앱과의 상호작용

---

## 18. 배포

초기 공식 배포 대상:

- **Windows x64 (`win-x64`)**
- Windows 11 우선 검증
- Windows 10은 가능한 범위에서 확인
- ARM64는 초기 범위 제외

초기 배포 명칭은 `Portable`이 아니라 **Standalone**으로 한다.

목표 Release Asset:

```text
nag_cat.exe
```

- Self-contained
- 가능하면 Single-file
- Installer 없이 실행
- .NET Runtime 포함
- 사용자 데이터는 LocalAppData에 분리

### 18.1 GitHub Actions

공식 Windows 빌드 머신은 GitHub Actions Windows Runner로 한다.

Tag `v*` Push 시 목표 흐름:

1. Checkout
2. Restore
3. Test
4. `win-x64` Self-contained Release Publish
5. Single-file `nag_cat.exe` 생성
6. GitHub Release 생성/업데이트
7. exe를 Release Asset으로 첨부

MacBook에 Windows용 로컬 컴파일 환경을 별도로 요구하지 않는다.

Installer는 Stable Install Path, 자동 업데이트, 기업 배포 정책, Code Signing, Uninstall 관리 필요가 실제로 생길 때 도입한다.

---

## 19. 최초 실행

Modal Wizard로 사용자의 작업을 막지 않는다.

1. Primary Work Area 우측 하단에 nag_cat 표시
2. 기본 Routine 3개 즉시 활성화
3. 짧은 Welcome Bubble로 기본 동작과 우클릭 메뉴 안내
4. Windows 자동 실행은 OFF
5. LLM Key 입력은 Scheduler 사용을 막는 필수 절차가 아님
6. LLM 기능을 사용하려는 사용자가 설정에서 최초 1회 인증정보 등록

---

## 20. Phase Plan

### Phase 1 — Cat Scheduler MVP

목표: **절대로 작업을 방해하지 않는 작은 데스크톱 펫 Scheduler**

- Transparent / Borderless Cat Window
- Always-on-top / No-Activate
- Drag / Position 저장
- Sleep / Wake / Talk 최소 상태
- Notification Bubble
- 기본 Scheduler 3종
- `했어` / `10분 뒤`
- Session / Display Awareness
- Tray / Hidden Mode / Quiet Mode
- Windows Notification
- LocalAppData 설정
- Windows Startup Toggle
- LLM 제외

### Phase 2 — Local Routine Manager

- Routine 데이터 구조
- Custom Routine CRUD
- Settings UI
- Personality 설정 데이터

### Phase 3 — LLM Conversation

- 사내 LLM API 연결
- 자연스러운 대화
- 말투 Variation
- 최근 Reminder Context

### Phase 4 — Routine Skill Agent

- 자연어 → 구조화된 Routine Tool Call
- Routine 생성/수정/비활성화
- 필요한 경우 시간 추가 질문
- 사용자 건강 Routine/Preference Memory

### Phase 5 — Polish / Distribution

- Animation 고도화
- Fullscreen/DND 대응
- Multi Monitor 안정화
- 자동 업데이트 검토
- Installer / Code Signing 검토

---

## 21. v1.0 기본값

| 항목 | 기본값 |
|---|---|
| Cat Size | 128 DIP |
| 첫 위치 | Primary Work Area 우측 하단, 24 DIP Margin |
| 일어나기 | Active-Use 60분 |
| 물 | Active-Use 120분 |
| 스트레칭 | 매일 12:30 |
| Bubble 표시 | 12초 |
| Snooze | 10분 |
| 무응답 Retry | Active-Use 15분 후 1회 |
| Unlock Grace | 20초 |
| Fixed-Time Pending TTL | 60분 |
| 기본 Personality | 다정한 잔소리 |
| Windows 자동 실행 | OFF |
| Quiet Hours | OFF |
| 오늘 조용히 | 다음 날짜 00:00 자동 해제 |
| Conversation Context | 최근 10 Turn |
| Reminder Context | 최대 10건 |
| 배포 대상 | Windows x64 |
| 배포 형태 | Standalone `nag_cat.exe` |

---

## 22. 변경하면 안 되는 핵심 원칙

구현상의 기술 문제가 생겨도 다음 원칙은 유지한다.

1. 자동 인터랙션은 사용자의 Focus를 훔치지 않는다.
2. Lock / Display Off 상태에서 고양이 UI를 띄우지 않는다.
3. 밀린 알림을 몰아서 보여주지 않는다.
4. LLM 장애가 Scheduler를 중단시키지 않는다.
5. Secret을 Git / Release Binary / 로그에 포함하지 않는다.
6. Character Master Reference v1의 시각적 일관성을 모든 후속 Cat Asset에 유지한다.
