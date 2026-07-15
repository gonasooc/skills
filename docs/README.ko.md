# Skills

Claude Code를 비롯한 코딩 에이전트에서 쓸 수 있는 [Agent Skills](https://agentskills.io) 모음입니다.

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

### Codex

위 skills.sh 인스톨러가 Codex 설치를 직접 지원합니다 — 설치 대상 선택 화면에서 Codex를 고르면 됩니다. 수동으로 설치하려면 스킬 폴더를 Codex의 스킬 경로에 복사(또는 심링크)하세요: `~/.agents/skills/`(사용자 전역) 또는 저장소 내부의 `.agents/skills/`. 호출은 `$interviewer`, `/skills` 피커, 혹은 자연어 요청으로 합니다.

## 스킬 목록

| 스킬 | 설명 |
| --- | --- |
| [interviewer](../skills/interviewer/SKILL.md) | 결과물 기반 기술 면접관. 저장소나 아티클 링크를 주면 그 결과물을 근거로 **당신을** 면접합니다 — 모든 답변을 원본 소스와 대조해 검증하고, 얕은 답변은 꼬리질문으로 파고들며, 세션이 끝나면 약점 지도 리포트를 남깁니다. AI와 함께 작업하며 쌓이는 인지 부채를 갚기 위해 만들어졌습니다: 요약을 읽는 것은 이해했다는 착각을 만들지만, 면접은 회상을 강제합니다. |

## 왜 만들었나

AI의 도움으로 완성한 작업에는 인지 부채가 쌓입니다 — 소유자가 설명하지 못하는 코드와 글. 수동적인 요약으로는 갚아지지 않습니다. 이 스킬들은 부채를 실제로 갚아주는 회상(retrieval)을 강제하기 위해 존재합니다.

## 라이선스

MIT
