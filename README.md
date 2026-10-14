# nk-explorer

Turn a formula into an offline page with sliders, a curve and practice questions graded on the result.

Ask your coding agent, for example:

> Use nk-explorer to explain a £200,000 loan at 5% interest. Let me change the years to pay it back and practise reading the monthly payment.

![nk-explorer: the bundled loan example, with an answer, slider and curve](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/results/nk-explorer.png)

**Re-checked against Python:** £1169.18/month at 25 years; £1045.02 at 32 years. The loan page passes [6/6 browser checks](#how-this-version-was-tried).

[Download the loan page](output/loan.html) and open it locally. Version **0.1.7**; the download has the updated axes.

## What it makes

For teachers and writers using a coding agent to explain one formula with a calculator,
one main slider and a graph. Generate an HTML page, then check it and inspect it in the browser you intend to use.
The page is designed to open locally and work offline in a modern browser.

**The loan example starts at £1169.18 per month.** Its comparison shows £1169.18 over 25 years versus £1045.02
over 32 years with the other inputs unchanged. The shipped [output/loan.html](output/loan.html) shows the
question, answer and curve, expandable calculation and settings, and practice questions graded on the result.
The picture above is that page. Regenerate it with the commands below after editing its concept.

## Try it from a checkout

No packages or build step are required. Run from the skill folder. The builder overwrites the named output file;
relative paths are relative to the working directory. Create the output and scratch folders first:

```bash
mkdir -p output scratch
export TMPDIR="$PWD/scratch"
python3 scripts/make_explorer.py assets/concepts/loan-payment.json -o output/loan.html
```

Then verify the generated file and the non-browser test fixtures:

```bash
python3 scripts/explorer_check.py output/loan.html
python3 scripts/concept.py --selftest
python3 scripts/explorer_check.py --selftest
python3 scripts/formula.py --selftest
python3 scripts/make_explorer.py --selftest
python3 scripts/numsrc.py --selftest
```

With Node available, also check the template's axis labels and scale without launching a browser:

```bash
node scripts/chart_check.js --selftest
node scripts/chart_check.js output/loan.html
```

This runs the page's formula and chart functions with a minimal SVG document; browser layout and interaction
still need the Chrome check and human inspection below. The builder and Python checks require no Node.

Open `output/loan.html` and inspect it. Where Chrome/Chromium and permission to launch it are available, also run:

```bash
python3 scripts/probe_check.py output/loan.html
python3 scripts/probe_check.py --selftest
```

These commands do not install the skill or edit agent configuration. They write the requested HTML, and self-tests
and the browser probe create temporary files. Python uses `TMPDIR` when set, otherwise its system temporary
directory. The browser probe creates an `explorer-probe-…` profile there and removes it in `finally`; its self-test
also creates temporary pages and fake browser executables. Builder self-tests create temporary concept/HTML files.
An interruption can leave temporary files. If `PROBE_CACHE` is set, the probe also keeps JSON reports in that
directory; leave it unset for an uncached check after edits. Chrome may have its own platform side effects.
`formula.py`, `concept.py` and `explorer_check.py` print results/findings without output files.
`python3 scripts/numsrc.py check output/loan.html` checks number-source markup without writing output files.
`python3 scripts/make_explorer.py --concept-of output/loan.html` prints the embedded JSON without writing a file.

## Make a page for a request

Ask an agent to use this skill with a request such as:

> Let me change a restaurant bill, tip percentage, and number of friends to see how much each person pays.

![nk-explorer](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/social/nk-explorer.png)

The procedure is in [SKILL.md](SKILL.md), the JSON fields in
[references/concept-format.md](references/concept-format.md), and the human walkthrough in
[references/acceptance.md](references/acceptance.md).

Write one formula and one to six sliders. Carry the request's units into the controls and result, and put unit
conversions in the formula. A percent result must be computed on a 0–100 scale; adding `%` does not convert a ratio.
Currency results default to two decimal places; physical quantities, percentages and dimensionless results use
the ordinary six-significant-digit formatter (whole numbers below 10^15 stay plain). Optional `result.decimals`
sets 0–8 fixed decimal places. Slider readouts retain their exact formatting. Grading uses the full result,
so choose a practice tolerance consistent with the precision asked for.

Write `question` as a short question in the person's words and choose `chart.x` as the slider worth trying first.
Open on an ordinary case. Optional `result.unavailable` explains an invalid setting and what to change, such as
zero speed on a running-time page. Otherwise the page explains the arithmetic error.

Give the first screen a reason to inspect the curve: `chart.comparison` can select two distinct slider positions
with `from` and `to`. Both must be on the slider grid and give finite answers at the opening settings. Pick a
relevant pair, or a threshold the person actually asked about. Without this field, the page chooses the opening
position and another about one fifth of the range away. The results update with the other settings and disclose
undefined endpoints. A linear formula can still show what another hundred sales earns; no curvature is invented.

The chart samples 201 points. Undefined samples break the curve. A sparse extreme tail can be omitted from the
curve and scale; a note gives the omitted count and visible bounds. The current answer remains in the headline,
and an off-range dot is omitted and explained. The exact rule and its limits are in the format reference.
Both axes identify the quantity and unit. The existing design tokens provide the colours and typography.
Axis labels use grouped numbers and only the decimals needed to distinguish ticks: `£15,000`, rather than
`£15000.00`. Set `chart.locale` to the request's language and region, such as `en-GB` or `de-DE`.
This affects axes only; headline, formula, slider readouts and answer parsing retain their existing rules.

## Layout target

The design places controls beside the curve above 760px and the curve after the controls at narrower widths.
Other settings expand inside the controls panel; formula working and its legend expand below the curve.

The opening layout is designed for **1366 × 768 CSS pixels**, at 100% zoom, a 16px root font, and closed expandable
sections: the answer and whole curve should fit without scrolling. The builder checked CSS placement and a
conservative height estimate. A subsequent reviewer inspected the first screen at that viewport and ran the
browser checker on five pages; the recorded browser results are described below. Those records do not establish
mobile readability, keyboard/focus behaviour or first-impression appeal. Long headings or notes need inspection.

## What the checks establish

`make_explorer.py` validates the concept before writing. `explorer_check.py` reads the finished page against
fourteen rules: concept validity, the shipped formula language, policy and script hashes, listed outside-resource
and markup-writing patterns, token colours, icon characters, footer disclosures and probe wiring.
The page carries its own formula parser, without `eval`, and a Content-Security-Policy naming its scripts by hash.
Click the result or a slider readout to inspect its number source.

`probe_check.py` launches Chrome with `?probe=1`, works the page's sliders and practice buttons, and compares the
recorded formula lines, headline formatting, readouts and grading against Python. Without a runnable browser,
report this check as unrun. Opening `page.html?probe=1` manually shows the page's own report; independent checking
still requires recomputation outside the page.

## How this version was tried

The 0.1.7 offline run passed concept 32/32, explorer_check 105/105, formula 44/44, make_explorer 12/12,
numsrc 50/50 and the new chart regressions 14/14. All three shipped concepts built with 0 file findings;
the number-source checker emitted only the documented warnings for control readouts created at runtime.
The new chart tests passed 6/14 on the old template, failing the grouped currency labels and loan scale cases.
Chrome was not launched for 0.1.7. The browser recordings below are historical 0.1.6 evidence.

The supplied reports describe two agents who did not build the skill, each choosing three requests. The first
tried bill sharing, running time and kinetic energy: its report records correct numbers, missing units and a
running-time chart flattened near zero speed; two pages felt like generic worksheets. The second tried
royalties, tank filling and a lens: its report records correct numbers but found the curve below the controls
despite the README saying it was beside them, and weak appeal in two examples. The build adds units and
formatting, disclosed chart limits, controls beside the curve on wide screens, and live comparisons.

The five retained original examples (bill sharing, running time, kinetic energy, population growth and
probability) were rebuilt at 0.1.6 for the browser checks below. The later royalty, tank and lens comparison
pages are representative reconstructions; the original second agent's pages were not supplied. The trial
reports and these pages remain integration evidence, not artifacts bundled in this checkout; the historical
trial findings are reported results, not a fresh independent reproduction of the original trials.

A reviewer ran the skill's browser checker in headless Chrome on bill sharing, running time, kinetic energy,
population growth and probability: the supplied reports total 25/25 checks passed. The first screen was also
inspected at 1366 × 768.

The 0.1.6 loan page then passed 6/6 browser checks, and the browser self-test passed 47/47 on Python 3.13.9
and 47/47 on Python 3.9.6, all with exit 0. These runs used macOS 15.7.3 and Google Chrome 156 beta on the
current tree and the fresh lab copy; their printed outputs are recorded below. One earlier run of the same
self-test passed 45/47; two repeat runs passed 47/47. The cause of the earlier failures was not established.
These are recorded results for those runs, not a guarantee of repeatability.

All five non-browser self-tests and the generated loan page's file check pass locally. These fixture results
and recorded browser checks do not complete human acceptance or establish release readiness. Keyboard and focus,
phone-size appearance, other browsers, Windows and Linux remain unchecked. Source-panel interaction,
appearance persistence, cache behaviour and a separate audit of temporary-profile cleanup were not checked.
The browser self-test checks an edited page whose script hashes no longer match its policy; broader policy
enforcement was not separately audited. Shared number-source files were compared during integration.

<details><summary>Command output recorded on 2026-10-11</summary>

The setup commands print nothing. Non-browser blocks retain the complete local command output. The two
browser blocks reproduce the reviewer’s recordings as printed, including their command and exit markers.

`mkdir -p output scratch` (exit 0):

```text
```

`export TMPDIR="$PWD/scratch"` (exit 0):

```text
```

`python3 scripts/concept.py --selftest` (exit 0):

```text
  PASS  a slider's third tenth is 0.3, not 0.30000000000000004
  PASS  positions: ten steps, 0.7 on the grid, 0.75 and 1.1 off it
  PASS  halves round up, as Math.round does (2.5 -> 3, -2.5 -> -2)
  PASS  a unit's singular is used for exactly 1 and -1
  PASS  a currency sign goes in front (after a minus), % sticks to the number, no unit is just the number
  PASS  money with one decimal gets a second; two or more decimals are left as they are
  PASS  currency headlines carry two fixed decimals without changing slider readouts
  PASS  physical units, percent and dimensionless headline defaults
  PASS  explicit precision and singular units are respected
  PASS  fixed decimals round the computed binary value, remove negative rounded zero and handle huge magnitudes
  PASS  domain errors have a helpful default and a concept-specific override
  PASS  goals: within is inclusive at both ends, at most and at least include the target
  PASS  invalid headline precisions are refused by the concept schema
  PASS  a finite comparison between two chart slider positions is accepted
  PASS  duplicate, off-grid, out-of-range and malformed comparison endpoints are refused
  PASS  the reach walk finds the first pass at n = 11 and the first fail at n = 0 (got {'P': 1000, 'r': 7, 'n': 11.0}, {'P': 1000, 'r': 7, 'n': 0.0})
  PASS  lowest: 11 years passes, 12 does not (it reaches the goal, but so does 11)
  PASS  without lowest, 12 years passes too
  PASS  lowest looks at every lower position: a ball thrown up at 20 m/s is within 1 m of 15 m at 3 s, but the lowest is 0.9 s
  PASS  a 5 that is written as years does not state a 5% rate
  PASS  each value paired with its own number: £2,000, 5%, 5 years
  PASS  name = value ties a number to its name, and a minus is read (x = -2)
  PASS  a unit word no control uses (metres) does not stop a number standing for its value
  PASS  a number that only says how to answer (2 decimal places, the nearest 10, 3 significant figures) stands for no value
  PASS  a number that only says how to answer or which question it is stands for no value, in 15 forms of the class
  PASS  and a number that is a value is still read as one: 'within 2 of 10', 'in 5 tries', a label and a precision beside the values
  PASS  the example at-least-one-success.json meets every concept rule
  PASS  the example compound-interest.json meets every concept rule
  PASS  the example loan-payment.json meets every concept rule
  PASS  a zero tolerance is refused with E06 (the rest of the rules are tested page by page in explorer_check.py)
concept selftest: 30/30 passed
```

`python3 scripts/explorer_check.py --selftest` (exit 0):

```text
  ✔ clean → want ['clean'], got ['clean']
  ✔ E01 no concept block → want ['E01'], got ['E01']
  ✔ E01 a concept that is a list, not an object → want ['E01'], got ['E01']
  ✔ E01 a control that is not an object → want ['E01'], got ['E01']
  ✔ E01 a label left empty → want ['E01'], got ['E01']
  ✔ E01 notes written as an object, not a list → want ['E01'], got ['E01']
  ✔ E01 a question of a type the page does not know → want ['E01'], got ['E01']
  ✔ E01 a question value written as text → want ['E01'], got ['E01']
  ✔ E01 a question value that is true → want ['E01'], got ['E01']
  ✔ E04 a step below zero, on a control the formula does not use → want ['E04'], got ['E04']
  ✔ E01 a misspelt field → want ['E01'], got ['E01']
  ✔ E01 a field the skill does not know → want ['E01'], got ['E01']
  ✔ E01 a required field left out → want ['E01'], got ['E01']
  ✔ E01 a number written as text → want ['E01'], got ['E01']
  ✔ E01 true where a number goes → want ['E01'], got ['E01']
  ✔ E01 a key written twice → want ['E01'], got ['E01']
  ✔ E01 a title longer than 90 characters → want ['E01'], got ['E01']
  ✔ E01 no notes → want ['E01'], got ['E01']
  ✔ E01 no practice questions → want ['E01'], got ['E01']
  ✔ E01 seven controls → want ['E01'], got ['E01']
  ✔ E02 a name that is not a control → want ['E02'], got ['E02']
  ✔ E02 a formula that does not parse → want ['E02'], got ['E02']
  ✔ E02 the result named like a control → want ['E02'], got ['E02']
  ✔ E02 the result named like a function → want ['E02'], got ['E02']
  ✔ E04 a starting value between positions → want ['E04'], got ['E04']
  ✔ E04 a step that does not divide the range → want ['E04'], got ['E04']
  ✔ E04 positions that need more than six digits → want ['E04'], got ['E04']
  ✔ E04 a control named __proto__ → want ['E04'], got ['E04']
  ✔ E04 two controls with one name → want ['E04'], got ['E04']
  ✔ E04 a control named like a function → want ['E04'], got ['E04']
  ✔ E04 a slider with more than 2000 steps → want ['E04'], got ['E04']
  ✔ E04 a slider with no range → want ['E04'], got ['E04']
  ✔ E03 a control the formula never uses → want ['E03'], got ['E03']
  ✔ E03 a control that never changes the result → want ['E03'], got ['E03']
  ✔ E05 no value at the starting values → want ['E05'], got ['E05']
  ✔ E05 the chart runs along something that is not a control → want ['E05'], got ['E05']
  ✔ E05 a chart with no curve to draw → want ['E05'], got ['E05']
  ✔ E06 a question that leaves a control out → want ['E06'], got ['E06']
  ✔ E06 a question value between positions → want ['E06'], got ['E06']
  ✔ E06 a question that never says a value it uses → want ['E06'], got ['E06']
  ✔ E06 a value the prompt gives to another control → want ['E06'], got ['E06']
  ✔ E06 a number that only says how to round, standing in for a value → want ['E06'], got ['E06']
  ✔ E06 a question label's number standing in for a value → want ['E06'], got ['E06']
  ✔ E06 the M of a question label's N of M standing in for a value → want ['E06'], got ['E06']
  ✔ E06 a precision's number standing in for a value → want ['E06'], got ['E06']
  ✔ E06 a bracketed count standing in for a value → want ['E06'], got ['E06']
  ✔ clean: a negative value stated as x = -2 → want ['clean'], got ['clean']
  ✔ E06 a question the formula has no answer for → want ['E06'], got ['E06']
  ✔ E06 a tolerance of zero → want ['E06'], got ['E06']
  ✔ E06 a tolerance that also passes a neighbour → want ['E06'], got ['E06']
  ✔ clean: a step function with a tight tolerance → want ['clean'], got ['clean']
  ✔ E06 a tolerance a step function hides → want ['E06'], got ['E06']
  ✔ E06 a tolerance a far position passes, on a curve that turns back → want ['E06'], got ['E06']
  ✔ E07 fixes something that is not a control → want ['E07'], got ['E07']
  ✔ E07 a fixed value between positions → want ['E07'], got ['E07']
  ✔ E07 three controls left free → want ['E07'], got ['E07']
  ✔ E07 a goal the page does not know → want ['E07'], got ['E07']
  ✔ E07 a within goal with no tolerance → want ['E07'], got ['E07']
  ✔ E07 a tolerance on an at-least goal → want ['E07'], got ['E07']
  ✔ E07 lowest and highest together → want ['E07'], got ['E07']
  ✔ E07 lowest names a fixed control → want ['E07'], got ['E07']
  ✔ E07 more settings than the checker walks → want ['E07'], got ['E07']
  ✔ E07 a question that never says its target → want ['E07'], got ['E07']
  ✔ E07 a within goal whose prompt gives another tolerance → want ['E07'], got ['E07']
  ✔ E07 a goal the starting values already pass → want ['E07'], got ['E07']
  ✔ E07 a goal no setting reaches → want ['E07'], got ['E07']
  ✔ E08 a changed formula language → want ['E08'], got ['E08']
  ✔ E08 the language's markers removed → want ['E08'], got ['E08']
  ✔ E09 the tokens linked, not written in → want ['E09'], got ['E09']
  ✔ E09 no Content-Security-Policy → want ['E09'], got ['E09']
  ✔ E09 a policy that lets images in from anywhere → want ['E09'], got ['E09']
  ✔ E09 a script changed after the page was sealed → want ['E09'], got ['E09']
  ✔ E09 a meta refresh → want ['E09'], got ['E09']
  ✔ E09 an @import in the styles → want ['E09'], got ['E09']
  ✔ E09 an image list from an outside address → want ['E09'], got ['E09']
  ✔ E09 a Google Fonts stylesheet → want ['E09'], got ['E09']
  ✔ E09 a script from a file → want ['E09'], got ['E09']
  ✔ E09 an image from an outside address → want ['E09'], got ['E09']
  ✔ E09 an outside address in the styles → want ['E09'], got ['E09']
  ✔ E09 a fetch → want ['E09'], got ['E09']
  ✔ E10 markup written into the page → want ['E10'], got ['E10']
  ✔ E10 eval → want ['E10'], got ['E10']
  ✔ E10 eval called indirectly → want ['E10'], got ['E10']
  ✔ E10 eval reached through a dot → want ['E10'], got ['E10']
  ✔ E10 a javascript: link → want ['E10'], got ['E10']
  ✔ clean: a question that says 'only =' → want ['clean'], got ['clean']
  ✔ E10 an inline event handler → want ['E10'], got ['E10']
  ✔ E11 a raw colour in the page's styles → want ['E11'], got ['E11']
  ✔ E11 a colour in an inline style → want ['E11'], got ['E11']
  ✔ E11 a colour in an SVG attribute → want ['E11'], got ['E11']
  ✔ E11 a named colour in the page's styles → want ['E11'], got ['E11']
  ✔ E11 a colour set from the script → want ['E11'], got ['E11']
  ✔ E11 a colour inside an image written into the styles → want ['E11'], got ['E11']
  ✔ E12 an icon character in the page → want ['E12'], got ['E12']
  ✔ E12 an icon written as an escape in the concept → want ['E12'], got ['E12']
  ✔ E12 an emoji punctuation mark → want ['E12'], got ['E12']
  ✔ clean: arrows and a copyright sign in a note → want ['clean'], got ['clean']
  ✔ E13 no footer → want ['E13'], got ['E13']
  ✔ E13 the not-advice line removed → want ['E13'], got ['E13']
  ✔ E13 the notes never written → want ['E13'], got ['E13']
  ✔ E14 a page that does not read its concept block → want ['E14'], got ['E14']
  ✔ E14 no probe battery → want ['E14'], got ['E14']
  ✔ E14 a title that is not the concept's → want ['E14'], got ['E14']
  ✔ E14 two titles → want ['E14'], got ['E14']
  ✔ E14 no language on the page → want ['E14'], got ['E14']
explorer_check selftest: 105/105 passed
```

`python3 scripts/formula.py --selftest` (exit 0):

```text
formula selftest · 44/44 passed
  ✔ compound interest: 1000 at 5% for 10 years
  ✔ power runs right to left: 2^3^2 = 2^9
  ✔ -x^2 is -(x^2); (-x)^2 is not
  ✔ - and / run left to right
  ✔ * before +, and parentheses change it
  ✔ a negative exponent needs no parentheses
  ✔ min and max take two or more arguments
  ✔ round is half away from zero (2.5 → 3, -2.5 → -3)
  ✔ round works on the number as written, checked against answers worked out by hand
  ✔ round, floor and ceil finish the working in decimal first, checked against answers worked out by hand
  ✔ e-notation and a leading decimal point
  ✔ ** is accepted as ^
  ✔ error: 1 / 0 → division by zero
  ✔ error: sqrt(-1) → square root of a negative
  ✔ error: ln(0) → not positive
  ✔ error: (-8)^(1/3) → fractional power
  ✔ error: 0^-1 → zero to a negative power
  ✔ error: q + 1 → not a control
  ✔ error: 10^400 → not a finite number
  ✔ error: exp(1000) → not a finite number
  ✔ refused or harmless: 'alert(1)'
  ✔ refused or harmless: 'constructor'
  ✔ refused or harmless: '__proto__'
  ✔ refused or harmless: 'x.y'
  ✔ refused or harmless: 'a = 1'
  ✔ refused or harmless: "'text'"
  ✔ refused or harmless: 'sqrt 4'
  ✔ refused or harmless: 'max(1)'
  ✔ refused or harmless: 'abs(1, 2)'
  ✔ refused or harmless: '1 +'
  ✔ refused or harmless: '(1'
  ✔ refused or harmless: '1 2'
  ✔ the names a formula reads, in order
  ✔ written back symbolically (P × (1 + r ÷ 100)^n)
  ✔ written back with the values in place
  ✔ a negative value is written in brackets, so the line works out to the result (1 × (-3)^2 + 2 × (-3) + 0 = 3)
  ✔ a number in the formula is written as it was typed, not to six digits (1000 × 1.000137^(365 × 10))
  ✔ numbers keep their own spelling (1.5e3, .5); ** is written as ^
  ✔ no value when a step of the working is not finite, even if the end would be: 1 / exp(x) at {'x': 1000}
  ✔ no value when a step of the working is not finite, even if the end would be: min(exp(x), 5) at {'x': 1000}
  ✔ no value when a step of the working is not finite, even if the end would be: 1 / 10^x at {'x': 400}
  ✔ no value when a step of the working is not finite, even if the end would be: min(5, x * x - x * x) at {'x': 1e+200}
  ✔ a number too large to be finite is refused where it is written
  ✔ numbers shown as the page shows them
```

`python3 scripts/make_explorer.py --selftest` (exit 0):

```text
  PASS  the starter's own concept meets every concept rule
  PASS  the page built from it has no findings in explorer_check.py
  PASS  the tokens are written into the page, not linked
  PASS  the page is sealed: its policy, ahead of every script, allows exactly its four scripts by hash (the three of the template and the number-sources runtime) and no eval
  PASS  the concept read back out of the built page is the concept that went in
  PASS  the example at-least-one-success.json builds with no findings
  PASS  the example compound-interest.json builds with no findings
  PASS  the example loan-payment.json builds with no findings
  PASS  a note containing </script> and markup survives the trip into the page and back
  PASS  a concept with a control the formula never uses is refused and nothing is written
  PASS  a template whose formula language was edited is refused
  PASS  a concept with a key written twice is refused
make_explorer selftest: 12/12 passed
```

`python3 scripts/numsrc.py --selftest` (exit 0):

```text
selftest: 50/50
```

`python3 scripts/make_explorer.py assets/concepts/loan-payment.json -o output/loan.html` (exit 0):

```text
built output/loan.html: M = L * (i/1200) / (1 - (1 + i/1200)^(-12*y))  ·  controls L, i, y  ·  2 practice question(s)  ·  105736 bytes
next: python3 scripts/explorer_check.py output/loan.html  and  python3 scripts/probe_check.py output/loan.html
```

`python3 scripts/explorer_check.py output/loan.html` (exit 0):

```text
✔ loan.html: 0 findings
```

`python3 scripts/probe_check.py output/loan.html` (exit 0, reviewer-recorded):

```text
$ python3 scripts/probe_check.py output/loan.html
  ✔ P1 L: 5 moves, each changed the formula line and the chart and marked itself; 5 changed the result
  ✔ P1 i: 5 moves, each changed the formula line and the chart and marked itself; 5 changed the result
  ✔ P1 y: 5 moves, each changed the formula line and the chart and marked itself; 5 changed the result
  ✔ P2 12 states: every formula line, result and readout is what the Python twin writes
  ✔ P3 question 1 (compute): answer 948.974; passes typed three ways and half a tolerance off, fails two tolerances off and empty; Show sets the controls
  ✔ P4 question 2 (reach): passes at i = 6, fails at the first failing setting and one step past the highest; the fixed controls lock
6/6 checks pass
[exit 0]
```

`python3 scripts/probe_check.py --selftest` (exit 0, reviewer-recorded):

```text
$ python3 scripts/probe_check.py --selftest
  ✔ the starter behaves: 6 checks, all pass
  ✔ the example at-least-one-success.json behaves: 5 checks, all pass
  ✔ the example loan-payment.json behaves: 6 checks, all pass
  ✔ the ball thrown up (a curve that turns back) behaves: 5 checks, all pass
  ✔ a report for another formula → want P0 red, got ['P0']
  ✔ a slider that is not wired → want P1 red, got ['P1', 'P2']
  ✔ a probe that skips a control → want P1 red, got ['P1']
  ✔ the wrong number marked → want P1 red, got ['P1']
  ✔ a chart that never redraws → want P1 red, got ['P1']
  ✔ a result that never updates → want P1 red, got ['P1', 'P2']
  ✔ a result rounded its own way → want P2 red, got ['P2']
  ✔ a formula line typed by hand → want P2 red, got ['P2']
  ✔ a line with values written its own way → want P2 red, got ['P2']
  ✔ a readout that drops the unit → want P2 red, got ['P2']
  ✔ a value that drifts from its slider → want P2 red, got ['P2']
  ✔ a probe that skips a state → want P2 red, got ['P2']
  ✔ a missing value explained its own way → want P2 red, got ['P2']
  ✔ a formula that goes wrong only at the question's values → want P3 red, got ['P3']
  ✔ a grader that refuses exactly the page's own answer → want P3 red, got ['P3']
  ✔ a grader that refuses an answer given to four decimal places → want P3 red, got ['P3']
  ✔ a grader that does not read a unit → want P3 red, got ['P3']
  ✔ a grader stricter than its tolerance → want P3 red, got ['P3']
  ✔ a compute grader with a wider tolerance than it says → want P3 red, got ['P3']
  ✔ a grader that passes an empty answer → want P3 red, got ['P3']
  ✔ a Show button that moves the numbers and not the sliders → want P3 red, got ['P3']
  ✔ a compute grader that wants the exact text → want P3 red, got ['P3']
  ✔ a probe that skips a question → want P4 red, got ['P4']
  ✔ a within goal graded tighter than its tolerance → want P4 red, got ['P4']
  ✔ a probe that walks the settings in another order → want P4 red, got ['P4']
  ✔ a Check that grades the starting values, not the sliders → want P4 red, got ['P4']
  ✔ a reach Check that does not fix the question's values → want P4 red, got ['P4']
  ✔ a Check that passes one wrong setting → want P4 red, got ['P4']
  ✔ a reach grader that forgets lowest → want P4 red, got ['P4']
  ✔ a probe that steps two past the answer → want P4 red, got ['P4']
  ✔ a state with a control the page does not have → want P2 red, got ['P2']
  ✔ a probe that steps past the end of the slider → want P4 red, got ['P4']
  ✔ a reach grader that looks only one step further → want P4 red, got ['P4']
  ✔ a probe that never looks past the answer's stretch → want P4 red, got ['P4']
  ✔ a page without the battery gives no report rather than a pass
  ✔ a page changed after it was sealed: the browser runs none of its scripts, and the script says why → the page's scripts are not the ones its Content-Se…
  ✔ a Chrome that dies before the page runs is named, not the page → Chrome was ended by signal 6 before the page could run, with…
  ✔ a Chrome that exits 0 and prints nothing is named, not the page → Chrome printed nothing, not even an empty page, before the p…
  ✔ a --chrome path that is not there is not swapped for another Chrome
  ✔ a Chrome that dies only inside its own sandbox gets a second try without it → 6 checks, the note says so
  ✔ a Chrome that fails with a message inside its own sandbox gets a second try without it → 6 checks, the note says so
  ✔ a Chrome that reports its sandbox failed inside its own sandbox gets a second try without it → 6 checks, the note says so
  ✔ a Chrome that prints nothing inside its own sandbox gets a second try without it → 6 checks, the note says so
probe_check selftest: 47/47 passed
[exit 0]
```

`python3 scripts/make_explorer.py --concept-of output/loan.html` (exit 0):

```text
{
  "title": "What a loan costs each month",
  "question": "A loan paid back in equal monthly payments: how big is each payment, and what moves it most?",
  "result": {
    "name": "M",
    "label": "Monthly payment",
    "unit": "£"
  },
  "formula": "L * (i/1200) / (1 - (1 + i/1200)^(-12*y))",
  "controls": [
    {
      "name": "L",
      "label": "Amount borrowed",
      "unit": "£",
      "min": 10000,
      "max": 500000,
      "step": 5000,
      "value": 200000
    },
    {
      "name": "i",
      "label": "Yearly interest rate",
      "unit": "%",
      "min": 0.5,
      "max": 12,
      "step": 0.25,
      "value": 5
    },
    {
      "name": "y",
      "label": "Years to pay it back",
      "unit": "years",
      "min": 1,
      "max": 35,
      "step": 1,
      "value": 25,
      "unit_one": "year"
    }
  ],
  "chart": {
    "x": "y",
    "title": "The monthly payment for each length of loan"
  },
  "notes": [
    "The rate is fixed for the whole loan.",
    "Interest is charged every month at a twelfth of the yearly rate.",
    "Fees, insurance and paying early are left out."
  ],
  "source": "the standard level-payment (annuity) formula used for repayment mortgages and personal loans.",
  "practice": [
    {
      "type": "compute",
      "prompt": "£150,000 borrowed at 4.5% a year, paid back over 20 years. What is the monthly payment?",
      "set": {
        "L": 150000,
        "i": 4.5,
        "y": 20
      },
      "tolerance": 0.5
    },
    {
      "type": "reach",
      "prompt": "£300,000 over 30 years: what is the highest rate at which the payment is £1,800 a month or less?",
      "fixed": {
        "L": 300000,
        "y": 30
      },
      "goal": "at most",
      "target": 1800,
      "highest": "i"
    }
  ]
}
```

`python3 scripts/numsrc.py check output/loan.html` (exit 0):

```text
N05 warning: control.i: source is unused
N05 warning: control.L: source is unused
N05 warning: control.y: source is unused
```

The source-markup check warns that the three control entries are unused; this does not verify clicking their readouts.
The unused-source warnings have no fixed order, so the same warnings can print in a different sequence.

</details>

## Limits

One formula and sliders only; no systems of equations, tables or derivations. The chart is a slice through the
current settings. Its scale heuristic can omit valid extremes, and sampling can miss narrow features or
discontinuities. Display rounding does not change grading; magnitudes of 10^21 or more use the ordinary formatter.
The page states what the formula leaves out, its source and that it is not advice; the checks cannot establish
whether the formula fits a person's case.

Checks cover the listed patterns, not every possible script edit. Rewriting hashes or weakening the policy can
change what a browser runs; recheck or rebuild after edits. The checker command-line exit paths have no mutation
coverage. Human inspection remains necessary before sharing a page.

## Provenance and license

The ideas of showing the calculation behind every interaction and grading on the result come from Query Mirror,
the author's SQL teaching tool. The appearance uses the shared design tokens. MIT.
