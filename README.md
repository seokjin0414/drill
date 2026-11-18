# drill

A skill for AI coding agents. While you work, it finds the places where your technical understanding is missing or off, checks them, points them out, and fills them in. It does not stop the work to lecture. It does what you asked first, then adds one question at the end of the response.

The skill body is written in Korean, so the agent asks in Korean and the arguments are Korean words. The example below is translated.

## What it looks like

```
User:  I put a composite index on (user_id, created_at), so range queries on created_at will be fast too.

AI:    (does the requested work first)
       If the only condition is on created_at, how does the query use that index?

User:  created_at is in the index, so it uses it, right?

AI:    Half right. The sort order is missing.
       In what order is created_at laid out inside the index?

User:  Grouped by user_id first, then ordered by created_at within each...

AI:    Correct.
       Then without a user_id condition, which part of the index has to be read to find a created_at range?
```

At the default intensity (`헬`, "hell"), each next question is built from the weakest part of the answer you just gave. While it is asking, it does not explain. It gives a one-line verdict and moves on. It stops when you hit the bottom (the point where you can no longer answer) or when you say stop. Then it sums up how far you got and where the answers went shallow, and explains from the shallow point.

## Why

- When AI builds it for you, the work gets done even if you do not understand it. This keeps you from shipping things you do not understand.
- The learning that used to come from building things by hand is gone. Repeatedly asking "do I actually know this?" takes its place.
- What you are missing is carried across sessions in a record.

## Install

The rules are plain natural language, so they are not tied to one tool.

**Tools that support skills (`SKILL.md`)**: clone into that tool's skills folder under the name `drill`. For Claude Code:

```bash
git clone https://github.com/seokjin0414/drill.git ~/.claude/skills/drill
```

To update, run `git -C ~/.claude/skills/drill pull`. The record files and `local.md` are ignored by git, so pulling never conflicts with them.

**Tools without skill support**: put the body of `SKILL.md` into the instruction file the tool always loads (`AGENTS.md` and the like) or into the system prompt. The record only carries over where the agent can write files.

## Usage

| Input | What it does |
|---|---|
| `/drill` | Applies until the end of the session |
| `/drill <topic>` | Goes deep on that topic |
| `/drill 복습 [area]` | Review: picks items from the record and asks again |
| `/drill 정리` | Summary: shows what you know and what is missing, by area |
| `/drill 노말` · `/drill 헬` | Switches the question intensity for the session (normal / hell) |
| `/drill 끄기` | Turns it off for the session |

- In tools without slash commands, say the same thing in words ("drill 복습", "노말로").
- Answering is optional. If you give a work instruction instead of an answer, it drops the question and does the work.
- It does not ask on turns where you made no new judgment, such as lookups or simple execution. It also stays quiet during incident response.
- If a question was pointless or off, say so in the conversation ("질문 별로"). It drops the question without arguing and does not ask that kind of question again for the rest of the session. It never edits its own files.
- The skill description says to turn on in technical sessions even when not invoked, so it may turn itself on. See "Adjusting for your environment" to change this.

## Intensity

| Intensity | How it asks |
|---|---|
| `헬` (hell, default) | A technical interviewer's cross-examination. It also asks when you make a decision or assert something, takes the weakest part of each answer as the next question, and goes down to the bottom. It does not stop after a fixed number of questions |
| `노말` (normal) | Asks only when you seem not to know, in the form "Do you know what X is?" It judges the answer, fills in the gap, and moves on, with at most one follow-up |

## Record

Only concepts you got wrong or did not know are written to `ledger.md`, one line each. The state moves from `헷갈림` (confused) to `설명함` (explained) to `확인됨` (confirmed). A concept is confirmed only when you get it right on a later day than the explanation, not right after hearing it. When the same concept comes up in a later session, it asks before explaining again. `복습` (review) picks its questions from this record.

The record contains the system names and values of your environment, so it is listed in `.gitignore` and never reaches the repository.

## Files

| File | Role |
|---|---|
| `SKILL.md` | The rules |
| `review.md` | Procedure for `정리` and `복습` (read only then) |
| `ledger.md` | The record. Created on first write |
| `ledger-archive.md` | Archive of old record lines. Created on first move |
| `local.md` | Settings kept only in your environment (optional). Not committed |

## Adjusting for your environment

- **Default intensity**: to make normal the default, change the first paragraph of the '질문' section in `SKILL.md` and the "(기본)" in its subheading.
- **Reference material for hell (optional)**: if you have material to calibrate the level of questioning against, such as notes of real interview questions, create `local.md` in the skill folder and write its path there. It works without one.
- **How it turns on**: to have it turn on only when invoked, delete the sentence in the `SKILL.md` `description` that says to turn on in technical sessions without being invoked ("기술 작업 세션에서는 부르지 않아도 … 적용한다"). To make sure it is on in every session, the description alone is not a guarantee, so add the line below to the instruction file that is always loaded in your environment (`CLAUDE.md`, `AGENTS.md`, and the like).

```markdown
- **이해 점검 상시 적용(기본 켜짐·강도 헬)**: 기술 작업 세션에서는 부르지 않아도 첫 기술 응답 전에 `drill` 스킬을 불러 세션 내내 적용한다(규칙은 스킬 본문). 그 세션만 낮추기 = "노말로", 끄기 = "drill 끄기"
```

## License

MIT. See [LICENSE](LICENSE).
