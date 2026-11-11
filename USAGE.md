# Using This Handbook

Copy-paste setups for pointing any AI coding agent (Claude Code, Codex, Cursor, Copilot, Gemini CLI, Aider, Windsurf, etc.) — or yourself — at this handbook. Pick **one** option.

Repository: `https://github.com/arpitpatel25/engineering-handbook`

---

## Option 1 — Zero setup: just tell the agent (one-off)

Paste this into any agent session:

```text
Before making design, architecture, or quality decisions for this task, read the engineering
handbook at https://github.com/arpitpatel25/engineering-handbook — start with AGENTS.md and
follow its process (profile the project, triage relevant sections, decide, record, verify).
Only read the sections relevant to this task.
```

If the agent can't browse, have it clone first:

```bash
git clone --depth 1 https://github.com/arpitpatel25/engineering-handbook /tmp/engineering-handbook
```

…and refer to `/tmp/engineering-handbook/AGENTS.md`.

## Option 2 — Per project, as a git submodule (recommended for teams & CI)

Makes the handbook available to every agent and teammate working in the repo, including cloud/CI sandboxes:

```bash
git submodule add https://github.com/arpitpatel25/engineering-handbook .handbook
git commit -m "chore: add engineering handbook"
```

Then add this to your project's `AGENTS.md` (and/or `CLAUDE.md`, `.cursor/rules`, `.github/copilot-instructions.md`, `GEMINI.md` — whatever your tools read):

```markdown
## Engineering handbook
For design, architecture, and quality decisions, consult `.handbook/AGENTS.md` and follow its
process. This file, the project's ADRs, and existing code conventions take precedence over it.
```

Update later with `git submodule update --remote .handbook`.
New clones need `git clone --recurse-submodules` (or `git submodule update --init`).

## Option 3 — Per project, vendored copy (no submodules)

```bash
git clone --depth 1 https://github.com/arpitpatel25/engineering-handbook .handbook && rm -rf .handbook/.git
```

Add the same `AGENTS.md` snippet as Option 2. Simple, but you update it manually.

## Option 4 — Machine-wide (all your projects)

Clone once to a stable path:

```bash
git clone https://github.com/arpitpatel25/engineering-handbook ~/engineering-handbook
```

Then reference it from your tool's **global** instructions file if you want every session to use it — e.g. Claude Code `~/.claude/CLAUDE.md`, Codex `~/.codex/AGENTS.md`, or your editor's global rules:

```markdown
For software design, architecture, or quality decisions, consult ~/engineering-handbook/AGENTS.md
and follow its process. Project-level instructions take precedence.
```

Keep it current with `git -C ~/engineering-handbook pull`.

## Option 5 — Start a new project from the templates

```bash
H=.handbook   # wherever the handbook lives
mkdir -p docs/adr .github
cp $H/templates/project-AGENTS.md AGENTS.md && echo "@AGENTS.md" > CLAUDE.md
cp $H/templates/project-profile.md docs/project-profile.md
cp $H/templates/adr.md docs/adr/0000-template.md
cp $H/templates/pull-request.md .github/pull_request_template.md
cp $H/templates/service-readme.md README.md   # if starting fresh
```

Fill in `AGENTS.md` and `docs/project-profile.md` first — they let agents calibrate how much rigor the project needs.

---

## Good prompts to use with it

```text
Profile this project using the handbook (00-project-profiling) and write docs/project-profile.md.
Mark anything you inferred as an assumption.
```

```text
Design <feature>. Use the handbook's design-review checklist; write a feature spec from
templates/feature-spec.md and ADRs for significant decisions. Don't implement yet.
```

```text
Review this diff against the handbook's checklists/code-review.md. Report only real issues.
```

```text
Which stage-gate items (checklists/stage-gates.md) are we missing for <stage>? List gaps by
priority with the effort for each.
```

## Tips

- Agents should read **only the sections relevant to the task** — `AGENTS.md` has a routing table.
- Project-specific instructions always override the handbook; that's intended.
- If the handbook is wrong or missing something for your context, record your deviation in an ADR — and consider contributing it back.
