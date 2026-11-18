# nk-explorer

Turn a formula into an offline page with sliders, a curve and practice questions graded on the result.

Ask your coding agent, for example:

> Use nk-explorer to explain a £200,000 loan at 5% interest. Let me change the years to pay it back and practise reading the monthly payment.

![nk-explorer: the bundled loan example, with an answer, slider and curve](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/results/nk-explorer.png)

**The loan example starts at £1169.18 per month** over 25 years, set beside £1045.02 over 32 years with the other
inputs unchanged. Every slider shows the sum it changes.
[Download the loan page](output/loan.html) and open it locally.

Part of [nickkk-skills](https://github.com/NickkkLian/nickkk-skills): 14 agent skills that work with numbers, check an agent's work, guard a repository, and write and present.

## What it makes

For teachers and writers using a coding agent to explain one formula with a calculator,
one main slider and a graph. Generate an HTML page, then check it and inspect it in the browser you intend to use.
The page is designed to open locally and work offline in a modern browser.

The shipped [output/loan.html](output/loan.html) is the page in the picture above: question, answer and curve,
expandable calculation and settings, and practice questions. Regenerate it with the commands below after editing
its concept.

## Try it from a checkout

No packages or build step are required. Run from the skill folder. The builder overwrites the named output file;
relative paths are relative to the working directory. Create the output and scratch folders first:

```bash
mkdir -p output scratch
export TMPDIR="$PWD/scratch"
python3 scripts/make_explorer.py assets/concepts/loan-payment.json -o output/loan.html
```

Open `output/loan.html` and inspect it, then check the generated file:

```bash
python3 scripts/explorer_check.py output/loan.html
```

The self-tests, the chart check (Node) and the browser check (Chrome) are under [Verify](#verify).

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
browser checker on five pages; the recorded browser results are in [VERIFICATION.md](VERIFICATION.md). Those records do not establish
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

## Verify

The non-browser test fixtures:

```bash
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
still need the Chrome check and human inspection. The builder and Python checks require no Node.

Where Chrome/Chromium and permission to launch it are available, also run:

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

This is version **0.1.7**. The loan figures above were re-checked against Python, and the loan page passed 6/6
browser checks in the 0.1.6 recordings. The full test record for this version, the browser recordings and the reports of the agents who tried
it are in [VERIFICATION.md](VERIFICATION.md).

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
