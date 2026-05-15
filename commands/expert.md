---
description: Show the symbols and modules a person owns or has the deepest history in — inverse of `/oxagen:who-knows`.
argument-hint: "<github-login | email>"
allowed-tools: mcp__oxagen__code_expertise
---

# /oxagen:expert — what does this person know?

Wraps `code.expertise` (M9, SPEC §7.3). Given a person, returns the
symbols / modules / files they have the strongest authorship and
review signal on.

## Steps

### 1 — Take the argument

The argument is a GitHub login (preferred) or an email tied to a
commit author in the workspace.

### 2 — Call `code.expertise`

```
mcp__oxagen__code_expertise({
  person: "<login-or-email>",
  limit: 25
})
```

SPEC §7.9 envelope. Rows carry `commits_authored`, `reviews_given`,
and `score`, scoped to the person.

### 3 — Format

```
Expertise: <login>  (<N> symbols)

  <score>  <file:line>  <symbol_name>   <commits>c / <reviews>r
  ...
```

Group by top-level module if results span more than one.

### 4 — Suggest follow-ups

- "Run `/oxagen:pr-history <symbol>` for the recent change log."
- "Run `/oxagen:who-knows <symbol>` to flip direction."

## Failure modes

| Symptom | Fix |
|---------|-----|
| Empty results | Person may have no graph signal — check the GitHub login spelling |
| Counterfactual_method shows `expertise_git_log` | This means we used the git-log fallback; results are still valid but coarser than the cached aggregator |
