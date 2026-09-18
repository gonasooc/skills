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

직접 설치한 스킬은 이름으로 호출합니다 — Claude Code에서는 `/interviewer`, Codex에서는 `$interviewer`. 제품의 스킬 피커를 사용해도 됩니다. Claude Code 플러그인으로 설치하면 네임스페이스가 붙으므로 `/gonasooc-skills:interviewer`로 호출합니다. `interviewer`는 자연어로 요청해도 실행됩니다. `agents/openai.yaml` 같은 에이전트 전용 파일은 이를 사용하지 않는 에이전트에서는 무시됩니다.

## 스킬 목록

| 스킬 | 설명 |
| --- | --- |
| [interviewer](../skills/interviewer/SKILL.md) | 결과물 기반 기술 면접관. 저장소나 아티클 링크를 주면 그 결과물을 근거로 **당신을** 면접합니다 — 모든 답변을 원본 소스와 대조해 검증하고, 얕은 답변은 꼬리질문으로 파고들며, 세션이 끝나면 약점 지도 리포트를 남깁니다. AI와 함께 작업하며 쌓이는 인지 부채를 갚기 위해 만들어졌습니다: 요약을 읽는 것은 이해했다는 착각을 만들지만, 면접은 회상을 강제합니다. |

## 라이선스

MIT
