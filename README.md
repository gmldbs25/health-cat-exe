# nag_cat 🐈

Windows 화면 한구석에서 생활하면서 사용자의 건강 루틴을 챙겨주는 작은 데스크톱 펫입니다.

공식 프로젝트/캐릭터 명칭은 **`nag_cat`(내그캣)**이고, 캐릭터의 닉네임은 **Nagi(나기)**입니다.

평소에는 조용히 잠들어 있다가, 일어나기·물 마시기·스트레칭 같은 건강 알림이 필요할 때 **현재 작업의 Focus를 빼앗지 않고** 깨어나 말풍선으로 알려주는 것을 목표로 합니다.

향후 사내 LLM API를 연결해 자연스러운 대화, 사용자별 건강 루틴 등록/수정, 고양이 말투 커스터마이징을 지원할 예정입니다.

## 현재 상태

**Product Design v1.0 완료 / Phase 1 개발 준비**

- Desktop: C# / .NET / Avalonia UI
- Target: Windows x64
- Distribution: GitHub Releases의 Standalone `nag_cat.exe`
- User data: `%LocalAppData%\HealthNaggingCat`
- Build: GitHub Actions Windows Runner 예정

## 핵심 원칙

- 자동 알림은 현재 Window의 Focus를 절대 가져가지 않음
- Lock / Display Off 상태에서는 고양이 UI를 띄우지 않음
- Active-Use 기반 알림과 Fixed-Time 알림을 분리
- LLM 장애와 기본 Scheduler 동작을 분리
- API Key/Token은 Git 또는 Release 바이너리에 포함하지 않음

## Documents

- [Product Design v1.0](docs/DESIGN.md)
- [Naming](docs/NAMING.md)
- Character Master Reference v1: `assets/cat/master/nag_cat_character_sheet_v1.png`

---

`nag_cat` = 내그캣. Nagi는 작고 조용하지만 꾸준히 건강을 챙기는 잔소리 고양이.
