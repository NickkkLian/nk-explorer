# nk-explorer

![nk-explorer](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/social/nk-explorer.png)

A [Claude Code](https://code.claude.com) skill. Turn a concept that can be written as one formula — compound interest, a loan's monthly payment, the chance of at least one success — into one self-contained HTML page where every slider shows the formula it changes, written with names and with the current numbers, beside the result and a curve, plus practice questions graded on the result rather than on how it was reached.

Part of [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) — agent skills that ship a self-test with every script; the Verify
section below says which of them were broken on purpose before release to prove they react.

![nk-explorer demo: one idea in, a finished page out](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/nk-explorer.gif)

## What it does

- Five invariants: every control is in the formula and every move shows (the formula line rewritten with the moved number marked, the result, the curve); the formula shown is the formula computed, with no eval, and the line with numbers works out to the result; a question is graded on the result, with a tolerance no other answer a slider gives can pass; a reach question asks for something the starting values do not already give; the page says what the formula leaves out, where it comes from, and that it is not advice.
- `assets/starter.html`, a working explainer (how compound interest grows), and `assets/concepts/`, three concepts to start from: compound interest, a loan's monthly payment, and the chance of at least one success in a number of tries. The page is built from the concept block alone; `assets/design-tokens.css` is the token file it shares with the other pages in this family.
- `scripts/make_explorer.py`: a concept JSON becomes one file, tokens written in, sealed with a Content-Security-Policy that names the page's own scripts by their SHA-256 — the browser runs those and nothing else, loads nothing from any file or address, and runs no string as code. Before it writes, it refuses a concept that breaks one of seven rules — a control the formula never uses or that never changes the result, a slider whose starting value is between its positions, a page that would open on an error, a question that leaves a control unset or does not state each value it uses (a 5 written as years does not state a 5% rate), a tolerance another answer would pass, a reach question nothing passes or that the starting values already pass.
- `scripts/explorer_check.py`: fourteen rules on the finished file — the seven again, read back out of the page, then the formula language unchanged, the page's policy in place and still matching its scripts (and the usual ways a page asks for something from outside named, so they can be taken out), text written as text with no eval, colours from the tokens, no icon characters, a footer that says what is left out, and the page wired to its concept.
- `scripts/probe_check.py`: opens the page in Chrome with `?probe=1`. The page moves every slider and answers every question through its own buttons; the script recomputes what it recorded — the formula line, the result and every readout at each state it records, and every grading — with `scripts/formula.py`, a second, Python implementation of the page's formula language, and `scripts/concept.py`, the concept rules. Without Chrome, open `page.html?probe=1` yourself; the page's own summary is at the foot.
- Standard library only (probe_check.py also needs Chrome). The page it writes has no dependencies and no build step.

The full procedure, the boundaries and where the rules came from are in [SKILL.md](SKILL.md).

## How it works

1. Pick one formula
2. Write the concept
3. Choose ranges a person would try
4. Write the practice questions
5. Build the page
6. Check the file
7. Run it
8. Open it and move everything

## Why it is built this way

**The idea.** A formula is easier to believe once you have moved it. Every line is written from the formula the page computes with, so what you read is what it calculated.

**Where it came from.** The two ideas at the centre come from [Query Mirror](https://github.com/NickkkLian/SQL-Viz-for-Edu), a SQL teaching tool of mine: every click on the table becomes the SQL behind it, and practice is graded on the result set rather than on how the query was written. This skill carries both over to formulas and none of the code.

**Evidence.** What was broken on purpose to show that the self-tests can fail is under [Verify](#verify); what was run end to end, and in which agent, is under [Compatibility](#compatibility).

## Install

Pick one of four ways: three for Claude Code, one for OpenAI Codex. Skills load when a session starts, so open a **new** session after installing.

### 1 · Terminal, one command

```bash
git clone https://github.com/NickkkLian/nk-explorer ~/.claude/skills/nk-explorer
```

1. Run the command above (for one project only, clone into `.claude/skills/nk-explorer` inside that project).
2. Start a new Claude Code session.
3. Check it loaded: type `/nk-explorer` — it appears in the slash-command menu. Or just ask for the task; the skill triggers on its own.

### 2 · Claude Code in a terminal session (plugin)

The plugin route goes through the [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) marketplace. Add it once; after that each skill is one command.

```
/plugin marketplace add NickkkLian/nickkk-skills
/plugin install nk-explorer@nickkk-skills
```

1. In a Claude Code session, run the first line (once per machine).
2. Run the second line.
3. Start a new session (or run `/reload-plugins`). The skill shows up as `nk-explorer:nk-explorer`.

Without opening a session, the same two steps work from a shell: `claude plugin marketplace add NickkkLian/nickkk-skills` then `claude plugin install nk-explorer@nickkk-skills`.

### 3 · Claude desktop app (Code tab)

**Add the marketplace first — Discover only searches marketplaces you have already added.**

<img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/panel-route.gif" alt="Adding the marketplace and installing a skill in the desktop app" width="640">

<sub>Recorded on 2026-09-16, when the marketplace listed ten skills, all at version 0.1.0; it lists more now. The repository list in this recording shows the recorder's own repositories because a GitHub account is connected; yours will show yours. Type the full name as in step 4.</sub>

1. In the chat box, type `/plugin marketplace` and press Enter (or open **Settings → Customize → Plugins**). The **Plugins** panel opens.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step1-type-plugin-marketplace.png" alt="/plugin marketplace typed in the chat box" width="480">
2. Top right, open **Add ▾** and choose **Add marketplace**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step2-add-menu.png" alt="The Add menu with Add marketplace" width="480">
3. Choose **Add from a repository**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step3-add-from-repository.png" alt="Add marketplace dialog: Add from a repository" width="480">
4. In **URL**, type the full `NickkkLian/nickkk-skills`. At the bottom of the list choose the row **Use "NickkkLian/nickkk-skills"**, then press **Sync**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step4-url-then-sync.png" alt="URL filled in, Sync button" width="480">
5. You land on **Discover**, filtered to the new marketplace (**Filter · 1**). Find **Nk explorer** and press **Add**. Installed ones show **✓ Added**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step5-discover-add.png" alt="Discover list with Added and Add buttons" width="480">
6. Close the panel and start a new session.

To try it for one session without installing anything: `claude --plugin-dir ./nk-explorer` from a clone.

### 4 · OpenAI Codex CLI

```bash
git clone https://github.com/NickkkLian/nk-explorer.git ~/.agents/skills/nk-explorer
```

1. Run the command above (for one project only, clone into `.agents/skills/nk-explorer` inside that project).
2. Start a new Codex session.
3. Check it loaded, without spending a model call: `codex debug prompt-input | grep -o -- '- nk-explorer[a-z0-9:-]*' | sort -u` prints `- nk-explorer:nk-explorer:`. Codex adds the `nk-explorer:` prefix because this repository also carries a Claude Code plugin manifest. Ask for the task and the skill triggers on its own, or type `$` and pick it from the list.

## Compatibility

| Agent | Tested | What was checked |
|---|---|---|
| Claude Code (CLI 2.1.173, macOS) | yes | In a fresh project with an isolated Claude config, inside a macOS sandbox that blocked reading the tester's ~/.claude folder (settings, session history, memory), Desktop, Documents and Downloads, SSH keys and git identity, a plain request that never names the skill triggered it and it ran its bundled script. The brief asked for one page for a driving-theory class where students move the speed, the reaction time and the braking and watch the stopping-distance formula change, with practice questions they can check themselves; it never named the skill. In nine turns the run read an example concept and the format reference, tried the formula with scripts/formula.py, wrote a concept of its own, built the page with no refusal, and ran scripts/explorer_check.py (0 findings) and scripts/probe_check.py, which could not start Chrome's own sandbox inside the test sandbox, ran again with --no-sandbox and said so: 7 of 7 checks passed. That was the version before the outside audit; its concept, built again with the version released after it, gives 0 findings and 7 of 7 as well. What no check catches: its second question says “double the speed to 40 m/s” after a first question at 30 m/s, the kind of wording a person reads for in references/acceptance.md. |
| OpenAI Codex CLI (0.155.0-alpha.9.2, gpt-5.6-sol, low reasoning, macOS) | partly | Copied into `~/.agents/skills` of a temporary home, in a fresh project, without the user's Codex config, with the same brief. Codex read SKILL.md, the concept format, the acceptance list and an example, wrote a concept that takes the speed in km/h, checked it with scripts/concept.py (0 problems), built the page and checked it with scripts/explorer_check.py (0 findings). scripts/probe_check.py got no report back from Chrome and said the page wrote none, as if the page were at fault; Codex then ran Chrome by hand, found it ended by signal 6 inside Codex's sandbox even without Chrome's own sandbox, and reported that the Chrome test could not run there. Its concept, built again with the version released after the outside audit, gives 0 findings and 6 of 6 probe checks. probe_check.py now names a Chrome that dies instead of blaming the page, and also retries without Chrome's sandbox when Chrome dies without a word; both were tested with a fake Chrome, not by running Codex again. |
| Cursor, Gemini CLI | no | Not tested. Their documentation says both read `~/.agents/skills`, the folder route 4 clones into; Gemini CLI asks before it activates a skill. |

This skill's frontmatter uses only name, description, license, compatibility and metadata.

## Verify

```bash
python3 scripts/concept.py --selftest
python3 scripts/explorer_check.py --selftest
python3 scripts/formula.py --selftest
python3 scripts/make_explorer.py --selftest
python3 scripts/probe_check.py --selftest
```

Python 3.9+, standard library only; probe_check.py also needs Google Chrome or Chromium.
90 checks in explorer_check.py and concept.py were broken on purpose, one at a time, in a sandbox copy,
each turning the sample written for it red, none by a crash; so were 40 checks in probe_check.py, each against
a page broken in the way that check is there to catch (or, for the checks on how Chrome is run, a stand-in Chrome).
The 120 lines of those scripts that the matrices' pattern takes for reporting a finding were read from the source, not listed by hand: 6 only pass on what other lines found, and 1 is left out of the matrices. The one left out (a control that is not an object) is broken and run anyway: on a malformed control alone it crashes,
but with a second problem in the same concept it drops its own message without a crash, and no sample catches that.
The lines that print the findings and set the exit code, and explorer_check.py's line that passes on concept.py's
findings, are not among the lines read; no matrix breaks them (see Limits). Opened in Chrome, a page with an outside font,
an image list, an @import and an indirect eval added loaded none of them and ran no eval; a meta refresh, which no
policy stops, is what explorer_check.py names. The page's formula language and its Python twin gave the same numbers
and the same text on 2084 cases; on both the line with numbers worked out to the result, and
both gave the round() answers in a table worked out by hand. A formatter planted in one twin was caught, and so, in both
at once, were the old writer that put 1 × -3^2 for x = -3 and the 0.1.0 round(), floor() and ceil(), which rounded the
binary working and gave 88.82 for round(8.5 × 10.45, 2) and 8 for ceil(2.1 ÷ 0.3), where the answers by hand are 88.83 and 7.
make_explorer.py has a self-test but no break matrix.

## Limits

- **One formula, one result, sliders only.** No systems of equations, no step-by-step derivations, no dropdowns or switches. Each control is a number on a grid of at most 2,000 steps, and a page has one to six of them.
- **Formulas only, not code or queries.** A concept that is really a query over a table (GROUP BY, a join) is not this version's; a table-and-query kind is planned for 0.2.
- **The chart moves one control and holds the rest where they are.** It is a slice, not a map of everything.
- **Answers are read as plain numbers.** The result's unit, a currency sign and % are removed; commas may only separate groups of three digits (2,318.55), and e-notation is read because the page writes very large and very small results that way. A decimal comma (2318,55), a misplaced comma (23,18.55) and hex (0x90F) are not numbers here: the page says "Type a number."
- **It cannot tell whether the formula fits a case.** The page says where the formula comes from and what it leaves out; whether it applies to somebody's own loan or experiment is theirs to judge, and the footer says so.
- **The checks read the page as built, and the policy does not hold every edit.** Tested in Chrome, the page's policy stops an outside font, image or stylesheet, an @import and eval, and a script changed without its hash does not run. It does not stop: an edit that removes or loosens the policy line (explorer_check.py refuses that page, E09); anything placed before the policy line, which makes Chrome ignore the policy (E09 names such a page); a meta refresh (E09 names it); a script that navigates away (`location.assign`, `location =`, `window.open`), which explorer_check.py does not name; and a script whose hashes are written again after the edit, which then runs. Text can be changed to say anything. Run explorer_check.py and probe_check.py again after any edit, or build the page again.
- **E10 covers the usual ways of writing markup, not all of them.** It names innerHTML, outerHTML, insertAdjacentHTML and the others it lists, written that way; `el["inner" + "HTML"] = …` or `setHTMLUnsafe(…)` added to a page with its hashes written again gets no finding.
- **The checkers' own command lines are not break-tested.** The break matrices cover the lines that report a finding, not the lines that print the findings and set the exit code, nor explorer_check.py's line that passes on concept.py's findings. Changing explorer_check.py's exit code to 0 left every self-test green; a run then prints its findings but exits 0.

## License

MIT. Read a script before letting it run in your environment.
