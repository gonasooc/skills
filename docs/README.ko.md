# Skills

Claude Code를 비롯한 코딩 에이전트에서 쓸 수 있는 [Agent Skills](https://agentskills.io) 모음입니다. 각 스킬은 그럴듯한 요약을 믿는 대신 결과물 자체와 대조하고, 검증하지 못한 것은 검증하지 못했다고 밝힙니다.

## 설치

[skills.sh](https://skills.sh) 인스톨러로 원하는 스킬을 골라 설치합니다:

```bash
npx skills@latest add gonasooc/skills
```

또는 Claude Code 플러그인으로 컬렉션 전체를 설치할 수 있습니다:

```
/plugin marketplace add gonasooc/skills
/plugin install gonasooc-skills@gonasooc
```

### Codex, Gemini CLI, Antigravity

이 스킬들은 오픈 [Agent Skills](https://agentskills.io) 표준을 따르므로, 표준을 지원하는 에이전트라면 수정 없이 그대로 동작합니다.

- **Codex** — 위 skills.sh 인스톨러가 Codex 설치를 직접 지원합니다. 설치 대상 선택 화면에서 Codex를 고르세요.
- **Gemini CLI / Google Antigravity / 수동 설치** — 스킬 폴더를 표준 공용 경로에 복사(또는 심링크)하세요: `~/.agents/skills/`(사용자 전역) 또는 저장소 내부의 `.agents/skills/`. Codex·Gemini CLI·Antigravity 모두 프로젝트 스킬을 이 경로에서 읽습니다. (전역 스킬은 에이전트별 자체 경로에 둡니다 — 예: Gemini CLI는 `~/.gemini/skills/`. 정확한 경로는 각 에이전트 문서를 확인하세요.)

호출은 이름으로 합니다 — Claude Code에서는 `/interviewer`, Codex에서는 `$interviewer`. `/skills` 피커에서 골라도 됩니다. `interviewer`는 자연어로 요청해도 실행되지만, `session-checkpoint`와 `session-resume`은 [명시적 호출 전용](#기기-간-작업)이라 이름을 직접 입력해야만 실행됩니다. `agents/openai.yaml` 같은 에이전트 전용 파일은 이를 사용하지 않는 에이전트에서는 무시됩니다.

## 스킬 목록

| 스킬 | 설명 |
| --- | --- |
| [interviewer](../skills/interviewer/SKILL.md) | 결과물 기반 기술 면접관. 저장소나 아티클 링크를 주면 그 결과물을 근거로 **당신을** 면접합니다 — 모든 답변을 원본 소스와 대조해 검증하고, 얕은 답변은 꼬리질문으로 파고들며, 세션이 끝나면 약점 지도 리포트를 남깁니다. AI와 함께 작업하며 쌓이는 인지 부채를 갚기 위해 만들어졌습니다: 요약을 읽는 것은 이해했다는 착각을 만들지만, 면접은 회상을 강제합니다. |
| [session-checkpoint](../skills/session-checkpoint/SKILL.md) | 지금 세션을 `./sessions/`에 저장해, 다른 기기나 다른 에이전트가 이어받을 수 있게 합니다. 목표, **검증된 것과 가정에 불과한 것**의 구분, 다음 할 일, 그 기기에서만 참인 환경 정보를 남깁니다 — 모델의 기억이 아니라 실제 git 상태에 근거해서. 그리고 이후 작업을 구속하는 결정은 `sessions/decisions/` 아래 별도 기록으로 승격시킵니다. |
| [session-resume](../skills/session-resume/SKILL.md) | `session-checkpoint`의 반대쪽. 세션 문서와 **아직 살아 있는 결정들**을 읽은 뒤, 그 주장을 현재 저장소와 하나씩 대조합니다 — 옮겨간 HEAD, 이 기기에 없는 커밋, 문서를 쓴 기기에 커밋 안 된 채 남은 작업. 어긋난 지점을 보고한 뒤, 확인을 받기 전까지 아무 작업도 시작하지 않습니다. |

### 기기 간 작업

`session-checkpoint`와 `session-resume`은 한 쌍입니다. 한 기기·한 에이전트로 작업하다 다른 쪽에서 이어가는 상황을 위해 만들었습니다. 모두 작업 중인 저장소 안에 쌓이므로 코드와 함께 git으로 따라다닙니다.

```
sessions/
├── 2026-07-28-1432-auth-retry.md      # 한 시점 — 다음 체크포인트가 대체
└── decisions/
    └── 2026-07-28-token-in-memory.md  # 제약 — 뒤집히기 전까지 유효
```

둘을 나눈 건 수명이 다르기 때문입니다. 체크포인트는 작업이 진행되는 순간 낡지만 결정은 계속 구속합니다. 결정을 7월 어느 화요일 이름이 붙은 파일에 묻어두면 다음 세션이 같은 논의를 처음부터 다시 합니다. `session-resume`이 결정들을 읽고 무엇이 아직 살아 있는지 보고합니다.

두 디렉터리 모두 추가만 하고 고치지 않습니다. 결정 기록에 고전적인 ADR처럼 순번을 매기지 않고 날짜를 쓰는 이유도 같습니다 — 순번은 중앙 발급자가 필요한데, 두 기기가 각자 오프라인에서 `004-`를 집으면 다음 pull에서 충돌합니다. 대체는 새 기록에 거는 단방향 링크로 표현하고, 옛 파일은 건드리지 않습니다.

둘 다 **명시적 호출 전용**입니다(`disable-model-invocation`, Codex는 `allow_implicit_invocation: false`). 에이전트가 알아서 "이 세션은 기록할 만하다"고 판단하는 일은 없습니다 — 남기기로 결정한 세션만 `sessions/`에 들어갑니다.

커밋과 푸시는 하지 않습니다. `session-checkpoint`는 파일만 쓰고 멈추므로, 직접 커밋·푸시해야 다른 기기까지 도달합니다.

이름을 두 단어로 지은 건 의도적입니다. 짧은 단일 단어는 CLI 벤더가 내장 커맨드로 가져가는 영역입니다 — `/checkpoint`는 이미 Claude Code `/rewind`의 별칭이고, `/resume`은 이전 대화로 돌아가는 커맨드입니다. 같은 이름의 스킬은 가려지거나 반대로 내장 커맨드를 덮어쓰는데, 이 스킬들은 이름을 직접 입력하는 것 외엔 호출 경로가 없어서 가려지면 아예 쓸 수 없게 됩니다.

## 라이선스

MIT
